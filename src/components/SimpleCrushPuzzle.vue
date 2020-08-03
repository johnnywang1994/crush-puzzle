<template>
  <div class="crush-puzzle">
    <div
      v-for="(col, i) in jewelList"
      :key="`jewel_board_${i}`"
      class="jewel-col"
    >
      <div
        v-for="(jewel, j) in col"
        :key="`jewel_item_${jewel.id}`"
        @click="onJewelClick($event, jewel, i, j)"
        :class="[
          'jewel',
          { tag: jewel.tag === 0 },
          { picked: pick && (jewel.id === selectJewel[0].jewel.id) }
        ]"
      >
        <img
          :src="icons[jewel.value]"
          v-show="jewel.tag === 1"
        />
        <img
          :src="crushIcon"
          v-show="jewel.tag === 0"
        />
      </div>
    </div>
  </div>
</template>

<script>
const icons = {
  1: 'https://vuejs.org/images/logo.png',
  2: 'https://www.ssa-frontend.com/media/1164/react.png',
  3: 'https://angular.io/assets/images/logos/angular/angular.svg',
};

export default {
  name: 'CrushPuzzle',
  emits: [
    'count',
  ],
  props: {
    match: {
      type: Number,
      default: 3,
      description: 'Minimun jewel number should match'
    },
    ratio: {
      type: Number,
      default: 5,
      description: 'Puzzle ratio'
    },
    icons: {
      type: Object,
      default: () => (icons),
      description: 'Jewel icon urls'
    },
    crushIcon: {
      type: String,
      default: 'https://img.icons8.com/ultraviolet/40/000000/explosion.png',
      description: 'Icon for jewel crashed'
    },
    jewelTypes: {
      type: Number,
      default: 3,
      description: 'Jewel type number used in puzzle. should <= stickers length'
    },
  },
  data() {
    return {
      // custom
      points: 0,
      // settings
      jewelList: [],
      selectJewel: [],
      jewelUid: 0,
      hasInit: false,
      pick: false,
    };
  },
  methods: {
    randomNum() {
      return Math.ceil(Math.random() * this.jewelTypes);
    },
    // Create Jewel
    // id: X => for vue key animation
    // tag: 0 => jewel tagged, 1 => normal jewel
    // value: X => for scaning
    createJewel(i) {
      const vm = this;
      const list = vm.jewelList;
      const lastJewel = list[i].length ? list[i][list[i].length - 1].value : 0;
      let value = vm.randomNum();
      // check if is same with last one(top)
      while (value === lastJewel) {
        value = vm.randomNum();
      }
      vm.jewelUid++;
      return {
        id: vm.jewelUid,
        tag: 1,
        value,
      };
    },
    initBoard() {
      const { ratio, jewelList } = this;
      for (let i=0;i<ratio;i++) {
        jewelList[i] = [];
        for (let j=0;j<ratio;j++) {
          jewelList[i][j] = this.createJewel(i);
        }
      }
      this.scanBoard();
    },
    // scan whole puzzle through each col & row
    // since the puzzle is reverse by css, loop from 0 is from puzzle button
    scanBoard() {
      const { ratio } = this;
      for (let i=0;i<ratio;i++) {
        for (let j=0;j<ratio;j++) {
          this.scan(i, j);
          this.replaceTag();
        }
      }
    },
    onJewelClick(ev, jewel, i, j) {
      this.pick = !this.pick;
      this.selectJewel.push({ ev, jewel, i, j });
      if (this.pick) {
        this.firstPick();
      } else {
        this.secondPick();
      }
    },
    firstPick() {
      // ...
    },
    secondPick() {
      this.exchange();
    },
    exchange() {
      const list = this.jewelList;
      const [jOne, jTwo] = this.selectJewel;
      const isNear = this.checkIsNear(jOne, jTwo);
      const changeJewel = () => {
        const temp = list[jOne.i][jOne.j].value;
        list[jOne.i][jOne.j].value = list[jTwo.i][jTwo.j].value;
        list[jTwo.i][jTwo.j].value = temp;
      };
      if (isNear) {
        changeJewel();
        // scan both selected
        const state1 = this.scan(jOne.i, jOne.j);
        const state2 = this.scan(jTwo.i, jTwo.j);
        if (state1 || state2) {
          // replace tags
          this.replaceTag(); 
        } else {
          changeJewel();
        }
      }
      this.selectJewel = [];
    },
    checkIsNear(jOne, jTwo) {
      // same col
      if (jOne.i === jTwo.i) {
        return Math.abs(jOne.j - jTwo.j) === 1;
      }
      // is next or prev col
      if (Math.abs(jOne.i - jTwo.i) === 1) {
        return jOne.j === jTwo.j;
      }
      return false;
    },
    scan(i, j) {
      const { ratio, match, jewelList: list } = this;
      const col = list[i];
      const { value } = list[i][j];
      let left = i, right = i;
      let top = j, down = j;
      // top, down is reverse by css
      while (col[top - 1] && col[top - 1].value === value) {
        top--;
      }
      while (col[down + 1] && col[down + 1].value === value) {
        if (down+1 >= ratio) break;
        down++;
      }
      while (list[left - 1]
        && list[left - 1][j]
        && list[left - 1][j].value === value
      ) {
        left--;
      }
      while (list[right + 1]
        && list[right + 1][j]
        && list[right + 1][j].value === value
      ) {
        right++;
      }
      // console.log('vertical:', top, down);
      this.removeColWithTag(i, top, down);
      // console.log('horizontal:', left, right);
      this.removeRowWithTag(j, left, right);
      return (down-top>=(match-1) || right-left>=(match-1));
    },
    removeColWithTag(i, top, down) {
      if (down-top < (this.match-1)) return;
      const list = this.jewelList;
      const needOffset = down+1 === this.ratio;
      const from = top;
      const to = (needOffset ? down+2 : down+1);
      const slices = list[i].slice(from, to);
      slices.forEach((jewel) => jewel.tag = 0);
      // tag col => 0
      list[i].splice(from, to-from, ...slices);
      this.$forceUpdate();
    },
    removeRowWithTag(j, left, right) {
      if (right-left < (this.match-1)) return;
      const list = this.jewelList;
      let from = left, to = right;
      // tag col => 0
      while (from <= to) {
        list[from][j].tag = 0;
        from++;
      }
      this.$forceUpdate();
    },
    replaceTagHandler() {
      const { ratio, jewelList: list, hasInit } = this;
      let scanAgain = false;
      let tagCount = 0;
      for (let i=0;i<ratio;i++) {
        const remain = list[i].filter((jewel) => jewel.tag !== 0);
        if (list[i].length !== remain.length) {
          scanAgain = true;
          tagCount += list[i].length - remain.length;
        }
        list[i] = remain;
        while (list[i].length < ratio) {
          // push from top(reversed)
          list[i].push(this.createJewel(i));
        }
      }
      hasInit && (this.points += tagCount);
      hasInit && (this.$emit('count', this.points));
      // check if scanAgain or can rerender view
      if (scanAgain) {
        this.scanBoard();
      } else {
        this.$forceUpdate();
      }
    },
    replaceTag() {
      const vm = this;
      // wait for animation done
      if (vm.hasInit) {
        setTimeout(vm.replaceTagHandler, 500);
      } else {
        vm.replaceTagHandler();
        vm.hasInit = true;
      }
    }
  },
  created() {
    this.initBoard();
  },
};
</script>

<style lang="scss">
@keyframes fadeIn {
  0% {
    opacity: 0;
    transform: translateY(-100%);
  }
}

.crush-puzzle {
  width: 300px;
  height: 300px;
  display: flex;
  margin: auto;
  background: #e2fff3;
  .jewel-col {
    display: flex;
    flex-wrap: wrap;
    flex-direction: column-reverse;
    flex: 1;
    .jewel {
      display: flex;
      align-items: center;
      justify-content: center;
      flex: 1 0;
      width: 100%;
      text-align: center;
      border: 1px solid #888;
      box-sizing: border-box;
      cursor: pointer;
      animation: fadeIn 0.3s linear 1;
      &.tag {
        color: #fff;
        background: #000;
      }
      &.picked {
        color: red;
        background: #8ad2ff;
      }
      > img {
        display: block;
        width: 100%;
      }
    }
  }
}
</style>

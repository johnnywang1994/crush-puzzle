<template>
  <div class="jewel-board">
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
          {
            bomb: jewel.tag === 0,
            col: jewel.tag === 2,
            row: jewel.tag === 3,
            same: jewel.tag === 4,
            picked: pick && (jewel.id === selectJewel[0].jewel.id)
          },
        ]"
      >
        <img
          :src="icons[jewel.value]"
          v-show="jewel.tag !== 0"
        />
        <img
          :src="icons.bomb"
          v-show="jewel.tag === 0"
        />
      </div>
    </div>
  </div>
</template>

<script>
// 'https://image.flaticon.com/icons/png/128/148/148905.png',
// 'https://image.flaticon.com/icons/png/128/1144/1144441.png',
const icons = {
  1: 'https://stickershop.line-scdn.net/stickershop/v1/sticker/14015619/android/sticker.png',
  2: 'https://stickershop.line-scdn.net/stickershop/v1/sticker/14015631/android/sticker.png',
  3: 'https://stickershop.line-scdn.net/stickershop/v1/sticker/14015629/android/sticker.png',
  4: 'https://stickershop.line-scdn.net/stickershop/v1/sticker/14015625/android/sticker.png',
  5: 'https://stickershop.line-scdn.net/stickershop/v1/sticker/14015614/android/sticker.png',
  bomb: 'https://img.icons8.com/office/80/000000/explosion.png',
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
    jewelTypes: {
      type: Number,
      default: 4,
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
    // tag:
    //  0 => jewel bomb tagged,
    //  1 => normal jewel,
    //  2 => special col,
    //  3 => special row
    //  4 => special same
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
    findJewelById(id) {
      let list = this.jewelList;
      let result = {};
      for (let i=0;i<this.ratio;i++) {
        for (let j=0;j<this.ratio;j++) {
          if (list[i][j].id === id) {
            result.i = i;
            result.j = j;
            break;
          }
        }
      }
      return result;
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
      const [jOne, jTwo] = this.selectJewel;
      const isNear = this.checkIsNear(jOne, jTwo);
      const isSuper = this.checkIsSuper(jOne, jTwo);
      if (isNear) {
        if (isSuper) {
          this.superHandler(jOne, jTwo);
          this.replaceTag();
        } else {
          this.changeJewel(jOne, jTwo);
          // scan both selected
          const state1 = this.scan(jOne.i, jOne.j);
          const state2 = this.scan(jTwo.i, jTwo.j);
          if (state1 || state2) {
            // replace tags
            this.replaceTag();
          } else {
            this.changeJewel(jOne, jTwo);
          }
        }
      }
      this.selectJewel = [];
    },
    changeJewel(jOne, jTwo) {
      const list = this.jewelList;
      const from = list[jOne.i][jOne.j];
      const to = list[jTwo.i][jTwo.j];
      const { value: tempV, tag: tempT } = from;
      from.value = to.value;
      from.tag = to.tag;
      to.value = tempV;
      to.tag = tempT;
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
    checkIsSuper(jOne, jTwo) {
      if (jOne.jewel.tag === 1 || jTwo.jewel.tag === 1) return false;
      return true;
    },
    superHandler(jOne, jTwo) {
      // same special
      if (jOne.jewel.tag === jTwo.jewel.tag) {
        if (jOne.jewel.tag === 2) {
          this.specialColHandle(jOne.i);
          this.specialColHandle(jTwo.i);
        }
        if (jOne.jewel.tag === 3) {
          this.specialRowHandle(jOne.j);
          this.specialRowHandle(jTwo.j);
        }
      // different special
      } else {
        this.specialColHandle(jOne.i);
        this.specialRowHandle(jOne.j);
      }
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
      this.removeColWithTag(i, j, top, down);
      // console.log('horizontal:', left, right);
      this.removeRowWithTag(i, j, left, right);
      return (down-top>=(match-1) || right-left>=(match-1));
    },
    specialColHandle(i) {
      this.jewelList[i].forEach((jewel) => {
        if (jewel.tag === 3) {
          const { j } = this.findJewelById(jewel.id);
          this.specialRowHandle(j);
        }
        if (jewel.tag === 4) {
          this.specialSameHandle(jewel.value);
        }
        jewel.tag = 0;
      });
    },
    specialRowHandle(j) {
      for (let i=0;i<this.ratio;i++) {
        const jewel = this.jewelList[i][j];
        if (jewel.tag === 2) {
          const { i } = this.findJewelById(jewel.id);
          this.specialColHandle(i);
        }
        if (jewel.tag === 4) {
          this.specialSameHandle(jewel.value);
        }
        jewel.tag = 0;
      }
    },
    specialSameHandle(value) {
      const list = this.jewelList;
      for (let i=0;i<this.ratio;i++) {
        for (let j=0;j<this.ratio;j++) {
          const jewel = list[i][j];
          if (jewel.value === value) {
            if (jewel.tag === 2) {
              this.specialColHandle(i);
            }
            if (jewel.tag === 3) {
              this.specialRowHandle(j);
            }
            jewel.tag = 0;
          }
        }
      }
    },
    removeColWithTag(i, j, top, down) {
      if (down-top < (this.match-1)) return;
      const list = this.jewelList;
      const from = top;
      const to = down+1;
      const slices = list[i].slice(from, to);
      slices.forEach((jewel) => {
        // normal handle
        if (jewel.tag === 1) {
          jewel.tag = 0;
        } else {
          // special handle
          if (jewel.tag === 2) {
            this.specialColHandle(i);
          } else if (jewel.tag === 3) {
            const { j } = this.findJewelById(jewel.id);
            this.specialRowHandle(j);
          } else if (jewel.tag === 4) {
            this.specialSameHandle(jewel.value);
          }
        }
      });
      list[i].splice(from, to-from, ...slices);
      // tag col special
      if (to-from > this.match) {
        list[i][j].tag = to-from === 4 ? 2 : 4;
      }
      this.$forceUpdate();
    },
    removeRowWithTag(i, j, left, right) {
      if (right-left < (this.match-1)) return;
      const list = this.jewelList;
      let from = left, to = right;
      // tag col => 0
      while (from <= to) {
        const jewel = list[from][j];
        // normal
        if (jewel.tag === 1) {
          jewel.tag = 0;
        } else {
          // special handle
          if (jewel.tag === 2) {
            const { i } = this.findJewelById(jewel.id);
            this.specialColHandle(i);
          }
          if (jewel.tag === 3) {
            this.specialRowHandle(j);
          }
          if (jewel.tag === 4) {
            this.specialSameHandle(jewel.value);
          }
        }
        from++;
      }
      // tag row special
      if (right-left+1 > this.match) {
        list[i][j].tag = right-left+1 === 4 ? 3 : 4;
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

@keyframes shiny {
  50% {
    filter: brightness(1.2);
  }
}

.jewel-board {
  width: 360px;
  height: 360px;
  display: flex;
  margin: auto;
  background: #fff6e2;
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
      &.bomb {
        color: #fff;
        background: #000;
      }
      &.col {
        background-image: repeating-linear-gradient(
          90deg,
          transparent 3px,
          rgb(255, 203, 203) 3px,
          rgb(255, 203, 203) 9px,
          transparent 9px,
          transparent 15px,
        );
      }
      &.row {
        background-image: repeating-linear-gradient(
          transparent 3px,
          rgb(255, 203, 203) 3px,
          rgb(255, 203, 203) 9px,
          transparent 9px,
          transparent 15px,
        );
      }
      &.same {
        background: rgb(255, 255, 64);
        animation: shiny 1s linear infinite;
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

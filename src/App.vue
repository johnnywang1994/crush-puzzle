<template>
  <div id="app">
    <p>Your points: {{ points }}</p>
    <crush-puzzle :ratio="7" @count="onCount" />
    <h1>Vue Crush Puzzle</h1>
    <p>
      A simple component for creating a crush puzzle game
      <br>
      <a href="#">
        vue-crush-puzzle documentation(Now working)
      </a>.
    </p>
    <h3>Essential Links</h3>
    <ul>
      <li><a href="https://vuejs.org" target="_blank" rel="noopener">Vue Core Docs</a></li>
    </ul>

    <!-- Footer -->
    <footer>
      Copyright &copy; 2020-present, Johnny Wang
    </footer>
  </div>
</template>

<script>
import CrushPuzzle from './components/CrushPuzzle.vue'

export default {
  name: 'App',
  components: {
    CrushPuzzle,
  },
  data() {
    return {
      points: 0,
    };
  },
  methods: {
    onCount(points) {
      this.points = points;
    },
  },
};
</script>

<style lang="scss">
body {
  margin: 0;
  padding: 0;
  padding-bottom: 50px;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;

  h3 {
    margin: 40px 0 0;
  }
  ul {
    list-style-type: none;
    padding: 0;
  }
  li {
    display: inline-block;
    margin: 0 10px;
  }
  a {
    color: #42b983;
  }

  .crush-puzzle {
    margin: auto;
  }

  footer {
    position: fixed;
    bottom: 0;
    width: 100%;
    color: #fff;
    padding: 4px 0;
    font-weight: 700;
    background: #333;
  }
}
</style>

<template>
  <div id="app">
    <div class="board_header">
      <h5>Flags remaining: {{flags}}</h5>
      <h3 class="elapsed_time">{{getTitle}}</h3>
    </div>

    <board @gameEnded="handleEnding" @updatedFlags="handleFlags"></board>
  </div>
</template>
<script>
import Board from "./components/Board";
export default {
  data() {
    return {
      seconds: 0,
      counter: undefined,
      gameEnded: false,
      won: false,
      flags: 50
    };
  },
  computed: {
    getTitle() {
      if (this.gameEnded)
        if (this.won) return "You Won!";
        else return "You Lost!";
      else return this.getTime;
    },
    getTime() {
      let minutes =
        this.seconds >= 60 ? "0" + Math.floor(this.seconds / 60) : "00";
      let seconds = this.seconds - minutes * 60;
      if (seconds < 10) seconds = "0" + seconds;

      return minutes + ":" + seconds;
    }
  },
  components: {
    Board
  },
  mounted() {
    let instance = this;
    this.counter = setInterval(() => {
      instance._data.seconds += 1;
    }, 1000);
  },

  methods: {
    handleEnding(won) {
      this.gameEnded = true;
      this.won = won;
      clearInterval(this.counter);
    },
    handleFlags(flags) {
      this.flags = flags;
    }
  }
};
</script>

<style scoped>
#app {
  width: 100vw;
  height: 100vh;
  margin: 0;
  background: #fff;
  user-select: none;
}

.board_header {
  width: 43.5rem;
  position: absolute;
  left: 50%;
  top: 5%;
  transform: translateX(-50%);
  border-radius: 0.7rem;
  background: #2393c4;
}

.elapsed_time {
  text-align: center;
  color: white;
  margin-top: 1.5rem;
  padding-bottom: 2rem;
  font-size: 2.2rem;
}

.board_header h5 {
  text-align: center;
  color: white;
  margin-top: 1.5rem;
  font-weight: 300;
}
</style>

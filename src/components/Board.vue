<template>
  <div id="board" :style="{height:'43.5rem', width:'43.5rem'}">
    <div v-for="rows in board">
      <button
        class="space"
        :class="spaceClass(item)"
        v-for="item in rows"
        @click="checkSpace(item.i, item.j)"
        :key="item.i + ' ' + item.j"
      >
        {{item.visited ? item.isBomb ? '' : item.nearbyBombs > 0 ? item.nearbyBombs : '' : ''}}
        <i
          class="fas fa-bomb"
          v-if="item.isBomb && !item.flagged && item.visited"
        ></i>

        <i class="fas fa-flag" v-if="item.flagged"></i>
      </button>
    </div>
  </div>
</template>
<script>
export default {
  data() {
    return {
      board: [],
      bombCount: 50,
      flags: 50,
      visitedPlaces: 0,
      gameEnded: false
    };
  },
  mounted() {
    for (let i = 0; i < 20; i++) {
      let row = [];
      for (let j = 0; j < 20; j++)
        row.push({
          value: i * 20 + j + 1,
          i,
          j,
          visited: false,
          flagged: false,
          isBomb: 0,
          nearbyBombs: 0
        });
      this.board.push(row);
    }

    this.randomizeBombs(this.bombCount);
    this.boardSetup();

    let instance = this;
    setTimeout(function() {
      instance.handleRightClick();
    }, 10);
  },
  watch: {
    flags(val) {
      this.$emit("updatedFlags", val);
    }
  },
  methods: {
    checkSpace(i, j) {
      let item = this.board[i][j];
      if (this.gameEnded) return;
      if (item.visited || item.flagged) return;
      item.visited = true;
      this.visitedPlaces++;
      if (item.isBomb) {
        this.endGame(0);
      } else if (item.nearbyBombs > 0) {
      } else {
        let dl = [-1, -1, 0, 1, 1, 1, 0, -1];
        let dc = [0, 1, 1, 1, 0, -1, -1, -1];

        for (let dir = 0; dir < 8; dir++) {
          let newRow = item.i + dl[dir];
          let newCol = item.j + dc[dir];

          if (newRow >= 0 && newCol >= 0 && newRow < 20 && newCol < 20)
            this.checkSpace(newRow, newCol);
        }
      }
      console.log(this.visitedPlaces);
      if (this.visitedPlaces == 350) this.endGame(1);
    },
    randomizeBombs(bombs) {
      while (bombs) {
        let ranX = Math.floor(Math.random() * 20);
        let ranY = Math.floor(Math.random() * 20);

        if (!this.board[ranX][ranY].isBomb) {
          this.board[ranX][ranY].isBomb = 1;
          bombs--;
        }
      }
    },
    boardSetup() {
      let dl = [-1, -1, 0, 1, 1, 1, 0, -1];
      let dc = [0, 1, 1, 1, 0, -1, -1, -1];

      for (let i = 0; i < 20; i++)
        for (let j = 0; j < 20; j++) {
          if (this.board[i][j].isBomb) {
            this.board[i][j].nearbyBombs = Infinity;
            continue;
          }
          let nearbyBombs = 0;
          for (let dir = 0; dir < 8; dir++) {
            let newRow = i + dl[dir];
            let newCol = j + dc[dir];

            if (newRow >= 0 && newCol >= 0 && newRow < 20 && newCol < 20)
              if (this.board[newRow][newCol].isBomb) {
                nearbyBombs++;
              }
          }

          this.board[i][j].nearbyBombs = nearbyBombs;
        }
    },
    endGame(won) {
      this.gameEnded = true;
      this.$emit("gameEnded", won);
      if (won) {
        console.log("You Won!");
      } else return console.log("You Lost! Better luck next time ;)");
    },
    handleRightClick() {
      let places = document.getElementsByClassName("space");
      let index = 0;
      let instance = this;
      for (let i = 0; i < 20; i++) {
        for (let j = 0; j < 20; j++) {
          places[index].addEventListener(
            "contextmenu",
            function(ev) {
              ev.preventDefault();
              if (instance._data.board[i][j].visited) return false;
              instance._data.board[i][j].flagged = !instance._data.board[i][j]
                .flagged;
              if (instance._data.board[i][j].flagged === false) {
                instance._data.flags++;
              } else instance._data.flags--;

              return false;
            },
            false
          );
          index++;
        }
      }
    },
    spaceClass(item) {
      return {
        visited: item.visited,
        flagged: item.flagged,
        bombed: item.isBomb,
        "bomb-1": item.nearbyBombs === 1 && !item.flagged,
        "bomb-2": item.nearbyBombs === 2 && !item.flagged,
        "bomb-3": item.nearbyBombs === 3 && !item.flagged,
        "bomb-4": item.nearbyBombs === 4 && !item.flagged,
        "bomb-5": item.nearbyBombs === 5 && !item.flagged,
        "bomb-6": item.nearbyBombs === 6 && !item.flagged,
        "bomb-7": item.nearbyBombs === 7 && !item.flagged,
        "bomb-8": item.nearbyBombs === 8 && !item.flagged
      };
    }
  }
};
</script>
<style scoped>
#board {
  position: absolute;
  left: 50%;
  top: 60%;
  transform: translate(-50%, -50%);
}
.space {
  display: inline-block;
  background: #4d595e;
  float: left;
  border: 1px solid black;
  width: 2rem;
  border-radius: 0.2rem;
  margin-left: 0.15rem;
  margin-bottom: 0.15rem;
  height: 2rem;
}

.space.visited {
  background: #b2ebff;
}

.space.flagged {
  background: #c4b923;
}

.space.visited.bombed {
  background: #9f5555;
}

button {
  font-weight: bolder;
}

.bomb-1 {
  color: #24c197;
}

.bomb-2 {
  color: #24a6c1;
}

.bomb-3 {
  color: #2475c1;
}

.bomb-4 {
  color: #9e24c1;
}

.bomb-5 {
  color: #82c124;
}

.bomb-6 {
  color: #bc831a;
}

.bomb-7 {
  color: #bc4a1a;
}

.bomb-8 {
  color: #bc1a73;
}
</style>



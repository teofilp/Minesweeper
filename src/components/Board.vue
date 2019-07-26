<template>
  <div id="board" :style="{height:'43.5rem', width:'43.5rem'}">
    <div v-for="(row, rowIndex) in board" :key="rowIndex">
      <button
        class="space"
        v-for="(item, columnIndex) in row"
        @click="checkSpace(rowIndex, columnIndex)"
        :key="rowIndex + ' ' + columnIndex"
        :class="spaceClass(item)"
        :style="{'width': 'calc(40rem / ' + boardSize, 'height': 'calc(40rem / ' + boardSize,
         'margin-left' : 'calc(3.5rem /' + boardSize, 'margin-bottom': 'calc(3.5rem / ' + boardSize}"
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
      bombCount: 30,
      boardSize: 15,
      flags: 50,
      visitedPlaces: 0,
      gameEnded: false
    };
  },
  mounted() {
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
      if (item.isBomb) this.endGame(0);

      item.visited = true;
      this.visitedPlaces++;

      if (item.nearbyBombs > 0) return;

      let dl = [-1, -1, 0, 1, 1, 1, 0, -1];
      let dc = [0, 1, 1, 1, 0, -1, -1, -1];

      for (let dir = 0; dir < 8; dir++) {
        let newRow = i + dl[dir];
        let newCol = j + dc[dir];

        if (
          newRow >= 0 &&
          newCol >= 0 &&
          newRow < this.boardSize &&
          newCol < this.boardSize
        )
          this.checkSpace(newRow, newCol);
      }
      if (this.visitedPlaces == 350) this.endGame(1);
    },
    randomizeBombs(bombs) {
      while (bombs) {
        let ranX = Math.floor(Math.random() * this.boardSize);
        let ranY = Math.floor(Math.random() * this.boardSize);

        if (!this.board[ranX][ranY].isBomb) {
          this.board[ranX][ranY].isBomb = 1;
          bombs--;
        }
      }
    },
    getRowOfSpaces() {
      let row = [];
      for (let j = 0; j < this.boardSize; j++) {
        row.push({
          visited: false,
          flagged: false,
          isBomb: 0,
          nearbyBombs: 0
        });
      }
      return row;
    },
    drawBoxes() {
      for (let i = 0; i < this.boardSize; i++) {
        this.board.push(this.getRowOfSpaces());
      }
    },
    countNearbyBombs(item, i, j) {
      let deltaY = [-1, -1, 0, 1, 1, 1, 0, -1];
      let deltaX = [0, 1, 1, 1, 0, -1, -1, -1];
      let nearbyBombs = 0;
      for (let dir = 0; dir < 8; dir++) {
        let newRow = i + deltaY[dir];
        let newCol = j + deltaX[dir];

        if (
          newRow >= 0 &&
          newCol >= 0 &&
          newRow < this.boardSize &&
          newCol < this.boardSize
        )
          if (this.board[newRow][newCol].isBomb) {
            nearbyBombs++;
          }
      }

      item.nearbyBombs = nearbyBombs;
    },
    boardSetup() {
      this.drawBoxes();
      this.randomizeBombs(this.bombCount);
      for (let i = 0; i < this.boardSize; i++)
        for (let j = 0; j < this.boardSize; j++) {
          var item = this.board[i][j];
          if (item.isBomb) {
            item.nearbyBombs = Infinity;
            continue;
          }
          this.countNearbyBombs(item, i, j);
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
      for (let i = 0; i < this.boardSize; i++) {
        for (let j = 0; j < this.boardSize; j++) {
          places[index].addEventListener(
            "contextmenu",
            function(ev) {
              ev.preventDefault();
              if (instance._data.board[i][j].visited) return false;
              if (
                instance._data.flags == 0 &&
                !instance._data.board[i][j].flagged
              )
                return;
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
  border-radius: 0.2rem;
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



<template>
  <div :class="{'game':true,'game--over':gameIsWon}">
    <div
      v-if="board"
      :class="{'board':true,'board--playable':!gameIsWon,'board--turn-1':turn===1,'board--turn-2':turn===2}"
      >
      <div
        v-for="(column, columnIndex) in board"
        :key="`board-column-${columnIndex}`"
        :class="{'column':true,'column--full':freeIndexPerCol[columnIndex]===-1,'column--highlighted':highlightedColumn===columnIndex}"
        @touchend="handleColumnTouchend(columnIndex)"
        @click="handleColumnClick(columnIndex)"
        >
          <div
            v-for="(cell, cellIndex) in column"
            :key="`board-column-${columnIndex}-cell-${cellIndex}`"
            :data-position="`${columnIndex}x${cellIndex}`"
            :class="{'cell':true,'cell--winner':cell.winner}"
            >
            <span :class="{'piece':true,'piece--1':cell.value===1,'piece--2':cell.value===2}">&nbsp;</span>
          </div>
          <div
            class="cell cell--preview"
            >
            <span :class="{'piece':true,'piece--1':turn===1,'piece--2':turn===2}">&nbsp;</span>
          </div>
      </div>
    </div>
    <div class="won">
      <p>{{ turn === 1 ? 'Red' : 'Yellow' }} wins!</p>
      <button class="startgame" @click="startgame">Play again</button>
    </div>
  </div>
</template>

<script>
import { Connect4 } from '../lib/Connect4'

export default {
  data() {
    return {
      game: null,
      board: null,
      turn: 1,
      winner: false,
      highlightedColumn: null,
      touchEnabled: false
    }
  },
  props: {
    cols: {
      type: Number,
      default: 7
    },
    rows: {
      type: Number,
      default: 6
    }
  },
  computed: {
    freeIndexPerCol() {
      return this.board.map(col => col.findIndex(cell => cell.value === null))
    },
    gameIsWon() {
      return !!this.winner
    }
  },
  methods: {
    startgame() {
      this.game = new Connect4(this.cols, this.rows)
      this.updateFromGame()
    },
    handleColumnTouchend(col) {
      this.touchEnabled = true

      if (this.gameIsWon || this.freeIndexPerCol[col] === -1) {
        return
      }

      if (this.highlightedColumn !== col) {
        this.highlightedColumn = col
      } else {
        this.makeMove(col)
        this.highlightedColumn = null
      }

    },
    handleColumnClick(col) {
      if (this.touchEnabled) {
        this.touchEnabled = false
        return
      }

      if (this.gameIsWon || this.freeIndexPerCol[col] === -1) return

      if (this.touchEnabled) {
        this.touchEnabled = false
      } else {
        this.makeMove(col)
      }
    },
    makeMove(col) {
      this.game.makeMove(col)
      this.updateFromGame()
    },
    updateFromGame() {
      this.turn = this.game.turn
      this.winner = this.game.winner

      const winningCoordinates = this.winner
        ? this.winner.winningCoordinates.map(c => c.toString())
        : undefined

      this.board = [...this.game.board.map(
        (col, x) => col.map(
          (cell, y) => {
            const value = cell
            const winner = winningCoordinates
              ? winningCoordinates.indexOf([x, y].toString()) !== -1
              : false

            return { value, winner }
        })
      )]
    }
  },
  mounted() {
    this.startgame()
  }
}
</script>

<style>
:root {
  text-align: center;
  font-family:'Courier New', Courier, monospace;
}
:root, body {
  padding: 0;
  margin: 0;
}
.game {
  position: relative;
  display: inline-block;
  margin: auto;
  text-align: left;
}
.won {
  margin-top: 3rem;
  font-size: 2rem;
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%,-50%);
  padding: 2rem;
  border: solid 2px #ccc;
  background-color: rgba(255, 255, 255, 0.9);
  box-shadow: 2px 2px 4px 2px rgba(0, 0, 0, 0.2);
  border-radius: 1rem;
  display: none;
  text-align: center;
}
.startgame {
  font-size: 3rem;
  background-color: #405FCA;
  color: white;
  text-transform: uppercase;
  border: none;
  border-radius: 0.5rem;
  letter-spacing: 0.2rem;
}
.game--over .won {
  display: inline-block;
}
.board {
  display: inline-block;

  display: flex;
  flex-direction: row;

  padding: 1rem;
  background-color: #405FCA;
  border-radius: 2rem;
  margin-top: 3rem;
  box-shadow: 2px 2px 4px 2px rgba(0, 0, 0, 0.2);
}
.column {
  display: flex;
  flex-direction: column-reverse;
  margin-top: -6rem;
}
/* .column--highlighted {
  background-color: rgba(124,0,0,0.5);
} */
.cell {
  margin: 1rem;
  height: 4rem;
  width: 4rem;
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
  border-radius: 50%;
  box-shadow: inset 4px 4px rgba(0, 0, 0, 0.1);
  border: solid 4px transparent;
}
.cell--winner:after {
  content: "";
  position: absolute;
  width: 100%;
  height: 100%;
  display: block;
  background-color: transparent;
  border: solid 4px white;
  border-radius: 50%;
}
.cell--preview {
  border: none;
  opacity: 0;
  margin-top: 0.5rem;
  box-shadow: 2px 2px 4px 2px rgba(0, 0, 0, 0.2);
}
.board--playable .column:not(.column--full):hover .cell--preview,
.board--playable .column--highlighted:not(.column--full) .cell--preview {
  opacity: 1;
}
.board--playable.board--turn-1 .column--highlighted:not(.column--full) .cell {
  background: radial-gradient(circle, rgba(255,0,0,1) 85%, rgba(124,0,0,1) 100%);
}
.board--playable.board--turn-2 .column--highlighted:not(.column--full) .cell {
  background: radial-gradient(circle, rgba(255,250,0,1) 85%, rgba(230,224,0,1) 100%);
}
.piece {
  background: radial-gradient(circle, rgba(255,255,255,1) 39%, rgba(230,230,230,1) 100%);
  display: inline-block;
  width: 100%;
  height: 100%;
  border-radius: 50%;
}
.cell:not(.cell--preview) .piece--1,
.cell:not(.cell--preview) .piece--2 {
  box-shadow: inset 4px 4px rgba(0, 0, 0, 0.1);
}
.piece--1 {
  background: rgb(255,0,0);
  background: radial-gradient(circle, rgba(255,0,0,1) 85%, rgba(124,0,0,1) 100%);
}
.piece--2 {
  background: rgb(255,250,0);
  background: radial-gradient(circle, rgba(255,250,0,1) 85%, rgba(230,224,0,1) 100%);
}
@media screen and (orientation:portrait) {
  :root {
    font-size: 1.8vw;
  }
  .game {
    margin-top: 5vw;
  }
}
@media screen and (orientation:landscape) {
  :root {
    font-size: 1.8vh;
  }
  .game {
    margin-top: 5vh;
  }
}
</style>

<template>
  <div :class="{'game':true,'game--over':gameIsWon}">
    <div
      v-if="board"
      :class="{'board':true,'board--playable':!gameIsWon}"
      >
      <div
        v-for="(column, columnIndex) in board"
        :key="`board-column-${columnIndex}`"
        :class="{'column':true,'column--full':freeIndexPerCol[columnIndex]===-1}"
        @click="makeMove(columnIndex)"
        >
          <div
            v-for="(cell, cellIndex) in column"
            :key="`board-column-${columnIndex}-cell-${cellIndex}`"
            :data-position="`${columnIndex}x${cellIndex}`"
            :class="{'cell':true,'cell--winner':cell.winner}"
            >
            {{ cell.value === null ? ' ' : cell.value }}
          </div>
          <div
            class="cell cell--preview"
          >{{turn}}</div>
      </div>
    </div>
    <div class="won">
      <p>Game won!</p>
      <button @click="startgame">Play again</button>
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
      winner: false
    }
  },
  props: {
    cols: {
      type: Number,
      default: 8
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
    makeMove(col) {
      if (!this.gameIsWon && this.freeIndexPerCol[col]>-1) {
        this.game.makeMove(col)
        this.updateFromGame()
      }
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

<style scoped>
.game {
  position: relative;
  display: inline-block;
}
.won {
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%,-50%);
  padding: 2rem;
  border: solid 1px #ccc;
  background-color: rgba(255, 255, 255, 0.8);
  box-shadow: rgb(32, 32, 32);
  display: none;
}
.game--over .won {
  display: inline-block;
}
.board {
  display: inline-block;
  font-size: 32px;

  display: flex;
  flex-direction: row;
}
.column {
  display: flex;
  flex-direction: column-reverse;
}
.cell {
  border: dashed 1px black;
  margin: 1rem;
  height: 4rem;
  width: 4rem;
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
}
.cell--winner:after {
  content: "★";
  position: absolute;
  top: 0;
  right: 0;
  font-size: 0.5rem;
}
.cell--preview {
  border: none;
  opacity: 0;
}
.board--playable .column:not(.column--full):hover .cell--preview {
  opacity: 1;
}
.coordinates {
  position: absolute;
  top: 0.2rem;
  right: 0.2rem;
  font-size: 0.7rem;
}
</style>

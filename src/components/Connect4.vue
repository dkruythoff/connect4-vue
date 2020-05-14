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

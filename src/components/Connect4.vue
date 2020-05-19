<template>
  <div
    :class="{
      'game': true,
      'game--over': gameState.gameover
    }">
    <div
      :class="{
        'board':true,
        'board--playable': !gameState.gameover,
        'board--turn-1': gameState.turn === 1,
        'board--turn-2': gameState.turn === 2
        }"
      >
      <div
        v-for="(column, columnIndex) in gameState.board"
        :key="`board-column-${columnIndex}`"
        :class="{
          'column': true,
          'column--full': !gameState.columnsPlayable,
          'column--highlighted': highlightedColumn === columnIndex
        }"
        @touchend="handleColumnTouchend(columnIndex)"
        @click="handleColumnClick(columnIndex)"
        >
          <div
            v-for="(cell, cellIndex) in column"
            :key="`board-column-${columnIndex}-cell-${cellIndex}`"
            :data-position="`${columnIndex}x${cellIndex}`"
            :class="{
              'cell':true,
              'cell--winner':cell.winner
            }"
            >
            <span :class="{
              'piece': true,
              'piece--1': cell.value === 1,
              'piece--2': cell.value === 2
            }">&nbsp;</span>
          </div>
          <div
            class="cell cell--preview"
            >
            <span :class="{
              'piece': true,
              'piece--1': gameState.turn === 1,
              'piece--2': gameState.turn === 2
            }">&nbsp;</span>
          </div>
      </div>
    </div>
    <div class="won">
      <p>{{ gameState.turn === 1 ? 'Red' : 'Yellow' }} wins!</p>
      <button class="startgame" @click="startgame">Play again</button>
    </div>
  </div>
</template>

<script>
import { play } from 'connect4-core'

export default {
  data() {
    return {
      gameState: null,
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
  methods: {
    startgame() {
      const {cols,rows} = this
      this.gameState = play({cols,rows})
    },
    handleColumnTouchend(columnIndex) {
      this.touchEnabled = true

      if (!this.columnIsPlayable(columnIndex)) {
        return
      }

      if (this.highlightedColumn !== columnIndex) {
        this.highlightedColumn = columnIndex
      } else {
        this.highlightedColumn = null
        this.makeMove(columnIndex)
      }

    },
    handleColumnClick(columnIndex) {
      if (this.touchEnabled) {
        this.touchEnabled = false
        return
      }

      if (!this.columnIsPlayable(columnIndex)) {
        return
      }

      this.makeMove(columnIndex)
    },
    makeMove(col) {
      this.$set(this,'gameState',play(this.gameState, col))
    },
    columnIsPlayable(columnIndex) {
      const {gameState: {gameover, columnsPlayable }} = this
      return !(gameover || !columnsPlayable[columnIndex])
    }
  },
  created() {
    this.startgame()
  }
}
</script>

<script setup>
import { ref } from 'vue'
import Chekerpieces from './Chekerpieces.vue' // ✅ import component

const BOARD_SIZE = 8

const createBoard = () => {
  const board = []
  for (let row = 0; row < BOARD_SIZE; row++) {
    const rowCells = []
    for (let col = 0; col < BOARD_SIZE; col++) {
      const isDark = (row + col) % 2 === 1

      rowCells.push({
        row,
        col,
        isDark
      })
    }
    board.push(rowCells)
  }
  return board
}

const board = ref(createBoard())

// ✅ ADD PIECES LOGIC HERE
const pieces = ref([])

const setupPieces = () => {
  const temp = []

  board.value.forEach(row => {
    row.forEach(cell => {
      // RED (top)
      if (cell.isDark && cell.row <= 2) {
        temp.push({
          row: cell.row,
          col: cell.col,
          color: 'red',
          king: false
        })
      }

      // BLACK (bottom)
      if (cell.isDark && cell.row >= 5) {
        temp.push({
          row: cell.row,
          col: cell.col,
          color: 'black',
          king: false
        })
      }
    })
  })

  pieces.value = temp
}

setupPieces()
</script>

<template>
  <div class="game-container">
    <h1>Checkers!</h1>

    <div class="board">
      <div v-for="(row, rowIndex) in board" :key="rowIndex" class="row">
        <div
          v-for="cell in row"
          :key="`${cell.row}-${cell.col}`"
          class="cell"
          :class="{
            dark: cell.isDark,
            light: !cell.isDark
          }"
        >
          <!-- ✅ THIS LINE WAS MISSING -->
          <Chekerpieces :cell="cell" :pieces="pieces" />
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.game-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20px;
}

.board {
  border: 4px solid #333;
}

.row {
  display: flex;
}

.cell {
  width: 60px;
  height: 60px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.cell.light {
  background-color: #f0dec1;
}

.cell.dark {
  background-color: #b58863;
}
</style>
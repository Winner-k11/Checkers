<script setup>
import { ref } from 'vue'

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

const pieces = ref([])

// initialize pieces
const setupPieces = () => {
  const temp = []

  board.value.forEach(row => {
    row.forEach(cell => {
      // RED pieces (top)
      if (cell.isDark && cell.row <= 2) {
        temp.push({
          row: cell.row,
          col: cell.col,
          color: 'red',
          king: false
        })
      }

      // BLACK pieces (bottom)
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
    'dark': cell.isDark,
    'light': !cell.isDark
  }"
>

  <!-- LOOP THROUGH PIECES -->
  <div
    v-for="piece in pieces"
    :key="piece.row + '-' + piece.col"
  >
    <div
      v-if="piece.row === cell.row && piece.col === cell.col"
      :class="['piece', piece.color, { king: piece.king }]"
    >
      <span v-if="piece.king">👑</span>
    </div>
  </div>

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
  font-family: Arial, sans-serif;
}

h1 {
  margin-bottom: 20px;
  color: #333;
}

.board {
  border: 4px solid #333;
  display: inline-block;
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
  cursor: pointer;
}

.cell.light {
  background-color: #f0d9b5;
}

.cell.dark {
  background-color: #b58863;
}


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
  background-color: #f0d9b5;
}

.cell.dark {
  background-color: #b58863;
}

/* ✅ YOUR PIECE STYLES */
.piece {
  width: 40px;
  height: 40px;
  border-radius: 50%;
}

.red {
  background-color: red;
}

.black {
  background-color: black;
}

.king {
  border: 3px solid gold;
}


</style>

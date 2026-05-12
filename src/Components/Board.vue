<script setup>
import { ref } from 'vue'
import Chekerpieces from './Chekerpieces.vue' // ✅ import component

const pieces = ref([])

const setupPieces = () => {
  const temp = []

  board.value.forEach(row => {
    row.forEach(cell => {
      if (cell.isDark && cell.row <= 2) {
        temp.push({
          row: cell.row,
          col: cell.col,
          color: 'red',
          king: false
        })
      }

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




<div
  v-for="cell in row"
  :key="`${cell.row}-${cell.col}`"
  class="cell"
  :class="{
    dark: cell.isDark,
    light: !cell.isDark
  }"
>
  <Chekerpieces :cell="cell" :pieces="pieces" />
</div>

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
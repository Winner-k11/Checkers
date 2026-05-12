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
        />
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
  background-color: #f0dec1;
}

.cell.dark {
  background-color: #b58863;
}

</style>
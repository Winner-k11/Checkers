<script setup>
import { computed, ref } from 'vue'

const BOARD_SIZE = 8
const currentTurn = ref('red')
const selectedPiece = ref(null)
const pieces = ref([])
const forceJump = ref(true)

const board = computed(() => {
  return Array.from({ length: BOARD_SIZE }, (_, row) =>
    Array.from({ length: BOARD_SIZE }, (_, col) => ({
      row,
      col,
      isDark: (row + col) % 2 === 1
    }))
  )
})

const resetPieces = () => {
  const temp = []

  for (let row = 0; row < BOARD_SIZE; row++) {
    for (let col = 0; col < BOARD_SIZE; col++) {
      if ((row + col) % 2 === 1) {
        if (row <= 2) {
          temp.push({ row, col, color: 'red', king: false })
        }
        if (row >= 5) {
          temp.push({ row, col, color: 'black', king: false })
        }
      }
    }
  }

  pieces.value = temp
}

resetPieces()

const getPieceAt = (row, col) => {
  return pieces.value.find(piece => piece.row === row && piece.col === col)
}

const canCapture = piece => {
  const directions = piece.king
    ? [[1, 1], [1, -1], [-1, 1], [-1, -1]]
    : piece.color === 'red'
      ? [[1, 1], [1, -1]]
      : [[-1, 1], [-1, -1]]

  for (const [dr, dc] of directions) {
    const jumpRow = piece.row + dr * 2
    const jumpCol = piece.col + dc * 2
    if (jumpRow >= 0 && jumpRow < 8 && jumpCol >= 0 && jumpCol < 8) {
      if (!getPieceAt(jumpRow, jumpCol)) {
        const enemyRow = piece.row + dr
        const enemyCol = piece.col + dc
        const enemy = getPieceAt(enemyRow, enemyCol)
        if (enemy && enemy.color !== piece.color) {
          return true
        }
      }
    }
  }

  return false
}

const anyCaptureAvailable = computed(() => {
  return pieces.value.some(piece => piece.color === currentTurn.value && canCapture(piece))
})

const isSelected = piece => {
  return selectedPiece.value === piece
}

const selectPiece = piece => {
  if (piece.color !== currentTurn.value) {
    return
  }

  selectedPiece.value = selectedPiece.value === piece ? null : piece
}

const canMoveTo = (row, col) => {
  if (!selectedPiece.value) {
    return false
  }

  if (getPieceAt(row, col)) {
    return false
  }

  const rowDiff = row - selectedPiece.value.row
  const colDiff = col - selectedPiece.value.col
  const absRow = Math.abs(rowDiff)
  const absCol = Math.abs(colDiff)

  if (absRow !== absCol) {
    return false // Must be diagonal
  }

  const requiresCapture = forceJump.value && anyCaptureAvailable.value

  if (selectedPiece.value.king) {
    // Kings can move one square diagonally or jump
    if (absRow === 1 && absCol === 1) {
      return !requiresCapture
    }
    if (absRow === 2 && absCol === 2) {
      const enemyRow = selectedPiece.value.row + rowDiff / 2
      const enemyCol = selectedPiece.value.col + colDiff / 2
      const enemy = getPieceAt(enemyRow, enemyCol)
      return !!enemy && enemy.color !== selectedPiece.value.color
    }
  } else {
    // Regular pieces
    if (selectedPiece.value.color === 'red') {
      if (rowDiff === 1 && absCol === 1) {
        return !requiresCapture
      }
      if (rowDiff === 2 && absCol === 2) {
        const enemy = getPieceAt(selectedPiece.value.row + 1, selectedPiece.value.col + colDiff / 2)
        return !!enemy && enemy.color === 'black'
      }
    }

    if (selectedPiece.value.color === 'black') {
      if (rowDiff === -1 && absCol === 1) {
        return !requiresCapture
      }
      if (rowDiff === -2 && absCol === 2) {
        const enemy = getPieceAt(selectedPiece.value.row - 1, selectedPiece.value.col + colDiff / 2)
        return !!enemy && enemy.color === 'red'
      }
    }
  }

  return false
}

const hasMoreCaptures = (piece) => {
  const directions = piece.king
    ? [[1, 1], [1, -1], [-1, 1], [-1, -1]]
    : piece.color === 'red'
      ? [[1, 1], [1, -1]]
      : [[-1, 1], [-1, -1]]

  for (const [dr, dc] of directions) {
    const jumpRow = piece.row + dr * 2
    const jumpCol = piece.col + dc * 2
    if (jumpRow >= 0 && jumpRow < 8 && jumpCol >= 0 && jumpCol < 8) {
      if (!getPieceAt(jumpRow, jumpCol)) {
        const enemyRow = piece.row + dr
        const enemyCol = piece.col + dc
        const enemy = getPieceAt(enemyRow, enemyCol)
        if (enemy && enemy.color !== piece.color) {
          return true
        }
      }
    }
  }
  return false
}

const movePiece = cell => {
  if (!selectedPiece.value) {
    return
  }

  if (!canMoveTo(cell.row, cell.col)) {
    return
  }

  const rowDiff = cell.row - selectedPiece.value.row
  const colDiff = cell.col - selectedPiece.value.col

  let captured = false
  if (Math.abs(rowDiff) === 2) {
    const enemyRow = selectedPiece.value.row + rowDiff / 2
    const enemyCol = selectedPiece.value.col + colDiff / 2
    const enemy = getPieceAt(enemyRow, enemyCol)

    if (enemy) {
      const index = pieces.value.indexOf(enemy)
      if (index !== -1) {
        pieces.value.splice(index, 1)
        captured = true
      }
    }
  }

  selectedPiece.value.row = cell.row
  selectedPiece.value.col = cell.col

  // Promote to king if reached opponent's side
  if (!selectedPiece.value.king) {
    if (selectedPiece.value.color === 'red' && cell.row === 7) {
      selectedPiece.value.king = true
    } else if (selectedPiece.value.color === 'black' && cell.row === 0) {
      selectedPiece.value.king = true
    }
  }

  // If captured and more captures possible, keep turn
  if (captured && hasMoreCaptures(selectedPiece.value)) {
    // Don't switch turn, keep selected
  } else {
    currentTurn.value = currentTurn.value === 'red' ? 'black' : 'red'
    selectedPiece.value = null
  }
}
</script>

<template>
  <div class="movement-game">
    <div class="status-row">
      <div class="status-bar">{{ currentTurn.charAt(0).toUpperCase() + currentTurn.slice(1) }} Turn</div>
      <div class="force-control">
        <label>
          <input type="checkbox" v-model="forceJump" /> Force jump
        </label>
        <div class="force-note" v-if="forceJump">
          {{ anyCaptureAvailable ? 'Capture is required when available' : 'No captures available right now' }}
        </div>
      </div>
    </div>

    <div class="board">
      <div v-for="(row, rowIndex) in board" :key="rowIndex" class="row">
        <div
          v-for="cell in row"
          :key="`${cell.row}-${cell.col}`"
          class="square"
          :class="{
            dark: cell.isDark,
            light: !cell.isDark,
            valid: canMoveTo(cell.row, cell.col)
          }"
          @click="movePiece(cell)"
        >
          <div
            v-if="getPieceAt(cell.row, cell.col)"
            class="piece"
            :class="[getPieceAt(cell.row, cell.col).color, { selected: isSelected(getPieceAt(cell.row, cell.col)), king: getPieceAt(cell.row, cell.col).king }]"
            @click.stop="selectPiece(getPieceAt(cell.row, cell.col))"
          >
            <span v-if="getPieceAt(cell.row, cell.col).king" class="crown">👑</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.movement-game {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 16px;
  padding: 16px;
}

.status-bar {
  font-size: 1.25rem;
  font-weight: 600;
}

.board {
  display: inline-grid;
  grid-template-columns: repeat(8, 60px);
  gap: 0;
  border: 3px solid #333;
}

.row {
  display: contents;
}

.square {
  width: 60px;
  height: 60px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: background-color 0.15s ease;
}

.light {
  background-color: #f0dec1;
}

.dark {
  background-color: #b58863;
}

.valid {
  box-shadow: 0 0 15px rgba(0, 255, 0, 0.6), inset 0 0 0 3px rgba(0, 255, 0, 0.3);
}

.status-row {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 16px;
  align-items: center;
}

.force-control {
  display: flex;
  flex-direction: column;
  align-items: center;
  font-size: 0.95rem;
}

.force-control label {
  display: flex;
  align-items: center;
  gap: 8px;
}

.force-note {
  margin-top: 4px;
  font-size: 0.9rem;
  color: #444;
}

.piece {
  width: 44px;
  height: 44px;
  border-radius: 50%;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.15);
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
}

.red {
  background-color: #d92f2f;
}

.black {
  background-color: #1f1f1f;
}

.selected {
  outline: 3px solid #ffd54f;
  outline-offset: 2px;
}

.crown {
  position: absolute;
  top: -5px;
  left: 50%;
  transform: translateX(-50%);
  font-size: 1.2rem;
}
</style>
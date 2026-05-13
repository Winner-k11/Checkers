<script setup>
import { computed, ref, watch } from 'vue'

const BOARD_SIZE = 8
const currentTurn = ref('red')
const selectedPiece = ref(null)
const pieces = ref([])
const forceJump = ref(true)
const redCaptured = ref(0)
const blackCaptured = ref(0)
const moveCount = ref(0)

defineProps(['redScore', 'blackScore'])
const emit = defineEmits(['game-over'])

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
  redCaptured.value = 0
  blackCaptured.value = 0
  moveCount.value = 0
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
        // Track captured pieces
        if (selectedPiece.value.color === 'red') {
          redCaptured.value++
        } else {
          blackCaptured.value++
        }
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
  moveCount.value++
}

const checkGameOver = () => {
  const redPieces = pieces.value.filter(p => p.color === 'red')
  const blackPieces = pieces.value.filter(p => p.color === 'black')

  // If no pieces left, opponent wins
  if (redPieces.length === 0) {
    return 'black'
  }
  if (blackPieces.length === 0) {
    return 'red'
  }

  // Check if current player has any valid moves
  const currentPlayerPieces = pieces.value.filter(p => p.color === currentTurn.value)
  const hasValidMove = currentPlayerPieces.some(piece => {
    const directions = piece.king
      ? [[1, 1], [1, -1], [-1, 1], [-1, -1]]
      : piece.color === 'red'
        ? [[1, 1], [1, -1]]
        : [[-1, 1], [-1, -1]]

    for (const [dr, dc] of directions) {
      for (let i = 1; i <= 2; i++) {
        const targetRow = piece.row + dr * i
        const targetCol = piece.col + dc * i
        if (targetRow >= 0 && targetRow < 8 && targetCol >= 0 && targetCol < 8) {
          if (i === 1 && !getPieceAt(targetRow, targetCol)) {
            return true
          }
          if (i === 2) {
            const midRow = piece.row + dr
            const midCol = piece.col + dc
            const enemy = getPieceAt(midRow, midCol)
            const target = getPieceAt(targetRow, targetCol)
            if (!target && enemy && enemy.color !== piece.color) {
              return true
            }
          }
        }
      }
    }
    return false
  })

  if (!hasValidMove) {
    return currentTurn.value === 'red' ? 'black' : 'red'
  }

  return null
}

// Watch for game over condition
watch([pieces, currentTurn], () => {
  const winner = checkGameOver()
  if (winner) {
    setTimeout(() => {
      emit('game-over', winner)
    }, 500)
  }
}, { deep: true })
</script>

<template>
  <div class="game-layout">
    <!-- LEFT PANEL - GAME STATS -->
    <div class="side-panel left-panel">
      <div class="panel-section stats-section">
        <h3 class="panel-title">STATISTICS</h3>
        <div class="stat-box">
          <div class="stat-row">
            <span class="stat-icon">🔴</span>
            <div class="stat-info">
              <div class="stat-label">Red</div>
              <div class="stat-content">{{ pieces.filter(p => p.color === 'red').length }} pieces</div>
            </div>
          </div>
          <div class="stat-row">
            <span class="stat-icon">⚫</span>
            <div class="stat-info">
              <div class="stat-label">Black</div>
              <div class="stat-content">{{ pieces.filter(p => p.color === 'black').length }} pieces</div>
            </div>
          </div>
        </div>

        <div class="stat-box">
          <div class="stat-row">
            <span class="stat-label">Captured</span>
          </div>
          <div class="stat-sub-row">
            <span class="capture-badge red">⚫ {{ redCaptured }}</span>
            <span class="capture-badge black">● {{ blackCaptured }}</span>
          </div>
        </div>

        <div class="stat-box">
          <div class="stat-label">Move #{{ moveCount }}</div>
        </div>
      </div>

      <div class="panel-section">
        <h3 class="panel-title">SESSION</h3>
        <div class="session-info">
          <div class="session-item">
            <div class="session-label">Red Wins</div>
            <div class="session-value red">{{ redScore }}</div>
          </div>
          <div class="session-item">
            <div class="session-label">Black Wins</div>
            <div class="session-value black">{{ blackScore }}</div>
          </div>
        </div>
      </div>
    </div>

    <!-- CENTER - GAME BOARD -->
    <div class="center-container">
      <!-- TOP BAR -->
      <div class="top-bar">
        <div class="turn-display" :class="currentTurn">
          {{ currentTurn.charAt(0).toUpperCase() + currentTurn.slice(1) }}'s Turn
        </div>
      </div>

      <!-- BOARD -->
      <div class="board-wrapper">
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
                <span v-if="getPieceAt(cell.row, cell.col).king" class="crown">♔</span>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- CONTROLS BAR -->
      <div class="controls-bar">
        <label class="control-item">
          <input type="checkbox" v-model="forceJump" />
          <span>Force Jump</span>
        </label>
        <div class="control-status">
          {{ anyCaptureAvailable ? '⚠️ Capture!' : '✓ OK' }}
        </div>
      </div>
    </div>

    <!-- RIGHT PANEL - FEATURES -->
    <div class="side-panel right-panel">
      <div class="panel-section">
        <h3 class="panel-title">GAME INFO</h3>
        <div class="info-box">
          <div class="info-item">
            <span class="info-label">Mode</span>
            <span class="info-value">{{ gameMode }}</span>
          </div>
          <div class="info-item">
            <span class="info-label">Difficulty</span>
            <span class="info-value">{{ gameDifficulty }}</span>
          </div>
        </div>
      </div>

      <div class="panel-section">
        <h3 class="panel-title">FEATURES</h3>
        <div class="feature-list">
          <div class="feature-item">
            <span class="feature-icon">🔄</span>
            <span class="feature-text">Dynamic Play</span>
          </div>
          <div class="feature-item">
            <span class="feature-icon">👑</span>
            <span class="feature-text">King Pieces</span>
          </div>
          <div class="feature-item">
            <span class="feature-icon">📊</span>
            <span class="feature-text">Stats Track</span>
          </div>
          <div class="feature-item">
            <span class="feature-icon">🏆</span>
            <span class="feature-text">Achievements</span>
          </div>
        </div>
      </div>

      <div class="panel-section">
        <h3 class="panel-title">QUICK TIPS</h3>
        <div class="tips-box">
          <div class="tip">• Control center</div>
          <div class="tip">• Capture pieces</div>
          <div class="tip">• Reach the end</div>
          <div class="tip">• Get a King!</div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
* {
  box-sizing: border-box;
}

.game-layout {
  display: flex;
  gap: 0;
  height: 100vh;
  background: #f5f5f5;
  overflow: hidden;
}

/* SIDE PANELS */
.side-panel {
  width: 280px;
  background: #ffffff;
  border-right: 1px solid #e0e0e0;
  padding: 20px;
  overflow-y: auto;
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.left-panel {
  border-right: 1px solid #ddd;
}

.right-panel {
  border-left: 1px solid #ddd;
  border-right: none;
}

.panel-title {
  font-size: 0.85rem;
  font-weight: 700;
  color: #333;
  text-transform: uppercase;
  letter-spacing: 1px;
  margin: 0 0 12px 0;
  border-bottom: 2px solid #000;
  padding-bottom: 8px;
}

.panel-section {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

/* STATS */
.stats-section {
  gap: 10px;
}

.stat-box {
  background: #f9f9f9;
  border: 1px solid #e0e0e0;
  border-radius: 6px;
  padding: 12px;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.stat-row {
  display: flex;
  align-items: center;
  gap: 10px;
  font-size: 0.9rem;
}

.stat-icon {
  font-size: 1.2rem;
  min-width: 25px;
}

.stat-info {
  flex: 1;
}

.stat-label {
  font-size: 0.75rem;
  color: #666;
  text-transform: uppercase;
  font-weight: 600;
}

.stat-content {
  font-weight: 700;
  color: #000;
  font-size: 0.9rem;
}

.stat-sub-row {
  display: flex;
  gap: 8px;
  font-size: 0.85rem;
}

.capture-badge {
  flex: 1;
  padding: 6px 8px;
  border-radius: 4px;
  font-weight: 600;
  text-align: center;
  background: #f0f0f0;
}

.capture-badge.red {
  color: #c41e3a;
  border: 1px solid #ddd;
}

.capture-badge.black {
  color: #333;
  border: 1px solid #ddd;
}

/* SESSION */
.session-info {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.session-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px;
  background: #f9f9f9;
  border-radius: 4px;
  border: 1px solid #e0e0e0;
}

.session-label {
  font-size: 0.85rem;
  color: #666;
  font-weight: 600;
}

.session-value {
  font-size: 1.3rem;
  font-weight: 700;
}

.session-value.red {
  color: #c41e3a;
}

.session-value.black {
  color: #000;
}

/* INFO BOX */
.info-box {
  background: #f9f9f9;
  border: 1px solid #e0e0e0;
  border-radius: 6px;
  padding: 12px;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.info-item {
  display: flex;
  justify-content: space-between;
  font-size: 0.85rem;
}

.info-label {
  color: #666;
  font-weight: 600;
  text-transform: uppercase;
}

.info-value {
  color: #000;
  font-weight: 700;
}

/* FEATURES */
.feature-list {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.feature-item {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 8px 10px;
  background: #f9f9f9;
  border-radius: 4px;
  border: 1px solid #e0e0e0;
  font-size: 0.85rem;
  color: #333;
  font-weight: 600;
}

.feature-icon {
  font-size: 1.1rem;
}

.feature-text {
  flex: 1;
}

/* TIPS */
.tips-box {
  background: #f9f9f9;
  border: 1px solid #e0e0e0;
  border-radius: 6px;
  padding: 12px;
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.tip {
  font-size: 0.8rem;
  color: #555;
  line-height: 1.4;
}

/* CENTER CONTAINER */
.center-container {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 10px;
  padding: 15px;
  background: #f5f5f5;
}

/* TOP BAR */
.top-bar {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 50px;
  background: #ffffff;
  border: 1px solid #ddd;
  border-radius: 6px;
}

.turn-display {
  font-size: 1.1rem;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 1px;
  padding: 10px 20px;
  border-radius: 4px;
  min-width: 150px;
  text-align: center;
}

.turn-display.red {
  background: #fff5f5;
  color: #c41e3a;
  border: 2px solid #c41e3a;
}

.turn-display.black {
  background: #f5f5f5;
  color: #000;
  border: 2px solid #000;
}

/* BOARD */
.board-wrapper {
  flex: 1;
  display: flex;
  justify-content: center;
  align-items: center;
  background: #fff;
  border: 1px solid #ddd;
  border-radius: 6px;
  padding: 10px;
}

.board {
  display: inline-grid;
  grid-template-columns: repeat(8, 1fr);
  gap: 0;
  border: 3px solid #333;
  background: #fff;
}

.row {
  display: contents;
}

.square {
  aspect-ratio: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.15s ease;
  position: relative;
  min-width: 45px;
  min-height: 45px;
}

.light {
  background: #f0f0f0;
}

.dark {
  background: #999;
}

.square.valid {
  animation: pulse 0.6s ease-in-out infinite;
}

.square.valid::before {
  content: '';
  position: absolute;
  width: 8px;
  height: 8px;
  background: #4a7c59;
  border-radius: 50%;
  box-shadow: 0 0 8px rgba(74, 124, 89, 0.6);
}

@keyframes pulse {
  0%, 100% {
    box-shadow: inset 0 0 0 2px rgba(74, 124, 89, 0.4);
  }
  50% {
    box-shadow: inset 0 0 0 2px rgba(74, 124, 89, 0.8);
  }
}

.piece {
  width: 85%;
  height: 85%;
  border-radius: 50%;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.25);
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s ease;
  cursor: grab;
  border: 2px solid;
}

.piece:hover {
  transform: scale(1.08);
  box-shadow: 0 6px 12px rgba(0, 0, 0, 0.3);
}

.piece:active {
  cursor: grabbing;
}

.red {
  background: linear-gradient(135deg, #e57373 0%, #c41e3a 100%);
  border-color: #b71c1c;
}

.black {
  background: linear-gradient(135deg, #424242 0%, #000 100%);
  border-color: #000;
}

.selected {
  outline: 3px solid #ffb300;
  outline-offset: 2px;
  transform: scale(1.12);
  box-shadow: 0 6px 14px rgba(255, 179, 0, 0.4), 0 4px 8px rgba(0, 0, 0, 0.25);
}

.crown {
  position: absolute;
  top: -6px;
  left: 50%;
  transform: translateX(-50%);
  font-size: 1.2rem;
  text-shadow: 0 2px 4px rgba(0, 0, 0, 0.3);
}

/* CONTROLS BAR */
.controls-bar {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 20px;
  height: 45px;
  background: #ffffff;
  border: 1px solid #ddd;
  border-radius: 6px;
  padding: 0 15px;
}

.control-item {
  display: flex;
  align-items: center;
  gap: 8px;
  font-weight: 600;
  font-size: 0.9rem;
  color: #333;
  cursor: pointer;
  user-select: none;
}

.control-item input[type="checkbox"] {
  width: 16px;
  height: 16px;
  cursor: pointer;
  accent-color: #000;
}

.control-status {
  font-weight: 700;
  font-size: 0.9rem;
  color: #333;
}

/* SCROLLBAR */
.side-panel::-webkit-scrollbar {
  width: 6px;
}

.side-panel::-webkit-scrollbar-track {
  background: #f1f1f1;
}

.side-panel::-webkit-scrollbar-thumb {
  background: #ccc;
  border-radius: 3px;
}

.side-panel::-webkit-scrollbar-thumb:hover {
  background: #999;
}

/* RESPONSIVE */
@media (max-width: 1400px) {
  .side-panel {
    width: 240px;
    padding: 15px;
  }

  .square {
    min-width: 40px;
    min-height: 40px;
  }
}

@media (max-width: 1200px) {
  .side-panel {
    width: 220px;
    padding: 12px;
    gap: 15px;
  }

  .square {
    min-width: 35px;
    min-height: 35px;
  }

  .panel-title {
    font-size: 0.8rem;
    margin-bottom: 10px;
  }
}
</style>
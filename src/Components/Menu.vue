<script setup>
import { ref } from 'vue'

const showRules = ref(false)

defineEmits(['start-game'])
</script>

<template>
  <div class="menu-container">
    <div class="menu-card">
      <div class="menu-header">
        <h1 class="title">♟ CHECKERS ♟</h1>
        <p class="subtitle">Master the Classic Board Game</p>
      </div>

      <div v-if="!showRules" class="menu-content">
        <p class="welcome-text">
          Challenge yourself in an exciting game of strategy and tactics!
        </p>

        <div class="button-group">
          <button class="btn btn-play" @click="$emit('start-game')">
            <span class="btn-icon">▶</span>
            START GAME
          </button>
          <button class="btn btn-rules" @click="showRules = true">
            <span class="btn-icon">📋</span>
            HOW TO PLAY
          </button>
        </div>

        <div class="features">
          <h3>Features:</h3>
          <ul>
            <li>🎮 Strategic gameplay</li>
            <li>📊 Score tracking</li>
            <li>🏆 Achievement system</li>
            <li>♔ Piece promotion</li>
          </ul>
        </div>
      </div>

      <div v-else class="rules-content">
        <h2>How to Play Checkers</h2>
        
        <div class="rules-text">
          <h3>Objective:</h3>
          <p>Capture all of your opponent's pieces or block them from moving.</p>

          <h3>Movement:</h3>
          <ul>
            <li>Pieces move diagonally forward to empty squares</li>
            <li>When a piece reaches the opposite end, it's promoted to a King</li>
            <li>Kings can move diagonally in any direction</li>
          </ul>

          <h3>Capturing:</h3>
          <ul>
            <li>Jump over an opponent's piece to capture it</li>
            <li>If multiple jumps are available, you must make them</li>
            <li>Continue capturing with the same piece if possible</li>
          </ul>

          <h3>Winning:</h3>
          <p>You win by capturing all opponent pieces or blocking all their moves!</p>
        </div>

        <button class="btn btn-back" @click="showRules = false">
          <span class="btn-icon">←</span>
          BACK
        </button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.menu-container {
  width: 100%;
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 20px;
  background: #f5f5f5;
}

.menu-card {
  background: white;
  border-radius: 10px;
  padding: 50px 40px;
  max-width: 500px;
  width: 100%;
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.1);
  animation: slideIn 0.5s ease-out;
  border: 1px solid #ddd;
}

@keyframes slideIn {
  from {
    opacity: 0;
    transform: translateY(30px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.menu-header {
  text-align: center;
  margin-bottom: 40px;
}

.title {
  font-size: 3.5rem;
  font-weight: 800;
  color: #000;
  margin-bottom: 10px;
  letter-spacing: 2px;
}

.subtitle {
  font-size: 1.1rem;
  color: #666;
  font-weight: 500;
}

.menu-content {
  display: flex;
  flex-direction: column;
  gap: 30px;
}

.welcome-text {
  font-size: 1.05rem;
  color: #555;
  text-align: center;
  line-height: 1.6;
}

.button-group {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.btn {
  padding: 15px 30px;
  font-size: 1.1rem;
  font-weight: 700;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
  letter-spacing: 1px;
}

.btn-icon {
  font-size: 1.3rem;
}

.btn-play {
  background: #000;
  color: white;
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.2);
}

.btn-play:hover {
  transform: translateY(-3px);
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
}

.btn-play:active {
  transform: translateY(-1px);
}

.btn-rules {
  background: #f5f5f5;
  color: #333;
  border: 2px solid #ddd;
}

.btn-rules:hover {
  background: #efefef;
  border-color: #000;
  color: #000;
}

.btn-back {
  background: #000;
  color: white;
  width: 100%;
  margin-top: 20px;
}

.btn-back:hover {
  transform: translateY(-3px);
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.2);
}

.features {
  background: #f9f9f9;
  padding: 20px;
  border-radius: 8px;
  border-left: 4px solid #000;
}

.features h3 {
  color: #333;
  margin-bottom: 12px;
  font-size: 1rem;
}

.features ul {
  list-style: none;
  padding: 0;
}

.features li {
  color: #666;
  padding: 6px 0;
  font-size: 0.95rem;
}

.rules-content {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.rules-content h2 {
  color: #000;
  font-size: 1.8rem;
  text-align: center;
}

.rules-text {
  max-height: 400px;
  overflow-y: auto;
  padding-right: 10px;
}

.rules-text h3 {
  color: #333;
  margin: 15px 0 8px 0;
  font-size: 1.05rem;
}

.rules-text p,
.rules-text ul {
  color: #555;
  margin-bottom: 12px;
  line-height: 1.6;
}

.rules-text ul {
  padding-left: 20px;
}

.rules-text li {
  margin-bottom: 6px;
}

/* Scrollbar styling */
.rules-text::-webkit-scrollbar {
  width: 6px;
}

.rules-text::-webkit-scrollbar-track {
  background: #f1f1f1;
  border-radius: 10px;
}

.rules-text::-webkit-scrollbar-thumb {
  background: #999;
  border-radius: 10px;
}

.rules-text::-webkit-scrollbar-thumb:hover {
  background: #666;
}
</style>

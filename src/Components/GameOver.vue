<script setup>
import { computed } from 'vue'

defineProps({
  winner: String,
  redScore: Number,
  blackScore: Number
})

defineEmits(['play-again', 'return-home'])

const getAchievements = (color, score) => {
  const achievements = []
  
  if (score === 1) {
    achievements.push('🏆 First Victory!')
  }
  if (score === 3) {
    achievements.push('⭐ Winning Streak')
  }
  if (score === 5) {
    achievements.push('👑 Champion')
  }
  
  return achievements
}

const winnerAchievements = computed(() => {
  return getAchievements(winner, winner === 'red' ? redScore : blackScore)
})
</script>

<template>
  <div class="gameover-container">
    <div class="gameover-card">
      <div class="header">
        <h1 class="title">GAME OVER!</h1>
      </div>

      <div class="winner-section" :class="winner">
        <div class="winner-text">
          <span class="winner-name">{{ winner.toUpperCase() }}</span>
          <span class="winner-title">WINS!</span>
        </div>
        <div class="trophy">🏆</div>
      </div>

      <div class="score-board">
        <div class="score-item red">
          <div class="score-label">Red Player</div>
          <div class="score-value">{{ redScore }}</div>
        </div>
        <div class="vs">VS</div>
        <div class="score-item black">
          <div class="score-label">Black Player</div>
          <div class="score-value">{{ blackScore }}</div>
        </div>
      </div>

      <div v-if="winnerAchievements.length > 0" class="achievements">
        <h3>🎖 Achievements Unlocked!</h3>
        <div class="achievement-list">
          <div v-for="(achievement, index) in winnerAchievements" :key="index" class="achievement-badge">
            {{ achievement }}
          </div>
        </div>
      </div>

      <div class="button-group">
        <button class="btn btn-play-again" @click="$emit('play-again')">
          <span class="btn-icon">🔄</span>
          PLAY AGAIN
        </button>
        <button class="btn btn-menu" @click="$emit('return-home')">
          <span class="btn-icon">🏠</span>
          BACK TO MENU
        </button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.gameover-container {
  width: 100%;
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 20px;
  background: #f5f5f5;
  animation: fadeIn 0.4s ease-out;
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

.gameover-card {
  background: white;
  border-radius: 10px;
  padding: 50px 40px;
  max-width: 550px;
  width: 100%;
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.1);
  animation: slideUp 0.5s ease-out;
  border: 1px solid #ddd;
}

@keyframes slideUp {
  from {
    opacity: 0;
    transform: translateY(50px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.header {
  text-align: center;
  margin-bottom: 30px;
}

.title {
  font-size: 2.5rem;
  font-weight: 800;
  color: #000;
  letter-spacing: 2px;
}

.winner-section {
  text-align: center;
  margin-bottom: 40px;
  padding: 25px;
  border-radius: 8px;
  background: #f9f9f9;
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 20px;
  border: 2px solid #ddd;
}

.winner-section.red {
  border-left: 5px solid #c41e3a;
}

.winner-section.black {
  border-left: 5px solid #000;
}

.winner-text {
  display: flex;
  flex-direction: column;
  gap: 5px;
  flex: 1;
}

.winner-name {
  font-size: 2rem;
  font-weight: 800;
  color: #000;
}

.winner-section.red .winner-name {
  color: #c41e3a;
}

.winner-title {
  font-size: 1.3rem;
  font-weight: 700;
  color: #666;
}

.trophy {
  font-size: 4rem;
  animation: bounce 1s infinite;
}

@keyframes bounce {
  0%, 100% {
    transform: translateY(0);
  }
  50% {
    transform: translateY(-10px);
  }
}

.score-board {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 30px;
  gap: 15px;
}

.score-item {
  flex: 1;
  padding: 20px;
  border-radius: 8px;
  text-align: center;
  background: #f9f9f9;
  border: 2px solid #ddd;
}

.score-item.red {
  border-color: #c41e3a;
}

.score-item.black {
  border-color: #000;
}

.score-label {
  font-size: 0.85rem;
  color: #666;
  margin-bottom: 8px;
  text-transform: uppercase;
  font-weight: 600;
}

.score-value {
  font-size: 2.5rem;
  font-weight: 800;
  color: #000;
}

.score-item.red .score-value {
  color: #c41e3a;
}

.vs {
  color: #999;
  font-weight: 700;
  font-size: 0.9rem;
}

.achievements {
  background: #fff9e6;
  padding: 20px;
  border-radius: 8px;
  margin-bottom: 30px;
  text-align: center;
  border: 2px solid #ffd700;
}

.achievements h3 {
  color: #333;
  margin-bottom: 15px;
  font-size: 1.1rem;
}

.achievement-list {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.achievement-badge {
  background: white;
  padding: 12px 20px;
  border-radius: 6px;
  font-weight: 600;
  color: #333;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.1);
  animation: popIn 0.4s ease-out;
}

@keyframes popIn {
  from {
    opacity: 0;
    transform: scale(0.8);
  }
  to {
    opacity: 1;
    transform: scale(1);
  }
}

.button-group {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.btn {
  padding: 15px 30px;
  font-size: 1.05rem;
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
  font-size: 1.2rem;
}

.btn-play-again {
  background: #000;
  color: white;
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.2);
}

.btn-play-again:hover {
  transform: translateY(-3px);
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
}

.btn-menu {
  background: #f5f5f5;
  color: #333;
  border: 2px solid #ddd;
}

.btn-menu:hover {
  background: #efefef;
  border-color: #000;
  color: #000;
}

.btn:active {
  transform: translateY(-1px);
}
</style>

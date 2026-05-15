<script setup>
import { ref } from 'vue'
import Menu from './Components/Menu.vue'
import Movement from './Components/Movement.vue'
import GameOver from './Components/GameOver.vue'

const gameState = ref('menu') // 'menu', 'playing', 'gameOver'
const winner = ref(null)
const redScore = ref(0)
const blackScore = ref(0)
const redCaptured = ref(0)
const blackCaptured = ref(0)
const redRemaining = ref(0)
const blackRemaining = ref(0)

const startGame = () => {
  redScore.value = 0
  blackScore.value = 0
  redCaptured.value = 0
  blackCaptured.value = 0
  redRemaining.value = 0
  blackRemaining.value = 0
  winner.value = null
  gameState.value = 'playing'
}

const endGame = (winnerColor, rCaptured, bCaptured, rRemaining, bRemaining) => {
  winner.value = winnerColor
  redCaptured.value = rCaptured
  blackCaptured.value = bCaptured
  redRemaining.value = rRemaining
  blackRemaining.value = bRemaining
  if (winnerColor === 'red') {
    redScore.value++
  } else {
    blackScore.value++
  }
  gameState.value = 'gameOver'
}

const returnToMenu = () => {
  gameState.value = 'menu'
}
</script>

<template>
  <div class="app">
    <Menu v-if="gameState === 'menu'" @start-game="startGame" />
    <Movement 
      v-if="gameState === 'playing'" 
      @game-over="endGame"
      :red-score="redScore"
      :black-score="blackScore"
    />
    <GameOver 
      v-if="gameState === 'gameOver'" 
      :winner="winner"
      :red-score="redScore"
      :black-score="blackScore"
      :red-captured="redCaptured"
      :black-captured="blackCaptured"
      :red-remaining="redRemaining"
      :black-remaining="blackRemaining"
      @play-again="startGame"
      @return-home="returnToMenu"
    />
  </div>
</template>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background: #f5f5f5;
  min-height: 100vh;
}

.app {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #f5f5f5;
}
</style>
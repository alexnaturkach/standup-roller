<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'

const props = defineProps<{
  participants?: string[]
}>()

const emit = defineEmits<{
  'winner-selected': [winner: string]
}>()

const names: string[] = props.participants ?? ['Alex', 'Casey', 'Robby', 'Sheila']
const colors = ['#FF6B6B', '#4ECDC4', '#45B7D1', '#96CEB4']
const avatars = ['👤', '👤', '👤', '👤'] // Generic avatars

const gameStarted = ref(false)
const showResult = ref(false)
const winner = ref('')
const currentHolder = ref(0)
const isPassing = ref(false)
const passInterval = ref<ReturnType<typeof setInterval> | null>(null)
const countdown = ref(0)
const countdownInterval = ref<ReturnType<typeof setInterval> | null>(null)

const startGame = () => {
  gameStarted.value = true
  showResult.value = false
  currentHolder.value = 0
  isPassing.value = true
  countdown.value = 0
  
  // Start passing the potato
  startPassing()
  
  // Start countdown
  startCountdown()
}

const startPassing = () => {
  passInterval.value = setInterval(() => {
    currentHolder.value = (currentHolder.value + 1) % names.length
  }, 100) // Pass every 100ms - ultra fast!
}

const startCountdown = () => {
  countdown.value = 15 // Fixed 15-second countdown
  countdownInterval.value = setInterval(() => {
    countdown.value -= 0.05 // Update every 50ms for smoother countdown
    if (countdown.value <= 0) {
      stopGame()
    }
  }, 50) // Update every 50ms
}

const stopGame = () => {
  isPassing.value = false
  
  if (passInterval.value) {
    clearInterval(passInterval.value)
    passInterval.value = null
  }
  
  if (countdownInterval.value) {
    clearInterval(countdownInterval.value)
    countdownInterval.value = null
  }
  
  const safeIndex = names.length > 0 ? (currentHolder.value % names.length) : 0
  winner.value = names[safeIndex] ?? ''
  
  setTimeout(() => {
    showResult.value = true
    
    setTimeout(() => {
      emit('winner-selected', winner.value)
    }, 2000)
  }, 1000)
}

const manualStop = () => {
  if (isPassing.value) {
    stopGame()
  }
}

onUnmounted(() => {
  if (passInterval.value) {
    clearInterval(passInterval.value)
  }
  if (countdownInterval.value) {
    clearInterval(countdownInterval.value)
  }
})
</script>

<template>
  <div class="hot-potato-game">
    <div class="game-header">
      <h2>🥔 Hot Potato</h2>
      <p>The potato is being passed around! Click "Stop!" when you want to freeze it, or wait 15 seconds!</p>
      <div v-if="!gameStarted" class="game-info">
        <p class="instruction">Click "Start Passing!" to begin the hot potato game</p>
        <button @click="startGame" class="start-button">
          Start Passing! 🥔
        </button>
      </div>
      <div v-else-if="isPassing" class="game-status">
        <div class="countdown">
          <div class="countdown-bar" :style="{ width: `${(countdown / 15) * 100}%` }"></div>
          <p class="countdown-text">{{ countdown.toFixed(2) }}s</p>
        </div>
        <button @click="manualStop" class="stop-button">
          Stop! 🛑
        </button>
      </div>
    </div>
    
    <div class="players-container">
      <div 
        v-for="(name, index) in names" 
        :key="name"
        class="player"
        :class="{ 
          'has-potato': currentHolder === index,
          'passing': isPassing && currentHolder === index
        }"
        :style="{ backgroundColor: colors[index] }"
      >
        <div class="player-avatar">
          <span class="player-emoji">{{ avatars[index] }}</span>
        </div>
        <div class="player-name">{{ name }}</div>
        <div v-if="currentHolder === index" class="potato">
          🥔
        </div>
      </div>
    </div>
    
    <div v-if="showResult" class="result-overlay">
      <div class="result-content">
        <h3>Selected Participant</h3>
        <div class="winner-name">{{ winner }}</div>
        <p>will start today's standup</p>
      </div>
    </div>
  </div>
</template>

<style scoped>
.hot-potato-game {
  width: 100vw;
  height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background: linear-gradient(135deg, #FF6B6B 0%, #4ECDC4 100%);
  color: white;
  position: relative;
}

.game-header {
  text-align: center;
  margin-bottom: 2rem;
  z-index: 10;
}

.game-header h2 {
  font-size: 2.5rem;
  margin-bottom: 1rem;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
}

.game-header p {
  font-size: 1.2rem;
  opacity: 0.9;
  margin-bottom: 1rem;
}

.game-info, .game-status {
  background: rgba(255, 255, 255, 0.1);
  padding: 1.5rem;
  border-radius: 15px;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
  backdrop-filter: blur(10px);
}

.instruction {
  font-size: 1.1rem;
  margin-bottom: 1rem;
  color: white;
}

.start-button, .stop-button {
  background: linear-gradient(45deg, #e74c3c, #c0392b);
  color: white;
  border: none;
  padding: 1rem 2rem;
  font-size: 1.2rem;
  font-weight: bold;
  border-radius: 50px;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.2);
}

.stop-button {
  background: linear-gradient(45deg, #f39c12, #e67e22);
  animation: pulse 1s infinite alternate;
}

.start-button:hover, .stop-button:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.3);
}

.countdown {
  position: relative;
  width: 200px;
  height: 20px;
  background: rgba(255, 255, 255, 0.2);
  border-radius: 10px;
  margin: 1rem auto;
  overflow: hidden;
}

.countdown-bar {
  height: 100%;
  background: linear-gradient(90deg, #f39c12, #e74c3c);
  border-radius: 10px;
  transition: width 0.1s linear;
}

.countdown-text {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  font-weight: bold;
  font-size: 0.9rem;
  color: white;
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.5);
}

.players-container {
  display: flex;
  gap: 2rem;
  flex-wrap: wrap;
  justify-content: center;
  align-items: center;
  min-height: 200px;
}

.player {
  position: relative;
  width: 120px;
  height: 150px;
  border-radius: 25px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  transition: all 0.3s ease;
  box-shadow: 
    0 8px 25px rgba(0, 0, 0, 0.15),
    inset 0 1px 0 rgba(255, 255, 255, 0.2);
  border: 3px solid transparent;
  backdrop-filter: blur(10px);
}

.player.has-potato {
  border-color: #f39c12;
  box-shadow: 0 0 20px rgba(243, 156, 18, 0.5);
  transform: scale(1.1);
}

.player.passing {
  animation: bounce 0.8s ease-in-out infinite;
}

.player-avatar {
  width: 60px;
  height: 60px;
  background: rgba(255, 255, 255, 0.3);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 0.5rem;
  border: 3px solid rgba(255, 255, 255, 0.5);
  box-shadow: inset 0 2px 4px rgba(0, 0, 0, 0.1);
}

.player-emoji {
  font-size: 1.8rem;
  filter: drop-shadow(0 1px 2px rgba(0, 0, 0, 0.2));
}

.player-name {
  font-weight: bold;
  font-size: 1.1rem;
  color: white;
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.5);
}

.potato {
  position: absolute;
  top: -20px;
  font-size: 2rem;
  animation: potatoBounce 0.8s ease-in-out infinite;
}

@keyframes bounce {
  0%, 100% { transform: scale(1.1) translateY(0); }
  50% { transform: scale(1.1) translateY(-10px); }
}

@keyframes potatoBounce {
  0%, 100% { transform: translateY(0) rotate(0deg); }
  50% { transform: translateY(-10px) rotate(10deg); }
}

@keyframes pulse {
  0% { transform: scale(1); }
  100% { transform: scale(1.05); }
}

.result-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.8);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
  animation: fadeIn 0.5s ease-out;
}

.result-content {
  background: rgba(255, 255, 255, 0.95);
  color: #333;
  padding: 3rem;
  border-radius: 20px;
  text-align: center;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
  backdrop-filter: blur(10px);
  animation: scaleIn 0.5s ease-out;
}

.result-content h3 {
  font-size: 2rem;
  margin-bottom: 1rem;
  color: #667eea;
}

.winner-name {
  font-size: 3rem;
  font-weight: bold;
  color: #4ecdc4;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.1);
  margin-bottom: 0.5rem;
}

.result-content p {
  font-size: 1.2rem;
  margin: 0;
  color: #666;
}

@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

@keyframes scaleIn {
  from {
    opacity: 0;
    transform: scale(0.8);
  }
  to {
    opacity: 1;
    transform: scale(1);
  }
}

/* Responsive design */
@media (max-width: 768px) {
  .players-container {
    gap: 1rem;
  }
  
  .player {
    width: 100px;
    height: 130px;
  }
  
  .player-avatar {
    width: 50px;
    height: 50px;
  }
  
  .player-emoji {
    font-size: 1.5rem;
  }
  
  .player-name {
    font-size: 1rem;
  }
}
</style>

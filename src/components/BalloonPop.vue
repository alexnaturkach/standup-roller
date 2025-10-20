<script setup lang="ts">
import { ref, onMounted } from 'vue'

const props = defineProps<{
  participants?: string[]
}>()

const emit = defineEmits<{
  'winner-selected': [winner: string]
}>()

const participantNames = props.participants || ['Alex', 'Casey', 'Robby', 'Sheila']
const balloons = ref(
  participantNames.map((name, index) => ({
    name,
    color: ['#FF6B6B', '#4ECDC4', '#45B7D1', '#96CEB4'][index],
    popped: false,
    id: index,
    revealed: false,
  })),
)

const gameStarted = ref(false)
const showResult = ref(false)
const winner = ref('')
const poppedCount = ref(0)

const startGame = () => {
  gameStarted.value = true
  showResult.value = false
  poppedCount.value = 0

  // Shuffle the participant names for this game
  const shuffledNames = [...participantNames].sort(() => Math.random() - 0.5)

  // Update existing balloons with shuffled names
  balloons.value.forEach((balloon, index) => {
    balloon.name = shuffledNames[index] || ''
    balloon.popped = false
    balloon.revealed = false
  })
}

const popBalloon = (balloonId: number) => {
  const balloon = balloons.value.find((b) => b.id === balloonId)
  if (!balloon || balloon.popped) return

  balloon.popped = true
  balloon.revealed = true
  poppedCount.value++

  // Check if this is the last balloon
  if (poppedCount.value === balloons.value.length - 1) {
    // Find the remaining balloon and reveal it
    const remainingBalloon = balloons.value.find((b) => !b.popped)
    if (remainingBalloon) {
      remainingBalloon.revealed = true
      winner.value = remainingBalloon.name

      setTimeout(() => {
        showResult.value = true

        setTimeout(() => {
          emit('winner-selected', winner.value)
        }, 2000)
      }, 1000)
    }
  }
}

const getRemainingCount = () => {
  return balloons.value.filter((b) => !b.popped).length
}

// Auto-start the game when component mounts
onMounted(() => {
  setTimeout(() => {
    startGame()
  }, 1000) // Small delay to let the component fully render
})
</script>

<template>
  <div class="balloon-game">
    <div class="game-header">
      <h2>🎈 Balloon Pop Challenge</h2>
      <p>Click the balloons to pop them and reveal the names! The last balloon standing wins!</p>
      <div v-if="!showResult" class="game-status">
        <p class="remaining-count">
          {{ getRemainingCount() }} balloon{{ getRemainingCount() !== 1 ? 's' : '' }} remaining
        </p>
        <p class="instruction">Click a balloon to pop it!</p>
      </div>
    </div>

    <div class="balloons-container">
      <div
        v-for="balloon in balloons"
        :key="balloon.id"
        class="balloon-wrapper"
        :class="{
          clickable: !balloon.popped,
        }"
        @click="popBalloon(balloon.id)"
      >
        <div class="balloon-string" :class="{ hidden: balloon.popped }"></div>
        <div
          class="balloon"
          :class="{
            popped: balloon.popped,
          }"
        >
          <div class="balloon-body" :style="{ backgroundColor: balloon.color }">
            <span class="balloon-name">{{ balloon.revealed ? balloon.name : '?' }}</span>
          </div>
          <div v-if="balloon.popped" class="pop-effect">
            <div class="confetti-piece" v-for="n in 8" :key="n"></div>
          </div>
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
.balloon-game {
  width: 100vw;
  height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background: linear-gradient(135deg, #87ceeb 0%, #98fb98 100%);
  color: #333;
  position: relative;
  overflow: hidden;
}

.game-header {
  text-align: center;
  margin-bottom: 2rem;
  z-index: 10;
}

.game-header h2 {
  font-size: 2.5rem;
  margin-bottom: 1rem;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.1);
  color: #2c3e50;
}

.game-header p {
  font-size: 1.2rem;
  margin-bottom: 1rem;
  color: #34495e;
}

.game-info,
.game-status {
  background: rgba(255, 255, 255, 0.9);
  padding: 1.5rem;
  border-radius: 15px;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
}

.instruction {
  font-size: 1.1rem;
  margin-bottom: 1rem;
  color: #2c3e50;
}

.remaining-count {
  font-size: 1.3rem;
  font-weight: bold;
  color: #e74c3c;
  margin-bottom: 0.5rem;
}

.balloons-container {
  display: flex;
  gap: 2rem;
  flex-wrap: wrap;
  justify-content: center;
  align-items: center;
  min-height: 300px;
}

.balloon-wrapper {
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.balloon {
  position: relative;
  cursor: pointer;
  transition: all 0.3s ease;
  animation: float 3s ease-in-out infinite;
  width: 80px;
  height: 100px;
  background: transparent;
  border: none;
  outline: none;
}

.balloon-wrapper.clickable:hover .balloon {
  transform: scale(1.1);
}

.balloon.popped {
  animation: pop 0.5s ease-out forwards;
  cursor: default;
}

.balloon-string {
  width: 1px;
  height: 100px;
  background-color: black;
  margin: 0 auto;
  position: relative;
  z-index: 1;
  border: none;
  outline: none;
  box-shadow: none;
  background-image: none;
  background-clip: content-box;
  animation: float 3s ease-in-out infinite;
}

.balloon-string.hidden {
  opacity: 0;
  visibility: hidden;
  transition:
    opacity 0.3s ease,
    visibility 0.3s ease;
}

.balloon-body {
  width: 100%;
  height: 100%;
  border-radius: 50% 50% 50% 50% / 60% 60% 40% 40%;
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
  z-index: 2;
  border: 2px solid rgba(255, 255, 255, 0.3);
  box-shadow:
    inset -10px -10px 20px rgba(0, 0, 0, 0.1),
    0 4px 15px rgba(0, 0, 0, 0.2);
  background-clip: padding-box;
}

.balloon-name {
  font-weight: bold;
  font-size: 1rem;
  color: white;
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.3);
}

.pop-effect {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  pointer-events: none;
}

.confetti-piece {
  position: absolute;
  width: 6px;
  height: 6px;
  background: #ffd700;
  animation: confettiFall 1s ease-out forwards;
}

.confetti-piece:nth-child(1) {
  animation-delay: 0s;
  transform: rotate(0deg);
}
.confetti-piece:nth-child(2) {
  animation-delay: 0.1s;
  transform: rotate(45deg);
}
.confetti-piece:nth-child(3) {
  animation-delay: 0.2s;
  transform: rotate(90deg);
}
.confetti-piece:nth-child(4) {
  animation-delay: 0.3s;
  transform: rotate(135deg);
}
.confetti-piece:nth-child(5) {
  animation-delay: 0.4s;
  transform: rotate(180deg);
}
.confetti-piece:nth-child(6) {
  animation-delay: 0.5s;
  transform: rotate(225deg);
}
.confetti-piece:nth-child(7) {
  animation-delay: 0.6s;
  transform: rotate(270deg);
}
.confetti-piece:nth-child(8) {
  animation-delay: 0.7s;
  transform: rotate(315deg);
}

@keyframes float {
  0%,
  100% {
    transform: translateY(0px);
  }
  50% {
    transform: translateY(-20px);
  }
}

@keyframes pop {
  0% {
    transform: scale(1);
    opacity: 1;
  }
  50% {
    transform: scale(1.3);
    opacity: 0.8;
  }
  100% {
    transform: scale(0);
    opacity: 0;
  }
}

@keyframes confettiFall {
  0% {
    transform: translate(0, 0) rotate(0deg);
    opacity: 1;
  }
  100% {
    transform: translate(var(--random-x, 50px), var(--random-y, 100px)) rotate(360deg);
    opacity: 0;
  }
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
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
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
  .balloons-container {
    gap: 1rem;
  }

  .balloon-body {
    width: 60px;
    height: 80px;
  }

  .balloon-string {
    height: 80px;
  }

  .balloon-name {
    font-size: 0.9rem;
  }
}
</style>

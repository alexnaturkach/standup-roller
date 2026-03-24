<script setup lang="ts">
import { ref } from 'vue'

const props = defineProps<{
  participants?: string[]
  onWinnerSelected: (winner: string) => void
}>()

const isSpinning = ref(false)
const rotation = ref(0)
const showResult = ref(false)
const winner = ref('')

const names = props.participants || ['Alex', 'Casey', 'Robby', 'Sheila']
const colors = ['#FF6B6B', '#4ECDC4', '#45B7D1', '#96CEB4']

const segmentAngle = 360 / names.length

const getColor = (index: number) => colors[index % colors.length]

/**
 * Which segment sits under the pointer (fixed at 12 o'clock).
 * Segments use transform rotate(i * sa) with origin at wheel center; segment 0 is the NW wedge,
 * so segment 0 starts at angle (360 - sa), not 0°. Equivalently:
 *   α = (360 - R)° CW from top = wheel angle at the pointer
 *   shifted = (α + sa) % 360  →  index = floor(shifted / sa)
 */
const getWinnerIndexByAngle = () => {
  const n = names.length
  const sa = segmentAngle
  const R = ((rotation.value % 360) + 360) % 360
  const alpha = (360 - R) % 360
  const shifted = (alpha + sa) % 360
  return Math.floor(shifted / sa) % n
}

const spinWheel = () => {
  if (isSpinning.value) return

  isSpinning.value = true
  showResult.value = false

  const n = names.length
  const sa = 360 / n

  const targetIndex = Math.floor(Math.random() * n)
  const margin = 0.02 * sa
  const shifted =
    targetIndex * sa + margin + Math.random() * (sa - 2 * margin)

  const alpha = (shifted - sa + 360) % 360
  const RMod = (360 - alpha + 360) % 360

  const current = rotation.value
  const currentMod = ((current % 360) + 360) % 360
  let delta = RMod - currentMod
  if (delta <= 0) delta += 360

  const spins = 5 + Math.floor(Math.random() * 5)
  rotation.value = current + 360 * spins + delta
}

const onWheelTransitionEnd = (event: TransitionEvent) => {
  if (event.propertyName !== 'transform') return
  if (!isSpinning.value) return

  winner.value = names[getWinnerIndexByAngle()] || ''

  setTimeout(() => {
    // Pause so the wheel "stops", then reveal the winner modal.
    isSpinning.value = false
    showResult.value = true

    // Notify parent once the modal is shown.
    props.onWinnerSelected(winner.value)
  }, 2000)
}
</script>

<template>
  <div class="spinning-wheel-game">
    <div class="game-header">
      <h2>🎡 Spinning Wheel of Fortune</h2>
      <p>Click the wheel to spin and see who goes first!</p>
    </div>

    <div class="wheel-container">
      <div
        class="wheel"
        :class="{ spinning: isSpinning }"
        :style="{ transform: `rotate(${rotation}deg)` }"
        @click="spinWheel"
        @transitionend="onWheelTransitionEnd"
      >
        <div
          v-for="(name, index) in names"
          :key="name"
          class="wheel-segment"
          :data-index="index"
          :style="{
            backgroundColor: getColor(index),
            transform: `rotate(${index * segmentAngle}deg)`,
          }"
        >
          <span class="segment-text">{{ name }}</span>
        </div>
        <div class="wheel-center"></div>
      </div>

      <div class="wheel-pointer"></div>
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
.spinning-wheel-game {
  width: 100vw;
  height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  position: relative;
}

.game-header {
  text-align: center;
  margin-bottom: 2rem;
}

.game-header h2 {
  font-size: 2.5rem;
  margin-bottom: 1rem;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
}

.game-header p {
  font-size: 1.2rem;
  opacity: 0.9;
}

.wheel-container {
  position: relative;
  margin: 2rem 0;
}

.wheel {
  width: 300px;
  height: 300px;
  border-radius: 50%;
  position: relative;
  cursor: pointer;
  transition: transform 3s cubic-bezier(0.25, 0.46, 0.45, 0.94);
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
  border: 8px solid white;
}

.wheel.spinning {
  cursor: not-allowed;
}

.wheel-segment {
  position: absolute;
  width: 50%;
  height: 50%;
  transform-origin: 100% 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 0 0 0 100%;
  overflow: hidden;
}

.segment-text {
  position: absolute;
  top: 20px;
  left: 20px;
  font-weight: bold;
  font-size: 1.2rem;
  color: white;
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.5);
  transform: rotate(45deg);
}

.wheel-center {
  position: absolute;
  top: 50%;
  left: 50%;
  width: 40px;
  height: 40px;
  background: white;
  border-radius: 50%;
  transform: translate(-50%, -50%);
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.3);
  z-index: 10;
  display: flex;
  align-items: center;
  justify-content: center;
  border: 2px solid #333;
}

.wheel-pointer {
  position: absolute;
  top: -10px;
  left: 50%;
  width: 0;
  height: 0;
  border-left: 15px solid transparent;
  border-right: 15px solid transparent;
  border-top: 30px solid #ffd700;
  transform: translateX(-50%);
  z-index: 20;
  filter: drop-shadow(0 2px 4px rgba(0, 0, 0, 0.3));
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
</style>

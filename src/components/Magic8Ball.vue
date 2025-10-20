<script setup lang="ts">
import { ref } from 'vue'

const props = defineProps<{
  participants?: string[]
  onWinnerSelected: (winner: string) => void
}>()

const isShaking = ref(false)
const showResult = ref(false)
const winner = ref<string>('')
const ballRotation = ref(0)
const particles = ref<Array<{ id: number; x: number; y: number; vx: number; vy: number; life: number }>>([])

const names: string[] = props.participants ?? ['Alex', 'Casey', 'Robby', 'Sheila']

const shakeBall = () => {
  if (isShaking.value) return
  
  isShaking.value = true
  showResult.value = false
  particles.value = []
  
  // Create particles
  for (let i = 0; i < 20; i++) {
    particles.value.push({
      id: i,
      x: Math.random() * window.innerWidth,
      y: Math.random() * window.innerHeight,
      vx: (Math.random() - 0.5) * 10,
      vy: (Math.random() - 0.5) * 10,
      life: 1
    })
  }
  
  // Shake animation
  let shakeCount = 0
  const shakeInterval = setInterval(() => {
    ballRotation.value += (Math.random() - 0.5) * 20
    shakeCount++
    
    if (shakeCount >= 20) {
      clearInterval(shakeInterval)
      
      // Select winner
      const pool = names.length > 0 ? names : ['']
      const idx = Math.floor(Math.random() * pool.length)
      winner.value = pool[idx] ?? ''
      
      setTimeout(() => {
        isShaking.value = false
        showResult.value = true
        
        setTimeout(() => {
          props.onWinnerSelected(winner.value)
        }, 2000)
      }, 1000)
    }
  }, 100)
  
  // Animate particles
  const animateParticles = () => {
    particles.value.forEach(particle => {
      particle.x += particle.vx
      particle.y += particle.vy
      particle.life -= 0.02
      particle.vx *= 0.98
      particle.vy *= 0.98
    })
    
    particles.value = particles.value.filter(p => p.life > 0)
    
    if (particles.value.length > 0) {
      requestAnimationFrame(animateParticles)
    }
  }
  
  animateParticles()
}
</script>

<template>
  <div class="magic-8ball-game">
    <div class="game-header">
      <h2>🔮 Magic 8-Ball Shake</h2>
      <p>Shake the mystical ball and let it reveal who goes first!</p>
      <button 
        v-if="!isShaking && !showResult" 
        @click="shakeBall"
        class="shake-button"
      >
        Shake the Ball! ✨
      </button>
    </div>
    
    <div class="ball-container">
      <div 
        class="magic-ball"
        :class="{ shaking: isShaking }"
        :style="{ transform: `rotate(${ballRotation}deg)` }"
      >
        <div class="ball-surface">
          <div class="ball-window">
            <div v-if="!isShaking && !showResult" class="question-mark">?</div>
            <div v-else-if="isShaking" class="shaking-text">Shaking...</div>
            <div v-else class="answer-text">{{ winner }}</div>
          </div>
        </div>
        
        <div class="ball-base"></div>
      </div>
      
      <!-- Particles -->
      <div 
        v-for="particle in particles" 
        :key="particle.id"
        class="particle"
        :style="{ 
          left: particle.x + 'px',
          top: particle.y + 'px',
          opacity: particle.life
        }"
      >
        ✨
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
.magic-8ball-game {
  width: 100vw;
  height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background: linear-gradient(135deg, #2c3e50 0%, #34495e 100%);
  color: white;
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
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
  background: linear-gradient(45deg, #FFD700, #FFA500);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.game-header p {
  font-size: 1.2rem;
  opacity: 0.9;
  margin-bottom: 1rem;
}

.shake-button {
  background: linear-gradient(45deg, #8e44ad, #9b59b6);
  color: white;
  border: none;
  padding: 1rem 2rem;
  font-size: 1.2rem;
  font-weight: bold;
  border-radius: 50px;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.2);
  animation: glow 2s infinite alternate;
}

.shake-button:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.3);
}

.ball-container {
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
}

.magic-ball {
  width: 200px;
  height: 200px;
  position: relative;
  transition: transform 0.1s ease;
}

.magic-ball.shaking {
  animation: shake 0.1s infinite;
}

.ball-surface {
  width: 100%;
  height: 100%;
  background: radial-gradient(circle at 30% 30%, #1a1a1a, #000);
  border-radius: 50%;
  position: relative;
  box-shadow: 
    inset -20px -20px 50px rgba(0, 0, 0, 0.8),
    inset 20px 20px 50px rgba(255, 255, 255, 0.1),
    0 10px 30px rgba(0, 0, 0, 0.5);
  border: 3px solid #333;
}

.ball-window {
  position: absolute;
  top: 50%;
  left: 50%;
  width: 80px;
  height: 80px;
  background: linear-gradient(45deg, #87CEEB, #4682B4);
  border-radius: 50%;
  transform: translate(-50%, -50%);
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 
    inset 0 0 20px rgba(0, 0, 0, 0.3),
    0 0 20px rgba(135, 206, 235, 0.5);
  border: 2px solid #4682B4;
}

.question-mark,
.shaking-text,
.answer-text {
  font-size: 1.5rem;
  font-weight: bold;
  color: white;
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.5);
}

.shaking-text {
  animation: pulse 0.5s infinite alternate;
}

.ball-base {
  position: absolute;
  bottom: -20px;
  left: 50%;
  width: 120px;
  height: 20px;
  background: linear-gradient(45deg, #2c3e50, #34495e);
  border-radius: 50%;
  transform: translateX(-50%);
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.3);
}

.particle {
  position: absolute;
  font-size: 1.5rem;
  pointer-events: none;
  z-index: 5;
}

@keyframes shake {
  0%, 100% { transform: translateX(0) translateY(0); }
  25% { transform: translateX(-5px) translateY(-5px); }
  50% { transform: translateX(5px) translateY(5px); }
  75% { transform: translateX(-3px) translateY(3px); }
}

@keyframes glow {
  0% { box-shadow: 0 4px 15px rgba(0, 0, 0, 0.2); }
  100% { box-shadow: 0 4px 15px rgba(142, 68, 173, 0.5); }
}

@keyframes pulse {
  0% { opacity: 0.5; }
  100% { opacity: 1; }
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
</style>

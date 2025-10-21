<script setup lang="ts">
import { ref, onMounted, computed } from 'vue'

const props = defineProps<{
  team: 'web' | 'mobile' | null
  participants: string[]
  onGameSelected: (gameName: string, participants: string[]) => void
}>()

const isRolling = ref(false)
const selectedGame = ref('')
const showProceedButton = ref(false)
const showParticipantManager = ref(false)

const allParticipants = computed(() => props.participants)
const presentParticipants = ref([...props.participants])

const games: Array<{ name: string; emoji: string; description: string }> = [
  { name: 'Spinning Wheel of Fortune', emoji: '🎡', description: 'Spin the wheel and see who it lands on!' },
  { name: 'Balloon Pop Challenge', emoji: '🎈', description: 'Pop balloons to reveal names - last one standing wins!' },
  { name: 'Hot Potato', emoji: '🥔', description: 'Pass the potato around for 15 seconds or stop it early!' },
  { name: 'Magic 8-Ball Shake', emoji: '🔮', description: 'Shake the mystical ball for your answer!' }
]

const toggleParticipant = (participant: string) => {
  const index = presentParticipants.value.indexOf(participant)
  if (index > -1) {
    presentParticipants.value.splice(index, 1)
  } else {
    presentParticipants.value.push(participant)
  }
}

const rollForGame = () => {
  isRolling.value = true
  showProceedButton.value = false
  
  // Simulate rolling animation
  setTimeout(() => {
    const poolLength = games.length
    const randomIndex = Math.floor(Math.random() * (poolLength > 0 ? poolLength : 1))
    selectedGame.value = games[randomIndex]?.name ?? ''
    isRolling.value = false
    showProceedButton.value = true
  }, 2000)
}

const proceedToGame = () => {
  if (presentParticipants.value.length === 0) {
    alert('Please make sure at least one person is present!')
    return
  }
  props.onGameSelected(selectedGame.value as string, presentParticipants.value as string[])
}

onMounted(() => {
  // Reset participants when component mounts with new team data
  presentParticipants.value = [...props.participants]
  
  // Auto-roll when component mounts
  setTimeout(() => {
    rollForGame()
  }, 500)
})
</script>

<template>
  <div class="game-selector">
    <div class="selector-content">
      <h2>🎲 Let's Pick Your Game!</h2>
      <p>The dice will choose which exciting game we'll play today...</p>
      
      <!-- Team Display -->
      <div v-if="props.team" class="team-display">
        <h3>{{ props.team === 'web' ? '🌐 Web Team' : '📱 Mobile Team' }}</h3>
      </div>
      
      <!-- Participant Management -->
      <div class="participant-section">
        <h3>👥 Who's present today?</h3>
        <div class="checkbox-list">
          <label 
            v-for="participant in allParticipants" 
            :key="participant"
            class="checkbox-item"
          >
            <input 
              type="checkbox" 
              :checked="presentParticipants.includes(participant)"
              @change="toggleParticipant(participant)"
            />
            <span class="participant-name">{{ participant }}</span>
          </label>
        </div>
        <div class="participant-count">
          {{ presentParticipants.length }} participant{{ presentParticipants.length !== 1 ? 's' : '' }} present
        </div>
      </div>
      
      <div class="rolling-area">
        <div v-if="isRolling" class="rolling-dice">
          <div class="dice dice-1">🎲</div>
          <div class="dice dice-2">🎲</div>
          <div class="dice dice-3">🎲</div>
        </div>
        
        <div v-else-if="selectedGame" class="game-result">
          <div class="selected-game">
            <h3>{{ games.find(g => g.name === selectedGame)?.emoji }} {{ selectedGame }}</h3>
            <p>{{ games.find(g => g.name === selectedGame)?.description }}</p>
          </div>
          
          <div v-if="showProceedButton" class="button-group">
            <button 
              @click="rollForGame"
              class="reroll-button"
            >
              Re-roll! 🎲
            </button>
            <button 
              @click="proceedToGame"
              class="proceed-button"
              :disabled="presentParticipants.length === 0"
            >
              Let's Play! 🚀
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.game-selector {
  width: 100vw;
  height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
}

.selector-content {
  text-align: center;
  max-width: 600px;
  padding: 2rem;
}

h2 {
  font-size: 2.5rem;
  margin-bottom: 1rem;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
}

p {
  font-size: 1.2rem;
  margin-bottom: 2rem;
  opacity: 0.9;
}

.participant-section {
  margin-bottom: 2rem;
  background: rgba(255, 255, 255, 0.1);
  padding: 1.5rem;
  border-radius: 15px;
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.2);
}

.participant-section h3 {
  font-size: 1.5rem;
  margin: 0 0 1rem 0;
  color: #4ecdc4;
  text-align: center;
}

.team-display {
  margin-bottom: 1.5rem;
  background: rgba(78, 205, 196, 0.2);
  padding: 1rem;
  border-radius: 15px;
  backdrop-filter: blur(10px);
  border: 1px solid rgba(78, 205, 196, 0.3);
}

.team-display h3 {
  font-size: 1.5rem;
  margin: 0;
  color: #4ecdc4;
  text-align: center;
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.3);
}

.checkbox-list {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(120px, 1fr));
  gap: 0.75rem;
  margin-bottom: 1rem;
}

.checkbox-item {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  cursor: pointer;
  padding: 0.5rem;
  border-radius: 8px;
  transition: background-color 0.3s ease;
}

.checkbox-item:hover {
  background: rgba(255, 255, 255, 0.1);
}

.checkbox-item input[type="checkbox"] {
  width: 18px;
  height: 18px;
  accent-color: #4ecdc4;
  cursor: pointer;
}

.participant-name {
  font-weight: bold;
  color: white;
  font-size: 1rem;
}

.participant-count {
  text-align: center;
  font-size: 1rem;
  color: #4ecdc4;
  font-weight: bold;
}

.rolling-area {
  min-height: 200px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.rolling-dice {
  display: flex;
  gap: 1rem;
  animation: bounce 0.6s infinite alternate;
}

.dice {
  font-size: 3rem;
  animation: spin 0.5s linear infinite;
}

.dice-1 {
  animation-delay: 0s;
}

.dice-2 {
  animation-delay: 0.2s;
}

.dice-3 {
  animation-delay: 0.4s;
}

.game-result {
  animation: slideIn 0.5s ease-out;
}

.selected-game {
  background: rgba(255, 255, 255, 0.1);
  padding: 2rem;
  border-radius: 20px;
  margin-bottom: 2rem;
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.2);
}

.selected-game h3 {
  font-size: 2rem;
  margin-bottom: 1rem;
  color: #4ecdc4;
}

.selected-game p {
  font-size: 1.1rem;
  opacity: 0.9;
  margin: 0;
}

.button-group {
  display: flex;
  gap: 1rem;
  justify-content: center;
  flex-wrap: wrap;
}

.reroll-button {
  background: linear-gradient(45deg, #95a5a6, #7f8c8d);
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

.reroll-button:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.3);
}

.proceed-button {
  background: linear-gradient(45deg, #4ecdc4, #44a08d);
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

.proceed-button:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.3);
}

.proceed-button:active {
  transform: translateY(0);
}

@keyframes bounce {
  0% { transform: translateY(0); }
  100% { transform: translateY(-20px); }
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
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

@keyframes slideDown {
  from {
    opacity: 0;
    transform: translateY(-20px);
    max-height: 0;
  }
  to {
    opacity: 1;
    transform: translateY(0);
    max-height: 500px;
  }
}

/* Responsive design */
@media (max-width: 768px) {
  .button-group {
    flex-direction: column;
    align-items: center;
  }
  
  .reroll-button, .proceed-button {
    width: 100%;
    max-width: 250px;
  }
}
</style>

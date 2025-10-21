<script setup lang="ts">
import { ref } from 'vue'
import TeamSelector from './components/TeamSelector.vue'
import GameSelector from './components/GameSelector.vue'
import SpinningWheel from './components/SpinningWheel.vue'
import BalloonPop from './components/BalloonPop.vue'
import HotPotato from './components/HotPotato.vue'
import Magic8Ball from './components/Magic8Ball.vue'

type GameState = 'team-selection' | 'selecting' | 'playing' | 'finished'
type GameType = 'Spinning Wheel of Fortune' | 'Balloon Pop Challenge' | 'Hot Potato' | 'Magic 8-Ball Shake'
type TeamType = 'web' | 'mobile'

const gameState = ref<GameState>('team-selection')
const selectedTeam = ref<TeamType | null>(null)
const selectedGame = ref<GameType | null>(null)
const finalWinner = ref('')
const showFinalResult = ref(false)
const selectedParticipants = ref<string[]>([])

const teams = {
  web: ['Alex', 'Casey', 'Robby', 'Sheila'],
  mobile: ['Ted', 'Kevin', 'Maria']
}

const handleTeamSelected = (team: TeamType) => {
  selectedTeam.value = team
  gameState.value = 'selecting'
}

const handleGameSelected = (gameName: string, participants: string[]) => {
  selectedGame.value = gameName as GameType
  selectedParticipants.value = participants
  gameState.value = 'playing'
}

const handleWinnerSelected = (winner: string) => {
  finalWinner.value = winner
  gameState.value = 'finished'
  showFinalResult.value = true
}

const resetGame = () => {
  gameState.value = 'team-selection'
  selectedTeam.value = null
  selectedGame.value = null
  finalWinner.value = ''
  showFinalResult.value = false
}

const getGameComponent = () => {
  switch (selectedGame.value) {
    case 'Spinning Wheel of Fortune':
      return SpinningWheel
    case 'Balloon Pop Challenge':
      return BalloonPop
    case 'Hot Potato':
      return HotPotato
    case 'Magic 8-Ball Shake':
      return Magic8Ball
    default:
      return null
  }
}
</script>

<template>
  <div class="app">
    <!-- Team Selection Phase -->
    <TeamSelector 
      v-if="gameState === 'team-selection'"
      @team-selected="handleTeamSelected"
    />
    
    <!-- Game Selection Phase -->
    <GameSelector 
      v-else-if="gameState === 'selecting'"
      :team="selectedTeam"
      :participants="selectedTeam ? teams[selectedTeam] : []"
      @game-selected="handleGameSelected"
    />
    
    <!-- Game Playing Phase -->
    <component 
      v-else-if="gameState === 'playing' && selectedGame"
      :is="getGameComponent()"
      :participants="selectedParticipants"
      @winner-selected="handleWinnerSelected"
    />
    
    <!-- Final Result Overlay -->
    <div v-if="showFinalResult" class="final-result-overlay">
      <div class="final-result-content">
        <h1>Standup Participant Selected</h1>
        <div class="final-winner">{{ finalWinner }}</div>
        <p>will start today's standup</p>
        <button @click="resetGame" class="play-again-button">
          Select Again 🔄
        </button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.app {
  width: 100vw;
  height: 100vh;
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

.final-result-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.9);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 2000;
  animation: fadeIn 0.5s ease-out;
}

.final-result-content {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  padding: 4rem;
  border-radius: 30px;
  text-align: center;
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.3);
  backdrop-filter: blur(10px);
  animation: scaleIn 0.5s ease-out;
  max-width: 500px;
  width: 90%;
}

.final-result-content h1 {
  font-size: 2.5rem;
  margin-bottom: 1.5rem;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
}

.final-winner {
  font-size: 4rem;
  font-weight: bold;
  color: #FFD700;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
  margin-bottom: 1rem;
  animation: glow 2s infinite alternate;
}

.final-result-content p {
  font-size: 1.5rem;
  margin-bottom: 2rem;
  opacity: 0.9;
}

.play-again-button {
  background: linear-gradient(45deg, #4ecdc4, #44a08d);
  color: white;
  border: none;
  padding: 1.2rem 2.5rem;
  font-size: 1.3rem;
  font-weight: bold;
  border-radius: 50px;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.2);
}

.play-again-button:hover {
  transform: translateY(-3px);
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.3);
}

.play-again-button:active {
  transform: translateY(-1px);
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

@keyframes glow {
  0% { 
    text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
  }
  100% { 
    text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3), 0 0 20px rgba(255, 215, 0, 0.5);
  }
}
</style>
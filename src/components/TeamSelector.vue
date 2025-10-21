<script setup lang="ts">
import { ref } from 'vue'

const emit = defineEmits<{
  'team-selected': [team: 'web' | 'mobile']
}>()

const selectedTeam = ref<'web' | 'mobile' | null>(null)

const teams = {
  web: {
    name: 'Web Team',
    emoji: '🌐',
    participants: ['Alex', 'Casey', 'Robby', 'Sheila']
  },
  mobile: {
    name: 'Mobile Team', 
    emoji: '📱',
    participants: ['Ted', 'Kevin', 'Maria', 'Sheila']
  }
}

const selectTeam = (team: 'web' | 'mobile') => {
  selectedTeam.value = team
  emit('team-selected', team)
}
</script>

<template>
  <div class="team-selector">
    <div class="selector-content">
      <h2>👥 Choose Your Team</h2>
      <p>Select which team will be participating in today's standup</p>
      
      <div class="teams-grid">
        <div 
          v-for="(team, teamKey) in teams" 
          :key="teamKey"
          class="team-card"
          :class="{ selected: selectedTeam === teamKey }"
          @click="selectTeam(teamKey as 'web' | 'mobile')"
        >
          <div class="team-emoji">{{ team.emoji }}</div>
          <h3>{{ team.name }}</h3>
          <div class="participants-list">
            <div 
              v-for="participant in team.participants" 
              :key="participant"
              class="participant-tag"
            >
              {{ participant }}
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.team-selector {
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
  max-width: 800px;
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

.teams-grid {
  display: grid;
  grid-template-columns: repeat(2, minmax(400px, 1fr));
  gap: 2rem;
  margin-top: 2rem;
  justify-content: center;
}

.team-card {
  background: rgba(255, 255, 255, 0.1);
  padding: 2rem;
  border-radius: 20px;
  cursor: pointer;
  transition: all 0.3s ease;
  backdrop-filter: blur(10px);
  border: 2px solid rgba(255, 255, 255, 0.2);
  text-align: center;
}

.team-card:hover {
  transform: translateY(-5px);
  background: rgba(255, 255, 255, 0.15);
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.2);
}

.team-card.selected {
  background: rgba(78, 205, 196, 0.2);
  border-color: #4ecdc4;
  transform: translateY(-5px);
  box-shadow: 0 10px 30px rgba(78, 205, 196, 0.3);
}

.team-emoji {
  font-size: 4rem;
  margin-bottom: 1rem;
}

.team-card h3 {
  font-size: 1.8rem;
  margin-bottom: 1.5rem;
  color: #4ecdc4;
}

.participants-list {
  display: flex;
  flex-wrap: nowrap;
  gap: 0.5rem;
  justify-content: center;
  align-items: center;
}

.participant-tag {
  background: rgba(255, 255, 255, 0.2);
  padding: 0.4rem 0.8rem;
  border-radius: 15px;
  font-size: 0.85rem;
  font-weight: bold;
  border: 1px solid rgba(255, 255, 255, 0.3);
  white-space: nowrap;
  flex-shrink: 0;
}

.team-card.selected .participant-tag {
  background: rgba(78, 205, 196, 0.3);
  border-color: #4ecdc4;
}

/* Responsive design */
@media (max-width: 900px) {
  .teams-grid {
    grid-template-columns: 1fr;
    gap: 1rem;
  }
  
  .team-card {
    padding: 1.5rem;
  }
  
  .team-emoji {
    font-size: 3rem;
  }
  
  h2 {
    font-size: 2rem;
  }
}

@media (max-width: 480px) {
  .selector-content {
    padding: 1rem;
  }
  
  .team-card {
    padding: 1rem;
  }
  
  .team-emoji {
    font-size: 2.5rem;
  }
  
  h2 {
    font-size: 1.8rem;
  }
  
  .participants-list {
    flex-wrap: wrap;
    gap: 0.3rem;
  }
  
  .participant-tag {
    padding: 0.3rem 0.6rem;
    font-size: 0.8rem;
  }
}
</style>

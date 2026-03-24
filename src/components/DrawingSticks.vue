<script setup lang="ts">
import { computed, ref } from 'vue'

const props = defineProps<{
  participants?: string[]
}>()

const emit = defineEmits<{
  'winner-selected': [winner: string]
}>()

type Stick = {
  id: string
  lengthType: 'normal' | 'short'
  assignedTo: string | null
}

const participantNames = props.participants ?? ['Alex', 'Casey', 'Robby', 'Sheila']
const shortStickIndex = Math.floor(Math.random() * Math.max(participantNames.length, 1))

const sticks = ref<Stick[]>(
  participantNames.map((_, index) => ({
    id: `stick-${index}`,
    lengthType: index === shortStickIndex ? 'short' : 'normal',
    assignedTo: null,
  })),
)

const draggedStickId = ref<string | null>(null)
const isFinished = ref(false)
const winner = ref('')

const assignedCount = computed(() => sticks.value.filter((stick) => stick.assignedTo !== null).length)

const getAssignedStick = (participant: string) =>
  sticks.value.find((stick) => stick.assignedTo === participant) ?? null

const getUnassignedSticks = computed(() => sticks.value.filter((stick) => stick.assignedTo === null))

const onDragStart = (stickId: string) => {
  if (isFinished.value) return
  draggedStickId.value = stickId
}

const onDragEnd = () => {
  draggedStickId.value = null
}

const assignDraggedStickToParticipant = (participant: string) => {
  if (isFinished.value || !draggedStickId.value) return

  const incomingStick = sticks.value.find((stick) => stick.id === draggedStickId.value)
  if (!incomingStick) return

  const currentStickForParticipant = getAssignedStick(participant)

  // If participant already has a stick, send it back to the pool.
  if (currentStickForParticipant) {
    currentStickForParticipant.assignedTo = null
  }

  incomingStick.assignedTo = participant
  draggedStickId.value = null
  maybeFinishGame()
}

const maybeFinishGame = () => {
  const allAssigned = sticks.value.every((stick) => stick.assignedTo !== null)
  if (!allAssigned) return

  const shortStick = sticks.value.find((stick) => stick.lengthType === 'short')
  if (!shortStick || !shortStick.assignedTo) return

  isFinished.value = true
  winner.value = shortStick.assignedTo

  setTimeout(() => {
    emit('winner-selected', winner.value)
  }, 2000)
}
</script>

<template>
  <div class="drawing-sticks-game">
    <div class="game-header">
      <h2>🪵 Drawing Sticks</h2>
      <p>Drag one stick to each participant, then reveal who got the short one.</p>
    </div>

    <div class="participants-row">
      <div
        v-for="participant in participantNames"
        :key="participant"
        class="participant-slot"
        @dragover.prevent
        @drop.prevent="assignDraggedStickToParticipant(participant)"
      >
        <div class="participant-name">{{ participant }}</div>

        <div
          class="participant-stick-area"
          :class="{ 'participant-stick-area--empty': !getAssignedStick(participant) }"
        >
          <div
            v-if="getAssignedStick(participant)"
            class="assigned-stick"
            :class="{
              short: getAssignedStick(participant)?.lengthType === 'short',
              covered: !isFinished,
            }"
            draggable="true"
            @dragstart="onDragStart(getAssignedStick(participant)!.id)"
            @dragend="onDragEnd"
          >
            <div
              v-if="!isFinished"
              class="stick-cover"
              :class="
                getAssignedStick(participant)?.lengthType === 'short'
                  ? 'stick-cover--short'
                  : 'stick-cover--long'
              "
            >
              👍
            </div>
          </div>
          <div v-else class="drop-hint">Drop stick here</div>
        </div>
      </div>
    </div>

    <div class="sticks-pool">
      <h3>Sticks</h3>
      <div class="sticks-row">
        <div
          v-for="stick in getUnassignedSticks"
          :key="stick.id"
          class="stick-pool-slot"
        >
          <div
            class="stick-card"
            :class="{ short: stick.lengthType === 'short', covered: !isFinished }"
            draggable="true"
            @dragstart="onDragStart(stick.id)"
            @dragend="onDragEnd"
          >
            <div
              v-if="!isFinished"
              class="stick-cover"
              :class="stick.lengthType === 'short' ? 'stick-cover--short' : 'stick-cover--long'"
            >
              👍
            </div>
          </div>
        </div>
      </div>
      <p class="assignment-status">
        {{ assignedCount }} / {{ participantNames.length }} assigned
      </p>
    </div>
  </div>
</template>

<style scoped>
.drawing-sticks-game {
  width: 100vw;
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  padding: 2rem;
  box-sizing: border-box;
}

.game-header {
  text-align: center;
  margin-bottom: 2rem;
}

.game-header h2 {
  font-size: 2.4rem;
  margin-bottom: 0.6rem;
}

.game-header p {
  font-size: 1.1rem;
  opacity: 0.9;
  margin: 0;
}

.participants-row {
  width: min(1200px, 95vw);
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(170px, 1fr));
  gap: 1rem;
  margin-bottom: 2rem;
}

.participant-slot {
  background: rgba(255, 255, 255, 0.12);
  border: 1px solid rgba(255, 255, 255, 0.25);
  border-radius: 14px;
  padding: 0.8rem;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.participant-name {
  font-weight: 700;
  margin-bottom: 0.75rem;
}

/* Fixed height so grid row size does not jump when more participants get sticks (long vs short). */
.participant-stick-area {
  width: 100%;
  min-height: 200px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: flex-start;
  flex-shrink: 0;
}

.participant-stick-area--empty {
  justify-content: center;
}

.drop-hint {
  font-size: 0.9rem;
  opacity: 0.75;
  text-align: center;
}

.sticks-pool {
  width: min(1200px, 95vw);
  background: rgba(255, 255, 255, 0.12);
  border-radius: 16px;
  border: 1px solid rgba(255, 255, 255, 0.25);
  padding: 1rem;
}

.sticks-pool h3 {
  margin: 0 0 0.8rem 0;
  text-align: center;
}

.sticks-row {
  display: flex;
  flex-wrap: wrap;
  gap: 7rem;
  justify-content: center;
  align-items: flex-start;
  /* Same height whether the pool has long sticks, a short stick, or one short stick left. */
  min-height: 200px;
}

/* Fixed width keeps sticks centered in each column; top-align so short and long share the same top edge. */
.stick-pool-slot {
  display: flex;
  align-items: flex-start;
  justify-content: center;
  width: 72px;
  min-height: 200px;
  flex: 0 0 auto;
}

.stick-card,
.assigned-stick {
  --stick-length: 120px;
  width: 20px;
  height: var(--stick-length);
  background: linear-gradient(180deg, #d4a574, #b8956a);
  border-radius: 999px;
  border: 2px solid #9a7349;
  position: relative;
  cursor: grab;
}

.stick-card.short,
.assigned-stick.short {
  --stick-length: 90px;
}

.stick-card:active,
.assigned-stick:active {
  cursor: grabbing;
}

.stick-cover {
  position: absolute;
  left: -50%;
  display: flex;
  align-items: flex-end;
  justify-content: center;
  font-size: 48px;
  line-height: 1;
  overflow: hidden;
  transform: translateX(-50%) scale(1.5);
  transform-origin: bottom center;
}

/* Long sticks: cover sits on the bottom edge of the stick (same as pool + participant columns). */
.stick-cover--long {
  bottom: -5px;
}

/* Short stick: offset down so the cover’s bottom lines up with long-stick covers when rows are top-aligned. */
.stick-cover--short {
  bottom: calc(-5px - (120px - var(--stick-length)));
}

.assignment-status {
  text-align: center;
  margin: 0.8rem 0 0 0;
  font-weight: 700;
}

@media (max-width: 700px) {
  .game-header h2 {
    font-size: 1.9rem;
  }

  .participants-row {
    grid-template-columns: repeat(auto-fit, minmax(140px, 1fr));
  }
}
</style>

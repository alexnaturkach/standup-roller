<script setup lang="ts">
import { ref, onMounted, onUnmounted, reactive } from 'vue'

const props = defineProps<{
  participants?: string[]
}>()

const emit = defineEmits<{
  'winner-selected': [winner: string]
}>()

// ── Constants ──────────────────────────────────────────────────────────────
const CW = 1300
const CH = 800
const BR = 12
const PW = 160
const PH = 14
const PY = CH - 50
const BLK_Y = 50
const BLK_H = 38
const BLK_GAP = 6
const COLS = 7
const BLOCKS_PER_PERSON = 7
const REVEAL_MS = 1800
const BASE_SPEED = 6

const COLORS = [
  '#FF6B6B', '#4ECDC4', '#45B7D1', '#96CEB4',
  '#F7DC6F', '#BB8FCE', '#F0A500', '#58D68D',
  '#EC407A', '#26C6DA', '#FFA726', '#AB47BC',
]

// ── State ──────────────────────────────────────────────────────────────────
const canvas = ref<HTMLCanvasElement | null>(null)
const allNames = props.participants ?? ['Alex', 'Casey', 'Robby', 'Sheila']

const scores = reactive<Record<string, number>>(
  Object.fromEntries(allNames.map(n => [n, 0]))
)
const winTarget = ref(5)
const activeNames = ref<string[]>([...allNames])
const phase = ref<'idle' | 'playing' | 'paused' | 'done'>('idle')
const winnerName = ref('')
const showResult = ref(false)
const overlayMsg = ref('')

// ── Mutable game objects ───────────────────────────────────────────────────
let raf = 0
let lastTs = 0
const ball = { x: CW / 2, y: CH / 2, vx: 0, vy: 0 }
const paddle = { x: CW / 2 - PW / 2 }

interface Block {
  x: number; y: number; w: number; h: number
  pIdx: number
  hidden: boolean
  revealEnd: number
  destroyed: boolean
}
let blocks: Block[] = []

// ── Setup ──────────────────────────────────────────────────────────────────
const buildBlocks = () => {
  const n = activeNames.value.length
  const bw = (CW - BLK_GAP * (COLS + 1)) / COLS

  // Each participant gets BLOCKS_PER_PERSON blocks; shuffle all assignments
  const assignments: number[] = []
  for (let i = 0; i < n; i++) {
    for (let j = 0; j < BLOCKS_PER_PERSON; j++) assignments.push(i)
  }
  for (let i = assignments.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1))
    const tmp = assignments[i]!
    assignments[i] = assignments[j]!
    assignments[j] = tmp
  }

  const rows = n // n rows × 7 cols = 7n total blocks
  blocks = []
  for (let row = 0; row < rows; row++) {
    for (let col = 0; col < COLS; col++) {
      blocks.push({
        x: BLK_GAP + col * (bw + BLK_GAP),
        y: BLK_Y + row * (BLK_H + BLK_GAP),
        w: bw,
        h: BLK_H,
        pIdx: assignments[row * COLS + col] ?? 0,
        hidden: true,
        revealEnd: 0,
        destroyed: false,
      })
    }
  }
}

const launch = () => {
  ball.x = CW / 2
  ball.y = PY - 150
  const speed = BASE_SPEED + (winTarget.value === 3 ? 0.8 : 0)
  const spread = (Math.random() * 40 - 20) * (Math.PI / 180)
  const side = Math.random() > 0.5 ? 1 : -1
  ball.vx = Math.sin(spread) * speed + side * speed * 0.15
  ball.vy = -Math.cos(spread) * speed
}

const startRound = (names: string[], target: number) => {
  cancelAnimationFrame(raf)
  activeNames.value = names
  winTarget.value = target
  allNames.forEach(n => (scores[n] = 0))
  buildBlocks()
  paddle.x = CW / 2 - PW / 2
  launch()
  overlayMsg.value = ''
  phase.value = 'playing'
  lastTs = performance.now()
  raf = requestAnimationFrame(loop)
}

// ── Input ──────────────────────────────────────────────────────────────────
const onMouseMove = (e: MouseEvent) => {
  if (phase.value !== 'playing' || !canvas.value) return
  const rect = canvas.value.getBoundingClientRect()
  const mx = (e.clientX - rect.left) * (CW / rect.width)
  paddle.x = Math.max(0, Math.min(CW - PW, mx - PW / 2))
}

const onTouchMove = (e: TouchEvent) => {
  if (phase.value !== 'playing' || !canvas.value) return
  e.preventDefault()
  const rect = canvas.value.getBoundingClientRect()
  const touch = e.touches[0]
  if (!touch) return
  const mx = (touch.clientX - rect.left) * (CW / rect.width)
  paddle.x = Math.max(0, Math.min(CW - PW, mx - PW / 2))
}

// ── Block collision ────────────────────────────────────────────────────────
const hitBlocks = (now: number) => {
  for (const b of blocks) {
    if (b.destroyed || !b.hidden) continue  // skip destroyed and already-hit blocks
    if (
      ball.x + BR > b.x && ball.x - BR < b.x + b.w &&
      ball.y + BR > b.y && ball.y - BR < b.y + b.h
    ) {
      const ol = ball.x + BR - b.x
      const or_ = b.x + b.w - (ball.x - BR)
      const ot = ball.y + BR - b.y
      const ob = b.y + b.h - (ball.y - BR)
      const min = Math.min(ol, or_, ot, ob)
      if (min === ot || min === ob) ball.vy = -ball.vy
      else ball.vx = -ball.vx

      // Reveal duration shrinks as speed increases so fast balls don't linger
      const hitSpeed = Math.sqrt(ball.vx ** 2 + ball.vy ** 2)
      const revealDuration = Math.max(300, REVEAL_MS / (hitSpeed / BASE_SPEED))
      b.hidden = false
      b.revealEnd = now + revealDuration

      const name = activeNames.value[b.pIdx]
      if (!name) return
      scores[name] = (scores[name] ?? 0) + 1

      if ((scores[name] ?? 0) >= winTarget.value) endGame(name)
      return
    }
  }
}

// ── End conditions ─────────────────────────────────────────────────────────
const endGame = (winner: string) => {
  phase.value = 'done'
  cancelAnimationFrame(raf)
  winnerName.value = winner
  setTimeout(() => {
    showResult.value = true
    setTimeout(() => emit('winner-selected', winner), 2000)
  }, 600)
}

const onBallLost = () => {
  phase.value = 'paused'
  cancelAnimationFrame(raf)

  const maxSc = Math.max(...activeNames.value.map(n => scores[n] ?? 0))
  const tied = activeNames.value.filter(n => (scores[n] ?? 0) === maxSc)

  if (tied.length === 1) {
    const sole = tied[0]
    if (sole) endGame(sole)
    return
  }

  const nextTarget = winTarget.value === 5 ? 3 : winTarget.value
  overlayMsg.value = `🔥 Tiebreaker! ${tied.join(' vs ')} — first to ${nextTarget}!`
  setTimeout(() => startRound(tied, nextTarget), 3000)
}

// ── Render ─────────────────────────────────────────────────────────────────
const draw = (now: number) => {
  const c = canvas.value?.getContext('2d')
  if (!c) return

  c.fillStyle = '#0d0d1a'
  c.fillRect(0, 0, CW, CH)

  c.strokeStyle = '#1e1e3a'
  c.lineWidth = 1
  c.beginPath()
  c.moveTo(0, BLK_Y - 14)
  c.lineTo(CW, BLK_Y - 14)
  c.stroke()

  for (const b of blocks) {
    if (b.destroyed) continue

    const revealed = !b.hidden && now < b.revealEnd
    const name = activeNames.value[b.pIdx]
    if (!name) continue
    const ci = allNames.indexOf(name) % COLORS.length
    const blockColor = COLORS[ci] ?? '#FF6B6B'

    c.fillStyle = revealed ? blockColor : '#252540'
    c.beginPath()
    c.roundRect(b.x, b.y, b.w, b.h, 5)
    c.fill()

    c.strokeStyle = revealed ? 'rgba(255,255,255,0.2)' : '#3a3a5a'
    c.lineWidth = 1.5
    c.stroke()

    c.fillStyle = revealed ? '#fff' : '#55558a'
    c.font = `bold ${revealed && name.length > 7 ? 13 : 15}px system-ui, sans-serif`
    c.textAlign = 'center'
    c.textBaseline = 'middle'
    c.fillText(revealed ? name : '?', b.x + b.w / 2, b.y + b.h / 2, b.w - 8)
  }

  // Paddle
  c.fillStyle = '#4ECDC4'
  c.beginPath()
  c.roundRect(paddle.x, PY, PW, PH, 8)
  c.fill()

  // Ball glow
  const grad = c.createRadialGradient(ball.x, ball.y, 0, ball.x, ball.y, BR + 8)
  grad.addColorStop(0, 'rgba(255,107,107,0.5)')
  grad.addColorStop(1, 'rgba(255,107,107,0)')
  c.fillStyle = grad
  c.beginPath()
  c.arc(ball.x, ball.y, BR + 8, 0, Math.PI * 2)
  c.fill()

  // Ball
  c.beginPath()
  c.arc(ball.x, ball.y, BR, 0, Math.PI * 2)
  c.fillStyle = '#FF6B6B'
  c.fill()

  // Border
  c.strokeStyle = '#252540'
  c.lineWidth = 2
  c.strokeRect(1, 1, CW - 2, CH - 2)
}

// ── Game loop ──────────────────────────────────────────────────────────────
const loop = (now: number) => {
  if (phase.value !== 'playing') return

  const dt = Math.min((now - lastTs) / 16.67, 3)
  lastTs = now

  // Revealed blocks are destroyed once the reveal timer expires
  for (const b of blocks) {
    if (!b.hidden && now >= b.revealEnd) b.destroyed = true
  }

  // Speed scales with blocks destroyed: +1.5 per every 2 blocks
  const destroyedCount = blocks.filter(b => b.destroyed).length
  const targetSpeed = BASE_SPEED + (winTarget.value === 3 ? 0.8 : 0) + Math.floor(destroyedCount / 2) * 1.5
  const curSpd = Math.sqrt(ball.vx ** 2 + ball.vy ** 2)
  if (curSpd > 0.01) {
    ball.vx = (ball.vx / curSpd) * targetSpeed
    ball.vy = (ball.vy / curSpd) * targetSpeed
  }

  ball.x += ball.vx * dt
  ball.y += ball.vy * dt

  if (ball.x - BR <= 0) { ball.x = BR; ball.vx = Math.abs(ball.vx) }
  if (ball.x + BR >= CW) { ball.x = CW - BR; ball.vx = -Math.abs(ball.vx) }
  if (ball.y - BR <= 0) { ball.y = BR; ball.vy = Math.abs(ball.vy) }

  if (
    ball.vy > 0 &&
    ball.y + BR >= PY && ball.y + BR <= PY + PH + 4 &&
    ball.x + BR >= paddle.x && ball.x - BR <= paddle.x + PW
  ) {
    const rel = (ball.x - paddle.x) / PW
    const angle = (rel - 0.5) * 2.2
    const spd = Math.sqrt(ball.vx ** 2 + ball.vy ** 2)
    ball.vx = Math.sin(angle) * spd
    ball.vy = -Math.abs(Math.cos(angle) * spd)
    ball.y = PY - BR
    if (Math.abs(ball.vy) < 2.5) ball.vy = -2.5
  }

  hitBlocks(now)

  if (ball.y - BR > CH) {
    draw(now)
    onBallLost()
    return
  }

  draw(now)

  if (phase.value === 'playing') {
    raf = requestAnimationFrame(loop)
  }
}

// ── Lifecycle ──────────────────────────────────────────────────────────────
onMounted(() => {
  canvas.value?.addEventListener('mousemove', onMouseMove)
  canvas.value?.addEventListener('touchmove', onTouchMove, { passive: false })
  draw(performance.now())
})

onUnmounted(() => {
  cancelAnimationFrame(raf)
  if (canvas.value) {
    canvas.value.removeEventListener('mousemove', onMouseMove)
    canvas.value.removeEventListener('touchmove', onTouchMove)
  }
})
</script>

<template>
  <div class="pong-game">
    <div class="game-header">
      <h2>🏓 Pong</h2>
      <div v-if="phase !== 'idle'" class="target-info">
        First to <strong>{{ winTarget }}</strong> wins
        <span v-if="winTarget === 3" class="tiebreaker-badge">TIEBREAKER</span>
      </div>
    </div>

    <div class="game-body">
      <div class="canvas-wrap">
        <canvas ref="canvas" :width="CW" :height="CH" />

        <div v-if="overlayMsg" class="overlay-banner">{{ overlayMsg }}</div>

        <div v-if="phase === 'idle'" class="overlay-start">
          <p class="start-hint">Move your mouse to control the paddle</p>
          <button class="start-btn" @click="startRound([...allNames], 5)">
            Start Game 🏓
          </button>
        </div>
      </div>

      <div class="scoreboard">
        <h3>Scoreboard</h3>
        <div class="score-target">Target: {{ winTarget }}</div>
        <div
          v-for="name in allNames"
          :key="name"
          class="score-row"
          :class="{ inactive: !activeNames.includes(name) }"
        >
          <div class="score-name">{{ name }}</div>
          <div class="bar-track">
            <div
              class="bar-fill"
              :style="{
                width: `${Math.min(((scores[name] ?? 0) / winTarget) * 100, 100)}%`,
                backgroundColor: COLORS[allNames.indexOf(name) % COLORS.length] ?? '#888',
              }"
            />
          </div>
          <div class="score-num">{{ scores[name] ?? 0 }}</div>
        </div>
      </div>
    </div>

    <div v-if="showResult" class="result-overlay">
      <div class="result-content">
        <h3>Selected Participant</h3>
        <div class="winner-name">{{ winnerName }}</div>
        <p>will start today's standup</p>
      </div>
    </div>
  </div>
</template>

<style scoped>
.pong-game {
  width: 100vw;
  height: 100vh;
  display: flex;
  flex-direction: column;
  background: linear-gradient(160deg, #0d0d1a 0%, #1a1a2e 100%);
  color: white;
  position: relative;
  overflow: hidden;
}

.game-header {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 1.5rem;
  padding: 0.7rem 1.5rem;
  flex-shrink: 0;
}

.game-header h2 {
  font-size: 1.8rem;
  margin: 0;
  text-shadow: 0 0 20px rgba(78, 205, 196, 0.5);
}

.target-info {
  font-size: 1rem;
  opacity: 0.8;
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.tiebreaker-badge {
  background: #FF6B6B;
  color: white;
  font-size: 0.7rem;
  font-weight: bold;
  padding: 2px 8px;
  border-radius: 20px;
  letter-spacing: 0.05em;
  animation: pulse 1s infinite alternate;
}

.game-body {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 1.5rem;
  padding: 0 1.5rem 1rem;
  min-height: 0;
}

.canvas-wrap {
  position: relative;
  border-radius: 12px;
  overflow: hidden;
  box-shadow:
    0 0 40px rgba(78, 205, 196, 0.12),
    0 8px 32px rgba(0, 0, 0, 0.5);
  flex-shrink: 0;
  line-height: 0;
}

.canvas-wrap canvas {
  display: block;
  max-width: calc(100vw - 220px);
  max-height: calc(100vh - 100px);
}

.overlay-banner {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background: rgba(0, 0, 0, 0.88);
  border: 2px solid #FFD700;
  color: #FFD700;
  font-size: 1.2rem;
  font-weight: bold;
  padding: 1rem 2rem;
  border-radius: 12px;
  text-align: center;
  white-space: nowrap;
  animation: fadeIn 0.3s ease-out;
  backdrop-filter: blur(8px);
  z-index: 10;
}

.overlay-start {
  position: absolute;
  inset: 0;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 1rem;
  background: rgba(13, 13, 26, 0.85);
  backdrop-filter: blur(4px);
}

.start-hint {
  font-size: 0.9rem;
  opacity: 0.6;
  margin: 0;
}

.start-btn {
  background: linear-gradient(45deg, #4ECDC4, #45B7D1);
  color: white;
  border: none;
  padding: 0.9rem 2.5rem;
  font-size: 1.3rem;
  font-weight: bold;
  border-radius: 50px;
  cursor: pointer;
  transition: all 0.2s;
  box-shadow: 0 4px 20px rgba(78, 205, 196, 0.4);
}

.start-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 25px rgba(78, 205, 196, 0.6);
}

.scoreboard {
  width: 165px;
  flex-shrink: 0;
  background: rgba(255, 255, 255, 0.04);
  border: 1px solid rgba(255, 255, 255, 0.08);
  border-radius: 12px;
  padding: 1rem;
}

.scoreboard h3 {
  font-size: 1rem;
  margin: 0 0 0.25rem 0;
  text-align: center;
  color: #4ECDC4;
  letter-spacing: 0.05em;
}

.score-target {
  font-size: 0.75rem;
  text-align: center;
  color: rgba(255, 255, 255, 0.35);
  margin-bottom: 0.8rem;
}

.score-row {
  display: flex;
  align-items: center;
  gap: 0.4rem;
  margin-bottom: 0.55rem;
  transition: opacity 0.3s;
}

.score-row.inactive {
  opacity: 0.3;
}

.score-name {
  font-size: 0.8rem;
  font-weight: bold;
  width: 52px;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  flex-shrink: 0;
}

.bar-track {
  flex: 1;
  height: 8px;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 4px;
  overflow: hidden;
}

.bar-fill {
  height: 100%;
  border-radius: 4px;
  transition: width 0.3s ease;
}

.score-num {
  font-size: 0.85rem;
  font-weight: bold;
  width: 16px;
  text-align: right;
  flex-shrink: 0;
}

.result-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.85);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
  animation: fadeIn 0.4s ease-out;
}

.result-content {
  background: linear-gradient(135deg, #1a1a2e, #16213e);
  border: 2px solid #4ECDC4;
  color: white;
  padding: 3rem 4rem;
  border-radius: 20px;
  text-align: center;
  box-shadow:
    0 0 60px rgba(78, 205, 196, 0.3),
    0 10px 30px rgba(0, 0, 0, 0.5);
  animation: popIn 0.4s ease-out;
}

.result-content h3 {
  font-size: 1.5rem;
  color: #4ECDC4;
  margin-bottom: 1rem;
}

.winner-name {
  font-size: 3rem;
  font-weight: bold;
  color: #FFD700;
  margin-bottom: 0.5rem;
  text-shadow: 0 0 20px rgba(255, 215, 0, 0.5);
}

.result-content p {
  font-size: 1.1rem;
  opacity: 0.7;
  margin: 0;
}

@keyframes fadeIn { from { opacity: 0 } to { opacity: 1 } }
@keyframes popIn  {
  from { opacity: 0; transform: scale(0.85) }
  to   { opacity: 1; transform: scale(1) }
}
@keyframes pulse  { 0% { opacity: 1 } 100% { opacity: 0.6 } }
</style>

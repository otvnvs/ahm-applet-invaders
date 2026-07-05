<template>
  <div class="a">
    <header class="b">
      <h1>Invaders</h1>
      <p>Score: <b>{{ s }}</b> | Status: <b :class="{ r: o && !w, g: w }">{{ o ? (w ? 'WINNER' : 'OVER') : 'ALIVE' }}</b></p>
    </header>

    <main class="c">
      <div v-if="o" class="o" @click="r">
        <h2>{{ w ? 'MISSION SUCCESS' : 'SYSTEM CRASH' }}</h2>
        <p>Tap inside grid to redeploy</p>
      </div>
      <canvas ref="g" width="320" height="360"></canvas>
    </main>

    <!-- Fixed Bottom Dual Circle Touch Controllers -->
    <footer class="f">
      <div class="ctrl-pad" ref="lPad" @touchstart.prevent="tl">
        <div class="btn-inner">FIRE</div>
      </div>
      <div class="ctrl-pad" ref="rPad" @touchstart.prevent="ts" @touchmove.prevent="tm" @touchend.prevent="te">
        <div class="btn-inner animate-pulse">STEER</div>
      </div>
    </footer>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

const s = ref(0), o = ref(false), w = ref(false), g = ref(null), lPad = ref(null), rPad = ref(null)
let cn, ctx, t, p = 145, m = [], b = [], e = 1, d = 2, sx = 0, au = null

// Native Web Audio Synthesizer Engine
const initAudio = () => {
  if (!au) au = new (window.AudioContext || window.webkitAudioContext)()
}

const playSound = (type) => {
  if (!au) return
  const osc = au.createOscillator()
  const gain = au.createGain()
  osc.connect(gain)
  gain.connect(au.destination)

  if (type === 'laser') {
    osc.type = 'sawtooth'
    osc.frequency.setValueAtTime(700, au.currentTime)
    osc.frequency.exponentialRampToValueAtTime(150, au.currentTime + 0.12)
    gain.gain.setValueAtTime(0.12, au.currentTime)
    gain.gain.linearRampToValueAtTime(0.01, au.currentTime + 0.12)
    osc.start(); osc.stop(au.currentTime + 0.12)
  } else if (type === 'boom') {
    osc.type = 'triangle'
    osc.frequency.setValueAtTime(120, au.currentTime)
    osc.frequency.linearRampToValueAtTime(30, au.currentTime + 0.25)
    gain.gain.setValueAtTime(0.35, au.currentTime)
    gain.gain.linearRampToValueAtTime(0.01, au.currentTime + 0.25)
    osc.start(); osc.stop(au.currentTime + 0.25)
  } else if (type === 'steer') {
    osc.type = 'sine'
    osc.frequency.setValueAtTime(220, au.currentTime)
    osc.frequency.linearRampToValueAtTime(280, au.currentTime + 0.04)
    gain.gain.setValueAtTime(0.04, au.currentTime)
    gain.gain.linearRampToValueAtTime(0.01, au.currentTime + 0.04)
    osc.start(); osc.stop(au.currentTime + 0.04)
  }
}

const r = () => {
  initAudio()
  s.value = 0; o.value = false; w.value = false; p = 145; b = []; m = []
  for (let y = 0; y < 3; y++) {
    for (let x = 0; x < 6; x++) m.push({ x: x * 40 + 30, y: y * 25 + 40, w: 24, h: 14, a: 1 })
  }
}

// Left Circle - Trigger Laser Beam & Play Sound Effects
const tl = () => {
  initAudio()
  if (!o.value && b.length < 4) {
    b.push({ x: p + 14, y: 325 })
    playSound('laser')
  }
}

// Right Circle - Horizontal Steering Engine with Client delta tracking
const ts = (evt) => {
  initAudio()
  if (evt.touches.length) sx = evt.touches[0].clientX
}

const tm = (evt) => {
  if (!cn || o.value || !evt.touches.length) return
  const dx = evt.touches[0].clientX - sx
  
  // Highly calibrated 1.8 shift sensitivity multiplier matching your preferred setup
  p = Math.max(10, Math.min(280, p + dx * 1.8))
  
  if (Math.abs(dx) > 1) playSound('steer')
  sx = evt.touches[0].clientX
}

const te = () => { sx = 0 }

const u = () => {
  if (o.value) return
  ctx.fillStyle = '#111112'
  ctx.fillRect(0, 0, 320, 360)

  // Bullet tracking processing
  b.forEach((l, idx) => {
    l.y -= 5
    if (l.y < 0) b.splice(idx, 1)
    ctx.fillStyle = '#fff'
    ctx.fillRect(l.x, l.y, 2, 8)
  })

  // Alien formation loops
  let h = false, alive = 0
  m.forEach(i => {
    if (!i.a) return
    alive++
    i.x += e * d
    if (i.x + i.w > 310 || i.x < 10) h = true
    
    b.forEach((l, lIdx) => {
      if (l.x > i.x && l.x < i.x + i.w && l.y > i.y && l.y < i.y + i.h) {
        i.a = 0; b.splice(lIdx, 1); s.value += 20
        playSound('boom')
      }
    })

    if (i.y + i.h >= 330) { o.value = true; playSound('boom') }

    ctx.fillStyle = '#3a3a3c'
    ctx.fillRect(i.x, i.y, i.w, i.h)
  })

  if (alive === 0) { w.value = true; o.value = true }

  if (h) {
    e *= -1
    m.forEach(i => { i.y += 10 })
  }

  // Draw modern sleek square player block asset
  ctx.fillStyle = '#3a3a3c'
  ctx.fillRect(p, 330, 30, 10)
  ctx.fillRect(p + 11, 325, 8, 5)
}

onMounted(() => {
  cn = g.value; ctx = cn.getContext('2d')
  r(); t = setInterval(u, 1000 / 60)
  
  window.addEventListener('keydown', evt => {
    initAudio()
    if (evt.key === ' ') tl()
    if (evt.key === 'ArrowLeft') { p = Math.max(10, p - 15); playSound('steer') }
    if (evt.key === 'ArrowRight') { p = Math.min(280, p + 15); playSound('steer') }
  })
})
onUnmounted(() => clearInterval(t))
</script>

<style scoped>
.a { width: 100vw; height: 100vh; background: #000; color: #fff; display: flex; flex-direction: column; align-items: center; justify-content: space-between; padding: 16px; font-family: sans-serif; box-sizing: border-box; user-select: none; overflow: hidden; }
.b { text-align: center; margin-bottom: 5px; width: 100%; }
h1 { margin: 0; font-size: 20px; text-transform: uppercase; color: #e2e8f0; letter-spacing: 1.5px; }
p { margin: 4px 0; color: #71717a; font-size: 13px; }
b { color: #fff; } b.r { color: #ef4444; } b.g { color: #22c55e; }
.c { width: 100%; max-width: 320px; aspect-ratio: 320 / 360; position: relative; border: 2px solid #1c1c1e; box-shadow: 0 20px 40px rgba(0,0,0,0.6); background: #111112; }
canvas { display: block; width: 100%; height: 100%; }
.o { position: absolute; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.95); display: flex; flex-direction: column; align-items: center; justify-content: center; z-index: 10; cursor: pointer; }
.o h2 { color: #ef4444; margin: 0; font-size: 20px; letter-spacing: 1px; text-align: center; }
.o p { color: #71717a; font-size: 12px; margin-top: 6px; }

/* Fixed Layout Control Sheet Base Layouts */
.f { width: 100%; max-width: 320px; display: flex; justify-content: space-between; padding: 16px 0; box-sizing: border-box; }
.ctrl-pad { width: 76px; height: 76px; background: #111112; border: 2px solid #2c2c2e; border-radius: 50%; display: flex; align-items: center; justify-content: center; touch-action: none; cursor: pointer; }
.ctrl-pad:active { background: #1c1c1e; border-color: #3a3a3c; }
.btn-inner { font-size: 10px; font-weight: 800; color: #71717a; letter-spacing: 0.5px; text-transform: uppercase; pointer-events: none; }
.ctrl-pad:active .btn-inner { color: #fff; }
@keyframes pulse { 0%, 100% { opacity: 0.6; } 50% { opacity: 1; } }
.animate-pulse { animation: pulse 2s cubic-bezier(0.4, 0, 0.6, 1) infinite; }
@media (max-width: 360px) { .c { max-width: 90vw; } .f { max-width: 90vw; } }
</style>

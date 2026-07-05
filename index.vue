<template>
  <div class="a" @touchmove="tm" @mousemove="mm">
    <header class="b">
      <h1>Invaders</h1>
      <p>Score: <b>{{ s }}</b> | Status: <b :class="{ r: o && w == false, g: w }">{{ o ? (w ? 'WINNER' : 'OVER') : 'ALIVE' }}</b></p>
    </header>

    <main class="c">
      <div v-if="o" class="o" @click="r">
        <h2>{{ w ? 'MISSION SUCCESS' : 'SYSTEM CRASH' }}</h2>
        <p>Tap inside grid to redeploy</p>
      </div>
      <canvas ref="g" width="320" height="400"></canvas>
    </main>

    <footer class="f">
      <p class="h">Slide screen horizontally to steer & fire</p>
    </footer>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

const s = ref(0), o = ref(false), w = ref(false), g = ref(null)
let c, ctx, t, p = 145, m = [], b = [], e = 1, d = 2

const r = () => {
  s.value = 0; o.value = false; w.value = false; p = 145; b = []; m = []
  for (let y = 0; y < 3; y++) {
    for (let x = 0; x < 6; x++) m.push({ x: x * 40 + 30, y: y * 25 + 40, w: 24, h: 14, a: 1 })
  }
}

const mm = (e) => { if (c) p = Math.max(10, Math.min(280, e.clientX - c.getBoundingClientRect().left - 15)) }
const tm = (e) => { if (c && e.touches[0]) p = Math.max(10, Math.min(280, e.touches[0].clientX - c.getBoundingClientRect().left - 15)) }

const u = () => {
  if (o.value) return
  ctx.fillStyle = '#111112'
  ctx.fillRect(0, 0, 320, 400)

  // Auto shoot intervals simulation
  if (Math.random() < 0.06 && b.length < 4) b.push({ x: p + 14, y: 365 })

  // Bullet tracking logic
  b.forEach((l, idx) => {
    l.y -= 5
    if (l.y < 0) b.splice(idx, 1)
    ctx.fillStyle = '#fff'
    ctx.fillRect(l.x, l.y, 2, 8)
  })

  // Alien march matrix calculations
  let h = false, alive = 0
  m.forEach(i => {
    if (!i.a) return
    alive++
    i.x += e * d
    if (i.x + i.w > 310 || i.x < 10) h = true
    
    // Laser crosscheck bounds
    b.forEach((l, lIdx) => {
      if (l.x > i.x && l.x < i.x + i.w && l.y > i.y && l.y < i.y + i.h) {
        i.a = 0; b.splice(lIdx, 1); s.value += 20
      }
    })

    // Ground breach condition
    if (i.y + i.h >= 370) o.value = true

    // Draw monochromatic alien box nodes
    ctx.fillStyle = '#3a3a3c'
    ctx.fillRect(i.x, i.y, i.w, i.h)
  })

  if (alive === 0) { w.value = true; o.value = true }

  if (h) {
    e *= -1
    m.forEach(i => { i.y += 10 })
  }

  // Draw modern sleek square player block
  ctx.fillStyle = '#3a3a3c'
  ctx.fillRect(p, 370, 30, 10)
  ctx.fillRect(p + 11, 365, 8, 5)
}

onMounted(() => {
  c = g.value; ctx = c.getContext('2d')
  r(); t = setInterval(u, 1000 / 60)
})
onUnmounted(() => clearInterval(t))
</script>

<style scoped>
.a { width: 100vw; height: 100vh; background: #000; color: #fff; display: flex; flex-direction: column; align-items: center; justify-content: space-between; padding: 24px; font-family: sans-serif; box-sizing: border-box; user-select: none; overflow: hidden; }
.b { text-align: center; margin-bottom: 5px; width: 100%; }
h1 { margin: 0; font-size: 20px; text-transform: uppercase; color: #e2e8f0; letter-spacing: 1.5px; }
p { margin: 4px 0; color: #71717a; font-size: 13px; }
b { color: #fff; } b.r { color: #ef4444; } b.g { color: #22c55e; }
.c { width: 100%; max-width: 320px; aspect-ratio: 320 / 400; position: relative; border: 2px solid #1c1c1e; box-shadow: 0 20px 40px rgba(0,0,0,0.6); background: #111112; }
canvas { display: block; width: 100%; height: 100%; }
.o { position: absolute; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.95); display: flex; flex-direction: column; align-items: center; justify-content: center; z-index: 10; cursor: pointer; }
.o h2 { color: #ef4444; margin: 0; font-size: 20px; letter-spacing: 1px; text-align: center; }
.o p { color: #71717a; font-size: 12px; margin-top: 6px; }
.f { width: 100%; text-align: center; }
.h { font-size: 11px; text-transform: uppercase; letter-spacing: 0.5px; color: #3a3a3c; }
@media (max-width: 360px) { .c { max-width: 90vw; } }
</style>

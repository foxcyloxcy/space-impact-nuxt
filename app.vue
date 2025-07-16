
// File: ~/pages/index.vue
<template>
  <div class="relative w-screen h-screen bg-black overflow-hidden">
    <div v-if="!gameStarted" class="absolute inset-0 flex flex-col items-center justify-center bg-black bg-opacity-80 z-50">
      <h1 class="text-white text-4xl mb-4">Space Impact</h1>
      <button @click="startGame" class="bg-green-500 px-6 py-2 rounded text-white">Start Game</button>
    </div>

    <div v-else>
      <div class="absolute top-4 left-4 text-white text-xl z-10">Score: {{ score }} | Level: {{ level }} | Lives: {{ lives }}</div>
      <div v-if="bosses.length" class="absolute top-16 left-4 w-64 bg-gray-700 border border-white">
        <div class="bg-purple-600 h-4" :style="{ width: bossHealthPercent + '%' }"></div>
      </div>
      <div class="absolute top-28 left-4 text-white text-sm z-10">Time: {{ elapsedTime }}s | High Score: {{ highScore }}</div>

      <!-- Mobile Controls -->
      <div v-if="isMobile" class="absolute bottom-4 left-4 z-50 flex gap-4">
        <button @touchstart.prevent="moveDir = 'left'" @touchend.prevent="moveDir = null" class="w-16 h-16 bg-white bg-opacity-20 rounded-full"></button>
        <button @touchstart.prevent="moveDir = 'right'" @touchend.prevent="moveDir = null" class="w-16 h-16 bg-white bg-opacity-20 rounded-full"></button>
        <button @touchstart.prevent="moveDir = 'up'" @touchend.prevent="moveDir = null" class="w-16 h-16 bg-white bg-opacity-20 rounded-full"></button>
        <button @touchstart.prevent="moveDir = 'down'" @touchend.prevent="moveDir = null" class="w-16 h-16 bg-white bg-opacity-20 rounded-full"></button>
        <button @touchstart.prevent="shootBullet" class="w-16 h-16 bg-yellow-400 rounded-full">Fire</button>
      </div>

      <canvas ref="canvas" class="w-full h-full"></canvas>
    </div>
  </div>
</template>

<script setup>
import { onMounted, ref } from 'vue'

const canvas = ref(null)
const score = ref(0)
const level = ref(1)
const kills = ref(0)
const elapsedTime = ref(0)
const bossHealth = ref(0)
const bossHealthPercent = ref(100)
const lives = ref(3)
const highScore = ref(0)

if (process.client && score.value > highScore.value) {
  localStorage.setItem('highScore', score.value)
}
const gameStarted = ref(false)
const isMobile = ref(false)
const moveDir = ref(null)
const enemies = []
const bosses = []
const keys = {}


const sfx = {
  shoot: null,
  hit: null,
  bossHit: null,
  death: null,
  music: null
}


const player = {
  x: 50,
  y: 200,
  width: 50,
  height: 50,
  speed: 5,
  bullets: [],
  powerUp: null,
  powerTimer: 0
}

const startGame = () => {
  gameStarted.value = true
  sfx.music.play()
  requestAnimationFrame(gameLoop)
  setInterval(() => {
    elapsedTime.value++
    if (player.powerTimer > 0) {
      player.powerTimer--
      if (player.powerTimer === 0) player.powerUp = null
    }
  }, 1000)

  setInterval(() => {
    if (bosses.length === 0 && kills.value >= 10) {
      enemies.length = 0
      spawnBoss()
    } else if (bosses.length === 0) {
      spawnEnemy()
    }
  }, 1500)
}

onMounted(() => {
  if (process.client) {
  sfx.shoot = new Audio('/shoot.wav')
  sfx.hit = new Audio('/hit.wav')
  sfx.bossHit = new Audio('/boss-hit.wav')
  sfx.death = new Audio('/death.wav')
  sfx.music = new Audio('/background.mp3')
  sfx.music.loop = true
}

  isMobile.value = /Mobi|Android/i.test(navigator.userAgent)
  const c = canvas.value
  const ctx = c.getContext('2d')
  c.width = window.innerWidth
  c.height = window.innerHeight

  document.addEventListener('keydown', (e) => (keys[e.key] = true))
  document.addEventListener('keyup', (e) => (keys[e.key] = false))
  document.addEventListener('keydown', (e) => {
    if (e.code === 'Space') shootBullet()
  })

  function drawPlayer() {
    ctx.fillStyle = 'cyan'
    ctx.fillRect(player.x, player.y, player.width, player.height)
  }

  function drawBullets() {
    ctx.fillStyle = 'yellow'
    player.bullets.forEach((bullet, index) => {
      bullet.x += bullet.speed
      ctx.fillRect(bullet.x, bullet.y, bullet.width, bullet.height)
      if (bullet.x > c.width) player.bullets.splice(index, 1)
    })
  }

  function drawEnemies() {
    ctx.fillStyle = 'red'
    enemies.forEach((enemy, eIndex) => {
      enemy.x -= enemy.speed
      ctx.fillRect(enemy.x, enemy.y, enemy.width, enemy.height)

      // Collision with player
      if (
        enemy.x < player.x + player.width &&
        enemy.x + enemy.width > player.x &&
        enemy.y < player.y + player.height &&
        enemy.y + enemy.height > player.y
      ) {
        alert(`Game Over! Final Score: ${score.value}`)
        location.reload()
      }

      // Bullet collision
      player.bullets.forEach((bullet, bIndex) => {
        if (
          bullet.x < enemy.x + enemy.width &&
          bullet.x + bullet.width > enemy.x &&
          bullet.y < enemy.y + enemy.height &&
          bullet.y + bullet.height > enemy.y
        ) {
          enemies.splice(eIndex, 1)
          player.bullets.splice(bIndex, 1)
          score.value += 10
        }
      })

      if (enemy.x + enemy.width < 0) enemies.splice(eIndex, 1)
    })
  }

function movePlayer() {
  if (keys['ArrowUp'] || moveDir.value === 'up') player.y -= player.speed
  if (keys['ArrowDown'] || moveDir.value === 'down') player.y += player.speed
  if (keys['ArrowLeft'] || moveDir.value === 'left') player.x -= player.speed
  if (keys['ArrowRight'] || moveDir.value === 'right') player.x += player.speed

  player.x = Math.max(0, Math.min(player.x, canvas.value.width - player.width))
  player.y = Math.max(0, Math.min(player.y, canvas.value.height - player.height))
}

  function shootBullet() {
    const bullet = {
      x: player.x + player.width,
      y: player.y + player.height / 2 - 5,
      width: 10,
      height: 5,
      speed: 8
    }
    player.bullets.push(bullet)
  }

  function spawnEnemy() {
    const enemy = {
      x: c.width,
      y: Math.random() * (c.height - 40),
      width: 40,
      height: 40,
      speed: 2 + Math.random() * 2
    }
    enemies.push(enemy)
  }

  setInterval(spawnEnemy, 1500)

  function gameLoop() {
    ctx.clearRect(0, 0, c.width, c.height)
    movePlayer()
    drawPlayer()
    drawBullets()
    drawEnemies()
    requestAnimationFrame(gameLoop)
  }

  gameLoop()
})
</script>

<style>
html, body {
  margin: 0;
  padding: 0;
  overflow: hidden;
}
</style>

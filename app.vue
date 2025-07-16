
// File: ~/pages/index.vue
<template>
  <div class="relative w-screen h-screen bg-black overflow-hidden">
    <div class="absolute top-4 left-4 text-white text-xl z-10">Score: {{ score }}</div>
    <canvas ref="canvas" class="w-full h-full"></canvas>
  </div>
</template>

<script setup>
import { onMounted, ref } from 'vue'

const canvas = ref(null)
const score = ref(0)

const player = {
  x: 50,
  y: 200,
  width: 50,
  height: 50,
  speed: 5,
  bullets: []
}

const enemies = []
const keys = {}

onMounted(() => {
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
    if (keys['ArrowUp'] && player.y > 0) player.y -= player.speed
    if (keys['ArrowDown'] && player.y + player.height < c.height) player.y += player.speed
    if (keys['ArrowLeft'] && player.x > 0) player.x -= player.speed
    if (keys['ArrowRight'] && player.x + player.width < c.width) player.x += player.speed
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

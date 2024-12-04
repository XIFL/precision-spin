<template>
  <div>
    <canvas ref="canvas" :width="canvasSize" :height="canvasSize"></canvas>
    <p v-if="!gameStarted" class="instructions">
      How To Play: When the red line overlaps with the yellow section, press spacebar.
    </p>
    <p v-if="!gameStarted" class="start-screen">Press Spacebar to Start</p>
    <p v-if="gameOver" class="game-over">Game Over! Your Final Score: {{ score }}</p>
    <p>Score: {{ score }}</p>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from "vue";

// Canvas dimensions
const canvasSize = 400;
const outerRadius = 150;
const innerRadius = 100;
const centerX = canvasSize / 2;
const centerY = canvasSize / 2;

const canvas = ref(null);
let animationFrame;

// Game control variables
let angle = 0; // Initial angle
let speed = 0.007; // Starting speed
const speedIncrement = 0.001; // Increase per rotation
const tolerance = Math.PI / 8; // Updated tolerance

const score = ref(0);
const gameOver = ref(false);
const gameStarted = ref(false);

// Detect if red line overlaps with the yellow target zone
const isLineInTarget = (angle) => {
  const targetCenterAngle = -Math.PI / 2;
  return Math.abs((angle % (2 * Math.PI)) - targetCenterAngle) <= tolerance;
};

// Draw the static game elements (before starting)
const drawStaticGame = (ctx) => {
  // Draw yellow target zone
  const targetStartAngle = -(Math.PI / 8) - Math.PI / 2; // Start slightly left of top center
  const targetEndAngle = (Math.PI / 8) - Math.PI / 2; // End slightly right of top center
  ctx.beginPath();
  ctx.arc(centerX, centerY, outerRadius, targetStartAngle, targetEndAngle);
  ctx.arc(centerX, centerY, innerRadius, targetEndAngle, targetStartAngle, true);
  ctx.closePath();
  ctx.fillStyle = "yellow";
  ctx.fill();

  // Draw outer circle
  ctx.beginPath();
  ctx.arc(centerX, centerY, outerRadius, 0, Math.PI * 2);
  ctx.strokeStyle = "#000";
  ctx.lineWidth = 4;
  ctx.stroke();

  // Draw inner circle
  ctx.beginPath();
  ctx.arc(centerX, centerY, innerRadius, 0, Math.PI * 2);
  ctx.stroke();

  // Draw static red line
  const x1 = centerX + innerRadius * Math.cos(angle);
  const y1 = centerY + innerRadius * Math.sin(angle);
  const x2 = centerX + outerRadius * Math.cos(angle);
  const y2 = centerY + outerRadius * Math.sin(angle);

  ctx.beginPath();
  ctx.moveTo(x1, y1);
  ctx.lineTo(x2, y2);
  ctx.strokeStyle = "red";
  ctx.lineWidth = 3;
  ctx.stroke();
};

// Animate the red line with increasing speed
const animate = (ctx) => {
  const step = () => {
    if (gameOver.value) return;

    angle -= speed; // Adjust angle counterclockwise
    ctx.clearRect(0, 0, canvasSize, canvasSize);
    drawStaticGame(ctx);

    animationFrame = requestAnimationFrame(step);
  };

  step();
};

// Handle spacebar press
const handleKeyPress = (event) => {
  if (event.code === "Space") {
    if (!gameStarted.value) {
      // Start the game
      gameStarted.value = true;
      const ctx = canvas.value.getContext("2d");
      animate(ctx);
    } else if (!gameOver.value) {
      if (isLineInTarget(angle)) {
        score.value += 1;
        speed += speedIncrement; // Increase speed
      } else {
        gameOver.value = true; // End game if miss
        cancelAnimationFrame(animationFrame);
      }
    }
  }
};

// Lifecycle hooks
onMounted(() => {
  const ctx = canvas.value.getContext("2d");
  drawStaticGame(ctx); // Draw the static game elements initially
  window.addEventListener("keydown", handleKeyPress);
});

onUnmounted(() => {
  cancelAnimationFrame(animationFrame);
  window.removeEventListener("keydown", handleKeyPress);
});
</script>

<style>
canvas {
  display: block;
  margin: auto;
  border: 1px solid #ccc;
  background-color: #f9f9f9;
}
.instructions {
  text-align: center;
  color: blue;
  font-size: 16px;
  margin-top: 10px;
}
.start-screen {
  text-align: center;
  color: red;
  font-weight: bold;
  margin-top: 10px;
}
.game-over {
  text-align: center;
  color: red;
  font-weight: bold;
  margin-top: 10px;
}
</style>

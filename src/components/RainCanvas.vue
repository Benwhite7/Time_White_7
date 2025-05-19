<template>
  <canvas id="rain-canvas" ref="canvas"></canvas>
</template>
<script setup>
import "../assets/styles/rain-canvas.css";
import { onMounted, onUnmounted, watch, ref } from "vue";

const props = defineProps({
  setTime: String,
});

const canvas = ref(null);
let ctx = null;
let drops = [];
let flakes = [];
let niebla = [];
let nubes = [];
let rayos = [];
let lightningTimer = 0;
let lightningOpacity = 0;
let isLightning = false;
let animationFrameId;

function initRain(type) {
  const el = canvas.value;
  ctx = el.getContext("2d");
  el.width = window.innerWidth;
  el.height = window.innerHeight;
  nubes = [];
  drops = [];
  flakes = [];
  niebla = [];
  rayos = [];
  if (type.includes("rain") ) {
    let gotasNumber = 100;
    let gotasLength = 20;
    let gotasSpeed = 4;
    if(type.includes('big')) {
      console.log('Se hace el cambio');
      gotasNumber = 150;
      gotasLength = 80;
      gotasSpeed = 20;
    }
    for (let i = 0; i < gotasNumber; i++) {
      drops.push({
        x: Math.random() * el.width,
        y: Math.random() * el.height,
        length: Math.random() * gotasLength + 10,
        speed: Math.random() * gotasSpeed + 2,
      });
    }
  } else if (type.includes("snow")) {
    for (let i = 0; i < 100; i++) {
      flakes.push({
        x: Math.random() * el.width,
        y: Math.random() * el.height,
        radius: Math.random() * 3 + 2,
        speedY: Math.random() * 4 + 1,
        speedX: Math.random() * 3 + 1,
      });
    }
  } else if (type.includes("cloudy")) {
    const cloudsNumber = type.includes("bg-cloudy-sun") ? 10 : 20;
    for (let i = 0; i < cloudsNumber; i++) {  
      nubes.push({
        x: Math.random() * el.width,
        y: Math.random() * el.height,
        speed: Math.random() * 1,
      });
    }
  } else if (type.includes("fog")) {
    for (let i = 0; i < 25; i++) {
      niebla.push({
        x: Math.random() * el.width,
        y: el.height - 100,
        size: Math.random() * 10,
        dx: Math.random() * 0.5 - 0.25,
      });
    }
  } else if (type.includes("storm")) {
    for (let i = 0; i < 6; i++) {
      rayos.push({
        x: Math.random() * el.width,
      });
    }
  }
}

function stormBackground() {
  const el = canvas.value;
  ctx.fillStyle = isLightning ? "rgba(255, 255, 255, 0.8)" : "rgba(20,20,30,1)";
  ctx.fillRect(0, 0, el.width, el.height);
}

function drawRain() {
  const el = canvas.value;
  ctx.clearRect(0, 0, el.width, el.height);
  ctx.strokeStyle = "rgba(174,194,224,0.5)";
  ctx.lineWidth = 2;
  ctx.beginPath();

  for (let i = 0; i < drops.length; i++) {
    const d = drops[i];
    ctx.moveTo(d.x, d.y);
    ctx.lineTo(d.x, d.y + d.length);
  }

  ctx.stroke();
  moveRain();
}

function drawSnow() {
  const el = canvas.value;
  ctx.clearRect(0, 0, el.width, el.height);
  ctx.fillStyle = "rgba(255, 255, 255, 0.8)";
  ctx.beginPath();

  for (const flake of flakes) {
    ctx.moveTo(flake.x, flake.y);
    ctx.arc(flake.x, flake.y, flake.radius, 0, Math.PI * 2, true);
  }

  ctx.fill();
  moveSnow();
}
// function drawSun() {
//   const el = canvas.value;
//   const radius = 50;
//   const gradient = ctx.createRadialGradient(60,60,10,60,60,radius);
//   gradient.addColorStop(0, '#fff700');
//   gradient.addColorStop(1, '#fca300');

//   ctx.beginPath();
//   ctx.arc(60, 60, radius, 0, Math.PI * 2);
//   ctx.fillStyle = gradient;
//   ctx.fill();

//   const rayLength = 20;
//   const rayCount = 12;
//   const time = Date.now() / 1000;

//   for (let i = 0; i < rayCount; i++) {
//     const angle = (i * (Math.PI * 2)) / rayCount + time * 0.5;

//     const x1 = 60 + Math.cos(angle) * (radius + 10);
//     const y1 = 60 + Math.sin(angle) * (radius + 10);
//     const x2 = 60 + Math.cos(angle) * (radius + 10 + rayLength);
//     const y2 = 60 + Math.sin(angle) * (radius + 10 + rayLength);

//     ctx.beginPath();
//     ctx.moveTo(x1, y1);
//     ctx.lineTo(x2, y2);
//     ctx.strokeStyle = 'rgba(255, 190, 0, 0.8)';
//     ctx.lineWidth = 3;
//     ctx.stroke();
//   }
// }
function drawHeatWaves() {
  const el = canvas.value;
  ctx.clearRect(0, 0, el.width, el.height);
  ctx.strokeStyle = "rgba(255, 162, 51, 0.5)";
  ctx.lineWidth = 2;

  const waveCount = 20;
  const time = Date.now() / 1000;

  for (let i = 0; i < waveCount; i++) {
    const x = (el.width / waveCount) * i;
    ctx.beginPath();
    const yRandom = Math.random() * 2;
    let randomYInteger = Math.floor(yRandom) + 1;

    for (let y = 0; y < el.height; y += 10) {
      const offset = Math.sin(y / 20 + time + i) * 3;
      ctx.lineTo(x + offset, y);
    }
    ctx.stroke();
  }
}

function drawClouds(bg) {
  const el = canvas.value;
  ctx.clearRect(0, 0, el.width, el.height);
  ctx.fillStyle = bg.includes('night') ? "rgb(99,99,99)" : "rgb(230,230,230)";
  for (let i = 0; i < nubes.length; i++) {
    const XCloud = nubes[i].x;
    const YCloud = nubes[i].y;
    ctx.beginPath();
    ctx.ellipse(XCloud, YCloud, 65, 50, 0, 0, Math.PI * 2, true);
    ctx.fill();
    ctx.beginPath();
    ctx.ellipse(XCloud + 50, YCloud + 10, 55, 40, 0, 0, Math.PI * 2, true);
    ctx.fill();
    ctx.beginPath();
    ctx.ellipse(XCloud - 50, YCloud + 10, 55, 40, 0, 0, Math.PI * 2, true);
    ctx.fill();
  }
  moveCloud();
}

function drawFogs() {
  const el = canvas.value;
  ctx.clearRect(0, 0, el.width, el.height);

  for (const p of niebla) {
    ctx.beginPath();
    ctx.fillStyle = `rgba(200,200,200,0.7)`;
    ctx.ellipse(p.x, p.y, 250, 150 + 10 * p.size, 0, 0, Math.PI * 2, true);
    ctx.fill();
  }

  moveFog();
}

function drawLightning() {
  const el = canvas.value;
  if (isLightning) {
    lightningOpacity = 1;
  }
  ctx.strokeStyle = `rgba(250,250,250,${lightningOpacity})`;
  ctx.lineWidth = 3;
  ctx.beginPath();

  for (const r of rayos) {
    let coorX = r.x;
    ctx.moveTo(coorX, 0);
    for (let y = 0; y < el.height; y += 30) {
      coorX += (Math.random() - 0.5) * 30;
      ctx.lineTo(coorX, y);
    }
  }

  ctx.stroke();
  if (lightningOpacity <= 0) return (lightningOpacity = 0);
  lightningOpacity -= 0.01;
}

function moveRain() {
  const el = canvas.value;
  for (let i = 0; i < drops.length; i++) {
    const d = drops[i];
    d.y += d.speed;
    if (d.y > el.height) {
      d.y = 0;
      d.x = Math.random() * el.width;
    }
  }
}

function moveSnow() {
  const el = canvas.value;
  for (const flake of flakes) {
    flake.y += flake.speedY;
    flake.x += flake.speedX;
    if (flake.y > el.height) {
      flake.y = -flake.radius;
      flake.x = Math.random() * el.width;
    }
    if (flake.x > el.width || flake.x < 0) {
      flake.speedX *= -1;
    }
  }
}

function moveCloud() {
  const el = canvas.value;
  for (let i = 0; i < nubes.length; i++) {
    const n = nubes[i];
    n.x += n.speed;
    if (n.x - 110 > el.width) {
      n.x = 0;
      n.y = Math.random() * el.height;
    }
  }
}

function moveFog() {
  const el = canvas.value;
  for (const p of niebla) {
    p.x += p.dx;
    if (p.x < 0 || p.x > el.width) p.dx *= -1;
  }
}

function updateLightning() {
  lightningTimer--;
  if (lightningTimer <= 0) {
    isLightning = Math.random() < 0.1;
    console.log(isLightning);
    lightningTimer = isLightning ? 5 : Math.random() * 200 + 100;
  }
}

function loop(type) {
  if (type.includes("rain")) {
    drawRain();
  } else if (type.includes("snow")) {
    drawSnow();
  } else if (type.includes("-bg-sunny-hot")) {
    drawHeatWaves();
  } else if (type.includes("cloudy") || type.includes("cloudy-sun")) {
    drawClouds(type);
  } else if (type.includes("fog")) {
    drawFogs();
  } else if (type.includes("storm")) {
    stormBackground();
    drawLightning();
    updateLightning();
  }
  animationFrameId = requestAnimationFrame(() => loop(type));
}

onUnmounted(() => {
  cancelAnimationFrame(animationFrameId);
});

watch(
  () => props.setTime,
  (newValue, oldValue) => {
    if (!canvas) return;
    if (animationFrameId) cancelAnimationFrame(animationFrameId);
    const el = canvas.value;
    ctx = el.getContext("2d");
    ctx.clearRect(0, 0, el.width, el.height);
    initRain(newValue);
    loop(newValue);
  }
);
</script>
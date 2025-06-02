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
let CTX = null;
let el;
let timerResize;
let arrayStation = [];
let timeToLightning = 200;
let lightningTimer = 0;
let lightningOpacity = 0;
let isLightning = false;
let animationFrameId;

function initStations(type) {
  el.width = window.innerWidth;
  el.height = window.innerHeight;
  arrayStation = [];
  if (type.includes("rain") ) {
    let gotasNumber = 100;
    let gotasLength = 20;
    let gotasSpeed = 4;
    if(type.includes('big')) {
      gotasNumber = 150;
      gotasLength = 80;
      gotasSpeed = 20;
    }
    for (let i = 0; i < gotasNumber; i++) {
      arrayStation.push({
        x: Math.random() * el.width,
        y: Math.random() * el.height,
        length: Math.random() * gotasLength + 10,
        speed: Math.random() * gotasSpeed + 2,
      });
    }
  } else if (type.includes("snow")) {
    for (let i = 0; i < 100; i++) {
      arrayStation.push({
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
      arrayStation.push({
        x: Math.random() * el.width,
        y: Math.random() * el.height,
        speed: Math.random() * 1,
      });
    }
  } else if (type.includes("fog")) {
    for (let i = 0; i < 25; i++) {
      arrayStation.push({
        x: Math.random() * el.width,
        y: el.height - 100,
        size: Math.random() * 10,
        dx: Math.random() * 0.5 - 0.25,
      });
    }
  } else if (type.includes("storm")) {
    for (let i = 0; i < 250; i++) {
      arrayStation.push({
        x: Math.random() * el.width,
        y: Math.random() * el.height,
        length: Math.random() * 80 + 10,
        speed: Math.random() * 20 + 2,
      });
    }
    for (let i = 0; i < 6; i++) {
      arrayStation.push({
        x: Math.random() * el.width,
      });
    }
  }
}

function stormBackground() {
  CTX.fillStyle = isLightning ? "rgba(255, 255, 255, 0.8)" : "rgb(3, 18, 46)";
  CTX.fillRect(0, 0, el.width, el.height);
}

function drawRain() {
  CTX.clearRect(0, 0, el.width, el.height);
  CTX.strokeStyle = "rgba(174,194,224,0.5)";
  CTX.lineWidth = 2;
  CTX.beginPath();
  for (let i = 0; i < arrayStation.length; i++) {
    const d = arrayStation[i];
    CTX.moveTo(d.x, d.y);
    CTX.lineTo(d.x, d.y + d.length);
  }
  CTX.stroke();
  moveRain();
}

function drawSnow() {
  CTX.clearRect(0, 0, el.width, el.height);
  CTX.fillStyle = "rgba(255, 255, 255, 0.8)";
  CTX.beginPath();
  for (const flake of flakes) {
    CTX.moveTo(flake.x, flake.y);
    CTX.arc(flake.x, flake.y, flake.radius, 0, Math.PI * 2, true);
  }
  CTX.fill();
  moveSnow();
}

function drawHeatWaves() {
  CTX.clearRect(0, 0, el.width, el.height);
  CTX.strokeStyle = "rgba(255, 162, 51, 0.5)";
  CTX.lineWidth = 2;

  const waveCount = 20;
  const time = Date.now() / 1000;

  for (let i = 0; i < waveCount; i++) {
    const x = (el.width / waveCount) * i;
    CTX.beginPath();
    for (let y = 0; y < el.height; y += 10) {
      const offset = Math.sin(y / 20 + time + i) * 3;
      CTX.lineTo(x + offset, y);
    }
    CTX.stroke();
  }
}

function drawClouds(bg) {
  CTX.clearRect(0, 0, el.width, el.height);
  CTX.fillStyle = bg.includes('night') ? "rgb(99,99,99)" : "rgb(230,230,230)";
  for (let i = 0; i < arrayStation.length; i++) {
    const XCloud = arrayStation[i].x;
    const YCloud = arrayStation[i].y;
    CTX.beginPath();
    CTX.ellipse(XCloud, YCloud, 65, 50, 0, 0, Math.PI * 2, true);
    CTX.fill();
    CTX.beginPath();
    CTX.ellipse(XCloud + 50, YCloud + 10, 55, 40, 0, 0, Math.PI * 2, true);
    CTX.fill();
    CTX.beginPath();
    CTX.ellipse(XCloud - 50, YCloud + 10, 55, 40, 0, 0, Math.PI * 2, true);
    CTX.fill();
  }
  moveCloud();
}

function drawFogs() {
  CTX.clearRect(0, 0, el.width, el.height);
  for (const p of arrayStation) {
    CTX.beginPath();
    CTX.fillStyle = `rgba(200,200,200,0.7)`;
    CTX.ellipse(p.x, p.y, 250, 150 + 10 * p.size, 0, 0, Math.PI * 2, true);
    CTX.fill();
  }
  moveFog();
}

function drawLightning() {
  if (isLightning) {
    lightningOpacity = 1;
  }
  CTX.strokeStyle = `rgba(250,250,250,${lightningOpacity})`;
  CTX.lineWidth = 3;
  CTX.beginPath();
  for (const r of arrayStation) {
    let coorX = r.x;
    CTX.moveTo(coorX, 0);
    for (let y = 0; y < el.height; y += 30) {
      coorX += (Math.random() - 0.5) * 30;
      CTX.lineTo(coorX, y);
    }
  }

  CTX.stroke();
  if (lightningOpacity <= 0) return (lightningOpacity = 0);
  lightningOpacity -= 0.01;
}

function moveRain() {
  for (let i = 0; i < arrayStation.length; i++) {
    const d = arrayStation[i];
    d.y += d.speed;
    if (d.y > el.height) {
      d.y = 0;
      d.x = Math.random() * el.width;
    }
  }
}

function moveSnow() {
  for (const flake of arrayStation) {
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
  for (let i = 0; i < arrayStation.length; i++) {
    const n = arrayStation[i];
    n.x += n.speed;
    if (n.x - 110 > el.width) {
      n.x = 0;
      n.y = Math.random() * el.height;
    }
  }
}

function moveFog() {
  for (const p of arrayStation) {
    p.x += p.dx;
    if (p.x < 0 || p.x > el.width) p.dx *= -1;
  }
}

function moveLightning() {
  for (const r of arrayStation) {
    r.x = Math.random() * el.width
  }
}

function updateLightning() {
  lightningTimer--;
  if (lightningTimer <= 0) {
    isLightning = Math.random() < 0.1;
    if(timeToLightning <= 0) {
      timeToLightning = isLightning ? 150 : 0;
    }
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
    if(timeToLightning > 0) {
      timeToLightning--;
      stormBackground();
      drawLightning();
      if(timeToLightning == 1) moveLightning();
    } else {
      drawRain();
    }
    updateLightning();
  }
  animationFrameId = requestAnimationFrame(() => loop(type));
}

onUnmounted(() => {
  cancelAnimationFrame(animationFrameId);
});

watch(
  () => props.setTime,
  (newValue) => {
    if (!canvas) return;
    if (animationFrameId) cancelAnimationFrame(animationFrameId);
    el = canvas.value;
    CTX = el.getContext("2d");
    CTX.clearRect(0, 0, el.width, el.height);
    initStations(newValue);
    loop(newValue);
    window.addEventListener('resize', () => {
      if(timerResize) clearTimeout(timerResize);
      timerResize = setTimeout(() => {
        initStations(newValue)
      }, 500);
    })
  }
);
</script>
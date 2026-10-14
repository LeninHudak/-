<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
<meta name="apple-mobile-web-app-capable" content="yes">
<title>Для тебя 💖</title>
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; -webkit-tap-highlight-color: transparent; }
  html, body {
    width: 100%; height: 100%; overflow: hidden;
    background: radial-gradient(circle at center, #2b0a1e 0%, #0d0209 100%);
    font-family: -apple-system, BlinkMacSystemFont, "SF Pro Display", sans-serif;
    color: #fff;
    user-select: none;
    -webkit-user-select: none;
  }

  #canvas {
    position: fixed; inset: 0; z-index: 1;
    pointer-events: none;
  }

  /* Большое сердце */
  #bigHeart {
    position: fixed;
    top: 50%; left: 50%;
    transform: translate(-50%, -50%);
    width: 70vw; max-width: 320px; aspect-ratio: 1/1;
    z-index: 2;
    cursor: pointer;
    animation: beat 1.1s ease-in-out infinite;
    filter: drop-shadow(0 0 40px rgba(255, 40, 100, 0.8))
            drop-shadow(0 0 80px rgba(255, 40, 100, 0.5));
    transition: opacity .5s, transform .5s;
  }
  #bigHeart svg { width: 100%; height: 100%; display: block; }

  @keyframes beat {
    0%, 100% { transform: translate(-50%, -50%) scale(1); }
    15%      { transform: translate(-50%, -50%) scale(1.12); }
    30%      { transform: translate(-50%, -50%) scale(0.98); }
    45%      { transform: translate(-50%, -50%) scale(1.08); }
    60%      { transform: translate(-50%, -50%) scale(1); }
  }

  #bigHeart.hide {
    opacity: 0;
    transform: translate(-50%, -50%) scale(0.2);
    pointer-events: none;
  }

  /* Подсказка */
  #hint {
    position: fixed;
    bottom: calc(30px + env(safe-area-inset-bottom));
    left: 0; right: 0;
    text-align: center;
    font-size: 15px;
    letter-spacing: 1px;
    color: rgba(255,255,255,.75);
    z-index: 3;
    animation: pulseText 2s ease-in-out infinite;
    transition: opacity .4s;
  }
  @keyframes pulseText { 0%,100%{opacity:.5} 50%{opacity:1} }
  #hint.hide { opacity: 0; }

  /* Экран с бумажками */
  #notesScreen {
    position: fixed; inset: 0;
    z-index: 10;
    display: none;
    flex-direction: column;
    align-items: center;
    padding: calc(20px + env(safe-area-inset-top)) 16px calc(20px + env(safe-area-inset-bottom));
    overflow-y: auto;
    -webkit-overflow-scrolling: touch;
    background: radial-gradient(circle at center, #2b0a1e 0%, #0d0209 100%);
  }
  #notesScreen.show { display: flex; }

  #notesScreen h1 {
    font-size: 26px;
    font-weight: 700;
    margin-bottom: 18px;
    background: linear-gradient(90deg, #ff4d8d, #ffb3c8, #ff4d8d);
    -webkit-background-clip: text;
    background-clip: text;
    color: transparent;
    animation: hueShift 4s linear infinite;
  }
  @keyframes hueShift { 0%{filter:hue-rotate(0)} 100%{filter:hue-rotate(360deg)} }

  .notes-grid {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 14px;
    width: 100%;
    max-width: 480px;
    padding-bottom: 20px;
  }

  .note {
    background: #fffdf5;
    border-radius: 12px;
    padding: 14px 12px;
    color: #333;
    box-shadow: 0 8px 24px rgba(0,0,0,.4), 0 1px 0 rgba(0,0,0,.1);
    transform: translateY(40px) rotate(var(--rot, 0deg)) scale(.6);
    opacity: 0;
    animation: popIn .7s cubic-bezier(.2,1.4,.5,1) forwards;
    animation-delay: var(--delay, 0s);
    position: relative;
  }
  .note:nth-child(1) { --rot: -3deg; --delay: .1s; }
  .note:nth-child(2) { --rot:  2deg; --delay: .25s; }
  .note:nth-child(3) { --rot:  3deg; --delay: .4s; }
  .note:nth-child(4) { --rot: -2deg; --delay: .55s; }

  @keyframes popIn {
    0%   { transform: translateY(40px) rotate(var(--rot)) scale(.6); opacity: 0; }
    60%  { transform: translateY(-6px) rotate(var(--rot)) scale(1.05); opacity: 1; }
    100% { transform: translateY(0) rotate(var(--rot)) scale(1); opacity: 1; }
  }

  .note .photo {
    width: 100%;
    aspect-ratio: 1/1;
    border-radius: 8px;
    background: #ffe0ea center/cover no-repeat;
    margin-bottom: 10px;
    background-image: linear-gradient(135deg, #ffb3c8, #ff4d8d);
  }

  .note .title {
    font-family: "Comic Sans MS", "Marker Felt", cursive;
    font-size: 15px;
    font-weight: bold;
    color: #e6005c;
    margin-bottom: 4px;
    line-height: 1.1;
  }
  .note .text {
    font-size: 13px;
    line-height: 1.3;
    color: #444;
  }

  #backBtn {
    margin-top: 10px;
    padding: 10px 26px;
    background: rgba(255,255,255,.1);
    border: 1px solid rgba(255,255,255,.25);
    color: #fff;
    border-radius: 30px;
    font-size: 14px;
    cursor: pointer;
    backdrop-filter: blur(10px);
    -webkit-backdrop-filter: blur(10px);
  }
  #backBtn:active { transform: scale(.95); }

  /* Вспышка */
  #flash {
    position: fixed; inset: 0;
    background: #fff;
    z-index: 5;
    opacity: 0;
    pointer-events: none;
  }
  #flash.go { animation: flashAnim .6s ease-out; }
  @keyframes flashAnim {
    0% { opacity: 0; }
    20% { opacity: .9; }
    100% { opacity: 0; }
  }
</style>
</head>
<body>

<canvas id="canvas"></canvas>

<div id="bigHeart">
  <svg viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg">
    <defs>
      <radialGradient id="hg" cx="50%" cy="35%" r="65%">
        <stop offset="0%" stop-color="#ff9ec2"/>
        <stop offset="50%" stop-color="#ff2d6f"/>
        <stop offset="100%" stop-color="#b10048"/>
      </radialGradient>
      <filter id="glow">
        <feGaussianBlur stdDeviation="1.2" result="b"/>
        <feMerge><feMergeNode in="b"/><feMergeNode in="SourceGraphic"/></feMerge>
      </filter>
    </defs>
    <path filter="url(#glow)" fill="url(#hg)"
      d="M50 88
         C 20 68, 6 52, 6 36
         C 6 22, 18 12, 30 12
         C 39 12, 46 17, 50 25
         C 54 17, 61 12, 70 12
         C 82 12, 94 22, 94 36
         C 94 52, 80 68, 50 88 Z"/>
  </svg>
</div>

<div id="hint">нажми на сердце 💗</div>

<div id="flash"></div>

<div id="notesScreen">
  <h1>Love is…</h1>
  <div class="notes-grid" id="notesGrid"><!-- JS заполнит --></div>
  <button id="backBtn">← обратно к сердцу</button>
</div>

<script>
/* ============ НАСТРОЙКИ ============ */
const NOTES = [
  { photo: "photo1.webp", title: "Love is…", text: "…твоя улыбка, которая делает мой день лучше." },
  { photo: "photo2.webp", title: "Love is…", text: "…обнимашки, когда холодно и грустно." },
  { photo: "photo3.webp", title: "Love is…", text: "…мы вместе. И пусть весь мир подождёт." },
  { photo: "photo4.webp", title: "Love is…", text: "…ты. Просто ты. 💕" },
]
;
/* ==================================== */

const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');
let W, H, DPR;
function resize() {
  DPR = Math.min(window.devicePixelRatio || 1, 2);
  W = canvas.clientWidth = window.innerWidth;
  H = canvas.clientHeight = window.innerHeight;
  canvas.width = W * DPR;
  canvas.height = H * DPR;
  ctx.setTransform(DPR, 0, 0, DPR, 0, 0);
}
window.addEventListener('resize', resize);
resize();

/* Рисуем маленькие сердечки */
function drawHeart(ctx, x, y, size, color, alpha) {
  ctx.save();
  ctx.globalAlpha = alpha;
  ctx.fillStyle = color;
  ctx.translate(x, y);
  ctx.scale(size, size);
  ctx.beginPath();
  ctx.moveTo(0, 0.3);
  ctx.bezierCurveTo(0, 0.05, -0.4, 0.05, -0.4, 0.35);
  ctx.bezierCurveTo(-0.4, 0.65, 0, 0.9, 0, 1.1);
  ctx.bezierCurveTo(0, 0.9, 0.4, 0.65, 0.4, 0.35);
  ctx.bezierCurveTo(0.4, 0.05, 0, 0.05, 0, 0.3);
  ctx.fill();
  ctx.restore();
}

const heartColors = ['#ff4d8d', '#ff85b3', '#ffb3c8', '#ff2d6f', '#ffd0dd'];
const flyingHearts = [];
const MAX_HEARTS = 40;

function spawnHeart() {
  const size = 6 + Math.random() * 14;
  flyingHearts.push({
    x: Math.random() * W,
    y: H + size * 2,
    vx: (Math.random() - 0.5) * 0.6,
    vy: -(0.5 + Math.random() * 1.2),
    size: size,
    rot: Math.random() * Math.PI * 2,
    vr: (Math.random() - 0.5) * 0.04,
    alpha: 0.4 + Math.random() * 0.6,
    color: heartColors[(Math.random() * heartColors.length) | 0],
    wobble: Math.random() * Math.PI * 2,
    wobbleSpd: 0.02 + Math.random() * 0.03,
    life: 1,
  });
}

/* Всплеск сердец при тапе */
function burstHearts(cx, cy, count = 40) {
  for (let i = 0; i < count; i++) {
    const angle = Math.random() * Math.PI * 2;
    const speed = 2 + Math.random() * 5;
    const size = 8 + Math.random() * 16;
    flyingHearts.push({
      x: cx, y: cy,
      vx: Math.cos(angle) * speed,
      vy: Math.sin(angle) * speed,
      size, rot: Math.random() * Math.PI * 2,
      vr: (Math.random() - 0.5) * 0.15,
      alpha: 1,
      color: heartColors[(Math.random() * heartColors.length) | 0],
      wobble: 0, wobbleSpd: 0,
      life: 1, gravity: 0.05, fade: 0.012,
    });
  }
}

function tick() {
  ctx.clearRect(0, 0, W, H);

  /* Автоспавн */
  if (flyingHearts.length < MAX_HEARTS && Math.random() < 0.15) spawnHeart();

  for (let i = flyingHearts.length - 1; i >= 0; i--) {
    const h = flyingHearts[i];
    h.wobble += h.wobbleSpd;
    h.x += h.vx + Math.sin(h.wobble) * 0.4;
    h.y += h.vy;
    h.rot += h.vr;

    if (h.gravity) h.vy += h.gravity;
    if (h.fade) h.life -= h.fade;

    if (h.y < -40 || h.y > H + 60 || h.life <= 0) {
      flyingHearts.splice(i, 1);
      continue;
    }

    drawHeart(ctx, h.x, h.y, h.size, h.color, h.alpha * h.life);
  }

  requestAnimationFrame(tick);
}
tick();

/* ====== UI логика ====== */
const bigHeart = document.getElementById('bigHeart');
const hint = document.getElementById('hint');
const notesScreen = document.getElementById('notesScreen');
const notesGrid = document.getElementById('notesGrid');
const flash = document.getElementById('flash');
const backBtn = document.getElementById('backBtn');

/* Строим бумажки */
function buildNotes() {
  notesGrid.innerHTML = '';
  NOTES.forEach((n, i) => {
    const el = document.createElement('div');
    el.className = 'note';
    el.innerHTML = `
      <div class="photo" style="background-image:url('${n.photo}'), linear-gradient(135deg,#ffb3c8,#ff4d8d)"></div>
      <div class="title">${n.title}</div>
      <div class="text">${n.text}</div>
    `;
    notesGrid.appendChild(el);
  });
}
buildNotes();

let opened = false;

function openNotes() {
  if (opened) return;
  opened = true;

  const rect = bigHeart.getBoundingClientRect();
  burstHearts(rect.left + rect.width / 2, rect.top + rect.height / 2, 50);

  flash.classList.remove('go');
  void flash.offsetWidth;
  flash.classList.add('go');

  bigHeart.classList.add('hide');
  hint.classList.add('hide');

  setTimeout(() => {
    notesScreen.classList.add('show');
  }, 250);
}

function closeNotes() {
  notesScreen.classList.remove('show');
  opened = false;
  bigHeart.classList.remove('hide');
  hint.classList.remove('hide');
}

bigHeart.addEventListener('click', openNotes);
bigHeart.addEventListener('touchstart', (e) => {
  e.preventDefault();
  openNotes();
}, { passive: false });

backBtn.addEventListener('click', closeNotes);

/* Вибрация на iPhone (если поддерживается) */
if (navigator.vibrate) {
  bigHeart.addEventListener('touchstart', () => navigator.vibrate([10, 30, 10]));
}
</script>
</body>
</html>

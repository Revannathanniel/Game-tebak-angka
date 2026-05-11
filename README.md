<!DOCTYPE html><html lang="id">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Neon Cyber Runner</title>
  <style>
    * {
      margin: 0;
      padding: 0;
      box-sizing: border-box;
      font-family: Arial, sans-serif;
    }body {
  overflow: hidden;
  background: black;
}

canvas {
  display: block;
  background: radial-gradient(circle at center, #111 0%, #000 100%);
}

.ui {
  position: absolute;
  top: 20px;
  left: 20px;
  color: white;
  z-index: 10;
  text-shadow: 0 0 10px cyan;
}

.title {
  font-size: 28px;
  color: cyan;
  margin-bottom: 10px;
}

.controls {
  position: absolute;
  bottom: 20px;
  width: 100%;
  text-align: center;
  color: #aaa;
  font-size: 14px;
}

.gameover {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  color: white;
  text-align: center;
  display: none;
  z-index: 20;
}

.gameover h1 {
  font-size: 60px;
  color: red;
  text-shadow: 0 0 20px red;
}

.gameover button {
  margin-top: 20px;
  padding: 12px 25px;
  border: none;
  background: cyan;
  color: black;
  font-size: 18px;
  cursor: pointer;
  border-radius: 10px;
  box-shadow: 0 0 20px cyan;
}

  </style>
</head>
<body><div class="ui">
  <div class="title">⚡ Neon Cyber Runner</div>
  <div>Score: <span id="score">0</span></div>
  <div>Highscore: <span id="highscore">0</span></div>
</div><div class="controls">
  Gerak: ⬅ ➡ | Lompat: SPACE
</div><div class="gameover" id="gameover">
  <h1>GAME OVER</h1>
  <p id="finalscore"></p>
  <button onclick="restartGame()">Main Lagi</button>
</div><canvas id="game"></canvas>

<script>
const canvas = document.getElementById('game');
const ctx = canvas.getContext('2d');

canvas.width = window.innerWidth;
canvas.height = window.innerHeight;

let score = 0;
let highscore = localStorage.getItem('neon_highscore') || 0;
document.getElementById('highscore').innerText = highscore;

const gravity = 0.7;

const player = {
  x: 100,
  y: 300,
  w: 50,
  h: 50,
  color: 'cyan',
  vx: 0,
  vy: 0,
  speed: 7,
  jumping: false
};

let obstacles = [];
let particles = [];
let gameRunning = true;

function createObstacle() {
  const size = Math.random() * 60 + 30;

  obstacles.push({
    x: canvas.width + size,
    y: canvas.height - size - 60,
    w: size,
    h: size,
    speed: 8 + Math.random() * 4,
    color: `hsl(${Math.random() * 360},100%,50%)`
  });
}

setInterval(() => {
  if (gameRunning) createObstacle();
}, 1200);

function drawPlayer() {
  ctx.save();
  ctx.shadowBlur = 30;
  ctx.shadowColor = player.color;
  ctx.fillStyle = player.color;
  ctx.fillRect(player.x, player.y, player.w, player.h);
  ctx.restore();
}

function drawObstacles() {
  obstacles.forEach((o, i) => {
    o.x -= o.speed;

    ctx.save();
    ctx.shadowBlur = 20;
    ctx.shadowColor = o.color;
    ctx.fillStyle = o.color;
    ctx.fillRect(o.x, o.y, o.w, o.h);
    ctx.restore();

    if (
      player.x < o.x + o.w &&
      player.x + player.w > o.x &&
      player.y < o.y + o.h &&
      player.y + player.h > o.y
    ) {
      endGame();
    }

    if (o.x + o.w < 0) {
      obstacles.splice(i, 1);
      score++;
      document.getElementById('score').innerText = score;

      if (score > highscore) {
        highscore = score;
        localStorage.setItem('neon_highscore', highscore);
        document.getElementById('highscore').innerText = highscore;
      }
    }
  });
}

function drawGround() {
  ctx.fillStyle = '#111';
  ctx.fillRect(0, canvas.height - 60, canvas.width, 60);

  for (let i = 0; i < canvas.width; i += 40) {
    ctx.strokeStyle = 'rgba(0,255,255,0.3)';
    ctx.beginPath();
    ctx.moveTo(i, canvas.height - 60);
    ctx.lineTo(i + 20, canvas.height);
    ctx.stroke();
  }
}

function createParticle(x, y) {
  particles.push({
    x,
    y,
    size: Math.random() * 4 + 1,
    vx: (Math.random() - 0.5) * 4,
    vy: Math.random() * -3,
    alpha: 1
  });
}

function drawParticles() {
  particles.forEach((p, i) => {
    p.x += p.vx;
    p.y += p.vy;
    p.alpha -= 0.02;

    ctx.save();
    ctx.globalAlpha = p.alpha;
    ctx.fillStyle = 'cyan';
    ctx.beginPath();
    ctx.arc(p.x, p.y, p.size, 0, Math.PI * 2);
    ctx.fill();
    ctx.restore();

    if (p.alpha <= 0) {
      particles.splice(i, 1);
    }
  });
}

function updatePlayer() {
  player.x += player.vx;
  player.y += player.vy;

  if (player.y + player.h < canvas.height - 60) {
    player.vy += gravity;
  } else {
    player.vy = 0;
    player.y = canvas.height - player.h - 60;
    player.jumping = false;
  }

  if (player.x < 0) player.x = 0;
  if (player.x + player.w > canvas.width)
    player.x = canvas.width - player.w;

  createParticle(player.x + player.w / 2, player.y + player.h);
}

function drawBackground() {
  ctx.fillStyle = 'rgba(0,0,0,0.25)';
  ctx.fillRect(0, 0, canvas.width, canvas.height);

  for (let i = 0; i < 80; i++) {
    ctx.fillStyle = 'rgba(255,255,255,0.08)';
    ctx.fillRect(
      Math.random() * canvas.width,
      Math.random() * canvas.height,
      2,
      2
    );
  }
}

function animate() {
  if (!gameRunning) return;

  requestAnimationFrame(animate);

  drawBackground();
  drawGround();
  updatePlayer();
  drawPlayer();
  drawObstacles();
  drawParticles();
}

animate();

window.addEventListener('keydown', (e) => {
  if (e.code === 'ArrowRight') {
    player.vx = player.speed;
  }

  if (e.code === 'ArrowLeft') {
    player.vx = -player.speed;
  }

  if (e.code === 'Space' && !player.jumping) {
    player.vy = -15;
    player.jumping = true;
  }
});

window.addEventListener('keyup', (e) => {
  if (e.code === 'ArrowRight' || e.code === 'ArrowLeft') {
    player.vx = 0;
  }
});

function endGame() {
  gameRunning = false;
  document.getElementById('gameover').style.display = 'block';
  document.getElementById('finalscore').innerText = `Score Kamu: ${score}`;
}

function restartGame() {
  obstacles = [];
  particles = [];
  score = 0;
  document.getElementById('score').innerText = score;
  player.x = 100;
  player.y = 300;
  player.vx = 0;
  player.vy = 0;
  gameRunning = true;

  document.getElementById('gameover').style.display = 'none';

  animate();
}

window.addEventListener('resize', () => {
  canvas.width = window.innerWidth;
  canvas.height = window.innerHeight;
});
</script></body>
</html>

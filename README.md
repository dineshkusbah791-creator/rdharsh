
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Mini 2D Platformer</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <style>
    html, body {
      margin: 0;
      padding: 0;
      background: #0b1220;
      color: #e5e7eb;
      font-family: Arial, sans-serif;
      height: 100%;
    }
    .wrap {
      display: grid;
      place-items: center;
      min-height: 100vh;
      gap: 10px;
    }
    .hud {
      display: flex;
      gap: 16px;
      font-size: 14px;
      opacity: 0.9;
    }
    canvas {
      background: linear-gradient(#0ea5e9, #0b1220);
      border: 2px solid #1f2933;
      border-radius: 12px;
      image-rendering: pixelated;
    }
    .tip {
      font-size: 12px;
      opacity: 0.7;
    }
  </style>
</head>
<body>
  <div class="wrap">
    <div class="hud">
      <div>Score: <b id="score">0</b></div>
      <div>Coins: <b id="coins">0</b></div>
    </div>
    <canvas id="game" width="640" height="360"></canvas>
    <div class="tip">A/D move, W or Space jump</div>
  </div>

  <script>
    const canvas = document.getElementById("game");
    const ctx = canvas.getContext("2d");
    const scoreEl = document.getElementById("score");
    const coinsEl = document.getElementById("coins");

    const GRAVITY = 0.6;
    const FRICTION = 0.8;

    const keys = {};
    window.addEventListener("keydown", e => keys[e.key.toLowerCase()] = true);
    window.addEventListener("keyup", e => keys[e.key.toLowerCase()] = false);

    const player = {
      x: 40, y: 0, w: 24, h: 24,
      vx: 0, vy: 0,
      onGround: false
    };

    const platforms = [
      { x: 0,   y: 330, w: 640, h: 30 }, // ground
      { x: 120, y: 260, w: 120, h: 16 },
      { x: 300, y: 210, w: 120, h: 16 },
      { x: 480, y: 160, w: 120, h: 16 },
    ];

    let coins = [
      { x: 150, y: 230, r: 6, taken: false },
      { x: 330, y: 180, r: 6, taken: false },
      { x: 510, y: 130, r: 6, taken: false },
    ];

    let score = 0;
    let collected = 0;

    function rectsCollide(a, b) {
      return (
        a.x < b.x + b.w &&
        a.x + a.w > b.x &&
        a.y < b.y + b.h &&
        a.y + a.h > b.y
      );
    }

    function update() {
      // input
      if (keys["a"] || keys["arrowleft"]) player.vx -= 0.6;
      if (keys["d"] || keys["arrowright"]) player.vx += 0.6;
      if ((keys["w"] || keys[" "]) && player.onGround) {
        player.vy = -10;
        player.onGround = false;
      }

      // physics
      player.vy += GRAVITY;
      player.vx *= FRICTION;

      player.x += player.vx;
      player.y += player.vy;

      // bounds
      if (player.x < 0) player.x = 0;
      if (player.x + player.w > canvas.width) player.x = canvas.width - player.w;

      // platform collision
      player.onGround = false;
      for (const p of platforms) {
        if (rectsCollide(player, p)) {
          if (player.vy > 0 && player.y + player.h - player.vy <= p.y) {
            player.y = p.y - player.h;
            player.vy = 0;
            player.onGround = true;
          }
        }
      }

      // coin pickup
      for (const c of coins) {
        if (!c.taken) {
          const dx = (player.x + player.w/2) - c.x;
          const dy = (player.y + player.h/2) - c.y;
          const dist = Math.hypot(dx, dy);
          if (dist < c.r + 10) {
            c.taken = true;
            score += 10;
            collected += 1;
            scoreEl.textContent = score;
            coinsEl.textContent = collected;
          }
        }
      }

      // fall reset
      if (player.y > canvas.height) {
        player.x = 40; player.y = 0; player.vx = 0; player.vy = 0;
      }
    }

    function draw() {
      ctx.clearRect(0, 0, canvas.width, canvas.height);

      // platforms
      ctx.fillStyle = "#0f172a";
      platforms.forEach(p => ctx.fillRect(p.x, p.y, p.w, p.h));

      // coins
      coins.forEach(c => {
        if (!c.taken) {
          ctx.beginPath();
          ctx.arc(c.x, c.y, c.r, 0, Math.PI * 2);
          ctx.fillStyle = "#fbbf24";
          ctx.fill();
        }
      });

      // player
      ctx.fillStyle = "#22c55e";
      ctx.fillRect(player.x, player.y, player.w, player.h);
    }

    function loop() {
      update();
      draw();
      requestAnimationFrame(loop);
    }

    loop();
  </script>
</body>
</html>

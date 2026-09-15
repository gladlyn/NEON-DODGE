<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">

  <title>Neon Dodge</title>

  <style>
    * {
      box-sizing: border-box;
    }

    body {
      margin: 0;
      background: #050510;
      color: white;
      font-family: Arial, sans-serif;
      display: flex;
      justify-content: center;
      align-items: center;
      min-height: 100vh;
    }

    .game-container {
      text-align: center;
    }

    h1 {
      color: #00ffff;
      text-shadow: 0 0 15px #00ffff;
      margin-bottom: 10px;
    }

    canvas {
      border: 2px solid #00ffff;
      box-shadow: 0 0 25px #00ffff;
      background: #080820;
      max-width: 100%;
    }

    .info {
      margin-top: 10px;
      font-size: 18px;
    }

    button {
      margin-top: 12px;
      padding: 10px 20px;
      border: 2px solid #00ffff;
      background: transparent;
      color: #00ffff;
      cursor: pointer;
      font-size: 16px;
      border-radius: 5px;
    }

    button:hover {
      background: #00ffff;
      color: black;
      box-shadow: 0 0 15px #00ffff;
    }
  </style>
</head>

<body>

  <div class="game-container">

    <h1>⚡ NEON DODGE ⚡</h1>

    <canvas id="game" width="900" height="560"></canvas>

    <div class="info">
      Score: <span id="score">0</span>
      &nbsp; | &nbsp;
      Best: <span id="best">0</span>
    </div>

    <button onclick="restartGame()">Restart</button>

    <p>Move with ← → or A / D</p>

  </div>


  <script>

    // =========================
    // CANVAS SETUP
    // =========================

    const canvas = document.getElementById("game");
    const ctx = canvas.getContext("2d");


    // =========================
    // SCORE
    // =========================

    let score = 0;

    let best = Number(
      localStorage.getItem("neonDodgeBest") || 0
    );

    document.getElementById("best").textContent = best;


    // =========================
    // PLAYER
    // =========================

    const player = {
      x: 450,
      y: 500,
      w: 46,
      h: 24,
      speed: 8
    };


    // =========================
    // ENEMIES
    // =========================

    let blocks = [];


    // =========================
    // GAME VARIABLES
    // =========================

    let running = true;

    let startTime = performance.now();

    let lastSpawn = 0;


    // =========================
    // KEYBOARD INPUT
    // =========================

    const keys = {};

    document.addEventListener("keydown", function(event) {
      keys[event.key.toLowerCase()] = true;
    });

    document.addEventListener("keyup", function(event) {
      keys[event.key.toLowerCase()] = false;
    });


    // =========================
    // CREATE ENEMY
    // =========================

    function spawn() {

      const width = 25 + Math.random() * 55;

      const height = 20 + Math.random() * 35;

      const x = Math.random() * (canvas.width - width);

      const speed =
        3.2 +
        Math.random() * 2.8 +
        score / 900;

      blocks.push({
        x: x,
        y: -height,
        w: width,
        h: height,
        speed: speed
      });
    }


    // =========================
    // COLLISION DETECTION
    // =========================

    function hit(a, b) {

      return (
        a.x < b.x + b.w &&
        a.x + a.w > b.x &&
        a.y < b.y + b.h &&
        a.y + a.h > b.y
      );

    }


    // =========================
    // GAME OVER
    // =========================

    function gameOver() {

      running = false;

      if (score > best) {

        best = score;

        localStorage.setItem(
          "neonDodgeBest",
          best
        );

        document.getElementById("best").textContent = best;
      }

    }


    // =========================
    // RESTART GAME
    // =========================

    function restartGame() {

      player.x = 450;

      blocks = [];

      score = 0;

      running = true;

      startTime = performance.now();

      lastSpawn = 0;

      requestAnimationFrame(loop);

    }


    // =========================
    // UPDATE GAME
    // =========================

    function update(time) {

      // Move player left
      if (keys["arrowleft"] || keys["a"]) {
        player.x -= player.speed;
      }


      // Move player right
      if (keys["arrowright"] || keys["d"]) {
        player.x += player.speed;
      }


      // Keep player inside canvas
      if (player.x < 0) {
        player.x = 0;
      }

      if (player.x + player.w > canvas.width) {
        player.x = canvas.width - player.w;
      }


      // Spawn enemies
      if (time - lastSpawn > 500) {

        spawn();

        lastSpawn = time;

      }


      // Move enemies
      for (let i = blocks.length - 1; i >= 0; i--) {

        const block = blocks[i];

        block.y += block.speed;


        // Collision
        if (hit(player, block)) {

          gameOver();

          return;

        }


        // Remove blocks that leave screen
        if (block.y > canvas.height) {

          blocks.splice(i, 1);

        }

      }


      // Calculate score
      score = Math.floor(
        (time - startTime) / 100
      );

      document.getElementById("score").textContent = score;

    }


    // =========================
    // DRAW BACKGROUND
    // =========================

    function drawBackground() {

      const gradient = ctx.createLinearGradient(
        0,
        0,
        0,
        canvas.height
      );

      gradient.addColorStop(0, "#05051a");

      gradient.addColorStop(1, "#120018");

      ctx.fillStyle = gradient;

      ctx.fillRect(
        0,
        0,
        canvas.width,
        canvas.height
      );


      // Grid
      ctx.strokeStyle = "rgba(0,255,255,0.08)";

      ctx.lineWidth = 1;


      for (let x = 0; x < canvas.width; x += 40) {

        ctx.beginPath();

        ctx.moveTo(x, 0);

        ctx.lineTo(x, canvas.height);

        ctx.stroke();

      }


      for (let y = 0; y < canvas.height; y += 40) {

        ctx.beginPath();

        ctx.moveTo(0, y);

        ctx.lineTo(canvas.width, y);

        ctx.stroke();

      }

    }


    // =========================
    // DRAW PLAYER
    // =========================

    function drawPlayer() {

      ctx.save();

      ctx.shadowColor = "#00ffff";

      ctx.shadowBlur = 25;

      ctx.fillStyle = "#00ffff";

      ctx.fillRect(
        player.x,
        player.y,
        player.w,
        player.h
      );

      ctx.restore();

    }


    // =========================
    // DRAW ENEMIES
    // =========================

    function drawBlocks() {

      for (const block of blocks) {

        ctx.save();

        ctx.shadowColor = "#ff0088";

        ctx.shadowBlur = 20;

        ctx.fillStyle = "#ff0088";

        ctx.fillRect(
          block.x,
          block.y,
          block.w,
          block.h
        );

        ctx.restore();

      }

    }


    // =========================
    // DRAW GAME OVER
    // =========================

    function drawGameOver() {

      ctx.fillStyle = "rgba(0,0,0,0.65)";

      ctx.fillRect(
        0,
        0,
        canvas.width,
        canvas.height
      );


      ctx.textAlign = "center";


      ctx.shadowColor = "#ff0088";

      ctx.shadowBlur = 20;

      ctx.fillStyle = "#ff0088";

      ctx.font = "bold 60px Arial";

      ctx.fillText(
        "GAME OVER",
        canvas.width / 2,
        canvas.height / 2 - 30
      );


      ctx.shadowBlur = 0;

      ctx.fillStyle = "white";

      ctx.font = "24px Arial";

      ctx.fillText(
        "Score: " + score,
        canvas.width / 2,
        canvas.height / 2 + 20
      );


      ctx.font = "18px Arial";

      ctx.fillText(
        "Press Restart to play again",
        canvas.width / 2,
        canvas.height / 2 + 60
      );

    }


    // =========================
    // MAIN GAME LOOP
    // =========================

    function loop(time) {

      if (!running) {

        drawBackground();

        drawBlocks();

        drawPlayer();

        drawGameOver();

        return;

      }


      update(time);


      drawBackground();

      drawBlocks();

      drawPlayer();


      requestAnimationFrame(loop);

    }


    // =========================
    // START GAME
    // =========================

    requestAnimationFrame(loop);

  </script>

</body>
</html>

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Game Ular</title>
  <style>
    body {
      background-color: #f0f0f0;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      height: 100vh;
      margin: 0;
      font-family: Arial, sans-serif;
      overflow: hidden;
    }
    canvas {
      background-color: #222;
      border: 5px solid #555;
    }
    #scoreboard {
      margin: 20px;
      font-size: 24px;
      font-weight: bold;
      color: #333;
    }
    #gameover {
      position: absolute;
      top: 50%;
      transform: translateY(-50%);
      background: rgba(0,0,0,0.8);
      color: white;
      padding: 30px;
      border-radius: 10px;
      font-size: 32px;
      display: none;
      text-align: center;
    }
    button {
      margin-top: 20px;
      padding: 10px 20px;
      font-size: 20px;
      background: #28a745;
      border: none;
      border-radius: 5px;
      color: white;
      cursor: pointer;
    }
    button:hover {
      background: #218838;
    }
  </style>
</head>
<body>
  <div id="scoreboard">
    Score: 0 | Best Score: 0
  </div>
  <canvas id="gameCanvas" width="600" height="600"></canvas>

  <div id="gameover">
    <div id="gameover-text">Game Over</div>
    <button onclick="restartGame()">Restart</button>
  </div>

  <script>
    const canvas = document.getElementById("gameCanvas");
    const ctx = canvas.getContext("2d");
    const gameoverDiv = document.getElementById("gameover");
    const scoreboard = document.getElementById("scoreboard");

    const grid = 20;
    let count = 0;
    let score = 0;
    let bestScore = localStorage.getItem("bestScore") || 0;
    let speed = 6;
    let foodEaten = 0;
    let particles = [];

    const snake = {
      x: 160,
      y: 160,
      dx: grid,
      dy: 0,
      cells: [],
      maxCells: 4
    };

    const food = {
      x: 320,
      y: 320
    };

    function getRandomInt(min, max) {
      return Math.floor(Math.random() * (max - min)) + min;
    }

    function updateScoreboard() {
      scoreboard.innerText = `Score: ${score} | Best Score: ${bestScore}`;
    }

    function restartGame() {
      snake.x = 160;
      snake.y = 160;
      snake.cells = [];
      snake.maxCells = 4;
      snake.dx = grid;
      snake.dy = 0;
      score = 0;
      foodEaten = 0;
      speed = 6;
      updateScoreboard();
      food.x = getRandomInt(0, canvas.width / grid) * grid;
      food.y = getRandomInt(0, canvas.height / grid) * grid;
      gameoverDiv.style.display = "none";
      requestAnimationFrame(gameLoop);
    }

    function spawnParticles(x, y) {
      for (let i = 0; i < 10; i++) {
        particles.push({
          x: x,
          y: y,
          dx: (Math.random() - 0.5) * 4,
          dy: (Math.random() - 0.5) * 4,
          life: 30
        });
      }
    }

    function drawParticles() {
      particles.forEach((p, index) => {
        ctx.fillStyle = "yellow";
        ctx.fillRect(p.x, p.y, 3, 3);
        p.x += p.dx;
        p.y += p.dy;
        p.life--;
        if (p.life <= 0) {
          particles.splice(index, 1);
        }
      });
    }

    function gameLoop() {
      if (gameoverDiv.style.display === "block") return;

      requestAnimationFrame(gameLoop);

      if (++count < speed) return;
      count = 0;

      ctx.clearRect(0, 0, canvas.width, canvas.height);

      snake.x += snake.dx;
      snake.y += snake.dy;

      // wrap around
      if (snake.x < 0) snake.x = canvas.width - grid;
      else if (snake.x >= canvas.width) snake.x = 0;
      if (snake.y < 0) snake.y = canvas.height - grid;
      else if (snake.y >= canvas.height) snake.y = 0;

      snake.cells.unshift({x: snake.x, y: snake.y});
      if (snake.cells.length > snake.maxCells) snake.cells.pop();

      // draw food
      ctx.fillStyle = "red";
      ctx.fillRect(food.x, food.y, grid-1, grid-1);

      drawParticles();

      // draw snake
      snake.cells.forEach((cell, index) => {
        ctx.fillStyle = index === 0 ? "limegreen" : "lime";
        ctx.fillRect(cell.x, cell.y, grid-1, grid-1);

        // eat food
        if (cell.x === food.x && cell.y === food.y) {
          snake.maxCells++;
          score += 10;
          foodEaten++;

          if (score > bestScore) {
            bestScore = score;
            localStorage.setItem("bestScore", bestScore);
          }

          // naikkan kecepatan setiap 3 buah
          if (foodEaten % 3 === 0 && speed > 2) {
            speed--;
          }

          updateScoreboard();

          food.x = getRandomInt(0, canvas.width / grid) * grid;
          food.y = getRandomInt(0, canvas.height / grid) * grid;

          spawnParticles(cell.x, cell.y);
        }

        // tabrakan diri
        for (let i = index + 1; i < snake.cells.length; i++) {
          if (cell.x === snake.cells[i].x && cell.y === snake.cells[i].y) {
            gameoverDiv.style.display = "block";
          }
        }
      });
    }

    let lastDirection = { dx: grid, dy: 0 };

    document.addEventListener("keydown", function(e) {
      if ((e.key === "a" || e.key === "ArrowLeft") && lastDirection.dx === 0) {
        snake.dx = -grid;
        snake.dy = 0;
      } else if ((e.key === "w" || e.key === "ArrowUp") && lastDirection.dy === 0) {
        snake.dy = -grid;
        snake.dx = 0;
      } else if ((e.key === "d" || e.key === "ArrowRight") && lastDirection.dx === 0) {
        snake.dx = grid;
        snake.dy = 0;
      } else if ((e.key === "s" || e.key === "ArrowDown") && lastDirection.dy === 0) {
        snake.dy = grid;
        snake.dx = 0;
      }

      // simpan arah terakhir agar tidak bisa balik arah 180°
      lastDirection = { dx: snake.dx, dy: snake.dy };
    });

    updateScoreboard();
    requestAnimationFrame(gameLoop);
  </script>
</body>
</html>

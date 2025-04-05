<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Cartoon Car Game</title>
  <style>
    * {
      margin: 0;
      padding: 0;
      box-sizing: border-box;
    }
    body {
      background: #87ceeb;
      overflow: hidden;
      font-family: Arial, sans-serif;
    }
    #gameArea {
      position: relative;
      width: 400px;
      height: 600px;
      margin: 20px auto;
      background: black;
      border: 5px solid #000;
      overflow: hidden;
      touch-action: none;
    }
    .lane-line {
      position: absolute;
      width: 10px;
      height: 40px;
      background: white;
      left: 195px;
      animation: dash 1.5s linear infinite;
    }

    @keyframes dash {
      0% { top: -40px; }
      100% { top: 600px; }
    }

    #car {
      position: absolute;
      width: 60px;
      height: 120px;
      background: red; /* Full red color for car */
      bottom: 20px;
      left: 170px;
      border-radius: 10px;
    }

    .obstacle {
      position: absolute;
      width: 60px;
      height: 120px;
      background: blue; /* Blue color for obstacles */
      top: -130px;
      border-radius: 10px;
    }

    #scoreboard {
      position: absolute;
      top: 10px;
      right: 20px;
      font-size: 18px;
      font-weight: bold;
      color: #fff;
      z-index: 10;
    }
    #congrats {
      display: none;
      text-align: center;
      font-size: 24px;
      color: green;
      font-weight: bold;
    }
    #gameOverScreen {
      display: none;
      position: absolute;
      width: 100%;
      height: 100%;
      background-color: rgba(0, 0, 0, 0.8);
      color: white;
      text-align: center;
      padding-top: 200px;
      font-size: 24px;
      z-index: 999;
    }
    #restartBtn {
      margin-top: 20px;
      font-size: 20px;
      padding: 10px 20px;
      background-color: #28a745;
      border: none;
      color: white;
      cursor: pointer;
      border-radius: 10px;
    }
  </style>
</head>
<body>
  <div id="gameArea">
    <div id="scoreboard">Score: 0 | High Score: 0</div>
    <div id="car"></div>
    <div id="gameOverScreen">
      🛑 <strong>Khel Samapta!</strong><br />
      <button id="restartBtn">Feri Khelxu</button>
    </div>
  </div>
  <div id="congrats">🎉 Congratulations! New High Score! 🎉</div>

  <script>
    const gameArea = document.getElementById("gameArea");
    const car = document.getElementById("car");
    const scoreboard = document.getElementById("scoreboard");
    const congrats = document.getElementById("congrats");
    const gameOverScreen = document.getElementById("gameOverScreen");
    const restartBtn = document.getElementById("restartBtn");

    let carX = 170;
    let score = 0;
    let highScore = 0;
    let gameRunning = true;

    const carSpeed = 30;
    const carLeftBound = 10;
    const carRightBound = 330;

    // Movement control
    document.addEventListener("keydown", (e) => {
      if (!gameRunning) return;
      if (e.key === "ArrowLeft" && carX > carLeftBound) carX -= carSpeed;
      if (e.key === "ArrowRight" && carX < carRightBound) carX += carSpeed;
      car.style.left = carX + "px";
    });

    let touchStartX = null;
    gameArea.addEventListener("touchstart", (e) => {
      touchStartX = e.touches[0].clientX;
    });

    gameArea.addEventListener("touchend", (e) => {
      if (!touchStartX) return;
      const touchEndX = e.changedTouches[0].clientX;
      const diffX = touchEndX - touchStartX;
      if (Math.abs(diffX) > 30) {
        if (diffX > 0 && carX < carRightBound) carX += carSpeed;
        else if (diffX < 0 && carX > carLeftBound) carX -= carSpeed;
        car.style.left = carX + "px";
      }
      touchStartX = null;
    });

    function createObstacle() {
      if (!gameRunning) return;
      const obstacle = document.createElement("div");
      obstacle.classList.add("obstacle");
      gameArea.appendChild(obstacle);

      // Obstacles come in alternating positions (left, middle, right)
      const positions = ["70px", "170px", "270px"];
      const randomPos = positions[Math.floor(Math.random() * positions.length)];
      obstacle.style.left = randomPos;

      let obstacleY = -130;
      const fallInterval = setInterval(() => {
        if (!gameRunning) return clearInterval(fallInterval);
        obstacleY += 6;
        obstacle.style.top = obstacleY + "px";

        const carRect = car.getBoundingClientRect();
        const obsRect = obstacle.getBoundingClientRect();
        const gameRect = gameArea.getBoundingClientRect();

        if (
          carRect.left < obsRect.right &&
          carRect.right > obsRect.left &&
          carRect.top < obsRect.bottom &&
          carRect.bottom > obsRect.top
        ) {
          endGame();
          clearInterval(fallInterval);
        }

        if (obstacleY > 600) {
          obstacle.remove();
          score++;
          if (score > highScore) {
            highScore = score;
            congrats.style.display = "block";
          } else {
            congrats.style.display = "none";
          }
          updateScore();
          clearInterval(fallInterval);
        }
      }, 20);
    }

    function createLines() {
      const line = document.createElement("div");
      line.classList.add("lane-line");
      gameArea.appendChild(line);
      setTimeout(() => line.remove(), 1500);
    }

    function updateScore() {
      scoreboard.innerText = `Score: ${score} | High Score: ${highScore}`;
    }

    function endGame() {
      gameRunning = false;
      gameOverScreen.style.display = 'block';
    }

    restartBtn.addEventListener("click", () => {
      gameOverScreen.style.display = 'none';
      gameRunning = true;
      score = 0;
      updateScore();
      congrats.style.display = 'none';
      gameArea.querySelectorAll(".obstacle").forEach(o => o.remove());
    });

    setInterval(() => { if (gameRunning) createObstacle(); }, 1200);
    setInterval(() => { if (gameRunning) createLines(); }, 200);
  </script>
</body>
</html>

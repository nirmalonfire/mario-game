# mario-game
<!DOCTYPE html>
<html>
  <head>
    <title>Mario Run</title>
    <link rel="stylesheet" type="text/css" href="mario.css" />
  </head>
  <body>
    <button id="start-btn">Start Game</button>

    <div id="game-container">
      <audio
        id="background-music"
        src="./assets/background-music.mp3"
        loop
      ></audio>
      <audio id="jump-sound" src="./assets/jump.mp3"></audio>
      <div id="score"></div>
      <img id="mario" src="./assets/mario.gif" />
      <img id="pipe" class="obstacle" src="./assets/pipe.png" />
      <img id="mushroom" class="obstacle" src="./assets/mushroom.png" />
    </div>
    <script src="mario.js"></script>
  </body>
</html>

# CSS
body {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  background: #6b88fe;
  margin: 0;
  padding: 0;
}

#start-btn {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  padding: 15px 30px;
  font-size: 1.5rem;
  color: #fff;
  background: #ff5252;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  outline: none;
  transition: all 0.3s ease;
}

#start-btn:hover {
  background: #ff8a80;
}

#game-container {
  position: relative;
  height: 500px;
  width: 900px;
  background: url("./assets/background.png");
  background-size: cover;
  background-position: bottom;
  background-position-y: -95px;
  /* border: 3px solid #ffffff; */
  border-bottom-width: 0px;
  display: none;
}

#score {
  position: fixed;
  top: 10px;
  right: 10px;
  font-size: 30px;
  color: #ffffff;
}

#mario {
  position: absolute;
  bottom: 32px;
  left: 50px;
  height: 100px;
  transition: all 0.2s;
}

.obstacle {
  position: absolute;
  bottom: 35px;
  width: 60px;
  transition: all 0.3s;
  display: none;
}

.flipped {
  transform: scaleX(-1);
  transition: all 0s;
}

#mushroom {
  width: 80px;
  bottom: 10px;
}

# JavaScript
var mario = document.getElementById("mario");
var pipe = document.getElementById("pipe");
var mushroom = document.getElementById("mushroom");
var score = document.getElementById("score");
var backgroundMusic = document.getElementById("background-music");
var jumpSound = document.getElementById("jump-sound");
var startBtn = document.getElementById("start-btn");

startBtn.addEventListener("click", function () {
  // Start the game here
  backgroundMusic.play();
  // Hide the start button
  startBtn.style.display = "none";
  document.getElementById("game-container").style.display = "block";
  startGame();
});

function startGame() {
  var marioJumping = false;
  var marioMovingRight = false;
  var marioMovingLeft = false;
  var obstacles = [pipe, mushroom];
  var gameScore = 0;
  var gameContainerWidth =
    document.getElementById("game-container").offsetWidth;
  var marioPosition = 50;

  function jump() {
    if (!marioJumping) {
      marioJumping = true;
      jumpSound.play(); // play jump sound

      var startPos = 32;
      var endPos = 150;
      var speed = 5;

      var jumpInterval = setInterval(function () {
        if (startPos < endPos) {
          startPos += speed;
          mario.style.bottom = startPos + "px";
        } else {
          clearInterval(jumpInterval);
          fall();
        }
      }, 20);
    }
  }

  function fall() {
    var startPos = 150;
    var endPos = 32;
    var speed = 8;

    var fallInterval = setInterval(function () {
      if (startPos > endPos) {
        startPos -= speed;
        mario.style.bottom = startPos + "px";
      } else {
        clearInterval(fallInterval);
        marioJumping = false;
      }
    }, 20);
  }

  function moveMario(direction) {
    var proposedPosition = marioPosition + (direction === "right" ? 20 : -20);
    var maxMarioPosition = gameContainerWidth - mario.offsetWidth; // mario.offsetWidth gives the width of the mario element
    if (proposedPosition >= 0 && proposedPosition <= maxMarioPosition) {
      marioPosition = proposedPosition;
      mario.style.left = marioPosition + "px";
      if (direction === "right") {
        mario.classList.remove("flipped");
      } else {
        mario.classList.add("flipped");
      }
    }
  }

  function checkCollision(obstaclePos) {
    return obstaclePos < marioPosition + 100 && obstaclePos > marioPosition;
  }

  function moveObstacle(obstacle) {
    var obstaclePos = gameContainerWidth;
    obstacle.style.left = obstaclePos + "px"; // set initial position

    var obstacleTimer = setInterval(function () {
      if (obstaclePos < -60) {
        obstacle.style.display = "none";
        obstaclePos = gameContainerWidth;
        setTimeout(() => {
          obstacle.style.display = "block";
        }, 100);
        gameScore++;
        score.innerText = gameScore;
      } else if (checkCollision(obstaclePos) && marioJumping) {
        obstaclePos -= 10; // increase speed
      } else if (checkCollision(obstaclePos) && !marioJumping) {
        clearInterval(obstacleTimer);
        score.innerText = "Game Over! Score: " + gameScore;
        obstacles.forEach(function (obstacle) {
          obstacle.style.animationPlayState = "paused";
        });
        if (confirm("Game Over!")) {
          location.reload();
        } else {
          location.reload();
        }
      } else {
        obstaclePos -= 10; // increase speed
      }

      obstacle.style.left = obstaclePos + "px";
    }, Math.random() * (200 - 50) + 50);
  }

  window.addEventListener("keydown", function (event) {
    switch (event.key) {
      case " ":
        jump();
        break;
      case "ArrowRight":
        marioMovingRight = true;
        break;
      case "ArrowLeft":
        marioMovingLeft = true;
        break;
    }
  });

  window.addEventListener("keyup", function (event) {
    switch (event.key) {
      case "ArrowRight":
        marioMovingRight = false;
        break;
      case "ArrowLeft":
        marioMovingLeft = false;
        break;
    }
  });

  setInterval(function () {
    if (marioMovingRight) {
      moveMario("right");
    } else if (marioMovingLeft) {
      moveMario("left");
    }
  }, 100);

  obstacles.forEach(function (obstacle, index) {
    setTimeout(function () {
      obstacle.style.display = "block";
      moveObstacle(obstacle);
    }, index * 2000); // 2000 ms (2 seconds) delay before each obstacle starts moving
  });
}

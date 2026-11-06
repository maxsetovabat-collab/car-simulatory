<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Simple 2D Racing Game</title>
<style>
    * {
        margin: 0;
        padding: 0;
        box-sizing: border-box;
    }

    body {
        background-color: #1a1e2b;
        display: flex;
        justify-content: center;
        align-items: center;
        min-height: 100vh;
        font-family: Arial, Helvetica, sans-serif;
        color: #ffffff;
        flex-direction: column;
    }

    #gameContainer {
        position: relative;
        display: flex;
        flex-direction: column;
        align-items: center;
    }

    #scoreBoard {
        font-size: 24px;
        font-weight: bold;
        margin-bottom: 12px;
        background-color: #2b3147;
        padding: 10px 30px;
        border-radius: 12px;
        letter-spacing: 2px;
        box-shadow: 0 4px 0 #0f111a;
        color: #f5e56b;
    }

    canvas {
        background-color: #2d313f;
        border-radius: 16px;
        box-shadow: 0 8px 0 #0f111a;
        display: block;
    }

    #instructions {
        margin-top: 16px;
        color: #9da7c0;
        font-size: 15px;
    }

    #gameOverScreen {
        position: absolute;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        background-color: rgba(10, 12, 20, 0.85);
        display: none;
        justify-content: center;
        align-items: center;
        flex-direction: column;
        border-radius: 16px;
        z-index: 10;
    }

    #gameOverScreen.active {
        display: flex;
    }

    #gameOverText {
        font-size: 48px;
        font-weight: bold;
        color: #ff6b6b;
        letter-spacing: 3px;
        margin-bottom: 10px;
        text-shadow: 0 4px 0 #4d1a1a;
    }

    #finalScoreText {
        font-size: 26px;
        color: #f5e56b;
        margin-bottom: 30px;
        letter-spacing: 1px;
    }

    #restartButton {
        background-color: #4f9eeb;
        color: #ffffff;
        border: none;
        padding: 14px 40px;
        font-size: 20px;
        font-weight: bold;
        border-radius: 50px;
        cursor: pointer;
        box-shadow: 0 6px 0 #2a5f9e;
        letter-spacing: 1px;
        transition: transform 0.05s ease;
    }

    #restartButton:hover {
        background-color: #6fb2ff;
    }

    #restartButton:active {
        transform: translateY(4px);
        box-shadow: 0 2px 0 #2a5f9e;
    }
</style>
</head>
<body>

<div id="gameContainer">
    <div id="scoreBoard">SCORE: <span id="scoreValue">0</span></div>
    <canvas id="gameCanvas" width="600" height="400"></canvas>
    <div id="instructions">Use LEFT / RIGHT arrow keys to move</div>

    <div id="gameOverScreen">
        <div id="gameOverText">GAME OVER</div>
        <div id="finalScoreText">Final Score: <span id="finalScoreValue">0</span></div>
        <button id="restartButton">RESTART</button>
    </div>
</div>

<script>
// ============================================================
// 1. SETUP AND REFERENCES
// ============================================================
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
const scoreValueSpan = document.getElementById('scoreValue');
const finalScoreValueSpan = document.getElementById('finalScoreValue');
const gameOverScreen = document.getElementById('gameOverScreen');
const restartButton = document.getElementById('restartButton');

// Canvas dimensions
const CANVAS_WIDTH = 600;
const CANVAS_HEIGHT = 400;

// Road boundaries (where the car can drive)
const ROAD_LEFT = 80;
const ROAD_RIGHT = 520;

// ============================================================
// 2. GAME STATE VARIABLES
// ============================================================
let gameRunning = true;
let score = 0;
let frameCounter = 0;
let spawnTimer = 0;

// Player car object
const playerCar = {
    x: CANVAS_WIDTH / 2,
    y: 330,
    width: 38,
    height: 56,
    speed: 6
};

// Obstacles array
let obstacles = [];// Keyboard state
let moveLeft = false;
let moveRight = false;

// Spawn interval (in frames)
const SPAWN_INTERVAL = 50;

// ============================================================
// 3. HELPER FUNCTIONS
// ============================================================

// Check if two rectangles overlap (AABB collision detection)
function rectanglesOverlap(rectA, rectB) {
    return (
        rectA.x < rectB.x + rectB.width &&
        rectA.x + rectA.width > rectB.x &&
        rectA.y < rectB.y + rectB.height &&
        rectA.y + rectA.height > rectB.y
    );
}

// Update the score display in the HTML
function updateScoreDisplay() {
    scoreValueSpan.textContent = Math.floor(score);
}

// Reset the entire game to starting state
function resetGame() {
    gameRunning = true;
    score = 0;
    frameCounter = 0;
    spawnTimer = 0;
    obstacles = [];
    playerCar.x = CANVAS_WIDTH / 2;
    moveLeft = false;
    moveRight = false;
    updateScoreDisplay();
    gameOverScreen.classList.remove('active');
}

// Show the game over screen
function showGameOver() {
    gameRunning = false;
    finalScoreValueSpan.textContent = Math.floor(score);
    gameOverScreen.classList.add('active');
}

// Spawn a new obstacle car at the top of the road
function spawnObstacle() {
    const obstacleWidth = 36;
    const obstacleHeight = 54;

    // Random horizontal position within road boundaries
    const minX = ROAD_LEFT + 10;
    const maxX = ROAD_RIGHT - obstacleWidth - 10;
    const randomX = minX + Math.random() * (maxX - minX);

    obstacles.push({
        x: randomX,
        y: -obstacleHeight,
        width: obstacleWidth,
        height: obstacleHeight,
        speed: 3.5
    });
}

// ============================================================
// 4. UPDATE LOGIC (runs every frame)
// ============================================================
function update() {
    if (!gameRunning) return;

    // --- Player movement ---
    if (moveLeft) {
        playerCar.x -= playerCar.speed;
    }
    if (moveRight) {
        playerCar.x += playerCar.speed;
    }

    // Keep player inside the road
    const playerHalfWidth = playerCar.width / 2;
    if (playerCar.x - playerHalfWidth < ROAD_LEFT) {
        playerCar.x = ROAD_LEFT + playerHalfWidth;
    }
    if (playerCar.x + playerHalfWidth > ROAD_RIGHT) {
        playerCar.x = ROAD_RIGHT - playerHalfWidth;
    }

    // --- Increase score over time ---
    frameCounter++;
    if (frameCounter % 6 === 0) {
        score += 1;
        updateScoreDisplay();
    }

    // --- Spawn obstacles ---
    spawnTimer++;
    if (spawnTimer >= SPAWN_INTERVAL) {
        spawnObstacle();
        spawnTimer = 0;
    }

    // --- Update obstacles and check collisions ---
    for (let i = obstacles.length - 1; i >= 0; i--) {
        const obstacle = obstacles[i];
        obstacle.y += obstacle.speed;

        // Build bounding boxes for collision
        const playerBox = {
            x: playerCar.x - playerCar.width / 2,
            y: playerCar.y - playerCar.height / 2,
            width: playerCar.width,
            height: playerCar.height
        };

        const obstacleBox = {
            x: obstacle.x,
            y: obstacle.y,
            width: obstacle.width,
            height: obstacle.height
        };

        // Check collision
        if (rectanglesOverlap(playerBox, obstacleBox)) {
            showGameOver();
            return;
        }

        // Remove obstacles that have moved off screen
        if (obstacle.y > CANVAS_HEIGHT + 60) {
            obstacles.splice(i, 1);
        }
    }
}

// ============================================================
// 5. DRAWING (renders everything on the canvas)
// ============================================================
function draw() {
    // Clear canvas
    ctx.clearRect(0, 0, CANVAS_WIDTH, CANVAS_HEIGHT);

    // --- Draw grass (sides) ---
    ctx.fillStyle = '#2c4a2c';
    ctx.fillRect(0, 0, ROAD_LEFT, CANVAS_HEIGHT);
    ctx.fillRect(ROAD_RIGHT, 0, CANVAS_WIDTH - ROAD_RIGHT, CANVAS_HEIGHT);// --- Draw road base ---
    ctx.fillStyle = '#2d313f';
    ctx.fillRect(ROAD_LEFT, 0, ROAD_RIGHT - ROAD_LEFT, CANVAS_HEIGHT);

    // --- Draw road edge lines ---
    ctx.strokeStyle = '#d1d8f0';
    ctx.lineWidth = 4;
    ctx.setLineDash([20, 20]);
    ctx.beginPath();
    ctx.moveTo(ROAD_LEFT, 0);
    ctx.lineTo(ROAD_LEFT, CANVAS_HEIGHT);
    ctx.stroke();
    ctx.beginPath();
    ctx.moveTo(ROAD_RIGHT, 0);
    ctx.lineTo(ROAD_RIGHT, CANVAS_HEIGHT);
    ctx.stroke();

    // --- Draw centre dashed line ---
    ctx.strokeStyle = '#f5e56b';
    ctx.lineWidth = 4;
    ctx.setLineDash([25, 30]);
    ctx.beginPath();
    ctx.moveTo(CANVAS_WIDTH / 2, 0);
    ctx.lineTo(CANVAS_WIDTH / 2, CANVAS_HEIGHT);
    ctx.stroke();

    // Reset line dash for other drawings
    ctx.setLineDash([]);

    // --- Draw obstacles (red cars) ---
    for (let i = 0; i < obstacles.length; i++) {
        const obs = obstacles[i];

        // Car body
        ctx.fillStyle = '#e34f4f';
        ctx.beginPath();
        ctx.roundRect(obs.x, obs.y, obs.width, obs.height, 8);
        ctx.fill();

        // Windows
        ctx.fillStyle = '#c0d4ff';
        ctx.beginPath();
        ctx.roundRect(obs.x + 5, obs.y + 8, obs.width - 10, 10, 4);
        ctx.fill();
        ctx.beginPath();
        ctx.roundRect(obs.x + 5, obs.y + obs.height - 20, obs.width - 10, 10, 4);
        ctx.fill();

        // Headlights
        ctx.fillStyle = '#f5e56b';
        ctx.beginPath();
        ctx.arc(obs.x + 8, obs.y + obs.height - 8, 4, 0, Math.PI * 2);
        ctx.fill();
        ctx.beginPath();
        ctx.arc(obs.x + obs.width - 8, obs.y + obs.height - 8, 4, 0, Math.PI * 2);
        ctx.fill();
    }

    // --- Draw player car (blue) ---
    const px = playerCar.x - playerCar.width / 2;
    const py = playerCar.y - playerCar.height / 2;

    // Car body
    ctx.fillStyle = '#4f9eeb';
    ctx.beginPath();
    ctx.roundRect(px, py, playerCar.width, playerCar.height, 8);
    ctx.fill();

    // Windscreen
    ctx.fillStyle = '#d4ebff';
    ctx.beginPath();
    ctx.roundRect(px + 6, py + 8, playerCar.width - 12, 12, 5);
    ctx.fill();
    ctx.beginPath();
    ctx.roundRect(px + 6, py + playerCar.height - 20, playerCar.width - 12, 12, 5);
    ctx.fill();

    // Headlights
    ctx.fillStyle = '#f5e56b';
    ctx.beginPath();
    ctx.arc(px + 8, py + playerCar.height - 6, 4, 0, Math.PI * 2);
    ctx.fill();
    ctx.beginPath();
    ctx.arc(px + playerCar.width - 8, py + playerCar.height - 6, 4, 0, Math.PI * 2);
    ctx.fill();
}

// Helper method to draw rounded rectangles
CanvasRenderingContext2D.prototype.roundRect = function(x, y, width, height, radius) {
    if (width < 2 * radius) radius = width / 2;
    if (height < 2 * radius) radius = height / 2;
    this.moveTo(x + radius, y);
    this.lineTo(x + width - radius, y);
    this.quadraticCurveTo(x + width, y, x + width, y + radius);
    this.lineTo(x + width, y + height - radius);
    this.quadraticCurveTo(x + width, y + height, x + width - radius, y + height);
    this.lineTo(x + radius, y + height);
    this.quadraticCurveTo(x, y + height, x, y + height - radius);
    this.lineTo(x, y + radius);
    this.quadraticCurveTo(x, y, x + radius, y);
    return this;
};

// ============================================================
// 6. GAME LOOP
// ============================================================
function gameLoop() {
    update();
    draw();
    requestAnimationFrame(gameLoop);
}

// ============================================================
// 7. EVENT LISTENERS
// ============================================================

// Keyboard down
window.addEventListener('keydown', function(event) {
    if (event.key === 'ArrowLeft') {
        moveLeft = true;
        event.preventDefault();
    } else if (event.key === 'ArrowRight') {
        moveRight = true;
        event.preventDefault();
    }
});// Keyboard up
window.addEventListener('keyup', function(event) {
    if (event.key === 'ArrowLeft') {
        moveLeft = false;
        event.preventDefault();
    } else if (event.key === 'ArrowRight') {
        moveRight = false;
        event.preventDefault();
    }
});

// Restart button click
restartButton.addEventListener('click', function() {
    resetGame();
});

// ============================================================
// 8. START THE GAME
// ============================================================
resetGame();
gameLoop();
</script>

</body>
</html>

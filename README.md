<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Hide and Seek Game</title>
    <style>
        body, html {
            margin: 0;
            padding: 0;
            height: 100%;
            overflow: hidden;
            font-family: Arial, sans-serif;
            background-color: #f0f8ff;
        }
        #startScreen {
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            height: 100%;
            background-color: #f3e5ab;
            text-align: center;
        }
        #startScreen h1 {
            font-size: 36px;
            margin-bottom: 20px;
            color: #444;
        }
        #startScreen input {
            padding: 10px;
            font-size: 18px;
            margin-bottom: 20px;
            border: 2px solid #888;
            border-radius: 5px;
        }
        #startScreen button {
            padding: 10px 20px;
            font-size: 18px;
            background-color: #4caf50;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
        }
        #startScreen button:hover {
            background-color: #45a049;
        }
        canvas {
            display: none;
            background: #b3e5fc;
        }
        #timer, #score {
            position: absolute;
            top: 10px;
            font-size: 20px;
            background: rgba(255, 255, 255, 0.8);
            padding: 10px;
            border-radius: 5px;
        }
        #timer {
            left: 10px;
        }
        #score {
            right: 10px;
        }
    </style>
</head>
<body>
    <div id="startScreen">
        <h1>ברוך הבא למשחק מחבואים</h1>
        <input type="text" id="playerName" placeholder="הכנס את שמך המלא" />
        <button id="startButton">התחל מחבואים</button>
    </div>

    <div id="timer">זמן נותר: <span id="time">60</span> שניות</div>
    <div id="score">חבויים נמצאו: <span id="found">0</span></div>
    <canvas id="gameCanvas"></canvas>

    <script>
        const startScreen = document.getElementById('startScreen');
        const startButton = document.getElementById('startButton');
        const playerNameInput = document.getElementById('playerName');
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const timerElement = document.getElementById('time');
        const scoreElement = document.getElementById('found');

        // Game variables
        const seeker = { x: 100, y: 100, size: 30, color: 'red', speed: 5 };
        const hiders = [];
        const obstacles = [];
        const hiderCount = 5;
        const obstacleCount = 10;
        let hidersFound = 0;
        let timeLeft = 60;
        let gameOver = false;

        // Generate random hiders
        function generateHiders() {
            for (let i = 0; i < hiderCount; i++) {
                hiders.push({
                    x: Math.random() * (canvas.width - 50) + 25,
                    y: Math.random() * (canvas.height - 50) + 25,
                    size: 20,
                    color: 'blue',
                    found: false
                });
            }
        }

        // Generate random obstacles
        function generateObstacles() {
            for (let i = 0; i < obstacleCount; i++) {
                obstacles.push({
                    x: Math.random() * (canvas.width - 100),
                    y: Math.random() * (canvas.height - 100),
                    width: 100,
                    height: 50,
                    color: 'gray'
                });
            }
        }

        // Initialize game
        function startGame() {
            const playerName = playerNameInput.value.trim();
            if (!playerName) {
                alert("נא להכניס שם מלא כדי להתחיל את המשחק!");
                return;
            }

            alert(`ברוך הבא, ${playerName}! המשחק מתחיל עכשיו!`);
            startScreen.style.display = 'none';
            canvas.style.display = 'block';
            canvas.width = window.innerWidth;
            canvas.height = window.innerHeight;

            generateHiders();
            generateObstacles();
            update();
            startTimer();
        }

        // Draw seeker
        function drawSeeker() {
            ctx.fillStyle = seeker.color;
            ctx.beginPath();
            ctx.arc(seeker.x, seeker.y, seeker.size, 0, Math.PI * 2);
            ctx.fill();
        }

        // Draw hiders
        function drawHiders() {
            hiders.forEach(hider => {
                if (!hider.found) {
                    ctx.fillStyle = hider.color;
                    ctx.beginPath();
                    ctx.arc(hider.x, hider.y, hider.size, 0, Math.PI * 2);
                    ctx.fill();
                }
            });
        }

        // Draw obstacles
        function drawObstacles() {
            obstacles.forEach(obstacle => {
                ctx.fillStyle = obstacle.color;
                ctx.fillRect(obstacle.x, obstacle.y, obstacle.width, obstacle.height);
            });
        }

        // Check collision with hiders
        function checkCollision() {
            hiders.forEach(hider => {
                const dist = Math.hypot(seeker.x - hider.x, seeker.y - hider.y);
                if (dist < seeker.size + hider.size && !hider.found) {
                    hider.found = true;
                    hidersFound++;
                    scoreElement.textContent = hidersFound;
                }
            });
        }

        // Check collision with obstacles
        function checkObstacleCollision(x, y) {
            return obstacles.some(obstacle =>
                x > obstacle.x &&
                x < obstacle.x + obstacle.width &&
                y > obstacle.y &&
                y < obstacle.y + obstacle.height
            );
        }

        // Update game state
        function update() {
            if (timeLeft <= 0 || hidersFound === hiderCount) {
                gameOver = true;
                alert(`המשחק נגמר! מצאת ${hidersFound} מתוך ${hiderCount} החבויים.`);
                return;
            }

            ctx.clearRect(0, 0, canvas.width, canvas.height);
            drawObstacles();
            drawSeeker();
            drawHiders();
            checkCollision();
        }

        // Move seeker
        window.addEventListener('keydown', (e) => {
            if (gameOver) return;

            let newX = seeker.x;
            let newY = seeker.y;

            if (e.key === 'ArrowUp') newY -= seeker.speed;
            if (e.key === 'ArrowDown') newY += seeker.speed;
            if (e.key === 'ArrowLeft') newX -= seeker.speed;
            if (e.key === 'ArrowRight') newX += seeker.speed;

            // Prevent movement through obstacles
            if (!checkObstacleCollision(newX, seeker.y)) seeker.x = newX;
            if (!checkObstacleCollision(seeker.x, newY)) seeker.y = newY;

            update();
        });

        // Timer
        function startTimer() {
            const timer = setInterval(() => {
                if (timeLeft > 0) {
                    timeLeft--;
                    timerElement.textContent = timeLeft;
                } else {
                    clearInterval(timer);
                    gameOver = true;
                    alert(`הזמן נגמר! מצאת ${hidersFound} מתוך ${hiderCount} החבויים.`);
                }
            }, 1000);
        }

        startButton.addEventListener('click', startGame);
    </script>
</body>
</html>

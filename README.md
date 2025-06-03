# breakout-game

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Breakout Game with Power-ups</title>
    <style>
        body {
            margin: 0;
            padding: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            background: linear-gradient(135deg, #1a2a6c, #b21f1f, #1a2a6c);
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            overflow: hidden;
        }

        .game-container {
            position: relative;
            width: 800px;
            text-align: center;
        }

        canvas {
            background: #111;
            border-radius: 8px;
            box-shadow: 0 0 20px rgba(0, 0, 0, 0.7);
        }

        .game-ui {
            display: flex;
            justify-content: space-between;
            margin-bottom: 15px;
            color: white;
            font-size: 20px;
            font-weight: bold;
            text-shadow: 0 0 5px rgba(0, 0, 0, 0.8);
        }

        .score, .lives {
            background: rgba(0, 0, 0, 0.5);
            padding: 10px 20px;
            border-radius: 30px;
            min-width: 120px;
        }

        .controls {
            margin-top: 15px;
            color: #ddd;
            font-size: 16px;
        }

        .message {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background: rgba(0, 0, 0, 0.85);
            color: white;
            padding: 30px 50px;
            border-radius: 10px;
            font-size: 36px;
            font-weight: bold;
            text-align: center;
            z-index: 10;
            display: none;
            box-shadow: 0 0 30px rgba(255, 255, 255, 0.2);
        }

        .message button {
            background: #4CAF50;
            color: white;
            border: none;
            padding: 12px 30px;
            font-size: 20px;
            border-radius: 30px;
            cursor: pointer;
            margin-top: 20px;
            transition: all 0.3s;
        }

        .message button:hover {
            background: #45a049;
            transform: scale(1.05);
        }

        h1 {
            color: white;
            text-shadow: 0 0 10px rgba(255, 255, 255, 0.5);
            margin-bottom: 10px;
            font-size: 42px;
        }

        .instructions {
            background: rgba(0, 0, 0, 0.7);
            padding: 15px;
            border-radius: 10px;
            margin-top: 15px;
            color: #ddd;
            font-size: 16px;
            line-height: 1.5;
        }
        
        .power-up-info {
            background: rgba(0, 0, 0, 0.7);
            padding: 10px;
            border-radius: 10px;
            margin-top: 10px;
            color: #ddd;
            font-size: 14px;
            text-align: left;
        }
        
        .power-up-info h3 {
            margin: 5px 0 10px 0;
            text-align: center;
            color: #FFEB3B;
        }
        
        .power-up-info ul {
            padding-left: 20px;
            margin: 0;
        }
        
        .power-up-info li {
            margin-bottom: 5px;
        }
        
        .fireball { color: #FF5252; }
        .paddle { color: #4CAF50; }
        .multiball { color: #2196F3; }
        .life { color: #9C27B0; }
    </style>
</head>
<body>
    <div class="game-container">
        <h1>BREAKOUT</h1>
        <div class="game-ui">
            <div class="score">Score: <span id="score">0</span></div>
            <div class="lives">Lives: <span id="lives">3</span></div>
        </div>
        <canvas id="gameCanvas" width="800" height="600"></canvas>
        <div class="controls">
            Move mouse to control paddle | Click to launch ball
        </div>
        <div class="instructions">
            Destroy all bricks to win! Don't let the ball fall below the paddle.
        </div>
        <div class="power-up-info">
            <h3>Power-ups</h3>
            <ul>
                <li><span class="fireball">Fireball</span>: Ball passes through bricks (temporary)</li>
                <li><span class="paddle">Long Paddle</span>: Makes your paddle longer (temporary)</li>
                <li><span class="multiball">Multiball</span>: Doubles the number of balls (instant)</li>
                <li><span class="life">Extra Life</span>: +1 life (max 3)</li>
            </ul>
        </div>
        <div id="message" class="message">
            <div id="messageText">Game Over</div>
            <button id="restartButton">Play Again</button>
        </div>
    </div>

    <script>
        // Game constants
        const NUM_ROWS = 8;
        const BRICK_TOP_OFFSET = 50;
        const BRICK_SPACING = 4;
        const NUM_BRICKS_PER_ROW = 10;
        const BRICK_HEIGHT = 25;
        const PADDLE_WIDTH = 100;
        const PADDLE_HEIGHT = 15;
        const PADDLE_OFFSET = 30;
        const BALL_RADIUS = 10;
        const COLORS = ['#FF5252', '#FF9800', '#FFEB3B', '#4CAF50', '#2196F3', '#9C27B0'];
        const MAX_LIVES = 3;
        
        // Power-up constants
        const POWERUP_PROBABILITY = 0.3; // 30% chance of a brick dropping a power-up
        const POWERUP_TYPES = {
            FIREBALL: 'fireball',
            PADDLE: 'paddle',
            MULTIBALL: 'multiball',
            EXTRA_LIFE: 'extra-life'
        };
        const POWERUP_COLORS = {
            [POWERUP_TYPES.FIREBALL]: '#FF5252',
            [POWERUP_TYPES.PADDLE]: '#4CAF50',
            [POWERUP_TYPES.MULTIBALL]: '#2196F3',
            [POWERUP_TYPES.EXTRA_LIFE]: '#9C27B0'
        };
        const POWERUP_WIDTH = 30;
        const POWERUP_HEIGHT = 15;
        const POWERUP_SPEED = 3;
        const FIREBALL_DURATION = 10000; // 10 seconds
        const LONG_PADDLE_DURATION = 10000; // 10 seconds

        // Game variables
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const scoreElement = document.getElementById('score');
        const livesElement = document.getElementById('lives');
        const messageElement = document.getElementById('message');
        const messageText = document.getElementById('messageText');
        const restartButton = document.getElementById('restartButton');

        let bricks = [];
        let paddle;
        let balls = [];
        let powerUps = [];
        let score = 0;
        let lives = 3;
        let gameState = 'ready'; // ready, playing, paused, gameover, win
        let fireballActive = false;
        let fireballTimeout = null;
        let longPaddleActive = false;
        let longPaddleTimeout = null;
        let originalPaddleWidth = PADDLE_WIDTH;

        // Initialize game
        function initGame() {
            createBricks();
            createPaddle();
            balls = [createBall()];
            powerUps = [];
            score = 0;
            lives = 3;
            fireballActive = false;
            longPaddleActive = false;
            clearTimeout(fireballTimeout);
            clearTimeout(longPaddleTimeout);
            paddle.width = originalPaddleWidth;
            updateScore();
            updateLives();
            hideMessage();
            gameState = 'ready';
        }

        // Create bricks
        function createBricks() {
            bricks = [];
            const spaceForBricks = canvas.width - (NUM_BRICKS_PER_ROW + 1) * BRICK_SPACING;
            const brickWidth = spaceForBricks / NUM_BRICKS_PER_ROW;

            for (let row = 0; row < NUM_ROWS; row++) {
                for (let col = 0; col < NUM_BRICKS_PER_ROW; col++) {
                    const colorIndex = Math.floor(row / 2) % COLORS.length;
                    const brick = {
                        x: BRICK_SPACING + col * (brickWidth + BRICK_SPACING),
                        y: BRICK_TOP_OFFSET + row * (BRICK_HEIGHT + BRICK_SPACING),
                        width: brickWidth,
                        height: BRICK_HEIGHT,
                        color: COLORS[colorIndex],
                        active: true
                    };
                    bricks.push(brick);
                }
            }
        }

        // Create paddle
        function createPaddle() {
            paddle = {
                x: canvas.width / 2 - PADDLE_WIDTH / 2,
                y: canvas.height - PADDLE_OFFSET - PADDLE_HEIGHT,
                width: PADDLE_WIDTH,
                height: PADDLE_HEIGHT,
                color: '#2196F3'
            };
            originalPaddleWidth = PADDLE_WIDTH;
        }

        // Create ball
        function createBall() {
            const ball = {
                x: canvas.width / 2,
                y: canvas.height - PADDLE_OFFSET - PADDLE_HEIGHT - BALL_RADIUS - 5,
                radius: BALL_RADIUS,
                dx: 0,
                dy: 0,
                color: '#FFFFFF',
                fireball: false
            };
            return ball;
        }

        // Create power-up
        function createPowerUp(x, y) {
            const powerUpTypes = Object.values(POWERUP_TYPES);
            const randomType = powerUpTypes[Math.floor(Math.random() * powerUpTypes.length)];
            
            return {
                x: x,
                y: y,
                width: POWERUP_WIDTH,
                height: POWERUP_HEIGHT,
                type: randomType,
                color: POWERUP_COLORS[randomType],
                active: true
            };
        }

        // Draw everything
        function draw() {
            // Clear canvas
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            // Draw bricks
            bricks.forEach(brick => {
                if (brick.active) {
                    ctx.fillStyle = brick.color;
                    ctx.fillRect(brick.x, brick.y, brick.width, brick.height);

                    // Add brick highlight
                    ctx.strokeStyle = 'rgba(255, 255, 255, 0.3)';
                    ctx.lineWidth = 2;
                    ctx.strokeRect(brick.x, brick.y, brick.width, brick.height);
                }
            });

            // Draw paddle
            ctx.fillStyle = paddle.color;
            ctx.fillRect(paddle.x, paddle.y, paddle.width, paddle.height);

            // Add paddle highlight
            ctx.strokeStyle = 'rgba(255, 255, 255, 0.5)';
            ctx.lineWidth = 2;
            ctx.strokeRect(paddle.x, paddle.y, paddle.width, paddle.height);

            // Draw balls
            balls.forEach(ball => {
                ctx.fillStyle = ball.fireball ? '#FF5722' : ball.color;
                ctx.beginPath();
                ctx.arc(ball.x, ball.y, ball.radius, 0, Math.PI * 2);
                ctx.fill();

                // Add ball highlight
                ctx.strokeStyle = ball.fireball ? 'rgba(255, 200, 0, 0.8)' : 'rgba(0, 0, 0, 0.5)';
                ctx.lineWidth = 1;
                ctx.stroke();

                // Draw center dot
                ctx.fillStyle = ball.fireball ? '#FFEB3B' : '#FF5722';
                ctx.beginPath();
                ctx.arc(ball.x, ball.y, ball.radius / 3, 0, Math.PI * 2);
                ctx.fill();
            });

            // Draw power-ups
            powerUps.forEach(powerUp => {
                if (powerUp.active) {
                    ctx.fillStyle = powerUp.color;
                    ctx.fillRect(powerUp.x, powerUp.y, powerUp.width, powerUp.height);
                    
                    // Draw power-up icon
                    ctx.fillStyle = '#FFFFFF';
                    ctx.font = '12px Arial';
                    ctx.textAlign = 'center';
                    ctx.textBaseline = 'middle';
                    
                    let symbol = '';
                    switch(powerUp.type) {
                        case POWERUP_TYPES.FIREBALL:
                            symbol = '🔥';
                            break;
                        case POWERUP_TYPES.PADDLE:
                            symbol = '📏';
                            break;
                        case POWERUP_TYPES.MULTIBALL:
                            symbol = '⚾⚾';
                            break;
                        case POWERUP_TYPES.EXTRA_LIFE:
                            symbol = '❤️';
                            break;
                    }
                    
                    ctx.fillText(symbol, powerUp.x + POWERUP_WIDTH/2, powerUp.y + POWERUP_HEIGHT/2);
                }
            });
        }

        // Update game state
        function update() {
            if (gameState !== 'playing') return;

            // Move balls
            balls.forEach(ball => {
                ball.x += ball.dx;
                ball.y += ball.dy;

                // Wall collisions
                if (ball.x + ball.radius > canvas.width || ball.x - ball.radius < 0) {
                    ball.dx = -ball.dx;
                }

                if (ball.y - ball.radius < 0) {
                    ball.dy = -ball.dy;
                }
            });

            // Move power-ups
            powerUps.forEach(powerUp => {
                if (powerUp.active) {
                    powerUp.y += POWERUP_SPEED;
                    
                    // Remove power-ups that go off screen
                    if (powerUp.y > canvas.height) {
                        powerUp.active = false;
                    }
                    
                    // Check collision with paddle
                    if (powerUp.y + powerUp.height > paddle.y && 
                        powerUp.y < paddle.y + paddle.height &&
                        powerUp.x + powerUp.width > paddle.x && 
                        powerUp.x < paddle.x + paddle.width) {
                        
                        applyPowerUp(powerUp);
                        powerUp.active = false;
                    }
                }
            });

            let bricksRemaining = 0;

            // Check ball collisions with bricks and paddle
            balls.forEach(ball => {
                // Paddle collision
                if (ball.y + ball.radius > paddle.y && 
                    ball.y - ball.radius < paddle.y + paddle.height &&
                    ball.x > paddle.x && 
                    ball.x < paddle.x + paddle.width) {

                    // Calculate bounce angle based on where ball hits paddle
                    const hitPosition = (ball.x - (paddle.x + paddle.width / 2)) / (paddle.width / 2);
                    ball.dx = hitPosition * 6;
                    ball.dy = -Math.abs(ball.dy);
                }

                // Brick collisions
                bricks.forEach(brick => {
                    if (!brick.active) return;

                    bricksRemaining++;

                    if (ball.x + ball.radius > brick.x && 
                        ball.x - ball.radius < brick.x + brick.width &&
                        ball.y + ball.radius > brick.y && 
                        ball.y - ball.radius < brick.y + brick.height) {

                        // Fireball passes through bricks without bouncing
                        if (!ball.fireball) {
                            // Determine bounce direction
                            const ballLeft = ball.x - ball.radius;
                            const ballRight = ball.x + ball.radius;
                            const ballTop = ball.y - ball.radius;
                            const ballBottom = ball.y + ball.radius;

                            const brickLeft = brick.x;
                            const brickRight = brick.x + brick.width;
                            const brickTop = brick.y;
                            const brickBottom = brick.y + brick.height;

                            // Determine which side was hit
                            const bottomHit = Math.abs(ballTop - brickBottom);
                            const topHit = Math.abs(ballBottom - brickTop);
                            const leftHit = Math.abs(ballRight - brickLeft);
                            const rightHit = Math.abs(ballLeft - brickRight);

                            const minHit = Math.min(bottomHit, topHit, leftHit, rightHit);

                            if (minHit === bottomHit || minHit === topHit) {
                                ball.dy = -ball.dy;
                            } else {
                                ball.dx = -ball.dx;
                            }
                        }

                        // Collision detected - remove brick
                        brick.active = false;
                        score += 10;
                        updateScore();

                        // Random chance to drop a power-up
                        if (Math.random() < POWERUP_PROBABILITY) {
                            powerUps.push(createPowerUp(
                                brick.x + brick.width/2 - POWERUP_WIDTH/2,
                                brick.y + brick.height/2
                            ));
                        }
                    }
                });
            });

            // Check if all bricks are destroyed
            if (bricksRemaining === 0) {
                showMessage("You Win!", true);
                gameState = 'win';
                return;
            }

            // Check if any ball is below paddle
            for (let i = balls.length - 1; i >= 0; i--) {
                const ball = balls[i];
                
                if (ball.y - ball.radius > canvas.height) {
                    balls.splice(i, 1);
                }
            }

            // If no balls left
            if (balls.length === 0) {
                lives--;
                updateLives();

                if (lives <= 0) {
                    showMessage("Game Over", false);
                    gameState = 'gameover';
                } else {
                    balls = [createBall()];
                    gameState = 'ready';
                }
            }
        }

        // Apply power-up effect
        function applyPowerUp(powerUp) {
            switch(powerUp.type) {
                case POWERUP_TYPES.FIREBALL:
                    activateFireball();
                    break;
                case POWERUP_TYPES.PADDLE:
                    activateLongPaddle();
                    break;
                case POWERUP_TYPES.MULTIBALL:
                    activateMultiball();
                    break;
                case POWERUP_TYPES.EXTRA_LIFE:
                    if (lives < MAX_LIVES) {
                        lives++;
                        updateLives();
                    }
                    break;
            }
        }

        // Activate fireball
        function activateFireball() {
            fireballActive = true;
            balls.forEach(ball => {
                ball.fireball = true;
            });
            
            // Clear any existing timeout
            if (fireballTimeout) clearTimeout(fireballTimeout);
            
            // Set timeout to deactivate fireball
            fireballTimeout = setTimeout(() => {
                fireballActive = false;
                balls.forEach(ball => {
                    ball.fireball = false;
                });
            }, FIREBALL_DURATION);
        }

        // Activate long paddle
        function activateLongPaddle() {
            if (!longPaddleActive) {
                paddle.width = originalPaddleWidth * 1.5;
                longPaddleActive = true;
            }
            
            // Clear any existing timeout
            if (longPaddleTimeout) clearTimeout(longPaddleTimeout);
            
            // Set timeout to deactivate long paddle
            longPaddleTimeout = setTimeout(() => {
                paddle.width = originalPaddleWidth;
                longPaddleActive = false;
            }, LONG_PADDLE_DURATION);
        }

        // Activate multiball
        function activateMultiball() {
            const newBalls = [];
            
            balls.forEach(ball => {
                // Create a new ball for each existing ball
                const newBall = {
                    x: ball.x,
                    y: ball.y,
                    radius: BALL_RADIUS,
                    dx: -ball.dx,
                    dy: -ball.dy,
                    color: '#FFFFFF',
                    fireball: ball.fireball
                };
                newBalls.push(newBall);
            });
            
            // Add the new balls to the game
            balls = balls.concat(newBalls);
        }

        // Game loop
        function gameLoop() {
            update();
            draw();
            requestAnimationFrame(gameLoop);
        }

        // Event listeners
        canvas.addEventListener('mousemove', e => {
            const rect = canvas.getBoundingClientRect();
            const mouseX = e.clientX - rect.left;

            // Move paddle with mouse
            paddle.x = Math.max(
                0, 
                Math.min(canvas.width - paddle.width, mouseX - paddle.width / 2)
            );

            // If game is ready, move ball with paddle
            if (gameState === 'ready' && balls.length > 0) {
                balls[0].x = paddle.x + paddle.width / 2;
            }
        });

        canvas.addEventListener('click', () => {
            if (gameState === 'ready' && balls.length > 0) {
                // Launch ball at random upward angle
                balls[0].dx = (Math.random() * 4 + 2) * (Math.random() > 0.5 ? 1 : -1);
                balls[0].dy = -5;
                gameState = 'playing';
            }
        });

        restartButton.addEventListener('click', () => {
            initGame();
        });

        // UI functions
        function updateScore() {
            scoreElement.textContent = score;
        }

        function updateLives() {
            livesElement.textContent = lives;
        }

        function showMessage(text, isWin) {
            messageText.textContent = text;
            messageText.style.color = isWin ? '#4CAF50' : '#FF5252';
            messageElement.style.display = 'block';
        }

        function hideMessage() {
            messageElement.style.display = 'none';
        }

        // Start the game
        initGame();
        gameLoop();
    </script>
</body>
</html>
UN uego para jugar cuando tengas ganar de jugar a jugar algo

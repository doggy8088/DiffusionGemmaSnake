# 貪食蛇遊戲

以下為使用 DiffusionGemma 產生此頁面的指令：

```sh
uv run python -m mlx_vlm.generate \
  --model mlx-community/diffusiongemma-26B-A4B-it-4bit \
  --max-tokens 25600 \
  --temperature 0.0 \
  --prompt "請用 VanillaJS 設計一款貪食蛇遊戲，畫面精美"
```

這是一個使用純 JavaScript (Vanilla JS)、HTML 和 CSS 編的貪食蛇遊戲。為了達到「畫面精美」，我採用了**霓虹風** (Neon Style)設計，並加入了流暢的動畫效果與陰影特效。

你可以直接將以下代碼儲存為 `.html` 檔並用瀏覽器開啟。

```html
<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>霓虹貪食蛇遊戲</title>
    <style>
        :root {
            --bg-color: #0a0a12;
            --snake-color: #00f2ff;
            --snake-head: #0088ff;
            --food-color: #ff007b;
            --text-color: #ffffff;
        }

        body {
            margin: 0;
            padding: 0;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            height: 100vh;
            background-color: var(--bg-color);
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            color: var(--text-color);
            overflow: hidden;
        }

        .game-container {
            position: relative;
            box-shadow: 0 0 50px rgba(0, 242, 255, 0.2);
            border: 4px solid #333;
            border-radius: 10px;
        }

        canvas {
            display: block;
            background: radial-gradient(circle, #161625 0%, #0a0a12 100%);
        }

        .ui-layer {
            position: absolute;
            top: -50px;
            width: 100%;
            display: flex;
            justify-content: space-between;
            font-size: 24px;
            font-weight: bold;
            text-shadow: 0 0 10px rgba(255,255,255,0.5);
        }

        .score-board {
            color: var(--snake-color);
        }

        .overlay {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            text-align: center;
            background: rgba(0, 0, 0, 0.8);
            padding: 2rem;
            border-radius: 15px;
            border: 1px solid var(--snake-color);
            display: none;
        }

        button {
            margin-top: 20px;
            padding: 10px 25px;
            font-size: 18px;
            background: var(--snake-color);
            border: none;
            border-radius: 5px;
            cursor: pointer;
            font-weight: bold;
            transition: transform 0.2s, background 0.2s;
        }

        button:hover {
            transform: scale(1.1);
            background: #00d8ef;
        }

        .instructions {
            margin-top: 20px;
            color: #666;
            font-size: 14px;
        }
    </style>
</head>
<body>

    <div class="game-container">
        <div class="ui-layer">
            <div class="score-board">SCORE: <span id="score">0</span></div>
        </div>
        <canvas id="gameCanvas" width="600" height="400"></canvas>

        <div id="gameOver" class="overlay">
            <h1 style="color: var(--food-color)">GAME OVER</h1>
            <p>你的分數： <span id="finalScore">0</span></p>
            <button onclick="resetGame()">重新遊戲</button>
        </div>
    </div>

    <div class="instructions">使用方向鍵控制控制蛇移動</div>

    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const scoreElement = document.getElementById('score');
        const finalScoreElement = document.getElementById('finalScore');
        const gameOverScreen = document.getElementById('gameOver');

        const grid = 20; // 格子大小
        let tileX = canvas.width / grid;
        let tileY = canvas.height / grid;

        let score = 0;
        let snake = [{x: 10, y: 10}];
        let food = {x: 5, y: 5};
        let dx = 0;
        let dy = 0;
        let nextDx = 0;
        let nextDy = 0;
        let gameSpeed = 100;

        function main() {
            if (checkGameOver()) {
                showGameOver();
                return;
            }

            setTimeout(function onTick() {
                clearCanvas();
                drawFood();
                moveSnake();
                drawSnake();
                main();
            }, gameSpeed);
        }

        function clearCanvas() {
            // 畫背景
            ctx.fillStyle = "#0a0a12";
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            // 畫淡淡的網格 (增加高級感)
            ctx.strokeStyle = "rgba(255,255,255,0.03)";
            for(let i=0; i<canvas.width; i+=grid) {
                ctx.beginPath(); ctx.moveTo(i,0); ctx.lineTo(i,canvas.height); ctx.stroke();
            }
            for(let i=0; i<canvas.height; i+=grid) {
                ctx.beginPath(); ctx.moveTo(0,i); ctx.lineTo(canvas.width,i); ctx.stroke();
            }
        }

        function drawSnake() {
            snake.forEach((part, index) => {
                const isHead = index === 0;
                ctx.fillStyle = isHead ? "#0088ff" : "#00f2ff";

                // 發光特效
                ctx.shadowBlur = 15;
                ctx.shadowColor = "#00f2ff";

                // 畫蛇身體
                ctx.fillRect(part.x * grid + 1, part.y * grid + 1, grid - 2, grid - 2);
                ctx.shadowBlur = 0; // 重置陰影，避免影響其他元素
            });
        }

        function moveSnake() {
            dx = nextDx;
            dy = nextDy;

            const head = {x: snake[0].x + dx, y: snake[0].y + dy };

            // 如果沒按鍵，則不移動
            if(dx === 0 && dy === 0) return;

            snake.unshift(head);

            // 檢查是否吃到食物
            if (head.x === food.x && head.y === food.y) {
                score += 10;
                scoreElement.innerText = score;
                generateFood();
                // 積分增加速度
                if(gameSpeed > 50) gameSpeed -= 2;
            } else {
                snake.pop();
            }
        }

        function generateFood() {
            food.x = Math.floor(Math.random() * (canvas.width / grid));
            food.y = Math.floor(Math.random() * (canvas.height / grid));
            // 確保食物不會出現在蛇體上
            snake.forEach(part => {
                if(part.x === food.x && part.y === food.y) generateFood();
            });
        }

        function drawFood() {
            ctx.fillStyle = "#ff007b";
            ctx.shadowBlur = 20;
            ctx.shadowColor = "#ff007b";
            ctx.beginPath();
            ctx.arc(food.x * grid + grid/2, food.y * grid + grid/2, grid/2 - 2, 0, Math.PI * 2);
            ctx.fill();
            ctx.shadowBlur = 0;
        }

        function checkGameOver() {
            const head = snake[0];
            // 碰撞牆壁
            if (head.x < 0 || head.x >= canvas.width / grid || head.y < 0 || head.y >= canvas.height / grid) return true;
            // 碰撞自己
            for (let i = 1; i < snake.length; i++) {
                if (head.x === snake[i].x && head.y === snake[i].y) return true;
            }
            return false;
        }

        function showGameOver() {
            gameOverScreen.style.display = 'block';
            finalScoreElement.innerText = score;
        }

        function resetGame() {
            score = 0;
            scoreElement.innerText = score;
            snake = [{x: 10, y: 10}];
            dx = 0; dy = 0;
            nextDx = 0; nextDy = 0;
            gameSpeed = 100;
            gameOverScreen.style.display = 'none';
            main();
        }

        window.addEventListener('keydown', e => {
            const key = e.key;
            // 防止反向轉向 180 度
            if (key === 'ArrowUp' && dy === 0) { nextDx = 0; nextDy = -1; }
            if (key === 'ArrowDown' && dy === 0) { nextDx = 0; nextDy = 1; }
            if (key === 'ArrowLeft' && dx === 0) { nextDx = -1; nextDy = 0; }
            if (key === 'ArrowRight' && dx === 0) { nextDx = 1; nextDy = 0; }
        });

        // 初始化啟遊戲
        generateFood();
        main();
    </script>
</body>
</html>
```

### 此版本的亮點：

1.  **霓虹視覺風格**：
    *   使用了 `shadowBlur` 和 `shadowColor` 讓蛇和食物產生如同「霓虹發光」的效果。
    *   背景採用了漸變色（Radial Gradient）並帶有細微的背景網格線，讓畫面看起來更有層次。
2.  **流暢的操控**：
    *   **輸入緩衝機制**：使用了 `nextDx/Dy` 變量。這解決了當你快速連續按下相反方向鍵時，蛇會因為判斷不及而「撞到自己」的經典 Bug。
3.  **動態難度**：
    *   每當吃到一顆食物，遊戲速度（`gameSpeed`）會稍微提升，增加挑戰性。
4.  **響響 UI**：
    *   設計了簡潔的遊戲結束覆層（Overlay），包含分數顯示與重新開始按鈕。

### 如何執行：
1. 複製代碼並存為 `snake.html`。
2. 使用任何現代瀏覽器（Chrome, Edge, Firefox 等）開啟。
3. 使用鍵盤的**方向鍵**控制遊戲。

```
==========
Prompt: 27 tokens, 433.313 tokens-per-sec
Generation: 2714 tokens, 63.230 tokens-per-sec
Peak memory: 18.607 GB
```

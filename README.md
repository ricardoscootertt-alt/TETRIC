<html lang="pt-PT">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>TetRic - Neon Edition</title>
    <style>
        :root {
            --bg-color: #050510;
            --panel-color: #12122b;
            --neon-cyan: #00f2ff;
            --neon-magenta: #ff007f;
            --neon-yellow: #f2ff00;
            --text-main: #ffffff;
            --shadow-glow: 0 0 15px rgba(0, 242, 255, 0.5);
        }

        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            touch-action: none; /* Previne scroll e zoom acidental no telemóvel */
        }

        body {
            background-color: var(--bg-color);
            color: var(--text-main);
            font-family: 'Segoe UI', system-ui, -apple-system, sans-serif;
            height: 100vh;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: space-between;
            overflow: hidden;
            padding: 10px;
        }

        /* Cabeçalho */
        header {
            text-align: center;
            padding: 5px 0;
        }

        .logo {
            font-size: 32px;
            font-weight: 900;
            letter-spacing: 4px;
            text-transform: uppercase;
            color: var(--text-main);
            text-shadow: var(--shadow-glow);
        }

        .logo span {
            color: var(--neon-magenta);
        }

        /* Estrutura do Jogo */
        .game-wrapper {
            display: flex;
            gap: 15px;
            align-items: flex-start;
            max-height: 60vh;
        }

        #game-board {
            border: 3px solid var(--neon-cyan);
            background-color: #000;
            box-shadow: var(--shadow-glow);
            height: 100%;
            aspect-ratio: 10 / 20;
        }

        /* Painel Lateral */
        .sidebar {
            display: flex;
            flex-direction: column;
            gap: 10px;
            min-width: 90px;
        }

        .data-card {
            background: var(--panel-color);
            padding: 10px;
            border-radius: 10px;
            border: 1px solid rgba(255, 255, 255, 0.1);
            text-align: center;
        }

        .data-label {
            font-size: 10px;
            text-transform: uppercase;
            color: var(--neon-cyan);
            margin-bottom: 5px;
        }

        .data-value {
            font-size: 18px;
            font-weight: bold;
        }

        #next-canvas {
            background: #000;
            border: 1px solid var(--neon-magenta);
            width: 60px;
            height: 60px;
            margin: 0 auto;
        }

        /* Controlos Adaptáveis */
        .controls-panel {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 10px;
            width: 100%;
            max-width: 340px;
            margin-bottom: 10px;
        }

        .control-btn {
            background: var(--panel-color);
            border: 1px solid var(--neon-cyan);
            color: white;
            padding: 20px;
            border-radius: 15px;
            font-size: 24px;
            display: flex;
            align-items: center;
            justify-content: center;
            cursor: pointer;
            user-select: none;
            transition: transform 0.1s, background 0.1s;
        }

        .control-btn:active {
            background: var(--neon-cyan);
            color: #000;
            transform: scale(0.95);
        }

        .btn-full {
            grid-column: span 3;
            font-size: 16px;
            font-weight: bold;
            padding: 15px;
            background: linear-gradient(45deg, var(--panel-color), #252545);
        }

        /* Ecrã de Game Over */
        #game-over-screen {
            position: fixed;
            inset: 0;
            background: rgba(0, 0, 0, 0.95);
            display: none;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            z-index: 100;
            text-align: center;
        }

        .retry-btn {
            margin-top: 25px;
            padding: 15px 30px;
            background: var(--neon-magenta);
            border: none;
            color: white;
            border-radius: 8px;
            font-weight: bold;
            cursor: pointer;
            font-size: 18px;
        }

        @media (max-height: 700px) {
            .logo { font-size: 24px; }
            .control-btn { padding: 15px; }
        }
    </style>
</head>
<body>

    <div id="game-over-screen">
        <h1 style="color: var(--neon-magenta); font-size: 40px; margin-bottom: 10px;">FIM DE JOGO</h1>
        <p style="font-size: 20px;">Pontuação Final no <strong>TetRic</strong>:</p>
        <p id="final-score" style="font-size: 32px; font-weight: bold; color: var(--neon-cyan);">0</p>
        <button class="retry-btn" onclick="startNewGame()">JOGAR NOVAMENTE</button>
    </div>

    <header>
        <div class="logo">Tet<span>Ric</span></div>
    </header>

    <div class="game-wrapper">
        <canvas id="game-board"></canvas>

        <div class="sidebar">
            <div class="data-card">
                <div class="data-label">Próxima</div>
                <canvas id="next-canvas"></canvas>
            </div>
            <div class="data-card">
                <div class="data-label">Pontos</div>
                <div id="score" class="data-value">0</div>
            </div>
            <div class="data-card">
                <div class="data-label">Nível</div>
                <div id="level" class="data-value">1</div>
            </div>
        </div>
    </div>

    <div class="controls-panel">
        <div class="control-btn" id="left-btn">←</div>
        <div class="control-btn" id="rotate-btn">↻</div>
        <div class="control-btn" id="right-btn">→</div>
        <div class="control-btn btn-full" id="hard-drop-btn">QUEDA TOTAL (SPACE)</div>
    </div>

    <script>
        const canvas = document.getElementById('game-board');
        const ctx = canvas.getContext('2d');
        const nextCanvas = document.getElementById('next-canvas');
        const nextCtx = nextCanvas.getContext('2d');

        const COLS = 10;
        const ROWS = 20;
        let blockSize = 0;

        const COLORS = [
            null,
            '#00f2ff', // I
            '#ff007f', // T
            '#f2ff00', // O
            '#00ff66', // S
            '#ff4d00', // Z
            '#0044ff', // J
            '#ffae00'  // L
        ];

        const TETROMINOS = {
            'I': [[0,0,0,0],[1,1,1,1],[0,0,0,0],[0,0,0,0]],
            'T': [[0,2,0],[2,2,2],[0,0,0]],
            'O': [[3,3],[3,3]],
            'S': [[0,4,4],[4,4,0],[0,0,0]],
            'Z': [[5,5,0],[0,5,5],[0,0,0]],
            'J': [[6,0,0],[6,6,6],[0,0,0]],
            'L': [[0,0,7],[7,7,7],[0,0,0]]
        };

        let arena = [];
        let player = { pos: {x: 0, y: 0}, matrix: null };
        let nextPieceMatrix = null;
        let score = 0;
        let level = 1;
        let dropCounter = 0;
        let dropInterval = 1000;
        let lastTime = 0;
        let isPaused = false;

        function resize() {
            const h = canvas.clientHeight;
            canvas.height = h;
            canvas.width = h * (COLS / ROWS);
            blockSize = canvas.width / COLS;
            
            nextCanvas.width = 60;
            nextCanvas.height = 60;
        }

        function createArena(w, h) {
            const matrix = [];
            while (h--) matrix.push(new Array(w).fill(0));
            return matrix;
        }

        function drawBlock(x, y, colorIndex, context, size, isGhost = false) {
            const xPos = x * size;
            const yPos = y * size;
            
            context.shadowBlur = isGhost ? 0 : 10;
            context.shadowColor = COLORS[colorIndex];
            context.fillStyle = isGhost ? 'rgba(255,255,255,0.1)' : COLORS[colorIndex];
            context.fillRect(xPos + 1, yPos + 1, size - 2, size - 2);
            
            if (!isGhost) {
                context.shadowBlur = 0;
                context.strokeStyle = 'rgba(255,255,255,0.3)';
                context.strokeRect(xPos + 1, yPos + 1, size - 2, size - 2);
            }
        }

        function rotate(matrix) {
            for (let y = 0; y < matrix.length; ++y) {
                for (let x = 0; x < y; ++x) {
                    [matrix[x][y], matrix[y][x]] = [matrix[y][x], matrix[x][y]];
                }
            }
            matrix.forEach(row => row.reverse());
        }

        function collide(arena, player) {
            const [m, o] = [player.matrix, player.pos];
            for (let y = 0; y < m.length; ++y) {
                for (let x = 0; x < m[y].length; ++x) {
                    if (m[y][x] !== 0 && (arena[y + o.y] && arena[y + o.y][x + o.x]) !== 0) {
                        return true;
                    }
                }
            }
            return false;
        }

        function merge(arena, player) {
            player.matrix.forEach((row, y) => {
                row.forEach((value, x) => {
                    if (value !== 0) arena[y + player.pos.y][x + player.pos.x] = value;
                });
            });
        }

        function arenaSweep() {
            let rowCount = 0;
            outer: for (let y = arena.length - 1; y >= 0; --y) {
                for (let x = 0; x < arena[y].length; ++x) {
                    if (arena[y][x] === 0) continue outer;
                }
                const row = arena.splice(y, 1)[0].fill(0);
                arena.unshift(row);
                ++y;
                rowCount++;
            }
            if (rowCount > 0) {
                score += [0, 10, 30, 60, 150][rowCount] * level;
                document.getElementById('score').innerText = score;
                if (score >= level * 100) {
                    level++;
                    dropInterval = Math.max(100, 1000 - (level - 1) * 100);
                    document.getElementById('level').innerText = level;
                }
            }
        }

        function drawGhost() {
            const ghost = { pos: { x: player.pos.x, y: player.pos.y }, matrix: player.matrix };
            while (!collide(arena, ghost)) {
                ghost.pos.y++;
            }
            ghost.pos.y--;
            ghost.matrix.forEach((row, y) => {
                row.forEach((value, x) => {
                    if (value !== 0) {
                        drawBlock(x + ghost.pos.x, y + ghost.pos.y, value, ctx, blockSize, true);
                    }
                });
            });
        }

        function playerReset() {
            const pieces = 'ILJOTSZ';
            player.matrix = nextPieceMatrix || TETROMINOS[pieces[pieces.length * Math.random() | 0]];
            nextPieceMatrix = TETROMINOS[pieces[pieces.length * Math.random() | 0]];
            
            player.pos.y = 0;
            player.pos.x = (COLS / 2 | 0) - (player.matrix[0].length / 2 | 0);

            if (collide(arena, player)) {
                isPaused = true;
                document.getElementById('final-score').innerText = score;
                document.getElementById('game-over-screen').style.display = 'flex';
            }
            drawNext();
        }

        function playerDrop() {
            player.pos.y++;
            if (collide(arena, player)) {
                player.pos.y--;
                merge(arena, player);
                playerReset();
                arenaSweep();
            }
            dropCounter = 0;
        }

        function playerHardDrop() {
            while (!collide(arena, player)) {
                player.pos.y++;
            }
            player.pos.y--;
            merge(arena, player);
            playerReset();
            arenaSweep();
            dropCounter = 0;
        }

        function playerMove(dir) {
            player.pos.x += dir;
            if (collide(arena, player)) player.pos.x -= dir;
        }

        function playerRotate() {
            const pos = player.pos.x;
            let offset = 1;
            rotate(player.matrix);
            while (collide(arena, player)) {
                player.pos.x += offset;
                offset = -(offset + (offset > 0 ? 1 : -1));
                if (offset > player.matrix[0].length) {
                    rotate(player.matrix); rotate(player.matrix); rotate(player.matrix);
                    player.pos.x = pos;
                    return;
                }
            }
        }

        function draw() {
            ctx.fillStyle = '#000';
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            // Arena estática
            arena.forEach((row, y) => {
                row.forEach((value, x) => {
                    if (value !== 0) drawBlock(x, y, value, ctx, blockSize);
                });
            });

            // Jogador e Sombra
            if (player.matrix) {
                drawGhost();
                player.matrix.forEach((row, y) => {
                    row.forEach((value, x) => {
                        if (value !== 0) drawBlock(x + player.pos.x, y + player.pos.y, value, ctx, blockSize);
                    });
                });
            }
        }

        function drawNext() {
            nextCtx.fillStyle = '#000';
            nextCtx.fillRect(0, 0, nextCanvas.width, nextCanvas.height);
            const size = 12;
            const offX = (nextCanvas.width - nextPieceMatrix[0].length * size) / 2;
            const offY = (nextCanvas.height - nextPieceMatrix.length * size) / 2;
            nextPieceMatrix.forEach((row, y) => {
                row.forEach((value, x) => {
                    if (value !== 0) {
                        nextCtx.fillStyle = COLORS[value];
                        nextCtx.fillRect(offX + x * size, offY + y * size, size - 1, size - 1);
                    }
                });
            });
        }

        function update(time = 0) {
            if (isPaused) return;
            const deltaTime = time - lastTime;
            lastTime = time;
            dropCounter += deltaTime;
            if (dropCounter > dropInterval) playerDrop();
            draw();
            requestAnimationFrame(update);
        }

        function startNewGame() {
            arena = createArena(COLS, ROWS);
            score = 0;
            level = 1;
            dropInterval = 1000;
            isPaused = false;
            nextPieceMatrix = null;
            document.getElementById('score').innerText = '0';
            document.getElementById('level').innerText = '1';
            document.getElementById('game-over-screen').style.display = 'none';
            playerReset();
            update();
        }

        // Controlos teclado
        document.addEventListener('keydown', e => {
            if (isPaused) return;
            if (e.key === 'ArrowLeft') playerMove(-1);
            if (e.key === 'ArrowRight') playerMove(1);
            if (e.key === 'ArrowDown') playerDrop();
            if (e.key === 'ArrowUp') playerRotate();
            if (e.key === ' ') { e.preventDefault(); playerHardDrop(); }
        });

        // Eventos Mobile
        const addEvent = (id, action) => {
            const btn = document.getElementById(id);
            btn.addEventListener('touchstart', (e) => { e.preventDefault(); action(); });
            btn.addEventListener('mousedown', (e) => { e.preventDefault(); action(); });
        };

        addEvent('left-btn', () => playerMove(-1));
        addEvent('right-btn', () => playerMove(1));
        addEvent('rotate-btn', () => playerRotate());
        addEvent('hard-drop-btn', () => playerHardDrop());

        window.addEventListener('resize', resize);
        
        // Inicialização
        resize();
        startNewGame();
    </script>
</body>
</html>


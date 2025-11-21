<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Jogo de Obstáculos</title>
    <style>
        /* Estilo base para garantir visualização em tela cheia e responsividade */
        body {
            font-family: 'Inter', sans-serif;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            margin: 0;
            background-color: #1a1a2e; /* Fundo escuro */
            color: #e4e4e4;
            padding: 10px;
            box-sizing: border-box;
            user-select: none; /* Desativa a seleção de texto para melhor experiência de jogo */
        }

        #game-container {
            width: 100%;
            max-width: 550px; /* Limite para não ficar muito largo em desktops */
            position: relative;
            text-align: center;
            background-color: #2e2e4a;
            padding: 20px;
            border-radius: 16px;
            box-shadow: 0 15px 30px rgba(0, 0, 0, 0.6);
            border: 3px solid #00bcd4;
        }

        h1 {
            color: #00bcd4; /* Azul ciano */
            margin-top: 0;
            margin-bottom: 5px;
            font-size: 1.5em;
        }

        p {
            font-size: 0.9em;
            margin-bottom: 15px;
            color: #ccc;
        }

        #gameCanvas {
            display: block;
            width: 100%;
            /* Altura será ajustada pelo JS, mas damos uma proporção base */
            aspect-ratio: 4 / 3; 
            background-color: #f0f0f0;
            border-radius: 8px;
            box-shadow: inset 0 0 10px rgba(0, 0, 0, 0.1);
        }

        .info-box {
            background-color: #161625;
            color: #fce47c;
            padding: 10px;
            border-radius: 8px;
            font-weight: bold;
            margin-top: 15px;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.3);
        }

        /* Estilo da tela de Fim de Jogo/Início (Tela de Carregamento) */
        #messageBox {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, 0.9);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            text-align: center;
            border-radius: 16px;
            z-index: 10;
            transition: opacity 0.3s ease;
        }

        .message-box h2 {
            color: #ff5722; /* Laranja de aviso */
            font-size: 2.2em;
            margin-bottom: 5px;
        }

        .message-box p {
            color: #ccc;
            font-size: 1.1em;
            margin-top: 5px;
        }

        .hidden {
            display: none;
        }
    </style>
</head>
<body>
    <div id="game-container">
        <h1>Evite os Blocos (Canvas Runner)</h1>
        <p>Use a barra de espaço ou toque na tela para saltar.</p>
        
        <!-- O Canvas onde o jogo é desenhado -->
        <canvas id="gameCanvas"></canvas>
        
        <!-- Elemento para mostrar mensagens de placar e fim de jogo -->
        <div id="scoreDisplay" class="info-box">Pontos: 0 | Recorde: 0</div>
        
        <!-- Tela de Início/Fim de Jogo -->
        <div id="messageBox" class="message-box">
            <h2 id="messageText">Toque para Iniciar</h2>
            <p>Seja rápido! Evite os obstáculos.</p>
        </div>
    </div>
    
    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const scoreDisplay = document.getElementById('scoreDisplay');
        const messageBox = document.getElementById('messageBox');

        // Variáveis do Jogo
        let isPlaying = false;
        let score = 0;
        let highScore = parseInt(localStorage.getItem('highScore') || 0);
        let animationFrameId;

        // Variáveis do Jogador
        const playerSize = 20;
        let playerX = 50; // Posição X fixa do jogador
        let playerY = 0;
        let playerVelocity = 0;
        const gravity = 0.5;
        const jumpStrength = -10;
        let isJumping = false;
        let groundY; // Altura do chão

        // Variáveis dos Obstáculos
        let obstacles = [];
        let obstacleSpeed = 4;
        let lastObstacleTime = 0;
        const obstacleInterval = 1000; // Tempo mínimo entre obstáculos (ms)
        let lastTime = 0;

        // --- Funções de Inicialização e Redimensionamento ---

        function resizeCanvas() {
            // Garante que o Canvas preencha o container e mantenha a proporção
            const container = document.getElementById('game-container');
            const containerWidth = container.clientWidth - 40; // Ajuste para o padding
            
            // Define o tamanho baseado em uma proporção de 4:3
            canvas.width = containerWidth;
            canvas.height = (containerWidth / 4) * 3; 
            
            // Recalcula o chão
            groundY = canvas.height - playerSize;
            
            // Posiciona o jogador no chão se o jogo não estiver rodando
            if (!isPlaying) {
                playerY = groundY;
                drawStartScreen();
            }
        }

        // --- Lógica do Jogo e Desenho ---

        function drawPlayer() {
            ctx.fillStyle = '#00bcd4'; // Cor do jogador
            ctx.fillRect(playerX, playerY, playerSize, playerSize);
        }

        function updatePlayer() {
            if (isPlaying) {
                // Aplica a gravidade à velocidade vertical
                playerVelocity += gravity;
                playerY += playerVelocity;

                // Verifica se o jogador atingiu o chão
                if (playerY >= groundY) {
                    playerY = groundY;
                    playerVelocity = 0;
                    isJumping = false;
                }
            }
        }

        function jump() {
            if (!isJumping && isPlaying) {
                isJumping = true;
                playerVelocity = jumpStrength;
            }
        }

        function drawObstacles() {
            ctx.fillStyle = '#ff5722'; // Cor dos obstáculos
            obstacles.forEach(obs => {
                ctx.fillRect(obs.x, obs.y, obs.width, obs.height);
            });
        }

        function updateObstacles(deltaTime) {
            // Atualiza a velocidade dos obstáculos
            // Multiplicamos por um fator baseado no deltaTime para um movimento mais consistente em diferentes taxas de quadros
            const distanceToMove = obstacleSpeed * (deltaTime / 16); 

            for (let i = 0; i < obstacles.length; i++) {
                obstacles[i].x -= distanceToMove;
            }

            // Remove obstáculos que saíram da tela
            obstacles = obstacles.filter(obs => obs.x + obs.width > 0);
        }

        function generateObstacle(timestamp) {
            if (timestamp - lastObstacleTime > obstacleInterval) {
                const obsWidth = 15 + Math.random() * 20;
                const obsHeight = 20 + Math.random() * 30;
                
                // Define a posição y para o chão (groundY + playerSize - obsHeight)
                const obsY = groundY + playerSize - obsHeight; 

                obstacles.push({
                    x: canvas.width,
                    y: obsY,
                    width: obsWidth,
                    height: obsHeight
                });

                lastObstacleTime = timestamp;
                
                // Aumenta a dificuldade ligeiramente
                obstacleSpeed += 0.05; 
            }
        }

        function checkCollision() {
            const playerBox = {
                x: playerX,
                y: playerY,
                width: playerSize,
                height: playerSize
            };

            for (const obs of obstacles) {
                // Colisão AABB (Axis-Aligned Bounding Box)
                if (playerBox.x < obs.x + obs.width &&
                    playerBox.x + playerBox.width > obs.x &&
                    playerBox.y < obs.y + obs.height &&
                    playerBox.y + playerBox.height > obs.y) {
                    
                    gameOver();
                    return true;
                }
            }
            return false;
        }

        // --- Loop Principal do Jogo ---

        function gameLoop(timestamp) {
            if (!isPlaying) return;

            const deltaTime = timestamp - lastTime;
            lastTime = timestamp;
            if (deltaTime > 100) { // Ignora grandes saltos de tempo (ex: ao trocar de aba)
                lastTime = timestamp;
                animationFrameId = requestAnimationFrame(gameLoop);
                return;
            }

            // 1. Limpa o Canvas
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            // 2. Atualiza o Estado
            updatePlayer();
            updateObstacles(deltaTime);
            
            // 3. Verifica Colisão
            if (checkCollision()) {
                return; 
            }
            
            // 4. Desenha o Jogo
            drawPlayer();
            drawObstacles();

            // 5. Gera Obstáculos e Pontua
            generateObstacle(timestamp);
            
            // 6. Aumenta o score a cada 100ms
            if (scoreDisplay.dataset.lastScoreTime === undefined || timestamp - parseInt(scoreDisplay.dataset.lastScoreTime) > 100) {
                score++;
                updateScoreDisplay();
                scoreDisplay.dataset.lastScoreTime = timestamp;
            }

            // 7. Próximo Frame
            animationFrameId = requestAnimationFrame(gameLoop);
        }

        // --- Funções de Controle e Interface ---

        function updateScoreDisplay() {
            scoreDisplay.textContent = `Pontos: ${score} | Recorde: ${highScore}`;
        }

        function drawStartScreen() {
            // Desenha uma tela simples de início
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            ctx.font = '24px Arial';
            ctx.fillStyle = '#333';
            ctx.textAlign = 'center';
            ctx.fillText('Toque ou Pressione ESPAÇO para Jogar', canvas.width / 2, canvas.height / 2);
        }

        function startGame() {
            if (isPlaying) return;

            // Resetar variáveis
            isPlaying = true;
            score = 0;
            obstacles = [];
            
            // Posiciona o jogador no chão
            playerY = groundY; 
            playerVelocity = 0;
            isJumping = false;
            
            lastObstacleTime = 0;
            obstacleSpeed = 4;
            
            messageBox.classList.add('hidden');
            updateScoreDisplay();
            
            // Inicia o loop do jogo
            lastTime = performance.now();
            animationFrameId = requestAnimationFrame(gameLoop);
        }

        function gameOver() {
            isPlaying = false;
            cancelAnimationFrame(animationFrameId);
            
            // Atualiza o recorde
            if (score > highScore) {
                highScore = score;
                localStorage.setItem('highScore', highScore); 
            }

            document.getElementById('messageText').textContent = `FIM DE JOGO! Pontos: ${score}`;
            messageBox.querySelector('p').textContent = `Recorde: ${highScore}. Toque ou Pressione ESPAÇO para recomeçar.`;
            messageBox.classList.remove('hidden');
            updateScoreDisplay();
        }

        // --- Eventos de Interação (Toque e Teclado) ---

        function handleInteraction() {
            if (!isPlaying) {
                startGame();
            } else {
                jump();
            }
        }

        // Suporte a toque no celular (na tela inteira do Canvas)
        canvas.addEventListener('touchstart', (e) => {
            e.preventDefault(); 
            handleInteraction();
        });
        messageBox.addEventListener('touchstart', (e) => {
            e.preventDefault(); 
            handleInteraction();
        });

        // Suporte a mouse (para desktop/notebook)
        canvas.addEventListener('mousedown', handleInteraction);
        messageBox.addEventListener('mousedown', handleInteraction);

        // Suporte a teclado (barra de espaço)
        document.addEventListener('keydown', (e) => {
            if (e.code === 'Space') {
                e.preventDefault(); 
                handleInteraction();
            }
        });

        // --- Inicialização ---

        window.onload = function() {
            resizeCanvas();
            updateScoreDisplay();
            // A tela de mensagem inicial já aparece por padrão no HTML
        }
        
        // Garante que o resize funcione ao girar o celular
        window.addEventListener('resize', resizeCanvas);
    </script>
</body>
</html>


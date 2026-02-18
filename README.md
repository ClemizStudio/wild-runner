<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Wild Runner Ultra | By Clemiz Studio</title>
    <script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1.5.1/dist/confetti.browser.min.js"></script>
    <style>
        :root { --primary: #00ff88; --bg: #050505; --danger: #ff3e3e; }
        body, html { margin: 0; padding: 0; height: 100%; font-family: 'Segoe UI', sans-serif; background: #000; color: #fff; overflow: hidden; touch-action: none; }

        #overlay { position: fixed; inset: 0; background: rgba(0,0,0,0.9); z-index: 100; display: flex; flex-direction: column; align-items: center; justify-content: center; text-align: center; }
        .box { background: #111; padding: 35px; border-radius: 30px; border: 1px solid #333; width: 90%; max-width: 500px; box-shadow: 0 0 50px rgba(0,255,136,0.1); }
        h1 { color: var(--primary); font-size: 2.5rem; letter-spacing: -2px; margin: 0; }
        
        .grid-char { display: grid; grid-template-columns: repeat(4, 1fr); gap: 15px; margin: 25px 0; }
        .char-opt { background: #222; font-size: 2.2rem; padding: 15px; border-radius: 15px; cursor: pointer; border: 2px solid transparent; transition: 0.3s; }
        .char-opt.active { border-color: var(--primary); transform: scale(1.1); background: #2a2a2a; box-shadow: 0 0 20px var(--primary); }
        
        input { width: 100%; padding: 15px; background: #000; border: 1px solid #444; color: #fff; border-radius: 12px; margin-bottom: 20px; text-align: center; font-size: 1.2rem; outline: none; }
        .btn { background: var(--primary); color: #000; padding: 18px; border: none; border-radius: 50px; font-weight: 900; cursor: pointer; width: 100%; text-transform: uppercase; font-size: 1.1rem; }

        #game-container { position: relative; width: 100vw; height: 100vh; overflow: hidden; display: none; }
        #canvas { display: block; width: 100%; height: 100%; }

        #ui { position: absolute; top: 30px; width: 100%; text-align: center; pointer-events: none; z-index: 50; }
        #chrono { font-size: 4rem; font-weight: 900; color: #fff; text-shadow: 0 4px 10px rgba(0,0,0,0.5); font-family: monospace; }
        #glide-ui { width: 200px; height: 8px; background: rgba(0,0,0,0.5); margin: 10px auto; border-radius: 10px; border: 1px solid rgba(255,255,255,0.2); overflow: hidden; }
        #glide-bar { width: 100%; height: 100%; background: var(--primary); transition: width 0.1s; }

        #death-screen { position: fixed; inset: 0; background: rgba(255, 0, 0, 0.3); display: none; flex-direction: column; align-items: center; justify-content: center; z-index: 110; backdrop-filter: blur(10px); }
        footer { position: fixed; bottom: 15px; width: 100%; text-align: center; font-size: 0.8rem; opacity: 0.6; z-index: 50; }
        .btn-menu { background: #555; margin-top: 10px; color: white; }
    </style>
</head>
<body>

    <div id="overlay">
        <div class="box">
            <h1>WILD RUNNER <span style="color:#fff">ULTRA</span></h1>
            <p style="opacity:0.5; margin-bottom:20px;">BY AVRIL OLA</p>
            <div class="grid-char">
                <div class="char-opt active" onclick="setChar('🦁','🤠','#f4a460','#8b4513','savannah')">🦁</div>
                <div class="char-opt" onclick="setChar('🦊','🐺','#1a1a2e','#050505','forest')">🦊</div>
                <div class="char-opt" onclick="setChar('🐼','🐯','#e0e0e0','#a0a0a0','snow')">🐼</div>
                <div class="char-opt" onclick="setChar('🐸','🐍','#2e8b57','#1a3a2a','marsh')">🐸</div>
            </div>
            <input type="text" id="nick" placeholder="VOTRE PSEUDO" autofocus onkeydown="if(event.key === 'Enter') startApp()">
            <button class="btn" onclick="startApp()">Entrer dans l'Arène</button>
        </div>
    </div>

    <div id="ui">
        <div id="chrono">00:00:00</div>
        <div id="glide-ui"><div id="glide-bar"></div></div>
    </div>

    <div id="death-screen">
        <h1 style="font-size: 3.5rem; color:#fff;">DÉVORÉ !</h1>
        <p id="death-msg" style="font-size: 1.2rem; margin: 20px 0;"></p>
        <button class="btn" style="width:250px" onclick="resetGame()">ESSAYER ENCORE</button>
        <button class="btn btn-menu" style="width:250px" onclick="location.reload()">RETOUR MENU</button>
    </div>

    <div id="game-container">
        <canvas id="canvas"></canvas>
    </div>

    <footer>'' Tous les droits réservés - By <strong>Avril Ola</strong> ''</footer>

    <script>
        const canvas = document.getElementById('canvas');
        const ctx = canvas.getContext('2d');
        const nickInput = document.getElementById('nick');

        let char = '🦁', pred = '🤠', sky = '#f4a460', ground = '#8b4513', biome = 'savannah';
        let isRunning = false, isPressing = false, playerY = 0, velocityY = 0, jumpCount = 0, glideEnergy = 100;
        let obstacles = [], backgroundElements = [], particles = [];
        let speed = 8, startTime = 0, nextObs = 0, lastTime = 0;

        const GRAVITY = 0.8;
        const GLIDE_GRAVITY = 0.18;
        const JUMP_FORCE = -16;

        function resize() {
            canvas.width = window.innerWidth;
            canvas.height = window.innerHeight;
            if (!isRunning) playerY = canvas.height * 0.8 - 60;
        }
        window.addEventListener('resize', resize);
        resize();

        function setChar(c, p, s, g, b) {
            char = c; pred = p; sky = s; ground = g; biome = b;
            document.querySelectorAll('.char-opt').forEach(el => el.classList.remove('active'));
            event.currentTarget.classList.add('active');
        }

        function startApp() {
            if(!nickInput.value.trim()) return alert("Pseudo !");
            document.getElementById('overlay').style.display = "none";
            document.getElementById('game-container').style.display = "block";
            resetGame();
        }

        function resetGame() {
            document.getElementById('death-screen').style.display = "none";
            obstacles = []; backgroundElements = []; particles = [];
            isRunning = true; 
            playerY = canvas.height * 0.8 - 60; velocityY = 0; jumpCount = 0; glideEnergy = 100;
            speed = 8; startTime = performance.now(); lastTime = performance.now();
            for(let i=0; i<15; i++) spawnBackground(Math.random() * canvas.width);
            requestAnimationFrame(update);
        }

        function spawnBackground(x = canvas.width) {
            const biomes = {
                'savannah': ['🌵', '🏜️', '☀️', '☁️'],
                'forest': ['🌲', '🌲', '🌑', '☁️'],
                'snow': ['🏔️', '❄️', '❄️', '☁️'],
                'marsh': ['🌿', '🍄', '🌱', '🌫️']
            };
            const icons = biomes[biome];
            backgroundElements.push({
                x, y: (canvas.height * 0.8) - (20 + Math.random() * 250),
                txt: icons[Math.floor(Math.random() * icons.length)],
                size: 20 + Math.random() * 60,
                v: 0.2 + Math.random() * 0.5 
            });
        }

        function createDust(x, y) {
            for(let i=0; i<3; i++) {
                particles.push({
                    x: x + 30, y: y + 50,
                    vx: -Math.random() * 3, vy: -Math.random() * 2,
                    life: 1.0, size: 5 + Math.random() * 5
                });
            }
        }

        function update(now) {
            if (!isRunning) return;
            const dt = (now - lastTime) / 16.67;
            lastTime = now;

            ctx.clearRect(0, 0, canvas.width, canvas.height);
            let grd = ctx.createLinearGradient(0, 0, 0, canvas.height * 0.8);
            grd.addColorStop(0, sky); grd.addColorStop(1, "#333");
            ctx.fillStyle = grd; ctx.fillRect(0, 0, canvas.width, canvas.height);
            ctx.fillStyle = ground; ctx.fillRect(0, canvas.height * 0.8, canvas.width, canvas.height * 0.2);

            document.getElementById('chrono').innerText = fmt(now - startTime);
            speed += 0.001 * dt;

            if (Math.random() < 0.03) spawnBackground();
            backgroundElements.forEach((el, i) => {
                el.x -= speed * el.v * dt;
                ctx.globalAlpha = 0.4; ctx.font = `${el.size}px serif`;
                ctx.fillText(el.txt, el.x, el.y);
                if (el.x < -100) backgroundElements.splice(i, 1);
            });
            ctx.globalAlpha = 1.0;

            particles.forEach((p, i) => {
                p.x += p.vx * dt; p.y += p.vy * dt; p.life -= 0.02 * dt;
                ctx.fillStyle = `rgba(255,255,255,${p.life})`;
                ctx.beginPath(); ctx.arc(p.x, p.y, p.size, 0, Math.PI*2); ctx.fill();
                if(p.life <= 0) particles.splice(i, 1);
            });

            let curGrav = (isPressing && velocityY > 0 && glideEnergy > 0) ? GLIDE_GRAVITY : GRAVITY;
            if (isPressing && velocityY > 0 && glideEnergy > 0) {
                glideEnergy -= 1.2 * dt;
                ctx.shadowBlur = 15; ctx.shadowColor = varColor();
            } else if (playerY >= canvas.height * 0.8 - 65) {
                if(glideEnergy < 100) glideEnergy += 0.5 * dt;
                if(Math.random() > 0.5) createDust(100, playerY);
            }
            document.getElementById('glide-bar').style.width = glideEnergy + "%";

            velocityY += curGrav * dt;
            playerY += velocityY * dt;

            if (playerY > canvas.height * 0.8 - 60) {
                playerY = canvas.height * 0.8 - 60; velocityY = 0; jumpCount = 0;
            }

            ctx.font = "60px serif";
            ctx.fillText(char, 100, playerY + 55);
            ctx.shadowBlur = 0;

            if (now > nextObs) {
                obstacles.push({ x: canvas.width });
                nextObs = now + (1500 + Math.random() * 1000) / (speed/8);
            }

            obstacles.forEach((obs, i) => {
                obs.x -= speed * dt;
                ctx.font = "55px serif";
                ctx.fillText(pred, obs.x, canvas.height * 0.8 - 5);

                let pRect = { x: 115, y: playerY + 10, w: 30, h: 45 };
                let oRect = { x: obs.x + 10, y: canvas.height * 0.8 - 45, w: 35, h: 40 };

                if (pRect.x < oRect.x + oRect.w && pRect.x + pRect.w > oRect.x &&
                    pRect.y < oRect.y + oRect.h && pRect.y + pRect.h > oRect.y) {
                    gameOver(now - startTime);
                }
                if (obs.x < -100) obstacles.splice(i, 1);
            });

            requestAnimationFrame(update);
        }

        function varColor() { return char === '🦁' ? '#ffcc00' : char === '🐸' ? '#00ff88' : '#fff'; }

        function handleInput(down) {
            if (down && isRunning) {
                if (jumpCount < 2) { velocityY = JUMP_FORCE; jumpCount++; }
                isPressing = true;
            } else { isPressing = false; }
        }

        function gameOver(score) {
            isRunning = false;
            document.getElementById('death-msg').innerText = `${nickInput.value}, ton ${char} a tenu ${fmt(score)}`;
            document.getElementById('death-screen').style.display = "flex";
            confetti({ particleCount: 100, spread: 70, origin: { y: 0.6 } });
        }

        window.addEventListener('keydown', e => { 
            if(e.code === 'Space' || e.code === 'ArrowUp') { 
                e.preventDefault(); 
                if(!isPressing) handleInput(true); 
            } 
        });
        window.addEventListener('keyup', e => { if(e.code === 'Space' || e.code === 'ArrowUp') handleInput(false); });
        canvas.addEventListener('touchstart', e => { e.preventDefault(); handleInput(true); });
        canvas.addEventListener('touchend', () => handleInput(false));

        function fmt(t) {
            let m = Math.floor(t/60000), s = Math.floor((t%60000)/1000), ms = Math.floor((t%1000)/10);
            return `${m.toString().padStart(2,'0')}:${s.toString().padStart(2,'0')}:${ms.toString().padStart(2,'0')}`;
        }
    </script>
</body>
</html>

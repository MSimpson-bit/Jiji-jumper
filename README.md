<!DOCTYPE html>
<html>
<head>
    <title>Jiji Jumper: Official</title>
    <style>
        body { margin: 0; overflow: hidden; background: #0a0a0a; font-family: 'Segoe UI', sans-serif; }
        canvas { display: block; }
        #ui { 
            position: absolute; top: 20px; width: 100%; text-align: center; 
            color: white; font-size: 24px; font-weight: bold; text-shadow: 0 0 10px rgba(0,255,255,0.8);
            pointer-events: none;
        }
    </style>
</head>
<body>
<div id="ui">JIJI JUMPER - 0%</div>
<canvas id="game"></canvas>

<script>
const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");
const ui = document.getElementById("ui");

canvas.width = window.innerWidth;
canvas.height = window.innerHeight;

// --- SETTINGS ---
const GROUND_Y = canvas.height / 2 + 100;
let gameOver = false;
let speed = 9;
let gravity = 0.9;
let isHolding = false;
let progress = 0;

// --- LOAD JIJI ---
const playerImg = new Image();
playerImg.src = 'Jiji-jumper.jpg'; 

// --- GAME OBJECTS ---
let player = {
    x: 150, y: GROUND_Y - 40, size: 40,
    velocityY: 0, rotation: 0, onGround: false
};

let spikes = [{x: 900}, {x: 1400}, {x: 1445}, {x: 2000}, {x: 2600}, {x: 2645}, {x: 2690}];
let particles = [];

// --- PARTICLE SYSTEM ---
function createExplosion(x, y) {
    for (let i = 0; i < 15; i++) {
        particles.push({
            x: x, y: y,
            vx: (Math.random() - 0.5) * 15,
            vy: (Math.random() - 0.5) * 15,
            size: Math.random() * 10 + 5,
            life: 1.0
        });
    }
}

function updateParticles() {
    for (let i = particles.length - 1; i >= 0; i--) {
        let p = particles[i];
        p.x += p.vx;
        p.y += p.vy;
        p.life -= 0.02;
        if (p.life <= 0) particles.splice(i, 1);
    }
}

function drawParticles() {
    particles.forEach(p => {
        ctx.fillStyle = `rgba(0, 255, 255, ${p.life})`;
        ctx.fillRect(p.x, p.y, p.size, p.size);
    });
}

// --- CORE LOOPS ---
function update() {
    if (gameOver) {
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        drawWorld();
        updateParticles();
        drawParticles();
        requestAnimationFrame(update);
        return;
    }

    ctx.clearRect(0, 0, canvas.width, canvas.height);
    
    // Physics
    player.velocityY += gravity;
    player.y += player.velocityY;

    if (player.y > GROUND_Y - player.size) {
        player.y = GROUND_Y - player.size;
        player.velocityY = 0;
        player.onGround = true;
        if (isHolding) jump();
        player.rotation = Math.round(player.rotation / 90) * 90;
    } else {
        player.onGround = false;
        player.rotation += 6; 
    }

    // World & Progress
    progress += 0.05;
    ui.innerText = `JIJI JUMPER - ${Math.min(100, Math.floor(progress))}%`;

    drawWorld();
    
    // Spikes Logic
    spikes.forEach(s => {
        s.x -= speed;
        // Draw Spike
        ctx.fillStyle = "#ff0055";
        ctx.shadowBlur = 10; ctx.shadowColor = "#ff0055";
        ctx.beginPath();
        ctx.moveTo(s.x, GROUND_Y);
        ctx.lineTo(s.x + 20, GROUND_Y - 40);
        ctx.lineTo(s.x + 40, GROUND_Y);
        ctx.fill();
        ctx.shadowBlur = 0;

        // Hitbox Collision (Tight for fairness)
        if (player.x + player.size - 10 > s.x + 5 && 
            player.x + 10 < s.x + 35 && 
            player.y + player.size - 5 > GROUND_Y - 35) {
            die();
        }

        if (s.x < -100) s.x = canvas.width + Math.random() * 500;
    });

    drawPlayer();
    requestAnimationFrame(update);
}

function drawPlayer() {
    ctx.save();
    ctx.translate(player.x + player.size/2, player.y + player.size/2);
    ctx.rotate(player.rotation * Math.PI / 180);
    
    ctx.shadowBlur = 15;
    ctx.shadowColor = "cyan";
    
    // Draw Jiji Face
    if (playerImg.complete) {
        ctx.drawImage(playerImg, -player.size/2, -player.size/2, player.size, player.size);
    } else {
        ctx.fillStyle = "cyan";
        ctx.fillRect(-player.size/2, -player.size/2, player.size, player.size);
    }
    
    ctx.strokeStyle = "white";
    ctx.lineWidth = 2;
    ctx.strokeRect(-player.size/2, -player.size/2, player.size, player.size);
    ctx.restore();
}

function drawWorld() {
    // Background Grid (GD Style)
    ctx.strokeStyle = "#111";
    for(let i=0; i<canvas.width; i+=50) {
        ctx.beginPath(); ctx.moveTo(i - (progress*10 % 50), 0); ctx.lineTo(i - (progress*10 % 50), canvas.height); ctx.stroke();
    }
    // Ground
    ctx.fillStyle = "#050505";
    ctx.fillRect(0, GROUND_Y, canvas.width, canvas.height);
    ctx.strokeStyle = "cyan";
    ctx.lineWidth = 4;
    ctx.strokeRect(-10, GROUND_Y, canvas.width+20, 5);
}

function jump() {
    if (player.onGround) {
        player.velocityY = -15;
        player.onGround = false;
    }
}

function die() {
    if (gameOver) return;
    gameOver = true;
    createExplosion(player.x + player.size/2, player.y + player.size/2);
    setTimeout(() => location.reload(), 2000);
}

// Controls
window.addEventListener("keydown", (e) => { if(e.code === "Space") { isHolding = true; jump(); } });
window.addEventListener("keyup", (e) => { if(e.code === "Space") isHolding = false; });
window.addEventListener("mousedown", () => { isHolding = true; jump(); });
window.addEventListener("mouseup", () => isHolding = false);

update();
</script>
</body>
</html>

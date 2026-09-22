# kartik-Bhaiya
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Protect My Sister</title>
<style>
    * { box-sizing: border-box; }

    body {
        margin: 0;
        overflow: hidden;
        background: #090909;
        font-family: Arial, sans-serif;
        color: white;
    }

    canvas {
        display: block;
        background: linear-gradient(#18251c, #080b09);
    }

    #ui {
        position: fixed;
        top: 15px;
        left: 15px;
        z-index: 5;
        background: rgba(0,0,0,.65);
        padding: 12px 16px;
        border-radius: 10px;
        font-size: 16px;
    }

    #message {
        position: fixed;
        inset: 0;
        display: flex;
        align-items: center;
        justify-content: center;
        text-align: center;
        pointer-events: none;
        font-size: 42px;
        font-weight: bold;
        text-shadow: 3px 3px 8px #000;
    }

    #help {
        position: fixed;
        bottom: 12px;
        left: 50%;
        transform: translateX(-50%);
        background: rgba(0,0,0,.65);
        padding: 8px 14px;
        border-radius: 8px;
        font-size: 14px;
    }
</style>
</head>
<body>

<canvas id="game"></canvas>

<div id="ui">
    ❤️ Brother: <span id="hp">100</span>
    &nbsp; | &nbsp;
    👧 Sister: <span id="sisterHp">100</span>
    &nbsp; | &nbsp;
    🧟 Zombies: <span id="score">0</span>
</div>

<div id="message"></div>

<div id="help">
    WASD / Arrow Keys = Move &nbsp; | &nbsp; Mouse = Aim &nbsp; | &nbsp; Click = Shoot
</div>

<script>
const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");

canvas.width = innerWidth;
canvas.height = innerHeight;

window.addEventListener("resize", () => {
    canvas.width = innerWidth;
    canvas.height = innerHeight;
});

const keys = {};

addEventListener("keydown", e => {
    keys[e.key.toLowerCase()] = true;

    if (gameOver && e.key.toLowerCase() === "r") {
        location.reload();
    }
});

addEventListener("keyup", e => {
    keys[e.key.toLowerCase()] = false;
});

let mouse = {
    x: canvas.width / 2,
    y: canvas.height / 2,
    down: false
};

canvas.addEventListener("mousemove", e => {
    mouse.x = e.clientX;
    mouse.y = e.clientY;
});

canvas.addEventListener("mousedown", () => mouse.down = true);
canvas.addEventListener("mouseup", () => mouse.down = false);

const brother = {
    x: canvas.width / 2,
    y: canvas.height / 2 + 100,
    r: 18,
    speed: 4,
    hp: 100,
    cooldown: 0
};

const sister = {
    x: canvas.width / 2,
    y: canvas.height / 2,
    r: 17,
    hp: 100
};

let bullets = [];
let zombies = [];
let score = 0;
let spawnTimer = 0;
let gameOver = false;

function distance(a,b) {
    return Math.hypot(a.x-b.x, a.y-b.y);
}

function spawnZombie() {
    const side = Math.floor(Math.random()*4);
    let x, y;

    if (side === 0) {
        x = Math.random()*canvas.width;
        y = -30;
    } else if (side === 1) {
        x = canvas.width+30;
        y = Math.random()*canvas.height;
    } else if (side === 2) {
        x = Math.random()*canvas.width;
        y = canvas.height+30;
    } else {
        x = -30;
        y = Math.random()*canvas.height;
    }

    zombies.push({
        x,
        y,
        r: 17,
        speed: 0.65 + Math.random()*0.7,
        hp: 2,
        attackCooldown: 0
    });
}

function shoot() {
    if (brother.cooldown > 0) return;

    const angle = Math.atan2(
        mouse.y - brother.y,
        mouse.x - brother.x
    );

    bullets.push({
        x: brother.x,
        y: brother.y,
        dx: Math.cos(angle)*10,
        dy: Math.sin(angle)*10,
        r: 4
    });

    brother.cooldown = 10;
}

function update() {
    if (gameOver) return;

    // Brother movement
    let dx = 0;
    let dy = 0;

    if (keys["w"] || keys["arrowup"]) dy--;
    if (keys["s"] || keys["arrowdown"]) dy++;
    if (keys["a"] || keys["arrowleft"]) dx--;
    if (keys["d"] || keys["arrowright"]) dx++;

    if (dx || dy) {
        const len = Math.hypot(dx,dy);
        brother.x += dx/len * brother.speed;
        brother.y += dy/len * brother.speed;
    }

    brother.x = Math.max(20, Math.min(canvas.width-20, brother.x));
    brother.y = Math.max(20, Math.min(canvas.height-20, brother.y));

    if (mouse.down) shoot();

    if (brother.cooldown > 0) brother.cooldown--;

    // Bullets
    bullets.forEach(b => {
        b.x += b.dx;
        b.y += b.dy;
    });

    bullets = bullets.filter(b =>
        b.x > -20 && b.x < canvas.width+20 &&
        b.y > -20 && b.y < canvas.height+20
    );

    // Spawn zombies
    spawnTimer--;

    if (spawnTimer <= 0) {
        spawnZombie();

        // Gets harder over time
        spawnTimer = Math.max(15, 55 - score * 0.5);
    }

    // Zombies
    zombies.forEach(z => {

        const target = distance(z, sister) < distance(z, brother)
            ? sister
            : brother;

        const angle = Math.atan2(target.y-z.y, target.x-z.x);

        z.x += Math.cos(angle)*z.speed;
        z.y += Math.sin(angle)*z.speed;

        if (z.attackCooldown > 0) z.attackCooldown--;

        if (distance(z, sister) < z.r + sister.r) {
            if (z.attackCooldown <= 0) {
                sister.hp -= 8;
                z.attackCooldown = 45;
            }
        }

        if (distance(z, brother) < z.r + brother.r) {
            if (z.attackCooldown <= 0) {
                brother.hp -= 6;
                z.attackCooldown = 45;
            }
        }
    });

    // Bullet collision
    for (let i = zombies.length-1; i >= 0; i--) {
        for (let j = bullets.length-1; j >= 0; j--) {

            if (distance(zombies[i], bullets[j]) <
                zombies[i].r + bullets[j].r) {

                zombies[i].hp--;
                bullets.splice(j,1);

                if (zombies[i].hp <= 0) {
                    zombies.splice(i,1);
                    score++;
                }

                break;
            }
        }
    }

    if (brother.hp <= 0 || sister.hp <= 0) {
        gameOver = true;

        document.getElementById("message").innerHTML =
            sister.hp <= 0
            ? "💀 Your sister was lost<br><small>Press R to restart</small>"
            : "💀 You were overwhelmed<br><small>Press R to restart</small>";
    }

    document.getElementById("hp").textContent =
        Math.max(0, Math.floor(brother.hp));

    document.getElementById("sisterHp").textContent =
        Math.max(0, Math.floor(sister.hp));

    document.getElementById("score").textContent = score;
}

function drawBackground() {
    ctx.fillStyle = "#101711";
    ctx.fillRect(0,0,canvas.width,canvas.height);

    // Ground grid
    ctx.strokeStyle = "rgba(255,255,255,.035)";
    ctx.lineWidth = 1;

    for (let x=0; x<canvas.width; x+=50) {
        ctx.beginPath();
        ctx.moveTo(x,0);
        ctx.lineTo(x,canvas.height);
        ctx.stroke();
    }

    for (let y=0; y<canvas.height; y+=50) {
        ctx.beginPath();
        ctx.moveTo(0,y);
        ctx.lineTo(canvas.width,y);
        ctx.stroke();
    }

    // Safe zone around sister
    ctx.beginPath();
    ctx.arc(sister.x,sister.y,100,0,Math.PI*2);
    ctx.strokeStyle = "rgba(80,255,120,.15)";
    ctx.stroke();
}

function drawCharacter(c, type) {
    ctx.save();
    ctx.translate(c.x,c.y);

    if (type === "brother") {
        // Body
        ctx.fillStyle = "#3567b7";
        ctx.fillRect(-13,-8,26,30);

        // Head
        ctx.fillStyle = "#d99b72";
        ctx.beginPath();
        ctx.arc(0,-18,12,0,Math.PI*2);
        ctx.fill();

        // Hair
        ctx.fillStyle = "#17120e";
        ctx.beginPath();
        ctx.arc(0,-22,12,Math.PI,Math.PI*2);
        ctx.fill();

        // Gun
        const angle = Math.atan2(mouse.y-c.y, mouse.x-c.x);
        ctx.rotate(angle);
        ctx.fillStyle = "#444";
        ctx.fillRect(8,-4,24,7);
    }

    if (type === "sister") {
        // Dress
        ctx.fillStyle = "#e7b6d8";
        ctx.beginPath();
        ctx.moveTo(-15,8);
        ctx.lineTo(15,8);
        ctx.lineTo(23,34);
        ctx.lineTo(-23,34);
        ctx.closePath();
        ctx.fill();

        // Head
        ctx.fillStyle = "#d99b72";
        ctx.beginPath();
        ctx.arc(0,-12,11,0,Math.PI*2);
        ctx.fill();

        // Hair
        ctx.fillStyle = "#281812";
        ctx.beginPath();
        ctx.arc(0,-16,12,Math.PI,Math.PI*2);
        ctx.fill();

        // Exclamation
        ctx.fillStyle = "white";
        ctx.font = "bold 18px Arial";
        ctx.textAlign = "center";
        ctx.fillText("!",0,-38);
    }

    ctx.restore();
}

function drawZombie(z) {
    ctx.save();
    ctx.translate(z.x,z.y);

    // Body
    ctx.fillStyle = "#5d8a55";
    ctx.fillRect(-13,-3,26,25);

    // Head
    ctx.fillStyle = "#759b68";
    ctx.beginPath();
    ctx.arc(0,-13,13,0,Math.PI*2);
    ctx.fill();

    // Eyes
    ctx.fillStyle = "#ff2222";
    ctx.beginPath();
    ctx.arc(-5,-15,2.5,0,Math.PI*2);
    ctx.arc(5,-15,2.5,0,Math.PI*2);
    ctx.fill();

    // Mouth
    ctx.fillStyle = "#210909";
    ctx.fillRect(-7,-7,14,4);

    ctx.restore();
}

function draw() {
    drawBackground();

    // Bullets
    ctx.fillStyle = "#ffd84a";

    bullets.forEach(b => {
        ctx.beginPath();
        ctx.arc(b.x,b.y,b.r,0,Math.PI*2);
        ctx.fill();
    });

    // Sister
    drawCharacter(sister,"sister");

    // Brother
    drawCharacter(brother,"brother");

    // Zombies
    zombies.forEach(drawZombie);

    // Vignette
    const gradient = ctx.createRadialGradient(
        canvas.width/2,
        canvas.height/2,
        100,
        canvas.width/2,
        canvas.height/2,
        Math.max(canvas.width,canvas.height)/1.2
    );

    gradient.addColorStop(0,"rgba(0,0,0,0)");
    gradient.addColorStop(1,"rgba(0,0,0,.55)");

    ctx.fillStyle = gradient;
    ctx.fillRect(0,0,canvas.width,canvas.height);
}

function loop() {
    update();
    draw();
    requestAnimationFrame(loop);
}

loop();
</script>

</body>
</html>
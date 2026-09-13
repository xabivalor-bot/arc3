# AmazingRetroCar-advance
from IPython.display import HTML, display

display(HTML(r"""
<style>
#retro-game-container {
    width: 420px;
    max-width: 100%;
    margin: auto;
    font-family: monospace;
    user-select: none;
}

#retro-game-container canvas {
    width: 400px;
    max-width: 100%;
    display: block;
    margin: auto;
    image-rendering: pixelated;
    border: 4px solid #222;
}

#controls {
    width: 400px;
    max-width: 100%;
    margin: 12px auto 0;
    display: flex;
    justify-content: center;
    gap: 25px;
}

.control-btn {
    width: 150px;
    height: 65px;
    background: #e32626;
    color: white;
    border: 4px solid #8d1111;
    border-radius: 12px;
    font-size: 38px;
    font-weight: bold;
    cursor: pointer;
    touch-action: none;
}

.control-btn:active {
    background: #a91515;
    transform: translateY(2px);
}
</style>

<div id="retro-game-container">

<canvas id="game" width="400" height="700"></canvas>

<div id="controls">
    <button class="control-btn" id="left">⇠</button>
    <button class="control-btn" id="right">⇢</button>
</div>

</div>

<script>
(() => {

const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");

ctx.imageSmoothingEnabled = false;

// =================================================
// GAME SETTINGS
// =================================================

const W = 400;
const H = 700;

const roadLeft = 35;
const roadRight = 365;

const lanes = 5;
const laneWidth = (roadRight - roadLeft) / lanes;

let playerLane = 2;
let playerX = roadLeft + laneWidth * 2.5;

const playerY = 570;

let lives = 3;
let score = 0;

let gameOver = false;

let roadOffset = 0;

let speed = 4;

let leftPressed = false;
let rightPressed = false;

let cars = [];
let trees = [];


// =================================================
// RETRO TEXT
// =================================================

function text(txt, x, y, size, color="#ffffff", align="left") {

    ctx.font = `bold ${size}px monospace`;
    ctx.fillStyle = color;
    ctx.textAlign = align;
    ctx.textBaseline = "middle";

    ctx.fillText(txt, x, y);
}


// =================================================
// RANDOM TREE
// =================================================

function makeTree(x, y) {

    return {
        x: x,
        y: y,
        size: 20 + Math.random() * 12
    };

}


// =================================================
// CREATE TREES
// =================================================

for(let i=0; i<18; i++) {

    trees.push(
        makeTree(
            Math.random() < 0.5
                ? 12 + Math.random()*15
                : 372 + Math.random()*15,

            Math.random()*700
        )
    );

}


// =================================================
// CREATE TRAFFIC CAR
// =================================================

function createCar(y) {

    let lane = Math.floor(Math.random()*lanes);

    return {
        lane: lane,
        x: roadLeft + laneWidth*(lane+0.5),
        y: y,
        speed: 2.5 + Math.random()*2,
        width: 42,
        height: 75
    };

}


// Initial traffic
for(let i=0; i<7; i++) {
    cars.push(createCar(-100 - i*150));
}


// =================================================
// DRAW TREE
// =================================================

function drawTree(tree) {

    let x = tree.x;
    let y = tree.y;
    let s = tree.size;

    // trunk
    ctx.fillStyle = "#714321";
    ctx.fillRect(x-4, y+10, 8, 18);

    // leaves
    ctx.fillStyle = "#3c7d32";
    ctx.fillRect(x-s/2, y-s/2, s, s);

    ctx.fillStyle = "#659b43";
    ctx.fillRect(x-s/3, y-s*0.7, s*0.65, s*0.5);

    // pixel highlights
    ctx.fillStyle = "#8aaa55";
    ctx.fillRect(x-s/3, y-s/3, 5, 5);
}


// =================================================
// DRAW CAR
// =================================================

function drawCar(x, y, color, player=false) {

    const w = 42;
    const h = 75;

    // shadow
    ctx.fillStyle = "#222";
    ctx.fillRect(x-w/2-3, y-h/2+4, w+6, h);

    // body
    ctx.fillStyle = color;
    ctx.fillRect(x-w/2, y-h/2, w, h);

    // hood / rear
    ctx.fillStyle =
        color === "#e32626"
        ? "#b91616"
        : "#d0d0d0";

    ctx.fillRect(x-w/2+4, y-h/2+5, w-8, 12);

    // windows
    ctx.fillStyle = "#26343b";

    ctx.fillRect(
        x-w/2+7,
        y-h/2+20,
        w-14,
        17
    );

    ctx.fillRect(
        x-w/2+7,
        y-h/2+42,
        w-14,
        11
    );

    // window shine
    ctx.fillStyle = "#65777c";

    ctx.fillRect(
        x-w/2+9,
        y-h/2+22,
        5,
        12
    );

    // wheels
    ctx.fillStyle = "#111";

    ctx.fillRect(
        x-w/2-5,
        y-h/2+12,
        7,
        18
    );

    ctx.fillRect(
        x+w/2-2,
        y-h/2+12,
        7,
        18
    );

    ctx.fillRect(
        x-w/2-5,
        y+h/2-27,
        7,
        18
    );

    ctx.fillRect(
        x+w/2-2,
        y+h/2-27,
        7,
        18
    );

    // headlights / tail lights
    if(player) {

        ctx.fillStyle = "#ffdddd";

        ctx.fillRect(
            x-w/2+5,
            y+h/2-12,
            8,
            5
        );

        ctx.fillRect(
            x+w/2-13,
            y+h/2-12,
            8,
            5
        );

    } else {

        ctx.fillStyle = "#ff4444";

        ctx.fillRect(
            x-w/2+5,
            y-h/2+3,
            8,
            5
        );

        ctx.fillRect(
            x+w/2-13,
            y-h/2+3,
            8,
            5
        );
    }

}


// =================================================
// DRAW ROAD
// =================================================

function drawRoad() {

    // grass
    ctx.fillStyle = "#789b49";
    ctx.fillRect(0,0,W,H);

    // grass pattern
    for(let y=-20; y<H; y+=25) {

        ctx.fillStyle = "#92aa5b";

        ctx.fillRect(8, y, 4, 12);
        ctx.fillRect(20, y+7, 3, 10);

        ctx.fillRect(377, y+3, 4, 12);
        ctx.fillRect(390, y+12, 3, 8);
    }

    // road
    ctx.fillStyle = "#777";
    ctx.fillRect(
        roadLeft,
        0,
        roadRight-roadLeft,
        H
    );

    // road darker strips
    ctx.fillStyle = "#696969";

    ctx.fillRect(roadLeft,0,5,H);
    ctx.fillRect(roadRight-5,0,5,H);

    // lane separators
    ctx.fillStyle = "#eeeeee";

    for(let i=1; i<lanes; i++) {

        let x = roadLeft + laneWidth*i;

        for(let y=-60 + roadOffset; y<H; y+=75) {

            ctx.fillRect(
                x-2,
                y,
                4,
                35
            );
        }
    }

    // road edges
    ctx.fillStyle = "#dddddd";

    ctx.fillRect(roadLeft,0,3,H);
    ctx.fillRect(roadRight-3,0,3,H);
}


// =================================================
// UI
// =================================================

function drawUI() {

    // top retro panel
    ctx.fillStyle = "#4c82d8";
    ctx.fillRect(0,0,W,70);

    text(
        "AmazingRetroCar",
        15,
        25,
        19,
        "#ffffff"
    );

    text(
        "SCORE " + score,
        W-15,
        25,
        18,
        "#ffffff",
        "right"
    );

    // lives
    text(
        "♥ ♥ ♥".slice(0, lives*2-1),
        15,
        52,
        19,
        "#ff3333"
    );

    text(
        "GameByZaheen",
        W-15,
        52,
        13,
        "#ffffff",
        "right"
    );
}


// =================================================
// COLLISION
// =================================================

function collision(a,b) {

    return (
        Math.abs(a.x-b.x) < 34 &&
        Math.abs(a.y-b.y) < 55
    );

}


// =================================================
// UPDATE
// =================================================

function update() {

    if(gameOver) return;

    roadOffset += speed;

    if(roadOffset > 75)
        roadOffset -= 75;


    // steering
    if(leftPressed) {

        playerX -= 5;

    }

    if(rightPressed) {

        playerX += 5;

    }


    // keep player on road
    playerX = Math.max(
        roadLeft + 25,
        Math.min(roadRight - 25, playerX)
    );


    // traffic
    for(let car of cars) {

        car.y += car.speed + speed*0.45;

        // passed car
        if(car.y > H+100) {

            car.y = -100 - Math.random()*300;

            car.lane = Math.floor(Math.random()*lanes);

            car.x =
                roadLeft +
                laneWidth*(car.lane+0.5);

            score += 10;

            // gradually increase speed
            speed = Math.min(
                9,
                4 + score/250
            );
        }


        // collision
        if(
            collision(
                {x:playerX,y:playerY},
                car
            )
        ) {

            lives--;

            // move crashed car away
            car.y = -200 - Math.random()*300;

            car.lane = Math.floor(Math.random()*lanes);

            car.x =
                roadLeft +
                laneWidth*(car.lane+0.5);

            playerX = W/2;

            if(lives <= 0) {

                gameOver = true;

            }
        }
    }


    // trees
    for(let tree of trees) {

        tree.y += speed;

        if(tree.y > H+50) {

            tree.y = -50;

            tree.x =
                Math.random() < 0.5
                ? 12 + Math.random()*15
                : 372 + Math.random()*15;
        }
    }
}


// =================================================
// DRAW
// =================================================

function draw() {

    ctx.clearRect(0,0,W,H);

    drawRoad();


    // trees
    for(let tree of trees)
        drawTree(tree);


    // traffic
    for(let car of cars)
        drawCar(car.x,car.y,"#f5f5f5");


    // player
    if(!gameOver) {

        drawCar(
            playerX,
            playerY,
            "#e32626",
            true
        );

    }


    drawUI();


    // GAME OVER
    if(gameOver) {

        ctx.fillStyle = "rgba(0,0,0,0.72)";
        ctx.fillRect(0,0,W,H);

        text(
            "GAME OVER",
            W/2,
            280,
            38,
            "#ff3333",
            "center"
        );

        text(
            "SCORE: " + score,
            W/2,
            330,
            24,
            "#ffffff",
            "center"
        );

        text(
            "TAP TO RESTART",
            W/2,
            385,
            19,
            "#ffff55",
            "center"
        );
    }
}


// =================================================
// GAME LOOP
// =================================================

function loop() {

    update();
    draw();

    requestAnimationFrame(loop);

}

loop();


// =================================================
// TOUCH CONTROLS
// =================================================

function holdButton(button, direction) {

    button.addEventListener("pointerdown", e => {

        e.preventDefault();

        if(direction === "left")
            leftPressed = true;
        else
            rightPressed = true;
    });


    button.addEventListener("pointerup", e => {

        e.preventDefault();

        if(direction === "left")
            leftPressed = false;
        else
            rightPressed = false;
    });


    button.addEventListener("pointerleave", () => {

        if(direction === "left")
            leftPressed = false;
        else
            rightPressed = false;
    });
}


holdButton(
    document.getElementById("left"),
    "left"
);

holdButton(
    document.getElementById("right"),
    "right"
);


// =================================================
// KEYBOARD
// =================================================

document.addEventListener("keydown", e => {

    if(e.key === "ArrowLeft")
        leftPressed = true;

    if(e.key === "ArrowRight")
        rightPressed = true;

    if(e.key.toLowerCase() === "r" && gameOver)
        restart();
});


document.addEventListener("keyup", e => {

    if(e.key === "ArrowLeft")
        leftPressed = false;

    if(e.key === "ArrowRight")
        rightPressed = false;
});


// =================================================
// RESTART
// =================================================

function restart() {

    lives = 3;
    score = 0;
    speed = 4;

    playerX = W/2;

    gameOver = false;

    cars = [];

    for(let i=0; i<7; i++)
        cars.push(createCar(-100-i*150));
}


// tap canvas to restart
canvas.addEventListener("pointerdown", () => {

    if(gameOver)
        restart();

});

})();
</script>
"""))

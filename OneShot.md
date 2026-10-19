# One Shot

## Subway runner



I created this miniature game that is inspired by the famous game subway surfers. I sent this prompt to chatgpt who created the html that includes the javascript code. I analyzed the code through tracing variables and understanding the physics of motion in a moving environment. I looked at the game set up–colors, borders, shapes–before looking at the actual functionality (do whiles, if statements, for loops). I noticed the recursion in the code that minimizes computer energy yet boosts productivity. The user input (either slide, jump, or change track), was crucial to my understanding the general mechanics of the code and recognizing that each input is a function by itself. This game is an accurate representation of language processing for machines because a seemingly simple input is linked to many logical structures that lengthen the code. 



**Code:**

```javascript
// Metro Runner: all game logic and drawing live in this file.

// The player stays near the bottom while obstacles move toward them.

const canvas = document.getElementById('game');

const ctx = canvas.getContext('2d');

const scoreText = document.getElementById('score');

const coinText = document.getElementById('coins');

const bestText = document.getElementById('best');

const overlay = document.getElementById('overlay');

const headline = document.getElementById('headline');

const message = document.getElementById('message');

const startButton = document.getElementById('start');

const WIDTH = canvas.width, HEIGHT = canvas.height;

const LANES = [210, 360, 510];

const PLAYER_Y = 427;

const GRAVITY = 1650;         // Pixels per second squared

const JUMP_VELOCITY = 650;    // Initial upward speed

let mode = 'ready', lane = 1, visualX = LANES[1];

let jumpHeight = 0, verticalVelocity = 0, slideTime = 0;

let score = 0, coins = 0, best = 0, distance = 0;

let obstacles = [], spawnTimer = 0, lastTime = 0;

let worldSpeed = 210;



function resetGame() {

  mode = 'playing'; lane = 1; visualX = LANES[1];

  jumpHeight = 0; verticalVelocity = 0; slideTime = 0;

  score = 0; coins = 0; distance = 0; obstacles = [];

  spawnTimer = 0.8; worldSpeed = 210;

  overlay.classList.add('hidden');

}



function move(action) {

  if (action === 'pause') {

    if (mode === 'playing') mode = 'paused';

    else if (mode === 'paused') mode = 'playing';

    return;

  }

  if (mode !== 'playing') return;

  if (action === 'left') lane = Math.max(0, lane - 1);

  if (action === 'right') lane = Math.min(2, lane + 1);

  if (action === 'jump' && jumpHeight === 0) {

    verticalVelocity = JUMP_VELOCITY;

    slideTime = 0;

  }

  if (action === 'slide') {

    slideTime = 0.7;

    if (jumpHeight > 0) verticalVelocity = -JUMP_VELOCITY;

  }

}



// Objects spawn above the horizon and travel toward the player.

function spawnRow() {

  const safeLane = Math.floor(Math.random() * 3);

  const hazardLanes = [0, 1, 2].filter(l => l !== safeLane);

  const count = Math.random() < 0.4 ? 2 : 1;

  for (let i = 0; i < count; i++) {

    const types = ['train', 'barrier', 'sign'];

    obstacles.push({lane: hazardLanes[i], depth: 0, type: types[Math.floor(Math.random() * types.length)]});

  }

  for (let i = 0; i < 4; i++) obstacles.push({lane: safeLane, depth: -0.17 - i * 0.12, type: 'coin'});

}



// depth=0 is far away; depth=1 is the player's position.

function projectedPosition(object) {

  const depth = Math.max(0, object.depth);

  const perspective = depth * depth;

  return {x: WIDTH / 2 + (LANES[object.lane] - WIDTH / 2) * (0.18 + 0.82 * perspective),

          y: 138 + perspective * (PLAYER_Y - 138),

          scale: 0.18 + perspective * 0.82};

}



function update(dt) {

  if (mode !== 'playing') return;

  distance += worldSpeed * dt;

  worldSpeed = Math.min(390, 210 + distance * 0.012);

  score += dt * worldSpeed * 0.06;

  visualX += (LANES[lane] - visualX) * Math.min(1, 13 * dt);

  // Euler integration: velocity changes due to gravity, then position changes due to velocity.

  if (jumpHeight > 0 || verticalVelocity > 0) {

    jumpHeight += verticalVelocity * dt;

    verticalVelocity -= GRAVITY * dt;

    if (jumpHeight <= 0) {jumpHeight = 0; verticalVelocity = 0;}

  }

  slideTime = Math.max(0, slideTime - dt);

  spawnTimer -= dt;

  if (spawnTimer <= 0) {spawnRow(); spawnTimer = Math.max(0.82, 1.65 - distance / 12000);}

  for (const object of obstacles) {

    object.depth += dt * worldSpeed / 480;

    if (!object.checked && object.depth >= 0.93) {

      object.checked = true;

      if (object.lane !== lane) continue;

      if (object.type === 'coin') {coins++; score += 30;}

      else if (object.type === 'train' || (object.type === 'barrier' && jumpHeight < 55) || (object.type === 'sign' && slideTime <= 0)) endGame();

    }

  }

  obstacles = obstacles.filter(o => o.depth < 1.2);

  scoreText.textContent = Math.floor(score);

  coinText.textContent = coins;

}



function endGame() {

  mode = 'over';

  best = Math.max(best, Math.floor(score));

  bestText.textContent = best;

  headline.textContent = 'Game Over';

  message.textContent = `Score: ${Math.floor(score)} · Coins: ${coins}. Try to beat your best run!`;

  startButton.textContent = 'Play Again';

  overlay.classList.remove('hidden');

}



function box(x, y, w, h, color) {ctx.fillStyle = color; ctx.fillRect(x, y, w, h);}

function line(x1, y1, x2, y2, color, width = 3) {

  ctx.strokeStyle = color; ctx.lineWidth = width;

  ctx.beginPath(); ctx.moveTo(x1, y1); ctx.lineTo(x2, y2); ctx.stroke();

}

function drawBackground() {

  const sky = ctx.createLinearGradient(0, 0, 0, HEIGHT);

  sky.addColorStop(0, '#24396e'); sky.addColorStop(0.55, '#eb8a83'); sky.addColorStop(1, '#ffe1a4');

  box(0, 0, WIDTH, HEIGHT, sky);

  ctx.fillStyle = '#ffdf9b'; ctx.beginPath(); ctx.arc(560, 90, 34, 0, Math.PI * 2); ctx.fill();

  for (let i = 0; i < 17; i++) {

    const x = i * 46 - 20, h = 65 + (i * 31 % 78);

    box(x, 138 - h, 39, h, '#30426b');

    for (let j = 0; j < 4; j++) box(x + 9 + j * 8, 148 - h, 4, 6, '#ffd98c');

  }

  // Trapezoid track creates a perspective illusion.

  ctx.fillStyle = '#515d70';ctx.beginPath();ctx.moveTo(310,138);ctx.lineTo(410,138);ctx.lineTo(700,HEIGHT);ctx.lineTo(20,HEIGHT);ctx.fill();

  for (let l = 0; l < 3; l++) {

    const x = LANES[l];

    line(360+(x-360)*0.18-20,138,x-28,HEIGHT,'#c4d1d9',5);

    line(360+(x-360)*0.18+20,138,x+28,HEIGHT,'#c4d1d9',5);

  }

  for (let z = (distance % 60) / 60; z < 1; z += 0.12) {

    const y = 138 + z*z*(HEIGHT-138);

    line(360-z*z*320,y,360+z*z*320,y,'#394657',2+z*5);

  }

}

function drawObject(o) {

  if (o.depth < 0) return;

  const p = projectedPosition(o), s = p.scale;

  ctx.save(); ctx.translate(p.x,p.y);ctx.scale(s,s);

  if (o.type === 'coin') {

    ctx.fillStyle='#ffd448';ctx.beginPath();ctx.ellipse(0,-45,18,23,0,0,Math.PI*2);ctx.fill();

    ctx.strokeStyle='#fff4b4';ctx.lineWidth=4;ctx.stroke();

    ctx.fillStyle='#a66d0a';ctx.font='bold 24px Arial';ctx.textAlign='center';ctx.fillText('$',0,-37);

  } else if (o.type === 'train') {

    box(-55,-165,110,165,'#e64c57');box(-44,-148,88,77,'#8bd2f1');box(-37,-140,74,61,'#294c74');

    box(-55,-58,110,15,'#f9cc58');box(-40,-28,22,13,'#fff0a3');box(18,-28,22,13,'#fff0a3');

  } else if (o.type === 'barrier') {

    box(-56,-64,112,64,'#ffbe3b');box(-60,-70,120,16,'#273b55');

    for (let i=0;i<4;i++) {ctx.fillStyle='#f9f5e8';ctx.beginPath();ctx.moveTo(-55+i*30,-54);ctx.lineTo(-36+i*30,-54);ctx.lineTo(-55+i*30,-9);ctx.lineTo(-74+i*30,-9);ctx.fill();}

  } else if (o.type === 'sign') {

    box(-58,-165,12,165,'#91a9b5');box(46,-165,12,165,'#91a9b5');box(-64,-160,128,55,'#d95380');

    ctx.fillStyle='white';ctx.font='bold 23px Arial';ctx.textAlign='center';ctx.fillText('SLIDE',0,-124);

  }

  ctx.restore();

}

function drawPlayer(time) {

  const x = visualX, y = PLAYER_Y - jumpHeight;

  const step = Math.sin(time / 90) * (mode === 'playing' ? 11 : 0);

  ctx.fillStyle='#0006';ctx.beginPath();ctx.ellipse(x,PLAYER_Y+16,31,9,0,0,Math.PI*2);ctx.fill();

  ctx.save();ctx.translate(x,y);

  if (slideTime > 0) {ctx.translate(0,23);ctx.scale(1,0.55);}

  ctx.lineCap='round';ctx.strokeStyle='#24314c';ctx.lineWidth=12;

  ctx.beginPath();ctx.moveTo(-11,-38);ctx.lineTo(-16+step,0);ctx.moveTo(11,-38);ctx.lineTo(16-step,0);ctx.stroke();

  ctx.strokeStyle='#ffb98e';ctx.lineWidth=10;

  ctx.beginPath();ctx.moveTo(-19,-93);ctx.lineTo(-27-step/2,-51);ctx.moveTo(19,-93);ctx.lineTo(27+step/2,-51);ctx.stroke();

  box(-22,-105,44,67,'#42d5bb');box(-17,-101,34,9,'#c0fff0');

  ctx.fillStyle='#ffbb90';ctx.beginPath();ctx.arc(0,-123,23,0,Math.PI*2);ctx.fill();

  ctx.fillStyle='#283452';ctx.beginPath();ctx.arc(0,-130,24,Math.PI,2*Math.PI);ctx.fill();

  box(-26,-136,52,9,'#fa5c7e');box(-10,-124,5,4,'#25334c');box(7,-124,5,4,'#25334c');

  ctx.restore();

}

function draw(time) {

  drawBackground();

  [...obstacles].sort((a,b)=>a.depth-b.depth).forEach(drawObject);

  drawPlayer(time);

  if (mode === 'paused') {

    box(0,0,WIDTH,HEIGHT,'#101a31aa');ctx.fillStyle='white';ctx.textAlign='center';ctx.font='bold 45px Arial';ctx.fillText('PAUSED',WIDTH/2,HEIGHT/2);

  }

}

function frame(time) {

  const dt = Math.min((time-lastTime)/1000 || 0,0.04);

  lastTime=time;update(dt);draw(time);requestAnimationFrame(frame);

}

startButton.addEventListener('click',resetGame);

document.querySelectorAll('[data-action]').forEach(button=>button.addEventListener('click',()=>move(button.dataset.action)));

document.addEventListener('keydown',event=>{

  const actions={ArrowLeft:'left',a:'left',ArrowRight:'right',d:'right',ArrowUp:'jump',w:'jump',' ':'jump',ArrowDown:'slide',s:'slide',p:'pause'};

  if (actions[event.key]) {event.preventDefault();move(actions[event.key]);}

  if (event.key==='Enter' && (mode==='ready'||mode==='over')) resetGame();

});

let touchX=0,touchY=0;

canvas.addEventListener('touchstart',e=>{touchX=e.touches[0].clientX;touchY=e.touches[0].clientY;},{passive:true});

canvas.addEventListener('touchend',e=>{

  const dx=e.changedTouches[0].clientX-touchX,dy=e.changedTouches[0].clientY-touchY;

  if(Math.max(Math.abs(dx),Math.abs(dy))<20)return;

  move(Math.abs(dx)>Math.abs(dy)?(dx>0?'right':'left'):(dy<0?'jump':'slide'));

},{passive:true});

requestAnimationFrame(frame);
```

### Code Analysis

- **Canvas:** The HTML contains a `<canvas>` element with a width of 800 pixels and a height of 640 pixels. JavaScript uses this canvas to draw the tracks, player, and obstacles. Changing its dimensions would affect the playing area that is visible to the players and potentially create graphic distortion if the drawing coordinates were not adjusted to fit the page. 
- **State Variables:** The language JavaScript tracks values such as `lane`, `jump`, `jumpVel`, `speed`, `score`, `coins`, and `state`. These determine the player's position, vertical movement, difficulty, and whether the game is running or over.
- **Game Loop:** The `requestAnimationFrame(loop)` function continuously updates and redraws the game, typically around 60 times per second. Each cycle calls `update(dt)` to calculate movement and check collisions, followed by `render()` to display the updated scene/number of coins you earned at the end. 
- **Input Handling:** A `keydown` event listener detects arrow keys. Left and right change the player's lane, while up initiates a jump by assigning an upward velocity. Down activates sliding for a limited duration which is at 23 but if i change it to 50 it lasts for longer
- **Collision Detection:** The code checks whether an obstacle is close to the player (`o.z < 7`) and occupies the same lane (`o.lane === lane`). A collision ends the game unless the player jumps high enough to clear a barrier or slides beneath a sign.
- **Scoring:** The `score` variable increases as the player travels farther, while collecting a coin adds 25 points. The game also increases its speed as the score rises, making obstacles progressively harder to avoid. It reaches a maximum of 2.4 times its initial speed, which can be changed by simply adjusting the value to 5 to increase speed. 



## Flappy Bird



I also created a flappy bird game by sending the prompt to chatgpt who created the html that includes the javascript code. After getting the game to run, I went through the code to figure out how the bird moves, how the pipes appear and move across the screen, and how the score is calculated. I paid particular attention to the variables controlling gravity and velocity, since small changes to these values can make the game much easier or harder. The final game allows the player to navigate through randomly spaced pipes while earning points for each pair passed.



**Code: **

```html
<!DOCTYPE html>

<html lang="en">

<head>

<meta charset="UTF-8">

<meta name="viewport" content="width=device-width, initial-scale=1.0">

<title>Flappy Bird</title>



<style>

* {

  box-sizing: border-box;

}



body {

  margin: 0;

  min-height: 100vh;

  display: flex;

  justify-content: center;

  align-items: center;

  flex-direction: column;

  background: #202b43;

  font-family: Arial, sans-serif;

}



h1 {

  color: white;

  margin-bottom: 15px;

}



#gameContainer {

  position: relative;

  width: 420px;

  max-width: 95vw;

}



canvas {

  display: block;

  width: 100%;

  border: 4px solid white;

  border-radius: 12px;

  background: #70c5ce;

  touch-action: manipulation;

}



#scoreDisplay {

  position: absolute;

  top: 18px;

  left: 0;

  width: 100%;

  text-align: center;

  color: white;

  font-size: 38px;

  font-weight: bold;

  text-shadow: 2px 3px 3px #333;

  pointer-events: none;

}



#message {

  position: absolute;

  top: 43%;

  left: 50%;

  transform: translate(-50%, -50%);

  width: 85%;

  padding: 20px;

  border-radius: 12px;

  background: rgba(20, 40, 60, 0.85);

  color: white;

  text-align: center;

}



#message h2 {

  margin-top: 0;

  color: #ffdf52;

}



button {

  padding: 12px 25px;

  border: none;

  border-radius: 8px;

  background: #ffcc33;

  font-size: 17px;

  font-weight: bold;

  cursor: pointer;

}



#instructions {

  color: white;

  margin-top: 15px;

  text-align: center;

}

</style>

</head>



<body>



<h1>Flappy Bird</h1>



<div id="gameContainer">

  <canvas id="gameCanvas" width="420" height="600"></canvas>

  <div id="scoreDisplay">0</div>



  <div id="message">

    <h2 id="messageTitle">Flappy Bird</h2>

    <p id="messageText">Press Space, Up Arrow, or click to flap!</p>

    <button id="startButton">Start Game</button>

  </div>

</div>



<p id="instructions">Space / ↑ / Click = Flap</p>



<script>

const canvas = document.getElementById("gameCanvas");

const ctx = canvas.getContext("2d");



const scoreDisplay = document.getElementById("scoreDisplay");

const message = document.getElementById("message");

const messageTitle = document.getElementById("messageTitle");

const messageText = document.getElementById("messageText");

const startButton = document.getElementById("startButton");



const WIDTH = canvas.width;

const HEIGHT = canvas.height;



// Game physics

const gravity = 0.38;

const flapStrength = -6.9;

const pipeSpeed = 2.5;

const pipeWidth = 70;

const pipeGap = 166;

const pipeInterval = 95;



// Game variables

const birdX = 112;

const birdRadius = 16;



let birdY = 230;

let velocity = 0;

let pipes = [];



let score = 0;

let bestScore = 0;



let gameOver = false;

let gameStarted = false;

let frameCount = 0;



// Reset game

function resetGame() {

  birdY = 230;

  velocity = 0;

  pipes = [];

  score = 0;

  frameCount = 0;



  gameOver = false;

  gameStarted = true;



  scoreDisplay.textContent = score;

  message.style.display = "none";

}



// Bird movement

function flap() {

  if (!gameStarted || gameOver) return;

  velocity = flapStrength;

}



// Keyboard controls

document.addEventListener("keydown", function(event) {

  if (event.code === "Space" || event.code === "ArrowUp") {

    event.preventDefault();



    if (!gameStarted || gameOver) {

      resetGame();

    }



    flap();

  }

});



// Mouse and touch controls

canvas.addEventListener("pointerdown", function() {

  if (!gameStarted || gameOver) {

    resetGame();

  }



  flap();

});



// Start button

startButton.addEventListener("click", function() {

  resetGame();

  flap();

});



// Generate pipes

function createPipe() {

  const minGapY = 130;

  const maxGapY = HEIGHT - 130 - pipeGap;



  const gapY =

    Math.random() * (maxGapY - minGapY) + minGapY;



  pipes.push({

    x: WIDTH,

    gapY: gapY,

    scored: false

  });

}



// Collision detection

function checkCollision(pipe) {

  const horizontalCollision =

    birdX + birdRadius > pipe.x &&

    birdX - birdRadius < pipe.x + pipeWidth;



  const verticalCollision =

    birdY - birdRadius < pipe.gapY ||

    birdY + birdRadius > pipe.gapY + pipeGap;



  if (horizontalCollision && verticalCollision) {

    endGame();

  }

}



// Update game physics

function update() {

  if (!gameStarted || gameOver) return;



  frameCount++;



  velocity += gravity;

  birdY += velocity;



  // Ceiling and ground collision

  if (

    birdY - birdRadius <= 0 ||

    birdY + birdRadius >= HEIGHT

  ) {

    endGame();

    return;

  }



  // Create pipes

  if (frameCount % pipeInterval === 0) {

    createPipe();

  }



  // Update pipes

  for (let pipe of pipes) {

    pipe.x -= pipeSpeed;



    checkCollision(pipe);



    if (gameOver) return;



    // Scoring

    if (

      !pipe.scored &&

      pipe.x + pipeWidth < birdX - birdRadius

    ) {

      pipe.scored = true;

      score++;

      scoreDisplay.textContent = score;

    }

  }



  // Remove off-screen pipes

  pipes = pipes.filter(function(pipe) {

    return pipe.x + pipeWidth > 0;

  });

}



// Draw clouds

function drawCloud(x, y) {

  ctx.fillStyle = "rgba(255,255,255,0.75)";

  ctx.beginPath();

  ctx.arc(x, y, 20, 0, Math.PI * 2);

  ctx.arc(x + 20, y - 10, 25, 0, Math.PI * 2);

  ctx.arc(x + 45, y, 20, 0, Math.PI * 2);

  ctx.fill();

}



// Draw background

function drawBackground() {

  // Sky

  ctx.fillStyle = "#70c5ce";

  ctx.fillRect(0, 0, WIDTH, HEIGHT);



  // Sun

  ctx.fillStyle = "#ffe28a";

  ctx.beginPath();

  ctx.arc(345, 95, 38, 0, Math.PI * 2);

  ctx.fill();



  // Clouds

  drawCloud(70, 110);

  drawCloud(270, 180);

  drawCloud(150, 65);



  // Ground

  ctx.fillStyle = "#ded895";

  ctx.fillRect(0, HEIGHT - 18, WIDTH, 18);



  ctx.fillStyle = "#80b94a";

  ctx.fillRect(0, HEIGHT - 18, WIDTH, 6);

}



// Draw bird

function drawBird() {

  ctx.save();

  ctx.translate(birdX, birdY);



  const rotation = Math.max(

    -0.5,

    Math.min(1.0, velocity * 0.07)

  );



  ctx.rotate(rotation);



  // Body

  ctx.fillStyle = "#ffdd38";

  ctx.beginPath();

  ctx.arc(0, 0, birdRadius, 0, Math.PI * 2);

  ctx.fill();



  ctx.strokeStyle = "#d89a19";

  ctx.lineWidth = 2;

  ctx.stroke();



  // Wing

  ctx.fillStyle = "#fff1a6";

  ctx.beginPath();

  ctx.ellipse(-7, 5, 10, 6, -0.3, 0, Math.PI * 2);

  ctx.fill();



  // Eye

  ctx.fillStyle = "white";

  ctx.beginPath();

  ctx.arc(7, -7, 7, 0, Math.PI * 2);

  ctx.fill();



  // Pupil

  ctx.fillStyle = "black";

  ctx.beginPath();

  ctx.arc(10, -7, 3, 0, Math.PI * 2);

  ctx.fill();



  // Beak

  ctx.fillStyle = "#ff823b";

  ctx.beginPath();

  ctx.moveTo(12, 2);

  ctx.lineTo(28, 7);

  ctx.lineTo(12, 11);

  ctx.closePath();

  ctx.fill();



  ctx.restore();

}



// Draw pipes

function drawPipes() {

  for (let pipe of pipes) {

    const topHeight = pipe.gapY;

    const bottomY = pipe.gapY + pipeGap;

    const bottomHeight = HEIGHT - bottomY;



    // Pipe bodies

    ctx.fillStyle = "#55bd43";

    ctx.fillRect(pipe.x, 0, pipeWidth, topHeight);

    ctx.fillRect(pipe.x, bottomY, pipeWidth, bottomHeight);



    // Pipe outlines

    ctx.strokeStyle = "#28752c";

    ctx.lineWidth = 3;

    ctx.strokeRect(pipe.x, 0, pipeWidth, topHeight);

    ctx.strokeRect(pipe.x, bottomY, pipeWidth, bottomHeight);



    // Pipe openings

    ctx.fillStyle = "#65d451";

    ctx.fillRect(pipe.x - 5, topHeight - 23, pipeWidth + 10, 23);

    ctx.fillRect(pipe.x - 5, bottomY, pipeWidth + 10, 23);



    // Opening outlines

    ctx.strokeStyle = "#28752c";

    ctx.strokeRect(pipe.x - 5, topHeight - 23, pipeWidth + 10, 23);

    ctx.strokeRect(pipe.x - 5, bottomY, pipeWidth + 10, 23);

  }

}



// Draw game

function draw() {

  ctx.clearRect(0, 0, WIDTH, HEIGHT);

  drawBackground();

  drawPipes();

  drawBird();

}



// Game over

function endGame() {

  if (gameOver) return;



  gameOver = true;

  bestScore = Math.max(bestScore, score);



  messageTitle.textContent = "Game Over!";

  messageText.innerHTML =

    "Score: " + score +

    "<br>Best Score: " + bestScore;



  startButton.textContent = "Play Again";

  message.style.display = "block";

}



// Game loop

const timestep = 1000 / 60;



let lastTime = 0;

let accumulator = 0;



function gameLoop(time) {

  if (!lastTime) {

    lastTime = time;

  }



  const deltaTime = Math.min(50, time - lastTime);



  lastTime = time;

  accumulator += deltaTime;



  while (accumulator >= timestep) {

    update();

    accumulator -= timestep;

  }



  draw();

  requestAnimationFrame(gameLoop);

}



// Start animation

requestAnimationFrame(gameLoop);

</script>



</body>

</html>
```

### Code Analysis

- **Canvas:** The HTML contains a `<canvas>` element with a width of 420 pixels and a height of 600 pixels. JavaScript uses this canvas to draw the bird, pipes, and background. Changing its dimensions would affect the visible playing area and potentially distort the graphics if the positions and sizes of the objects were not adjusted accordingly.
- **State Variables:** The language JavaScript tracks values such as `birdY`, `velocity`, `gravity`, `flapStrength`, `pipeSpeed`, `score`, and `gameOver`. These determine the bird's vertical position, how quickly it rises or falls, the movement of the pipes, and whether the game is running or over.
- **Game Loop:** The `requestAnimationFrame(gameLoop)` function continuously redraws the game, while `update()` calculates movement and checks collisions 60 times per second. Each cycle updates the bird's position based on gravity and velocity, moves the pipes across the screen, and displays the updated score.
- **Input Handling:** A `keydown` event listener detects when the player presses the spacebar or up arrow. This changes the bird's vertical velocity to `-6.9`, making it move upward. I noticed that changing this value to `-10` would make the bird jump higher, while a value closer to zero would produce a smaller jump.
- **Collision Detection:** The code compares the bird's position and radius with the boundaries of each pipe. If the bird overlaps a pipe or touches the ground or ceiling, `gameOver` becomes true and the game ends. The distance between the pipes is controlled by `pipeGap = 166`, which could be increased to 200 to make the game easier by providing more space for the bird to pass through.
- **Scoring:** The `score` variable increases by one whenever the bird successfully passes a pair of pipes. The condition `pipe.x + pipeWidth < birdX - birdRadius` checks whether the pipe has completely passed the bird, while `pipe.scored` prevents the same pipe from being counted twice. The pipes move left at a speed of 2.5 pixels per update, controlled by `pipeSpeed`. Increasing this value to 4 would make the pipes approach faster, giving the player less time to react.





## Goals for this week: **



I want to investigate how JavaScript, Python, and C++ differ in computational efficiency when executing the same algorithms. My goal is to distinguish between time complexity, which describes how an algorithm's operations scale with its input size, and execution time, which can vary depending on the language, compiler, or runtime environment. I plan to implement identical algorithms in all three languages, including linear search, binary search, and sorting, then test them with progressively larger datasets. In past experiences, I have noticed c++ to be the most efficient because it's typically compiled directly into machine code before execution. Python typically runs code through a virtual machine, while JavaScript often compiles code as the program is running. This way, c++ can avoid some runtime overhead, thus is particularly effective for larger chunks in code found in game development. Additionally, with the same outcome, I have noticed the c++ code to be the shortest and require less index especially when working with #include \<set> or #include \<vector>, which you can use vectorname.size() to get the amount of values inputted. Furthermore, since the same algorithm generally retains its time complexity regardless of the language, I am interested in understanding why execution times are still different. I want to look more closely at factors like memory capacity, data structures used, built-in functions, and how they influence performance. Another goal I have is modifying the visual design of my game. After looking at the code for Flappy Bird and Subway Surfers, I want to try to change their colors and visual elements like obstacle width to increase difficulty. I want to find the sections of JavaScript responsible for adjusting properties such as background-color, fillStyle, and strokeStyle.

# Hex Color ID Reference Sheet:

## Basic colors

Red
#FF0000

Orange
#FFA500

Yellow
#FFFF00

Green
#008000

Blue
#0000FF

Purple
#800080

Pink
#FFC0CB

White
#FFFFFF

Black
#000000

Gray
#808080


# Subway Surfers in a Different Language?

## JavaScript — C++

**C++**

```cpp
// Metro Runner - C++ / raylib port of the JavaScript Canvas game.
// Build (macOS with Homebrew raylib): c++ -std=c++17 metro_runner.cpp -o metro_runner $(pkg-config --cflags --libs raylib)
// Build (Linux with raylib installed): g++ -std=c++17 metro_runner.cpp -o metro_runner $(pkg-config --cflags --libs raylib)
#include "raylib.h"
#include <algorithm>
#include <cmath>
#include <cstdlib>
#include <ctime>
#include <string>
#include <vector>



constexpr int WIDTH = 720;
constexpr int HEIGHT = 540;
constexpr float LANES[3] = {210.0f, 360.0f, 510.0f};
constexpr float PLAYER_Y = 427.0f;
constexpr float GRAVITY = 1650.0f;
constexpr float JUMP_VELOCITY = 650.0f;
constexpr float PI2 = 6.28318530718f;



enum class Mode { Ready, Playing, Paused, Over };
enum class Kind { Train, Barrier, Sign, Coin };
struct Obstacle { int lane; float depth; Kind type; bool checked = false; };
struct Projection { float x, y, scale; };



static Color Hex(unsigned rgb, unsigned char alpha = 255) {
   return Color{static_cast<unsigned char>((rgb >> 16) & 255),
                static_cast<unsigned char>((rgb >> 8) & 255),
                static_cast<unsigned char>(rgb & 255), alpha};
}
static void Rect(float x, float y, float w, float h, Color c) { DrawRectangleRec({x,y,w,h}, c); }
static void Line(float x1,float y1,float x2,float y2,Color c,float thick=3) {
   DrawLineEx({x1,y1},{x2,y2},thick,c);
}
static void Ellipse(float x,float y,float rx,float ry,Color c) {
   DrawEllipse(static_cast<int>(x),static_cast<int>(y),rx,ry,c);
}
static void CenterText(const std::string& s,int y,int size,Color c) {
   DrawText(s.c_str(), (WIDTH-MeasureText(s.c_str(),size))/2, y,size,c);
}



class Game {
public:
   Mode mode = Mode::Ready;
   int lane = 1, coins = 0, best = 0;
   float visualX = LANES[1], jumpHeight = 0, verticalVelocity = 0, slideTime = 0;
   float score = 0, distance = 0, spawnTimer = 0, worldSpeed = 210;
   std::vector<Obstacle> obstacles;



   void Reset() {
       mode = Mode::Playing; lane = 1; visualX = LANES[1];
       jumpHeight = verticalVelocity = slideTime = 0;
       score = distance = 0; coins = 0; obstacles.clear();
       spawnTimer = 0.8f; worldSpeed = 210;
   }
   void Move(int action) { // -1 left, 1 right, 2 jump, 3 slide, 4 pause
       if (action == 4) {
           if (mode == Mode::Playing) mode = Mode::Paused;
           else if (mode == Mode::Paused) mode = Mode::Playing;
           return;
       }
       if (mode != Mode::Playing) return;
       if (action == -1) lane = std::max(0,lane-1);
       if (action == 1) lane = std::min(2,lane+1);
       if (action == 2 && jumpHeight == 0) {
           verticalVelocity = JUMP_VELOCITY; slideTime = 0;
       }
       if (action == 3) {
           slideTime = 0.7f;
           if (jumpHeight > 0) verticalVelocity = -JUMP_VELOCITY;
       }
   }
   void SpawnRow() {
       int safeLane = GetRandomValue(0,2);
       int hazardLanes[2], j=0;
       for (int l=0;l<3;l++) if (l!=safeLane) hazardLanes[j++]=l;
       int count = GetRandomValue(0,999)<400 ? 2 : 1;
       for (int i=0;i<count;i++)
           obstacles.push_back({hazardLanes[i],0.0f,static_cast<Kind>(GetRandomValue(0,2)),false});
       for (int i=0;i<4;i++) obstacles.push_back({safeLane,-0.17f-i*0.12f,Kind::Coin,false});
   }
   Projection Project(const Obstacle& o) const {
       float depth = std::max(0.0f,o.depth), p = depth*depth;
       return {WIDTH/2.0f+(LANES[o.lane]-WIDTH/2.0f)*(0.18f+0.82f*p),
               138.0f+p*(PLAYER_Y-138.0f), 0.18f+p*0.82f};
   }
   void EndGame() { mode = Mode::Over; best = std::max(best,static_cast<int>(score)); }
   void Update(float dt) {
       if (mode!=Mode::Playing) return;
       distance += worldSpeed*dt;
       worldSpeed = std::min(390.0f,210.0f+distance*0.012f);
       score += dt*worldSpeed*0.06f;
       visualX += (LANES[lane]-visualX)*std::min(1.0f,13.0f*dt);
       if (jumpHeight>0 || verticalVelocity>0) {
           jumpHeight += verticalVelocity*dt;
           verticalVelocity -= GRAVITY*dt;
           if (jumpHeight<=0) jumpHeight=verticalVelocity=0;
```

To compare the time complexity and efficiency of JavaScript and C++, I converted my Metro Runner game into C++ using ChatGPT and looked into how the two versions handle obstacles and drawing. I started off by comparing the code that updates each obstacle and checks whether it has reached the player.

**JavaScript:**

```javascript
for (const object of obstacles) {
  object.depth += dt * worldSpeed / 480;
  if (!object.checked && object.depth >= 0.93) {
    object.checked = true;
    if (object.lane !== lane) continue;
  }
}
```

**C++:**

```cpp
for (auto& o : obstacles) {
    o.depth += dt * worldSpeed / 480.0f;
    if (!o.checked && o.depth >= 0.93f) {
        o.checked = true;
        if (o.lane != lane) continue;
    }
}
```

Both loops have a time complexity of O(n), in which n represents the number of obstacles. This is because the program goes through every obstacle once per frame. If the number of obstacles doubles, the amount of work in this loop doubles roughly too. Although the time complexity is the same, C++ runs the loop faster because of the way it stores information. In JavaScript, each obstacle is stored as an object inside an array, while C++ uses structs inside a std::vector. A vector keeps its elements next to each other in memory, so it's faster for the computer to access them when it's going through one after another, meaning that c++ can complete the same number of operations in less time.

Then, I looked at the part of the code that sorts obstacles before drawing them.

**JavaScript:**

```javascript
[...obstacles].sort((a,b) => a.depth-b.depth).forEach(drawObject);
```

**C++:**

```cpp
auto sorted = obstacles;

std::sort(
    sorted.begin(),
    sorted.end(),
    [](const Obstacle& a, const Obstacle& b) {
        return a.depth < b.depth;
    }
);

for (const auto& o : sorted) {
    DrawObject(o);
}
```

Both of these  versions sort obstacles by their depth so that objects farther away are drawn before objects closer to the player. Sorting takes O(n log n) time, making it the most costly part of the game through time complexity. Since the other main operations take O(n) or O(1), the overall time complexity of both games is O(n log n) per frame. Something I noticed is that both versions create a copy of the obstacles before sorting. In JavaScript, [...obstacles] creates a new array, and  auto sorted = obstacles in c++ copies the vector. This requires extra time and memory in both languages.

The last part I looked at was how the game removes obstacles after they pass the player.

**JavaScript:**

```javascript
obstacles = obstacles.filter(o => o.depth < 1.2);
```

**C++:**

```cpp
obstacles.erase(
    std::remove_if(
        obstacles.begin(),
        obstacles.end(),
        [](const Obstacle& o) {
            return o.depth >= 1.2f;
        }
    ),
    obstacles.end()
);
```

Both versions have a time complexity of O(n) because the program has to go through each obstacle to check whether it should be removed. The difference is that JavaScript's filter() creates a new array with the remaining obstacles, while C++ removes them directly from the existing vector. Because C++ doesn't need to create a new collection every frame, it could save some time spent on processing and memory.

Overall, converting Metro Runner from JavaScript to C++ didn't change its time complexity because I was still using the same algorithms. What changed was how each language stored and processed the information. C++ is compiled into machine code before the program runs, while JavaScript uses a browser engine that can compile and optimize the code as it runs. From comparing the code, I could see why C++ might be more efficient, especially when it comes to memory usage. However, I can't conclude that C++ is faster just from looking at the code, I would need a wider variety of codes to compare it more precisely.

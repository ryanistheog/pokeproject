<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Geometry Dash Neon Adventure</title>
<style>
*{box-sizing:border-box}
body{margin:0;background:#09051b;color:white;font-family:Arial,sans-serif;text-align:center}
header{padding:18px;background:#100a2b;border-bottom:2px solid #00f6ff}
h1{color:#00f6ff;text-shadow:0 0 15px #00eaff;margin:8px}
button{background:#17143b;color:white;border:2px solid #00f6ff;border-radius:10px;padding:11px 17px;margin:5px;font-weight:bold;cursor:pointer}
button:hover{background:#30306a;box-shadow:0 0 12px #00f6ff}
button:disabled{opacity:.4}
.screen{display:none;padding:18px}
.screen.active{display:block}
.panel{max-width:950px;margin:18px auto;padding:18px;border:1px solid #5149a8;border-radius:16px;background:#110c2c}
.grid{display:grid;grid-template-columns:repeat(auto-fit,minmax(140px,1fr));gap:9px;max-width:850px;margin:15px auto}
.skin{padding:10px;border:1px solid #45417a;border-radius:10px;cursor:pointer}
.skin.selected{border:2px solid #00f6ff;background:#22214c}
.icon{width:30px;height:30px;margin:4px auto;border:3px solid white;box-shadow:0 0 8px currentColor}
#gameCanvas{display:block;width:min(100%,900px);height:auto;margin:12px auto;border:2px solid #00f6ff;border-radius:10px;background:#10072d;touch-action:manipulation}
#status{color:#ffe35a;min-height:24px;font-weight:bold}
.small{color:#c9c4e8;font-size:14px}
.level{min-height:62px}
.birthday-title {color:#ffe35a;text-shadow:0 0 20px #ffea00;font-size:3rem;margin:20px 0}
.birthday-btn {font-size:1.5rem;padding:15px 35px;background:#ff32dc;border-color:#ff32dc;text-shadow:0 0 5px #fff}
.birthday-btn:hover {background:#e02ec2;box-shadow:0 0 20px #ff32dc}
</style>
</head>
<body>
<header>
<h1>GEOMETRY DASH: NEON ADVENTURE</h1>
<div>27 levels • All skins free • Gamemode portals</div>
</header>

<section id="birthday" class="screen active">
<div class="panel" style="padding:50px 20px;">
<h1 class="birthday-title">🎁 HAPPY BIRTHDAY EVAN! 🎂</h1>
<p style="font-size:1.2rem; color:#00f6ff;">Your custom Geometry Dash arcade machine is ready.</p>
<br>
<button class="birthday-btn" onclick="show('home')">CONTINUE ▶</button>
</div>
</section>

<section id="home" class="screen">
<div class="panel">
<h2>Ready to jump?</h2>
<p>Dodge spikes, leap over blocks, enter portals and change your vehicle physics mid-air!</p>
<button onclick="show('levels')">▶ PLAY LEVELS</button>
<button onclick="show('skins')">◇ CUBE SKINS</button>
</div>
<div class="panel">
<h3>Game features</h3>
<p>⚡ Neon graphics · 🔺 Spikes · ▰ Platforms · 🌀 Portals</p>
<p>Cube · Ship · UFO · Wave · Ball · Robot</p>
</div>
</section>

<section id="levels" class="screen">
<div class="panel">
<h2>Choose Your Level</h2>
<p>Every level is unlocked.</p>
<div id="levelGrid" class="grid"></div>
<button onclick="show('home')">BACK</button>
</div>
</section>

<section id="skins" class="screen">
<div class="panel">
<h2>Cube Customizer</h2>
<p>All icons and colors are FREE!</p>
<div id="skinGrid" class="grid"></div>
<button onclick="show('home')">BACK</button>
</div>
</section>

<section id="game" class="screen">
<div class="panel">
<h2 id="levelTitle">Level 1</h2>
<div id="status">Tap, click or press SPACE to jump.</div>
<canvas id="gameCanvas" width="900" height="390"></canvas>
<div>
<button onpointerdown="jump()" style="padding:15px 30px; background:#00f6ff; color:#000;">JUMP / ACTION</button>
<button onclick="togglePause()">PAUSE / RESUME</button>
<button onclick="restart()">RESTART</button>
<button onclick="show('levels');stopGame()">LEVELS</button>
</div>
<p class="small">Keyboard: SPACE / ↑ / W to jump. Wave & Ship require holding or rapid tapping!</p>
</div>
</section>

<script>
const $=id=>document.getElementById(id);
function show(id){
document.querySelectorAll('.screen').forEach(s=>s.classList.remove('active'));
$(id).classList.add('active');
if(id!=='game')stopGame();
}
const colors=['#00f6ff','#ff32dc','#ffeb3b','#76ff03','#ff5722','#bd70ff',
'#ffffff','#ff4081','#00e676','#448aff','#ff9100','#e040fb'];
const skinNames=['Neon Blue','Pink Flash','Golden Star','Toxic Green','Magma','Purple Shadow',
'Classic White','Candy','Emerald','Electric Blue','Sunburst','Violet'];
let skinIndex=0, selectedLevel=1;
const modes=['CUBE','SHIP','UFO','WAVE','BALL','ROBOT'];
let mode='CUBE';
const canvas=$('gameCanvas'),ctx=canvas.getContext('2d');
let player,obstacles,particles,frame=0,progress=0,running=false,paused=false,raf=0;
let isHolding=false;

const levelNames = [
  "Stereo Start", "Neon Steps", "Pixel Rush", "Gravity Run", "Laser Road",
  "Stereo Start 2", "Neon Steps 2", "Pixel Rush 2", "Gravity Run 2", "Laser Road 2",
  "Stereo Start 3", "Neon Steps 3", "Pixel Rush 3", "Gravity Run 3", "Laser Road 3",
  "Stereo Start 4", "Neon Steps 4", "Pixel Rush 4", "Gravity Run 4", "Laser Road 4",
  "Stereo Start 5", "Neon Steps 5", "Pixel Rush 5", "Gravity Run 5", "Laser Road 5",
  "Neon Finale", "Deadlocked"
];

function buildSkins(){
$('skinGrid').innerHTML='';
skinNames.forEach((name,i)=>{
let d=document.createElement('div');
d.className='skin'+(i===skinIndex?' selected':'');
d.innerHTML='<div class="icon" style="background:'+colors[i]+';color:'+colors[i]+'"></div><b>'+name+'</b><p class="small">FREE</p>';
d.onclick=()=>{skinIndex=i;buildSkins()};
$('skinGrid').appendChild(d);
});
}

function buildLevels() {
  const grid = document.getElementById("levelGrid");
  grid.innerHTML = "";
  levelNames.forEach((name, index) => {
    const levelNumber = index + 1;
    const button = document.createElement("button");
    button.className = "level";
    button.innerHTML = "<b>LEVEL " + levelNumber + "</b><br><span class='small'>" + name + "</span>";
    button.onclick = () => startLevel(levelNumber);
    grid.appendChild(button);
  });
}
buildSkins();buildLevels();

function startLevel(n){
selectedLevel=n;show('game');startGame();
}
function stopGame(){running=false;cancelAnimationFrame(raf)}

function startGame(){
stopGame();running=true;paused=false;frame=0;progress=0;mode='CUBE';
player={x:100,y:285,size:30,vy:0,onGround:true,rotation:0,targetRot:0};
obstacles=[];particles=[];
isHolding=false;

let currentX=450;
let modesPool=['SHIP','UFO','WAVE','BALL','ROBOT','CUBE'];
for(let i=0;i<20;i++){
    let typeRand=Math.random();
    if(i>0&&i%4===0){
        let targetMode=modesPool[(i/4-1)%modesPool.length];
        obstacles.push({x:currentX,y:170,w:35,h:130,type:'portal',targetMode:targetMode,used:false});
        currentX+=350;
    }else if(typeRand<0.4){
        obstacles.push({x:currentX,y:320,w:32,h:34,type:'spike'});
        currentX+=220+Math.random()*100;
    }else if(typeRand<0.75){
        obstacles.push({x:currentX,y:260,w:60,h:60,type:'block'});
        if(Math.random()>0.5){
            obstacles.push({x:currentX+15,y:200,w:30,h:32,type:'spike'});
        }
        currentX+=260+Math.random()*100;
    }else{
        obstacles.push({x:currentX,y:320,w:32,h:34,type:'spike'});
        obstacles.push({x:currentX+35,y:320,w:32,h:34,type:'spike'});
        currentX+=280;
    }
}
obstacles.push({x:currentX+200,y:170,w:40,h:130,type:'finish'});

$('levelTitle').textContent='LEVEL '+selectedLevel+' — '+mode;
$('status').textContent='Dodge the neon hazards!';
raf=requestAnimationFrame(loop);
}

function restart(){if($('game').classList.contains('active'))startGame()}

function jump(){
if(!running||paused||!player)return;
if(mode==='CUBE'||mode==='ROBOT'){
    if(player.onGround){
        player.vy=mode==='ROBOT'?-14:-11.5;
        player.onGround=false;
    }
}else if(mode==='UFO'){
    player.vy=-8.5;
    player.onGround=false;
}else if(mode==='BALL'){
    if(player.onGround){
        player.gravityInverted=!player.gravityInverted;
        player.onGround=false;
    }
}
}

function togglePause(){
if(!running)return;
paused=!paused;
if(!paused){
    raf=requestAnimationFrame(loop);
    $('status').textContent='Resumed!';
}else{
    $('status').textContent='Paused — press RESUME to continue';
}
}

document.addEventListener('keydown',e=>{
if(['Space','ArrowUp','KeyW'].includes(e.code)){
    e.preventDefault();
    if(!isHolding){isHolding=true;jump()}
}
});
document.addEventListener('keyup',e=>{
if(['Space','ArrowUp','KeyW'].includes(e.code)){isHolding=false}
});
canvas.addEventListener('pointerdown',e=>{e.preventDefault();isHolding=true;jump()});
canvas.addEventListener('pointerup',e=>{e.preventDefault();isHolding=false});

function drawBackground(){
let g=ctx.createLinearGradient(0,0,0,390);
g.addColorStop(0,'#110933');g.addColorStop(1,'#040514');
ctx.fillStyle=g;ctx.fillRect(0,0,900,390);
ctx.strokeStyle='rgba(0,246,255,.08)';ctx.lineWidth=1;
let offset=(progress)%40;
for(let x=-40;x<940;x+=40){ctx.beginPath();ctx.moveTo(x-offset,0);ctx.lineTo(x-offset,320);ctx.stroke()}
for(let y=0;y<320;y+=40){ctx.beginPath();ctx.moveTo(0,y);ctx.lineTo(900,y);ctx.stroke()}
ctx.fillStyle='#0f0b29';ctx.fillRect(0,320,900,70);
ctx.strokeStyle='#00f6ff';ctx.lineWidth=3;ctx.beginPath();ctx.moveTo(0,320);ctx.lineTo(900,320);ctx.stroke();

for(let i=0;i<30;i++){
    let x=(i*137-progress*.2+9000)%900,y=(i*73)%240;
    ctx.fillStyle='rgba(255,255,255,.4)';ctx.fillRect(x,y,2,2);
}
}

function drawPlayer(){
ctx.save();
ctx.translate(player.x+15,player.y+15);
ctx.rotate(player.rotation);
ctx.fillStyle=colors[skinIndex];
ctx.shadowBlur=15;
ctx.shadowColor=colors[skinIndex];

if(mode==='WAVE'){
    ctx.beginPath();ctx.moveTo(-15,0);ctx.lineTo(15,-10);ctx.lineTo(15,10);ctx.closePath();ctx.fill();
}else if(mode==='SHIP'){
    ctx.beginPath();ctx.moveTo(-18,-8);ctx.lineTo(18,0);ctx.lineTo(-18,8);ctx.lineTo(-8,0);ctx.closePath();ctx.fill();
    ctx.fillStyle='#ffffff';ctx.fillRect(-10,-4,6,6);
}else if(mode==='BALL'){
    ctx.beginPath();ctx.arc(0,0,15,0,Math.PI*2);ctx.fill();
    ctx.lineWidth=3;ctx.strokeStyle='#10102b';ctx.beginPath();ctx.moveTo(-15,0);ctx.lineTo(15,0);ctx.moveTo(0,-15);ctx.lineTo(0,15);ctx.stroke();

# Super Mario Remix (Super-mari00)

[![Pages Status](https://github.com/rowyn11056-byte/Super-mari00/actions/workflows/pages.yml/badge.svg)](https://github.com/rowyn11056-byte/Super-mari00/actions/workflows/pages.yml)

Live demo: https://rowyn11056-byte.github.io/Super-mari00/

---

This repository contains a small Super Mario–style HTML game. I added a placeholder demo page and an automatic GitHub Pages deployment workflow so you can host a live preview.

✅ What I added

- `docs/index.html` — **placeholder** demo page with instructions for enabling the playable demo.
- `.github/workflows/pages.yml` — GitHub Actions workflow that deploys the `docs/` folder to GitHub Pages on push to `main`.
- `.nojekyll` — prevents Jekyll processing in Pages.

> NOTE: The game's HTML/JS is currently present in the original `README.md`. To enable the live playable demo, move the full HTML into `docs/index.html` (replace the placeholder), commit, and push to `main`. The workflow will deploy it automatically and the demo will be available at the URL above.

---

## How to preview locally

1. Copy/merge the game's HTML into `docs/index.html`.
2. Run a simple local server from the repository root:

```bash
python -m http.server --directory docs 8000
```

3. Open http://localhost:8000 in your browser.

---

The playable demo has been moved into `docs/index.html` and is now a working, lightweight playable demo.

To preview locally, run:

```bash
python -m http.server --directory docs 8000
```

Open http://localhost:8000 to try the demo. The README no longer contains the full game HTML (it's been moved to `docs/index.html`).
<style>
body{margin:0;background:#5c94fc;overflow:hidden;}
canvas{width:360px;height:640px;image-rendering:pixelated;display:block;margin:auto;}

/* D-Pad */
#left,#right,#up,#down{
  width:50px;height:50px;background:#555;color:#fff;border:3px solid #000;border-radius:8px;font-size:20px;
  position:fixed;text-align:center;line-height:50px;
}
#left{bottom:70px;left:20px;} 
#right{bottom:70px;left:90px;}
#up{bottom:130px;left:55px;} 
#down{bottom:10px;left:55px;}

/* Action Buttons */
#A,#B{
  width:50px;height:50px;border-radius:25px;background:#d00;color:#fff;border:3px solid #800;font-size:20px;
  position:fixed;text-align:center;line-height:50px;
}
#A{bottom:70px;right:80px;} 
#B{bottom:130px;right:50px;}

/* Start / Select */
#start,#select{
  width:40px;height:30px;border-radius:5px;background:#222;color:#fff;border:2px solid #000;font-size:14px;
  position:fixed;text-align:center;line-height:30px;
}
#start{bottom:200px;right:50px;} 
#select{bottom:200px;right:10px;}
</style>
</head>
<body>
<canvas id="game" width="180" height="320"></canvas>

<button id="left">←</button>
<button id="right">→</button>
<button id="up">↑</button>
<button id="down">↓</button>
<button id="A">A</button>
<button id="B">B</button>
<button id="start">Start</button>
<button id="select">Select</button>

<script>
const c=document.getElementById("game"),x=c.getContext("2d");
x.imageSmoothingEnabled=false;

// ----- STATE -----
let gameState="title",level=1,score=0,lives=3,cam=0,win=false,paused=false;
const p={x:20,y:250,w:10,h:10,dx:0,dy:0,ground:false};
const g=0.4;let platforms=[],coins=[],enemies=[],blocks=[],flag={x:600,y:260};
let L=false,R=false,U=false,D=false,A=false,B=false,Start=false,Select=false;
let gamepadIndex=null,big=false;

// ----- UTILS -----
function beep(f,t){let a=new AudioContext(),o=a.createOscillator();o.frequency.value=f;o.connect(a.destination);o.start();o.stop(a.currentTime+t);}
function resetPlayer(){p.x=20;p.y=250;p.dy=0;big=false;}
function drawPlayer(px,py){x.fillStyle="#ff0000";x.fillRect(px,py,10,10);x.fillStyle="#0000ff";x.fillRect(px+2,py+5,6,5);}
function drawEnemy(ex,ey){x.fillStyle="#aa0000";x.fillRect(ex,ey,10,10);}
function saveGame(){localStorage.setItem("mario_remix",JSON.stringify({level,score,lives}));}
function loadGame(){let data=localStorage.getItem("mario_remix");if(data){data=JSON.parse(data);level=data.level;score=data.score;lives=data.lives;loadLevel(level);}}

// ----- LEVELS -----
function loadLevel(n){
  win=false;resetPlayer();blocks=[];coins=[];enemies=[];
  if(n===1){
    platforms=[{x:0,y:300,w:1400,h:20},{x:120,y:240,w:60,h:10}];
    coins=[{x:130,y:220,col:false}];
    enemies=[{x:300,y:290,w:10,h:10,d:1}];
    flag={x:500,y:260};
  }
  if(n===2){
    platforms=[{x:0,y:300,w:1600,h:20},{x:200,y:220,w:80,h:10},{x:360,y:180,w:80,h:10}];
    coins=[{x:220,y:200,col:false},{x:380,y:160,col:false}];
    enemies=[{x:420,y:290,w:10,h:10,d:-1}];
    flag={x:650,y:260};
  }
  if(n===3){
    platforms=[{x:0,y:300,w:1800,h:20},{x:300,y:200,w:100,h:10}];
    coins=[{x:320,y:180,col:false}];
    enemies=[{x:480,y:290,w:10,h:10,d:1}];
    flag={x:800,y:260};
  }
}

// ----- CONTROLS -----
const btnMap = {left:L,right:R,up:U,down:D,A:A,B:B,start:Start,select:Select};
["left","right","up","down","A","B","start","select"].forEach(id=>{
  document.getElementById(id).addEventListener("touchstart",()=>btnMap[id]=true);
  document.getElementById(id).addEventListener("touchend",()=>btnMap[id]=false);
});

window.addEventListener("keydown",e=>{
  if(e.code==="ArrowLeft")L=true;
  if(e.code==="ArrowRight")R=true;
  if(e.code==="ArrowUp")U=true;
  if(e.code==="ArrowDown")D=true;
  if(e.code==="KeyZ")A=true;
  if(e.code==="KeyX")B=true;
  if(e.code==="Enter")Start=true;
  if(e.code==="ShiftRight")Select=true;
});
window.addEventListener("keyup",e=>{
  if(e.code==="ArrowLeft")L=false;
  if(e.code==="ArrowRight")R=false;
  if(e.code==="ArrowUp")U=false;
  if(e.code==="ArrowDown")D=false;
  if(e.code==="KeyZ")A=false;
  if(e.code==="KeyX")B=false;
  if(e.code==="Enter")Start=false;
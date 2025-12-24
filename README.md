<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
<title>Sorpresa para Mayli 💕</title>

<script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1.6.0/dist/confetti.browser.min.js"></script>

<style>
:root{
  --accent:#fb6f92;
  --glow:#ffb3d9;
}

body{
  margin:0;
  height:100vh;
  display:flex;
  flex-direction:column;
  justify-content:center;
  align-items:center;
  background:#ffe5ec;
  font-family:Arial, sans-serif;
  overflow:hidden;
  touch-action:none;
}

.instructions{
  color:var(--accent);
  font-size:20px;
  font-weight:bold;
  margin-bottom:15px;
  text-align:center;
}

#main-wrapper{
  padding:12px;
  border-radius:30px;
  background:linear-gradient(45deg,#ff85a1,#fbb1bd,#85e3ff,#ff85a1);
  background-size:400% 400%;
  animation:led 5s linear infinite;
  box-shadow:0 0 25px var(--glow);
}

@keyframes led{
  0%{background-position:0% 50%}
  50%{background-position:100% 50%}
  100%{background-position:0% 50%}
}

.grid{
  display:grid;
  grid-template-columns:repeat(2,140px);
  grid-template-rows:repeat(3,140px);
  gap:10px;
  background:#fff;
  padding:15px;
  border-radius:22px;
}

.cell{
  position:relative;
  width:140px;
  height:140px;
  border-radius:15px;
  display:flex;
  flex-direction:column;
  justify-content:center;
  align-items:center;
  border:1px solid #ffe5ec;
}

.icon{ font-size:50px; }

.msg{
  font-size:11px;
  color:#aaa;
  font-weight:bold;
  margin-top:6px;
}

canvas{
  position:absolute;
  inset:0;
  border-radius:15px;
  cursor:crosshair;
}

#win{
  position:fixed;
  inset:0;
  background:rgba(255,255,255,.97);
  display:none;
  flex-direction:column;
  justify-content:center;
  align-items:center;
  text-align:center;
  z-index:100;
}

.big{
  font-size:90px;
  animation:bounce 1s infinite;
}

@keyframes bounce{
  0%,100%{transform:scale(1)}
  50%{transform:scale(1.2)}
}

.heart{
  position:fixed;
  top:-20px;
  font-size:18px;
  pointer-events:none;
  animation:fall linear forwards;
}

@keyframes fall{
  to{transform:translateY(110vh);opacity:0}
}
</style>
</head>

<body>

<!-- 🎶 Música romántica (autoplay + bajo volumen) -->
<audio id="bg-music" loop autoplay muted>
  <source src="https://cdn.pixabay.com/audio/2023/03/27/audio_fdbb1a62b7.mp3" type="audio/mpeg">
</audio>

<audio id="win-sound">
  <source src="https://assets.mixkit.co/active_storage/sfx/2013/2013-preview.mp3">
</audio>

<audio id="scratch-sound">
  <source src="https://assets.mixkit.co/active_storage/sfx/1105/1105-preview.mp3">
</audio>

<div class="instructions">✨ Rasca con cariño y descubre la sorpresa ✨</div>

<div id="main-wrapper">
  <div class="grid" id="grid"></div>
</div>

<div id="win">
  <div class="big">🎒</div>
  <h1 style="color:#fb6f92;">Para ti, Mayli 💕</h1>
  <p style="font-size:18px;">
    No es solo una mochila…<br>
    es algo que pensé con amor para ti 💖<br>
    Gracias por existir 💕
  </p>
</div>

<script>
const grid = document.getElementById("grid");
const bgMusic = document.getElementById("bg-music");
const winSound = document.getElementById("win-sound");
const scratchSound = document.getElementById("scratch-sound");

bgMusic.volume = 0.15;
scratchSound.volume = 0.25;

let musicStarted = false;
let found = 0;

function startMusic(){
  if(!musicStarted){
    bgMusic.muted = false;
    bgMusic.play().catch(()=>{});
    musicStarted = true;
  }
}

const layout = ["P","X","X","P","X","P"].sort(()=>Math.random()-0.5);

function createCell(type){
  const cell = document.createElement("div");
  cell.className = "cell";

  const icon = document.createElement("div");
  icon.className = "icon";
  icon.textContent = type==="P"?"🎁":(Math.random()>0.5?"🐱":"🐩");

  const msg = document.createElement("div");
  msg.className="msg";
  msg.textContent=type==="P"?"¡Uno más!":"Sigue rascando";

  cell.append(icon,msg);

  const canvas=document.createElement("canvas");
  canvas.width=140; canvas.height=140;
  const ctx=canvas.getContext("2d");

  const g=ctx.createLinearGradient(0,0,140,140);
  g.addColorStop(0,"#fbc2eb");
  g.addColorStop(1,"#a6c1ee");
  ctx.fillStyle=g;
  ctx.fillRect(0,0,140,140);

  ctx.fillStyle="#fff";
  ctx.font="bold 14px Arial";
  ctx.textAlign="center";
  ctx.fillText("RASPA AQUÍ 💕",70,75);
  ctx.globalCompositeOperation="destination-out";

  let draw=false,reveal=false;

  function scratch(x,y){
    if(reveal) return;
    startMusic();
    if(scratchSound.paused) scratchSound.play();
    ctx.beginPath();
    ctx.arc(x,y,26,0,Math.PI*2);
    ctx.fill();
    check();
  }

  function check(){
    const d=ctx.getImageData(0,0,140,140).data;
    let t=0;
    for(let i=3;i<d.length;i+=4) if(d[i]===0) t++;
    if(t>140*140*0.4){
      reveal=true;
      canvas.style.opacity=0;
      setTimeout(()=>canvas.remove(),500);
      if(type==="P" && ++found===3) win();
    }
  }

  canvas.addEventListener("mousedown",()=>draw=true);
  window.addEventListener("mouseup",()=>draw=false);
  canvas.addEventListener("mousemove",e=>{
    if(!draw) return;
    const r=canvas.getBoundingClientRect();
    scratch(e.clientX-r.left,e.clientY-r.top);
  });

  canvas.addEventListener("touchstart",e=>{
    draw=true;
    const r=canvas.getBoundingClientRect();
    const t=e.touches[0];
    scratch(t.clientX-r.left,t.clientY-r.top);
  });

  canvas.addEventListener("touchmove",e=>{
    e.preventDefault();
    if(!draw) return;
    const r=canvas.getBoundingClientRect();
    const t=e.touches[0];
    scratch(t.clientX-r.left,t.clientY-r.top);
  },{passive:false});

  cell.appendChild(canvas);
  return cell;
}

function win(){
  winSound.play();
  document.getElementById("win").style.display="flex";
  const end=Date.now()+5000;
  (function frame(){
    confetti({particleCount:6,spread:60,origin:{x:0}});
    confetti({particleCount:6,spread:60,origin:{x:1}});
    if(Date.now()<end) requestAnimationFrame(frame);
  })();
}

layout.forEach(t=>grid.appendChild(createCell(t)));

setInterval(()=>{
  const h=document.createElement("div");
  h.className="heart";
  h.textContent=["💖","💕","🌸","✨"][Math.floor(Math.random()*4)];
  h.style.left=Math.random()*100+"vw";
  h.style.animationDuration=(3+Math.random()*3)+"s";
  document.body.appendChild(h);
  setTimeout(()=>h.remove(),5000);
},450);
</script>

</body>
</html>

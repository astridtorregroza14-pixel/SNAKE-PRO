
<!DOCTYPE html>  <html lang="es">  
<head>  
<meta charset="UTF-8">  
<title>Snake PRO+ v14</title>  <link rel="icon" href="icon.png">  
<link rel="apple-touch-icon" href="icon.png">  
<meta name="theme-color" content="#000000">  <style>  
body {margin:0; background:#0f172a; display:flex; justify-content:center; align-items:center; height:100vh; font-family:Arial; color:white; overflow:hidden; flex-direction:column;}  
canvas {background:#020617; border:3px solid #00f0ff; box-shadow:0 0 20px #00f0ff;}  
button {margin:5px; padding:10px 20px; border:none; border-radius:10px; background:#00f0ff; color:black; font-weight:bold; cursor:pointer; font-size:16px;}  
#menu, #skinMenu, #gameUI, #exchangeMenu, #infoMenu {display:flex; flex-direction:column; align-items:center;}  
#skinPreview, #menuPreview {margin-top:10px; border:2px solid #00f0ff;}  
.row {display:flex; gap:10px;}  
.btn {width:70px;height:70px;border-radius:50%; background:linear-gradient(#00f0ff,#0077ff); display:flex;align-items:center;justify-content:center; font-size:24px; cursor:pointer;}  
.controls {display:none; position:absolute; bottom:20px; width:100%; align-items:center; flex-direction:column;}  
.controls .row {display:flex; gap:10px; justify-content:center;}  
#redeem input {padding:5px; border-radius:5px; border:none; font-size:14px;}  
#redeem button {padding:5px 10px; font-size:14px;}  
</style>  </head>  <body>  <!-- MENÚ PRINCIPAL -->  <div id="menu">  
  <h1>🐍 Snake PRO+</h1>  
  <canvas id="menuPreview" width="120" height="80"></canvas>  
  <button onclick="startGame()">Jugar</button>  
  <button onclick="openSkinMenu()">Skins</button>  
  <button onclick="openExchangeMenu()">Cambios</button>  
  <button onclick="openInfo()">ℹ️ Info</button>    <div id="redeem">  
    <h3>Canjear código</h3>  
    <input type="text" id="codeInput" placeholder="Ingresa tu código">  
    <button onclick="redeemCode()">Canjear</button>  
    <div id="codeMessage" style="margin-top:5px; color:yellow;"></div>  
  </div>  
</div>  <!-- APARTADO DE INFO -->   <!-- APARTADO DE INFO -->
<div id="infoMenu" style="display:none; flex-direction:column; align-items:center;">
  <h2>ℹ️ Información</h2>
  <p>Versión actual: 1.0</p>
  <p>Próxima actualización: 1.1</p>
  <p>Actualización en: <span id="updateCountdown">Cargando...</span></p>
  
  <div style="margin-top:10px; background:#001122; padding:10px; border:2px solid #00f0ff; border-radius:10px; text-align:center;">
    <h3>🎉 ¡Detalles importantes! 🎉</h3>
    <p>• 5 skins nuevas</p>
    <p>• De esas 5 skins, hay una nueva calidad</p>
    <p>• La nueva skin ultra legendaria</p>
    <p>• Próximamente: opciones, misiones y tienda</p>
    <div style="font-size:30px; margin-top:5px;">🛍️✨</div>
  </div>

  <button onclick="closeInfo()" style="margin-top:15px;">🏠 Volver</button>
</div>
</div>  <!-- MENU DE SKINS -->  <div id="skinMenu" style="display:none;">  
  <h2>Elige tu Skin</h2>  
  <div id="skinButtons"></div>  
  <canvas id="skinPreview" width="100" height="100"></canvas>  
  <button onclick="closeSkinMenu()">🏠 Volver</button>  
</div>  <!-- MENU DE INTERCAMBIO -->  <div id="exchangeMenu" style="display:none; flex-direction:column; align-items:center;">  
  <h2>Intercambiar Puntos por Gemas</h2>  
  <div>Puntos: <span id="exchangeScore">0</span></div>  
  <div>Gemas: <span id="gems">0</span></div>  
  <button onclick="exchangePoints()">Intercambiar 10 puntos → 5 gemas</button>  
  <button onclick="closeExchangeMenu()">🏠 Volver</button>  
</div>  <!-- UI DEL JUEGO -->  <div id="gameUI" style="display:none;">  
  <div>Puntos: <span id="score">0</span></div>  
  <div>🏆 Récord: <span id="high">0</span></div>  
  <button onclick="togglePause()">⏸️ Pausa</button>  
  <button onclick="restart()">🔄 Reintentar</button>  
  <button onclick="goHome()">🏠 Menú</button>  
</div>  <canvas id="game" width="400" height="400" style="display:none;"></canvas>

<!-- CONTROLES TÁCTILES -->  <div class="controls">  
  <div class="btn" id="up">⬆️</div>  
  <div class="row">  
    <div class="btn" id="left">⬅️</div>  
    <div class="btn" id="down">⬇️</div>  
    <div class="btn" id="right">➡️</div>  
  </div>  
</div>  <script>  
// --- VARIABLES ---  
const canvas = document.getElementById("game");  
const ctx = canvas.getContext("2d");  
const grid = 20;  
let snake, dx, dy, nextDx, nextDy, food, score, speed, gameOver=false, paused=false;  
let goldenApple=null, applesEaten=0, flashTimer=0;  
  
const skins = [  
  {name:"Clásico", colors:["lime","green"], requiredScore:0, unlocked:true},  
  {name:"Neón", colors:["#00ffff","#ff00ff"], requiredScore:30, unlocked:false},  
  {name:"Fuego", colors:["red","orange"], requiredScore:40, unlocked:false},  
  {name:"Hielo", colors:["#00bfff","#ffffff"], requiredScore:50, unlocked:false},  
  {name:"Arcoíris", colors:["rainbow","rainbow"], requiredScore:100, unlocked:false}  
];  
let currentSkin = localStorage.getItem("snakeSkin") || "Clásico";  
score = parseInt(localStorage.getItem("score")) || 0;  
let gems = parseInt(localStorage.getItem("gems")) || 0;  
let highScore = parseInt(localStorage.getItem("snakeHigh")) || 0;  
document.getElementById("high").innerText = highScore;  
  
// --- MENÚ PRINCIPAL ---  
const menuCanvas = document.getElementById("menuPreview");  
const menuCtx = menuCanvas.getContext("2d");  
let menuSnake = [{x:10,y:30},{x:30,y:30},{x:50,y:30}];  
let menuDir = 2;  
function drawMenuSnake(){  
  menuCtx.clearRect(0,0,menuCanvas.width,menuCanvas.height);  
  const skin = skins.find(s=>s.name===currentSkin);  
  menuSnake.forEach((s,i)=>{  
    let color;  
    if(skin.colors[0]==="rainbow"){ color=`hsl(${(Date.now()/10+i*30)%360},100%,50%)`; }  
    else color=i===0?skin.colors[1]:skin.colors[0];  
    menuCtx.fillStyle=color;  
    menuCtx.fillRect(s.x,s.y,15,15);  
  });  
  menuSnake.unshift({x:menuSnake[0].x+menuDir, y:menuSnake[0].y});  
  menuSnake.pop();  
  if(menuSnake[0].x>=menuCanvas.width-15||menuSnake[0].x<=0) menuDir*=-1;  
  requestAnimationFrame(drawMenuSnake);  
}  
drawMenuSnake();  
  
function startGame(){  
  document.getElementById("menu").style.display="none";  
  document.getElementById("gameUI").style.display="flex";  
  canvas.style.display="block";  
  document.querySelector(".controls").style.display="flex";  
  init();  
}  
  
// --- SKINS ---  
function openSkinMenu(){  
  document.getElementById("menu").style.display="none";  
  document.getElementById("skinMenu").style.display="flex";  
  renderSkinButtons();  
  renderSkinPreview(currentSkin);  
}  
function closeSkinMenu(){ document.getElementById("skinMenu").style.display="none"; document.getElementById("menu").style.display="flex"; }  
  
function renderSkinButtons(){  
  const container=document.getElementById("skinButtons"); container.innerHTML="";  
  skins.forEach(s=>{  
    const unlocked=s.unlocked || s.requiredScore===0;  
    const btn=document.createElement("button");  
    btn.innerText = unlocked? `${s.name} ✅ (${s.requiredScore} pts)`:`${s.name} 🔒 ${s.requiredScore} pts`;  
    btn.disabled=!unlocked;  
    btn.onclick=()=>{ if(unlocked){ currentSkin=s.name; localStorage.setItem("snakeSkin",currentSkin); renderSkinButtons(); renderSkinPreview(currentSkin); } };  
    container.appendChild(btn);  
  });  
}  
  
function renderSkinPreview(skinName){  
  const preview=document.getElementById("skinPreview");  
  const ctxP=preview.getContext("2d");  
  ctxP.clearRect(0,0,preview.width,preview.height);  
  const skin=skins.find(s=>s.name===skinName);  
  [{x:10,y:40},{x:30,y:40},{x:50,y:40}].forEach((b,i)=>{  
    let color;  
    if(skin.colors[0]==="rainbow"){ color=`hsl(${(i*120+Date.now()/10)%360},100%,50%)`; }  
    else color=i===0?skin.colors[1]:skin.colors[0];  
    ctxP.fillStyle=color; ctxP.fillRect(b.x,b.y,15,15);  
  });  
}  
  
// --- CÓDIGOS ---  
function redeemCode(){  
  const code=document.getElementById("codeInput").value.trim().toUpperCase();  
  const message=document.getElementById("codeMessage");  
  if(localStorage.getItem("redeemed_"+code)){ message.innerText="❌ Este código ya fue canjeado"; return; }  
  
  const codes={  
    "RAINBOW50": ()=>{ skins.find(s=>s.name==="Arcoíris").unlocked=true; localStorage.setItem("snakeSkin","Arcoíris"); message.innerText="🌈 ¡Skin Arcoíris desbloqueada!"; renderSkinButtons(); renderSkinPreview("Arcoíris"); flashScreen(); },  
    "BONUS20": ()=>{ score+=20; localStorage.setItem("score",score); updateUI(); message.innerText="💎 20 puntos añadidos!"; flashScreen(); },  
    "BONUS50": ()=>{ score+=50; localStorage.setItem("score",score); updateUI(); message.innerText="💎 50 puntos añadidos!"; flashScreen(); }  
  };  
  
  if(codes[code]){ codes[code](); localStorage.setItem("redeemed_"+code,"true"); }  
  else message.innerText="❌ Código inválido";  
  document.getElementById("codeInput").value="";  
}  
  
// --- INTERCAMBIO ---  
function openExchangeMenu(){ document.getElementById("menu").style.display="none"; document.getElementById("exchangeMenu").style.display="flex"; updateExchangeUI(); }  
function closeExchangeMenu(){ document.getElementById("exchangeMenu").style.display="none"; document.getElementById("menu").style.display="flex"; }  
function updateExchangeUI(){ document.getElementById("exchangeScore").innerText=score; document.getElementById("gems").innerText=gems; }  
  
function exchangePoints(){  
  if(score>=10){  
    const times=Math.floor(score/10);  
    const used=times*10;  
    const gemsEarned=times*5;  
    score-=used; gems+=gemsEarned;  
    localStorage.setItem("score",score);  
    localStorage.setItem("gems",gems);  
    updateUI(); updateExchangeUI();  
    flashScreen();  
    alert(`💎 Has recibido ${gemsEarned} gemas por ${used} puntos!`);  
  } else { alert("❌ No tienes suficientes puntos para intercambiar"); }  
}  
  
// --- INFO ---  
function openInfo(){ document.getElementById("menu").style.display="none"; document.getElementById("infoMenu").style.display="flex"; }  
function closeInfo(){ document.getElementById("infoMenu").style.display="none"; document.getElementById("menu").style.display="flex"; }  
  
// --- FLASH ---  
function flashScreen(){ flashTimer=20; }  
function drawFlash(){ if(flashTimer>0){ ctx.fillStyle=`rgba(255,255,0,${flashTimer/20*0.5})`; ctx.fillRect(0,0,canvas.width,canvas.height); flashTimer--; } }  
  
// --- CONTADOR ---  
const updateDate = new Date();   
updateDate.setDate(updateDate.getDate() + 10);  
function updateCountdown(){  
  const now = new Date();  
  let diff = Math.max(0, updateDate - now);  
  const days = Math.floor(diff / (1000*60*60*24));  
  diff -= days * 1000*60*60*24;  
  const hours = Math.floor(diff / (1000*60*60));  
  diff -= hours * 1000*60*60;  
  const minutes = Math.floor(diff / (1000*60));  
  diff -= minutes * 1000*60;  
  const seconds = Math.floor(diff / 1000);  
  document.getElementById("updateCountdown").innerText = `${days}d ${hours}h ${minutes}m ${seconds}s`;  
}  
setInterval(updateCountdown,1000);  
updateCountdown();  
  
// --- JUEGO ---  
function init(){  
  snake=[{x:200,y:200}]; dx=grid; dy=0; nextDx=dx; nextDy=dy;  
  food=randomFood(); goldenApple=null; applesEaten=0; speed=200; gameOver=false; paused=false;  
  updateUI();  
}  
function randomFood(){ return { x: Math.floor(Math.random()*20)*grid, y: Math.floor(Math.random()*20)*grid }; }  
function drawSnake(){  
  const skin=skins.find(s=>s.name===currentSkin);  
  snake.forEach((s,i)=>{  
    let color;  
    if(skin.colors[0]==="rainbow"){ color=`hsl(${(Date.now()/10+i*30)%360},100%,50%)`; }  
    else color=i===0?skin.colors[1]:skin.colors[0];  
    ctx.fillStyle=color; ctx.fillRect(s.x,s.y,grid-2,grid-2);  
  });  
}  
function gameLoop(){  
  if(gameOver||paused){ draw(); requestAnimationFrame(gameLoop); return; }  
  setTimeout(()=>requestAnimationFrame(gameLoop),speed);  
  dx=nextDx; dy=nextDy;  
  const head={x:snake[0].x+dx, y:snake[0].y+dy};  
  snake.unshift(head);  
  
  if(head.x===food.x && head.y===food.y){  
    score++; applesEaten++; food=randomFood();  
    if(applesEaten%10===0) goldenApple=randomFood();  
    localStorage.setItem("score",score);  
    updateUI();  
  } else { snake.pop(); }  
  
  if(goldenApple && head.x===goldenApple.x && head.y===goldenApple.y){  
    score+=10; goldenApple=null; localStorage.setItem("score",score); flashScreen(); updateUI();  
  }  
  
  if(head.x<0||head.x>=canvas.width||head.y<0||head.y>=canvas.height||snake.slice(1).some(s=>s.x===head.x&&s.y===head.y)){  
    gameOver=true; if(score>highScore){ highScore=score; localStorage.setItem("snakeHigh",highScore); }  
  }  
  draw();  
}  
function draw(){  
  ctx.clearRect(0,0,canvas.width,canvas.height);  
  ctx.fillStyle="cyan"; ctx.fillRect(food.x,food.y,grid,grid);  
  if(goldenApple){ ctx.fillStyle="gold"; ctx.fillRect(goldenApple.x,goldenApple.y,grid,grid); }  
  drawSnake();  
  drawFlash();  
  if(paused){ ctx.fillStyle="white"; ctx.font="20px Arial"; ctx.fillText("⏸️ PAUSA",150,200);}  
  if(gameOver){ ctx.fillStyle="red"; ctx.font="25px Arial"; ctx.fillText("💀 GAME OVER",120,200);}  
}  
function updateUI(){  
  document.getElementById("score").innerText=score;  
  document.getElementById("high").innerText=highScore;  
  skins.forEach(s=>{ if(score>=s.requiredScore) s.unlocked=true; });  
}  
  
// --- CONTROLES ---  
document.addEventListener("keydown",e=>{  
  if(e.key==="ArrowUp" && dy===0){ nextDx=0; nextDy=-grid; }  
  if(e.key==="ArrowDown" && dy===0){ nextDx=0; nextDy=grid; }  
  if(e.key==="ArrowLeft" && dx===0){ nextDx=-grid; nextDy=0; }  
  if(e.key==="ArrowRight" && dx===0){ nextDx=grid; nextDy=0; }  
});  
  
["up","down","left","right"].forEach(dir=>{  
  document.getElementById(dir).ontouchstart = e=>{  
    e.preventDefault();  
    if(dir==="up"&&dy===0) nextDx=0,nextDy=-grid;  
    if(dir==="down"&&dy===0) nextDx=0,nextDy=grid;  
    if(dir==="left"&&dx===0) nextDx=-grid,nextDy=0;  
    if(dir==="right"&&dx===0) nextDx=grid,nextDy=0;  
  }  
});  
  
function togglePause(){ paused=!paused; }  
function restart(){ init(); }  
function goHome(){  
  document.getElementById("gameUI").style.display="none";   
  canvas.style.display="none";   
  document.querySelector(".controls").style.display="none";   
  paused=false;   
  gameOver=false;  
  document.getElementById("menu").style.display="flex";  
}  
  
// Iniciar loop del juego  
gameLoop();  
</script>  </body>  
</html>  
<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no">
<meta name="theme-color" content="#f8eddb">
<title>ARROW RUSH</title>
<style>
:root{--cream:#f8eddb;--board:#f1e2cc;--brown:#64452f;--muted:#ddc6a7;--ink:#503723}
*{box-sizing:border-box}body{margin:0;background:#e9dcc9;color:var(--ink);font-family:system-ui,-apple-system,Segoe UI,Roboto,sans-serif;display:flex;justify-content:center;min-height:100vh}
.app{width:min(100%,480px);background:var(--cream);min-height:100vh;padding:16px;display:flex;flex-direction:column}
header{display:flex;justify-content:space-between;align-items:center;margin-bottom:16px}.eyebrow{font-size:10px;letter-spacing:3px;font-weight:900;opacity:.7}.brand{font-size:29px;line-height:1.15;font-weight:950;letter-spacing:1px}.brand span{color:#a77b4f}
button{font:inherit;cursor:pointer;touch-action:manipulation}.sound{width:42px;height:42px;border:1px solid var(--muted);background:#f4e5d0;border-radius:50%;font-size:20px;color:var(--brown)}
.stats{display:grid;grid-template-columns:repeat(3,1fr);gap:8px;margin-bottom:12px}.stat{background:#efe0c8;border-radius:12px;padding:9px 5px;text-align:center}.stat small{font-size:10px;letter-spacing:1px;font-weight:850}.stat strong{display:block;font-size:21px;font-variant-numeric:tabular-nums}
.track{height:7px;border-radius:9px;background:#e5d3ba;overflow:hidden;margin-bottom:15px}.progress{height:100%;width:100%;background:#98704a;transition:width .2s linear,background .2s}
.boardwrap{position:relative;background:var(--board);border:1px solid var(--muted);border-radius:16px;padding:10px;flex:0 0 auto}
.board{display:grid;grid-template-columns:repeat(6,minmax(0,1fr));gap:5px;width:100%;aspect-ratio:1}
.arrow{display:flex;align-items:center;justify-content:center;min-width:0;min-height:0;padding:0;border:1.5px solid #bda68b;background:#f5e8d4;color:var(--brown);border-radius:7px;font-size:clamp(19px,5.7vw,29px);font-weight:950;line-height:1;box-shadow:0 2px 0 #d7c3a6;transition:transform .12s}
.arrow:active{transform:scale(.91)}.arrow.gone{visibility:hidden;pointer-events:none;box-shadow:none}.arrow:focus-visible{outline:3px solid #b58148;outline-offset:1px}
.overlay{position:absolute;inset:0;background:#f8eddbed;border-radius:15px;display:flex;flex-direction:column;align-items:center;justify-content:center;text-align:center;padding:20px;z-index:2}.overlay .symbol{font-size:45px;margin-bottom:4px}.overlay h1{font-size:26px;margin:0;font-weight:950}.overlay p{font-size:13px;line-height:1.6;max-width:260px;margin:9px 0 18px}
.primary{background:var(--brown);color:#fff6e8;border:0;border-radius:12px;padding:12px 19px;font-weight:850}
.bottom{display:flex;justify-content:space-between;align-items:center;gap:8px;margin-top:15px}.how{font-size:12px;line-height:1.6}.how span{opacity:.72}.restart{background:transparent;border:1px solid #c9b294;border-radius:10px;padding:9px 12px;font-weight:800;font-size:12px;color:var(--brown)}.message{text-align:center;font-size:12px;min-height:30px;margin-top:10px;opacity:.85}
@media(min-height:760px){.app{padding-top:24px}.boardwrap{margin-top:3px}.bottom{margin-top:20px}}
</style>
</head>
<body>
<main class="app">
<header><div><div class="eyebrow">BRAIN PUZZLE</div><div class="brand">ARROW <span>RUSH</span></div></div><button id="sound" class="sound" aria-label="Toggle sound">♪</button></header>
<section class="stats"><div class="stat"><small>LEVEL</small><strong id="level">1</strong></div><div class="stat"><small>TIME LEFT</small><strong id="clock">60</strong></div><div class="stat"><small>ARROWS LEFT</small><strong id="left">18</strong></div></section>
<div class="track"><div id="progress" class="progress"></div></div>
<section class="boardwrap"><div id="board" class="board" role="group" aria-label="Arrow puzzle board"></div><div id="overlay" class="overlay"><div class="symbol">↗</div><h1 id="ovtitle">READY?</h1><p id="ovtext">Tap an arrow with a clear path to send it flying off the board. Clear every arrow before time runs out!</p><button id="start" class="primary">START GAME →</button></div></section>
<div class="bottom"><div class="how"><b>HOW TO PLAY</b><br><span>Clear path = arrow escapes</span></div><button id="restart" class="restart">↻ RESTART</button></div>
<div id="message" class="message" aria-live="polite">Clear all arrows to unlock the next level.</div>
</main>
<script>
(()=>{'use strict';
const $=id=>document.getElementById(id),dirs=[{dx:0,dy:-1,g:'↑',name:'up'},{dx:1,dy:0,g:'→',name:'right'},{dx:0,dy:1,g:'↓',name:'down'},{dx:-1,dy:0,g:'←',name:'left'}];
let level=1,seconds=60,remaining=0,arrows=[],active=false,started=false,timer=null,runId=0,sound=true;
function config(){return{n:Math.min(6+Math.floor((level-1)/2),9),count:Math.min(18+(level-1)*5,64),time:Math.max(24,60-(level-1)*4)}}
function shuffle(a){for(let i=a.length-1;i>0;i--){const j=Math.floor(Math.random()*(i+1));[a[i],a[j]]=[a[j],a[i]]}return a}
function clearRay(a,grid,n){let x=a.x+a.d.dx,y=a.y+a.d.dy;while(x>=0&&y>=0&&x<n&&y<n){if(grid[y*n+x])return false;x+=a.d.dx;y+=a.d.dy}return true}
function makePuzzle(){const c=config(),n=c.n,grid=Array(n*n).fill(null),items=[],candidates=[];for(let y=0;y<n;y++)for(let x=0;x<n;x++)for(const d of dirs)candidates.push({x,y,d});shuffle(candidates);
for(const a of candidates){if(items.length>=c.count)break;const k=a.y*n+a.x;if(grid[k]||!clearRay(a,grid,n))continue;grid[k]=a;items.push({...a,id:items.length,alive:true})}
arrows=items;remaining=items.length;seconds=c.time;$('board').style.gridTemplateColumns=`repeat(${n},minmax(0,1fr))`;$('board').innerHTML='';
for(let i=0;i<n*n;i++){const a=grid[i],b=document.createElement('button');b.type='button';b.className='arrow'+(!a?' gone':'');b.setAttribute('aria-label',a?`Arrow pointing ${a.d.name}`:'Empty space');b.textContent=a?a.d.g:'';if(a){b.dataset.id=a.id;b.addEventListener('click',()=>tap(a.id,b))}$('board').appendChild(b)}updateStats();$('message').textContent='Find a clear path. Watch the timer!'}
function updateStats(){$('level').textContent=level;$('clock').textContent=seconds;$('left').textContent=remaining;$('progress').style.width=(seconds/Math.max(1,config().time)*100)+'%';$('progress').style.background=seconds<=10?'#b54836':'#98704a';$('clock').style.color=seconds<=10?'#b54836':'#503723'}
function beep(f,d){if(!sound)return;try{const C=window.AudioContext||window.webkitAudioContext;if(!C)return;const c=new C(),o=c.createOscillator(),g=c.createGain();o.frequency.value=f;g.gain.value=.035;o.connect(g);g.connect(c.destination);o.start();g.gain.exponentialRampToValueAtTime(.001,c.currentTime+d);o.stop(c.currentTime+d);o.onended=()=>c.close()}catch(e){}}
function tap(id,btn){if(!active)return;const a=arrows.find(z=>z.id===id&&z.alive);if(!a)return;const n=config().n,grid=Array(n*n).fill(null);for(const z of arrows)if(z.alive&&z.id!==id)grid[z.y*n+z.x]=z;
if(!clearRay(a,grid,n)){$('message').textContent='Path blocked! Clear the arrows in front first.';btn.animate([{transform:'translateX(-3px)'},{transform:'translateX(3px)'},{transform:'translateX(0)'}],{duration:150});beep(180,.045);return}
a.alive=false;remaining--;btn.classList.add('gone');btn.disabled=true;btn.setAttribute('aria-label','Arrow cleared');beep(520,.045);updateStats();if(!remaining)win()}
function show(title,txt,label,fn){active=false;clearInterval(timer);timer=null;$('ovtitle').textContent=title;$('ovtext').textContent=txt;$('start').textContent=label;$('overlay').style.display='flex';$('start').onclick=fn}
function win(){active=false;clearInterval(timer);timer=null;$('message').textContent=`Level ${level} completed!`;show('LEVEL CLEARED!',`Great job! You cleared every arrow with ${seconds} seconds left. Ready for a tougher puzzle?`,'NEXT LEVEL →',()=>{level++;begin()})}
function lose(){active=false;clearInterval(timer);timer=null;show('TIME’S UP!','No worries! Try Level '+level+' again and find the clear paths faster.','TRY AGAIN ↻',()=>begin())}
function begin(){started=true;runId++;makePuzzle();active=true;$('overlay').style.display='none';clearInterval(timer);const thisRun=runId;timer=setInterval(()=>{if(!active||thisRun!==runId)return;seconds=Math.max(0,seconds-1);updateStats();if(seconds<=0)lose()},1000)}
$('start').onclick=()=>begin();$('restart').onclick=()=>{level=1;begin()};$('sound').onclick=()=>{sound=!sound;$('sound').textContent=sound?'♪':'♩';$('message').textContent=sound?'Sound effects enabled.':'Sound effects muted.'};makePuzzle();
})();
</script>
</body></html>
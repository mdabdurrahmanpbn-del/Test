<!DOCTYPE html>
<html>
<head>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Writing Practice</title>

<!-- ✅ বাংলা ফন্ট FIX -->
<link href="https://fonts.googleapis.com/css2?family=Noto+Sans+Bengali:wght@400;700&display=swap" rel="stylesheet">

<style>
body{
font-family:'Noto Sans Bengali', Arial, sans-serif;
margin:0;
background:#f0f8ff;
text-align:center;
}

header{
background:#2196f3;
color:white;
padding:10px;
display:flex;
justify-content:space-between;
align-items:center;
}

.homeBtn{
background:#ff5722;
color:white;
border:none;
padding:8px 12px;
border-radius:8px;
}

.category button{
margin:5px;
padding:8px 10px;
border:none;
border-radius:8px;
background:#ff9800;
color:white;
}

.letters{
display:flex;
flex-wrap:wrap;
justify-content:center;
}

.letters button{
margin:4px;
padding:8px;
border:none;
border-radius:6px;
background:#4caf50;
color:white;
}

.canvasWrap{
position:relative;
width:95%;
max-width:500px;
height:300px;
margin:10px auto;
border:3px solid black;
border-radius:10px;
background:white;
}

canvas{
position:absolute;
left:0;
top:0;
width:100%;
height:100%;
touch-action:none;
}

.tools{ margin-top:5px; }

.toolsRow{
display:flex;
justify-content:center;
flex-wrap:wrap;
gap:6px;
margin-top:5px;
}

.sliderRow{
display:flex;
justify-content:center;
gap:15px;
flex-wrap:wrap;
margin-top:5px;
}

.sliderBox{
display:flex;
flex-direction:column;
align-items:center;
font-size:14px;
}

.palette button{
width:28px;
height:28px;
border:none;
border-radius:50%;
margin:3px;
}

#resultBox{
position:fixed;
top:50%;
left:50%;
transform:translate(-50%,-50%) scale(0);
background:white;
padding:20px;
border-radius:15px;
box-shadow:0 0 15px rgba(0,0,0,0.3);
transition:0.3s;
font-size:22px;
z-index:1000;
}
#resultBox.show{
transform:translate(-50%,-50%) scale(1);
}
</style>
</head>

<body>

<header>
<h3>✍️ Writing Practice</h3>
<button class="homeBtn" onclick="goHome()">🏠</button>
</header>

<div class="category">
<button onclick="loadSet('eng')">ABC</button>
<button onclick="loadSet('bn_vowel')">স্বর</button>
<button onclick="loadSet('bn_con')">ব্যঞ্জন</button>
<button onclick="loadSet('num')">123</button>
<button onclick="loadSet('bn_num')">১২৩</button>
</div>

<div class="letters" id="letters"></div>

<div>📊 Progress: <span id="progress">0%</span> | ⭐ Stars: <span id="stars">0</span></div>

<div class="canvasWrap">
<canvas id="guideCanvas"></canvas>
<canvas id="drawCanvas"></canvas>
</div>

<div class="tools">

<input type="color" id="color" value="#000000">

<div class="palette">
<button onclick="setColor('#000')" style="background:black;"></button>
<button onclick="setColor('#f00')" style="background:red;"></button>
<button onclick="setColor('#0f0')" style="background:green;"></button>
<button onclick="setColor('#00f')" style="background:blue;"></button>
</div>

<div class="sliderRow">
<div class="sliderBox">
🔤 Letter Size  
<input type="range" id="letterSize" min="100" max="300" value="200" oninput="drawGuide()">
</div>

<div class="sliderBox">
✏️ Pen Size  
<input type="range" id="penSize" min="2" max="40" value="8">
</div>
</div>

<div class="toolsRow">
<button onclick="setPen()">✏️</button>
<button onclick="setEraser()">🧽</button>
<button onclick="clearCanvas()">🗑</button>
<button onclick="nextLetter()">➡️</button>
<button onclick="togglePreview()">👁️</button>
<button onclick="saveImage()">📸 Save</button>
<button onclick="checkWriting()">✅ Check</button>
<button onclick="toggleRainbow()">🌈 Rainbow</button>
</div>

</div>

<div id="resultBox"></div>

<script>

let guideCanvas=document.getElementById("guideCanvas");
let guideCtx=guideCanvas.getContext("2d");
let drawCanvas=document.getElementById("drawCanvas");
let ctx=drawCanvas.getContext("2d");

guideCanvas.width = drawCanvas.width = 500;
guideCanvas.height = drawCanvas.height = 300;

/* ✅ Smooth Pen */
ctx.lineJoin="round";
ctx.lineCap="round";

let previewMode=false;
let rainbow=false;
let hue=0;

let correct=0;
let total=0;
let stars=0;

function clickSound(){
new Audio("https://actions.google.com/sounds/v1/ui/click.ogg").play();
}

function goHome(){ clickSound(); window.location.href="index.html"; }
function setColor(c){ clickSound(); document.getElementById("color").value=c; }

let eng="ABCDEFGHIJKLMNOPQRSTUVWXYZ".split("");
let bn_vowel=["অ","আ","ই","ঈ","উ","ঊ","এ","ঐ","ও","ঔ"];
let bn_con=["ক","খ","গ","ঘ","ঙ","চ","ছ","জ","ঝ","ঞ","ট","ঠ","ড","ঢ","ণ","ত","থ","দ","ধ","ন","প","ফ","ব","ভ","ম","য","র","ল","শ","ষ","স","হ"];
let num="1234567890".split("");
let bn_num=["১","২","৩","৪","৫","৬","৭","৮","৯","০"];

let currentSet=eng;
let index=0;

function loadSet(type){
clickSound();
if(type=="eng") currentSet=eng;
if(type=="bn_vowel") currentSet=bn_vowel;
if(type=="bn_con") currentSet=bn_con;
if(type=="num") currentSet=num;
if(type=="bn_num") currentSet=bn_num;

index=0;
showLetters();
clearCanvas();
speak();
}

function showLetters(){
let div=document.getElementById("letters");
div.innerHTML="";
currentSet.forEach((l,i)=>{
let btn=document.createElement("button");
btn.innerText=l;
btn.onclick=()=>{ clickSound(); index=i; clearCanvas(); speak(); };
div.appendChild(btn);
});
}

function drawGuide(){
guideCtx.clearRect(0,0,500,300);
let size=document.getElementById("letterSize").value;
guideCtx.font="bold "+size+"px 'Noto Sans Bengali'";
guideCtx.textAlign="center";
guideCtx.textBaseline="middle";
guideCtx.fillStyle="rgba(0,0,0,0.15)";
guideCtx.fillText(currentSet[index],250,150);
}

function nextLetter(){ clickSound(); index=(index+1)%currentSet.length; clearCanvas(); speak(); }
function togglePreview(){ previewMode=!previewMode; guideCanvas.style.display = previewMode?"none":"block"; }

function speak(){
let msg=new SpeechSynthesisUtterance(currentSet[index]);

if(/[অ-হ]/.test(currentSet[index])) msg.lang="bn-BD";
else msg.lang="en-US";

speechSynthesis.cancel();
speechSynthesis.speak(msg);
}

/* ✅ Easy Check */
function checkWriting(){
let data = ctx.getImageData(0,0,500,300).data;
let pixels=0;

for(let i=0;i<data.length;i+=4){ if(data[i]!==0) pixels++; }

total++;
let ok=pixels>1000;

if(ok){
correct++;
showResult("⭐ Great!");
playSound(true);
launchBalloons();
nextLetter();

if(correct%5===0){
stars++;
document.getElementById("stars").innerText=stars;
}

}else{
showResult("❌ Try Again!");
playSound(false);
clearCanvas();
}

updateProgress();
}

function launchBalloons(){
for(let i=0;i<10;i++){
let b=document.createElement("div");
b.className="balloon";
b.style.left=Math.random()*100+"%";
b.innerText="🎈";
document.body.appendChild(b);
setTimeout(()=>b.remove(),3000);
}
}

function updateProgress(){
let percent=Math.floor((correct/total)*100);
document.getElementById("progress").innerText=percent+"%";
}

function showResult(t){
let box=document.getElementById("resultBox");
box.innerText=t;
box.classList.add("show");
setTimeout(()=>box.classList.remove("show"),2000);
}

function playSound(ok){
let audio=new Audio(ok?
"https://actions.google.com/sounds/v1/cartoon/clang_and_wobble.ogg":
"https://actions.google.com/sounds/v1/cartoon/wood_plank_flicks.ogg");
audio.play();
}

function toggleRainbow(){ rainbow=!rainbow; }

let drawing=false,lastX=0,lastY=0,erasing=false;

function getPos(e){
let rect=drawCanvas.getBoundingClientRect();
return {
x:(e.clientX-rect.left)*(drawCanvas.width/rect.width),
y:(e.clientY-rect.top)*(drawCanvas.height/rect.height)
};
}

drawCanvas.addEventListener("pointerdown",e=>{
drawing=true;
let p=getPos(e);
lastX=p.x;
lastY=p.y;
ctx.beginPath();
ctx.moveTo(lastX,lastY);
});

drawCanvas.addEventListener("pointermove",e=>{
if(!drawing) return;

let p=getPos(e);

ctx.lineTo(p.x,p.y);
ctx.lineWidth=document.getElementById("penSize").value;
ctx.lineCap="round";

if(rainbow){
ctx.strokeStyle=`hsl(${hue},100%,50%)`;
hue+=5;
}else{
ctx.strokeStyle=erasing?"white":document.getElementById("color").value;
}

ctx.stroke();

lastX=p.x;
lastY=p.y;
});

drawCanvas.addEventListener("pointerup",()=>drawing=false);
drawCanvas.addEventListener("pointerleave",()=>drawing=false);

function clearCanvas(){ ctx.clearRect(0,0,500,300); drawGuide(); }
function setPen(){ clickSound(); erasing=false; }
function setEraser(){ clickSound(); erasing=true; }

function saveImage(){
let temp=document.createElement("canvas");
temp.width=500; temp.height=300;
let t=temp.getContext("2d");
t.fillStyle="white";
t.fillRect(0,0,500,300);
t.drawImage(guideCanvas,0,0);
t.drawImage(drawCanvas,0,0);
let link=document.createElement("a");
link.download="writing.png";
link.href=temp.toDataURL();
link.click();
}

loadSet("eng");

</script>

</body>
</html>

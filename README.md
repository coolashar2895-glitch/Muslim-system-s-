<!DOCTYPE html>
<html>
<head>
<title>Shadow System</title>

<style>
body {
  background: radial-gradient(circle, #0a0a0a, #000);
  color: #00ffe1;
  font-family: Arial;
  text-align: center;
}

.container {
  border: 1px solid #00ffe1;
  padding: 15px;
  margin: 10px auto;
  width: 320px;
  border-radius: 10px;
  box-shadow: 0 0 15px #00ffe1;
}

button {
  background: black;
  color: #00ffe1;
  border: 1px solid #00ffe1;
  padding: 5px;
  margin: 3px;
  cursor: pointer;
}

input, select {
  margin: 5px;
  padding: 5px;
  background: black;
  color: #00ffe1;
  border: 1px solid #00ffe1;
}

.quest {
  border: 1px solid #00ffe1;
  margin: 5px;
  padding: 5px;
}

canvas {
  margin-top: 10px;
}
</style>
</head>

<body>

<h1>⚔️ SHADOW SYSTEM ⚔️</h1>

<div class="container">
<h2 id="rank">Rank: E</h2>
<h3 id="xp">XP: 0</h3>
<p id="streak">🔥 Streak: 0</p>

<p>Iman: <span id="iman">50</span></p>
<p>Strength: <span id="strength">50</span></p>
<p>Discipline: <span id="discipline">50</span></p>
<p>Knowledge: <span id="knowledge">50</span></p>
<p><b>Total Power: <span id="total">50</span></b></p>
</div>

<canvas id="chart" width="300" height="300"></canvas>

<h2>⚔️ Custom Quests</h2>
<input id="questName" placeholder="Quest name">
<input id="xpValue" placeholder="XP">

<select id="type">
<option value="iman">Iman</option>
<option value="strength">Strength</option>
<option value="discipline">Discipline</option>
<option value="knowledge">Knowledge</option>
</select>

<button onclick="addQuest()">Add</button>

<div id="quests"></div>

<h2>🏆 Titles</h2>
<div id="titles"></div>

<script>

// RANK SYSTEM
let ranks = [
  {name:"E", need:300},
  {name:"D", need:800},
  {name:"D+", need:1500},
  {name:"C", need:2500},
  {name:"C+", need:4000},
  {name:"B", need:6000},
  {name:"B+", need:9000},
  {name:"A", need:13000},
  {name:"A+", need:18000},
  {name:"S", need:25000},
  {name:"SSS", need:999999}
];

let currentRank = 0;

// DATA
let xp = 0, dailyXP = 0, streak = 0, lastDate = "";
let stats = {iman:50,strength:50,discipline:50,knowledge:50};
let titles = [];
let questList = [];

// SAVE / LOAD
function save(){
  localStorage.setItem("system", JSON.stringify({
    xp,dailyXP,streak,lastDate,stats,titles,currentRank,questList
  }));
}

function load(){
  let d = JSON.parse(localStorage.getItem("system"));
  if(d){
    xp=d.xp; dailyXP=d.dailyXP; streak=d.streak;
    lastDate=d.lastDate; stats=d.stats;
    titles=d.titles; currentRank=d.currentRank;
    questList=d.questList || [];
  }
  checkDay();
  questList.forEach(renderQuest);
}

// DAILY RESET
function checkDay(){
  let today = new Date().toDateString();
  if(lastDate !== today){
    if(dailyXP>0) streak++; else streak=0;
    dailyXP=0;
    lastDate=today;
    alert("🌅 New Day | Streak: "+streak);
  }
}

// UI UPDATE
function updateUI(){
  document.getElementById("rank").innerText = "Rank: "+ranks[currentRank].name;
  document.getElementById("xp").innerText = "XP: "+xp+" / "+ranks[currentRank].need;
  document.getElementById("streak").innerText = "🔥 Streak: "+streak;

  document.getElementById("iman").innerText = stats.iman.toFixed(1);
  document.getElementById("strength").innerText = stats.strength.toFixed(1);
  document.getElementById("discipline").innerText = stats.discipline.toFixed(1);
  document.getElementById("knowledge").innerText = stats.knowledge.toFixed(1);

  let total = (stats.iman+stats.strength+stats.discipline+stats.knowledge)/4;
  document.getElementById("total").innerText = total.toFixed(1);
}

// RANK UP
function checkRank(){
  if(xp >= ranks[currentRank].need){
    xp = 0;
    currentRank++;
    alert("⚡ RANK UP → "+ranks[currentRank].name);
  }
}

// TITLES
function checkTitle(){
  let t="";
  if(dailyXP>=1000) t="Shadow Monarch";
  else if(dailyXP>=700) t="Shadow Soldier";
  else if(dailyXP>=400) t="Elite";
  else if(dailyXP>=200) t="Warrior";
  else if(dailyXP>=100) t="Disciplined";

  if(t && !titles.includes(t)){
    titles.push(t);
    let div=document.createElement("div");
    div.innerText="🏆 "+t;
    document.getElementById("titles").appendChild(div);
    alert("🏆 "+t);
  }
}

// QUEST SYSTEM
function addQuest(){
  let name=document.getElementById("questName").value;
  let xpVal=parseInt(document.getElementById("xpValue").value);
  let type=document.getElementById("type").value;

  let q={id:Date.now(),name,xp:xpVal,type};
  questList.push(q);
  renderQuest(q);
  save();
}

function renderQuest(q){
  let div=document.createElement("div");
  div.className="quest";
  div.innerHTML=`${q.name} (${q.type}) +${q.xp}XP
  <button onclick='complete(${q.id},this)'>✔</button>
  <button onclick='del(${q.id},this)'>❌</button>`;
  document.getElementById("quests").appendChild(div);
}

function complete(id,btn){
  let q=questList.find(x=>x.id===id);
  xp+=q.xp; dailyXP+=q.xp;

  stats[q.type]=Math.min(100,stats[q.type]+q.xp/10);

  checkRank();
  checkTitle();
  updateUI();
  drawChart();
  save();

  btn.parentElement.remove();
}

function del(id,btn){
  questList=questList.filter(x=>x.id!==id);
  save();
  btn.parentElement.remove();
}

// GRAPH
function drawChart(){
  let c=document.getElementById("chart");
  let ctx=c.getContext("2d");
  ctx.clearRect(0,0,300,300);

  let v=[stats.iman,stats.strength,stats.discipline,stats.knowledge];
  let cx=150,cy=150,r=100;

  for(let i=0;i<4;i++){
    let a=(Math.PI*2/4)*i;
    ctx.beginPath();
    ctx.moveTo(cx,cy);
    ctx.lineTo(cx+r*Math.cos(a),cy+r*Math.sin(a));
    ctx.stroke();
  }

  ctx.beginPath();
  for(let i=0;i<4;i++){
    let a=(Math.PI*2/4)*i;
    let val=v[i]/100;
    let x=cx+r*val*Math.cos(a);
    let y=cy+r*val*Math.sin(a);
    if(i==0) ctx.moveTo(x,y); else ctx.lineTo(x,y);
  }
  ctx.closePath();
  ctx.stroke();
}

// INIT
load();
updateUI();
drawChart();

</script>

</body>
</html>

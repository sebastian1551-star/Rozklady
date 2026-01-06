<html lang="pl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<style>
:root{
--bg:#f4f6f8;--text:#222;--card:#fff;
--sat:#c7f3d1;--sun:#f8c7dc;--top:#1f2937;
--blink:#facc15;
}
body.dark{
--bg:#0f172a;--text:#e5e7eb;--card:#1e293b;
--sat:#14532d;--sun:#701a75;--top:#020617;
--blink:#fde047;
}
body{
font-family:system-ui,Arial;
background:var(--bg);color:var(--text);
margin:0;padding:20px;
}
h1{text-align:center}
.subtitle{text-align:center;color:gray;margin-bottom:10px}
.top-box{
background:var(--top);color:#fff;
padding:14px;border-radius:14px;
text-align:center;font-size:1.2em;margin-bottom:12px
}
.controls{text-align:center;margin-bottom:12px}
select,button{
padding:8px 12px;border-radius:10px;border:none;
font-size:1em;margin:4px
}
.grid{
display:grid;
grid-template-columns:repeat(auto-fill,minmax(90px,1fr));
gap:12px;max-width:620px;margin:0 auto
}
.tile{
padding:14px;text-align:center;font-size:1.05em;
border-radius:14px;font-weight:600;
background:var(--card);
box-shadow:0 4px 8px rgba(0,0,0,.15);
}
.saturday{background:var(--sat)}
.sunday{background:var(--sun)}
.weekday{background:var(--card)}
.past{
text-decoration:line-through;
opacity:.35;
filter:blur(0.7px);
}
.blink{
animation:blink 1s infinite;
background:var(--blink)!important;
}
@keyframes blink{
0%,50%{opacity:1}
25%,75%{opacity:.4}
}
.footer{text-align:center;margin-top:22px;color:gray;font-size:.85em}
</style>
</head>

<body>

<h1>Linia A2</h1>
<div class="subtitle">Interaktywny rozkład jazdy</div>

<div class="top-box" id="countdown">Ładowanie…</div>

<div class="controls">
<select id="daySelect" onchange="renderSchedule()">
<option value="auto">Dziś (auto)</option>
<option value="weekday">Dzień powszedni</option>
<option value="saturday">Sobota</option>
<option value="sunday">Niedziela</option>
</select>

<select id="stopSelect" onchange="renderSchedule()">
<option value="teklin">Teklin II → ORLA</option>
<option value="powrot">ORLA → Teklin II</option>
</select>

<button onclick="toggleDark()">🌙 / ☀️</button>
</div>

<div class="grid" id="schedule"></div>

<div class="footer">
✔ live • ✔ miganie &lt;12 min • ✔ tryb nocny
</div>

<script>
const schedules={
teklin:{
weekday:["04:13","05:04","05:59","06:27","06:58","07:34","08:00","08:33","09:04","09:33","10:33","11:29","12:29","13:33","14:09","14:33","15:03","15:34","16:04","16:29","17:04","18:08","19:07","20:06","21:34","23:04"],
saturday:["05:04","07:04","08:34","10:04","11:34","13:04","14:34","16:04","17:34","19:04","21:04"],
sunday:["07:34","09:34","11:34","13:34","15:34","17:34","19:34"]
},
powrot:{
weekday:["04:48","05:30","06:10","06:50","07:16","07:50","08:20","08:50","09:20","10:15","11:05","12:20","13:20","13:55","14:20","14:50","15:20","15:45","16:15","16:50","17:20","18:20","19:25","20:25","21:55","23:25"],
saturday:["05:20","07:15","08:50","10:20","11:50","13:20","14:50","16:30","17:50","19:20","21:25"],
sunday:["08:10","10:15","12:20","14:20","16:20","18:15","20:20"]
}
};

function autoDay(){
const d=new Date().getDay();
if(d===6)return"saturday";
if(d===0)return"sunday";
return"weekday";
}
function getDay(){
return daySelect.value==="auto"?autoDay():daySelect.value;
}
function nextTimeSec(){
const now=new Date();
const nowSec=now.getHours()*3600+now.getMinutes()*60+now.getSeconds();
return schedules[stopSelect.value][getDay()]
.map(t=>{const[h,m]=t.split(":");return h*3600+m*60})
.find(t=>t>nowSec)||null;
}
function updateCountdown(){
const next=nextTimeSec();
if(!next){countdown.textContent="Brak dalszych kursów";return;}
const now=new Date();
const diff=next-(now.getHours()*3600+now.getMinutes()*60+now.getSeconds());
countdown.textContent=`Najbliższy odjazd za ${Math.floor(diff/60)} min ${diff%60} s`;
}
function renderSchedule(){
schedule.innerHTML="";
const now=new Date();
const nowMin=now.getHours()*60+now.getMinutes();
const next=nextTimeSec();
schedules[stopSelect.value][getDay()].forEach(t=>{
const[h,m]=t.split(":");
const mins=h*60+ +m;
const div=document.createElement("div");
div.className="tile "+getDay();
if(mins<nowMin)div.classList.add("past");
if(next && mins*60===next && next-(nowMin*60)<720)div.classList.add("blink");
div.textContent=t;
schedule.appendChild(div);
});
}
function toggleDark(){
document.body.classList.toggle("dark");
localStorage.setItem("dark",document.body.classList.contains("dark"));
}
if(localStorage.getItem("dark")==="true")document.body.classList.add("dark");

renderSchedule();
updateCountdown();
setInterval(()=>{updateCountdown();renderSchedule();},1000);
</script>
</body>
</html>

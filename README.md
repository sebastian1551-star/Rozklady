<!DOCTYPE html>
<html lang="pl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Moje rozkłady</title>

<style>
:root{
--bg:#f4f6f8;--text:#222;--card:#ffffff;
--top:#1f2937;--blink:#facc15;
--skm:#dc2626;--km:#16a34a;--bus:#e5e7eb;
}
body.dark{
--bg:#0f172a;--text:#e5e7eb;--card:#1e293b;
--top:#020617;--blink:#fde047;
--skm:#b91c1c;--km:#15803d;--bus:#334155;
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
gap:12px;max-width:1000px;margin:0 auto
}

.tile{
padding:14px;text-align:center;
font-size:1.05em;border-radius:14px;
font-weight:600;box-shadow:0 4px 8px rgba(0,0,0,.15);
position:relative;cursor:default;
}

.skm{background:var(--skm);color:#fff}
.km{background:var(--km);color:#fff}
.bus{background:var(--bus)}

.past{text-decoration:line-through;opacity:.35}

.blink{
animation:blink 1s infinite;
background:var(--blink)!important;color:#000!important
}
@keyframes blink{
0%,50%{opacity:1}
25%,75%{opacity:.4}
}

/* dymki */
.tile::after{
content:attr(data-tip);
position:absolute;
bottom:110%;left:50%;
transform:translateX(-50%);
background:#000;color:#fff;
padding:4px 8px;border-radius:6px;
font-size:.75em;opacity:0;
white-space:nowrap;pointer-events:none;
}
.tile:hover::after{opacity:1}

.footer{
text-align:center;margin-top:22px;
color:gray;font-size:.85em
}
</style>
</head>

<body>

<h1>Moje rozkłady</h1>
<div class="subtitle">Autobus A2 + SKM + KM</div>

<div class="top-box" id="countdown">Ładowanie…</div>

<div class="controls">
<select id="daySelect" onchange="renderSchedule()">
<option value="auto">Dziś (auto)</option>
<option value="weekday">Dzień powszedni</option>
<option value="saturday">Sobota</option>
<option value="sunday">Niedziela</option>
</select>

<select id="routeSelect" onchange="renderSchedule()">
<option value="os_all">Otwock → Warszawa Stadion (SKM + KM)</option>
<option value="so_skm">Warszawa Stadion → Otwock (SKM)</option>
<option value="a2_teklin">A2 Teklin II → ORLA</option>
<option value="a2_orla">A2 ORLA → Teklin II</option>
</select>

<button onclick="toggleDark()">🌙 / ☀️</button>
</div>

<div class="grid" id="schedule"></div>

<div class="footer">
🟥 SKM • 🟩 KM • 🚌 A2 • ⏱ pociąg 35 min • ⏱ autobus 10 min
</div>

<script>
const t2m=t=>{const[a,b]=t.split(":");return +a*60+ +b};
const fmt=m=>String(Math.floor(m/60)).padStart(2,"0")+":"+String(m%60).padStart(2,"0");
const autoDay=()=>{const d=new Date().getDay();return d===6?"saturday":d===0?"sunday":"weekday"};
const getDay=()=>daySelect.value==="auto"?autoDay():daySelect.value;

// ===== SKM =====
function gen30(start,end,type){
let out=[],c=t2m(start),e=t2m(end);
while(c<=e){out.push({time:fmt(c),type});c+=30;}
return out;
}
const SKM_OS=gen30("04:29","23:29","skm");
const SKM_SO=gen30("03:58","23:28","skm");

// ===== KM =====
const KM_WEEK=["04:24","05:13","05:38","05:50","06:13","06:24","06:38","07:13","07:20","07:38","08:13","08:20","08:38","09:13","09:39","10:13","10:38","11:12","11:39","12:12","12:38","13:12","13:39","14:12","14:38","15:09","15:20","15:39","16:13","16:38","17:13","17:20","17:38","18:13","18:20","18:38","19:13","19:20","19:38","20:13","20:38","21:39","22:38"];
const KM_WEEKEND=["04:24","05:38","06:38","07:13","07:38","08:38","09:39","10:38","11:39","12:38","13:39","14:38","15:39","16:38","17:38","18:41","19:11","19:38","20:38","22:28"];

// ===== A2 =====
const A2={
teklin:{
weekday:["04:13","05:04","05:59","06:27","06:58","07:34","08:00","08:33","09:04","09:33","10:33","11:29","12:29","13:33","14:09","14:33","15:03","15:34","16:04","16:29","17:04","18:08","19:07","20:06","21:34","23:04"],
saturday:["05:04","07:04","08:34","10:04","11:34","13:04","14:34","16:04","17:34","19:04","21:04"],
sunday:["07:34","09:34","11:34","13:34","15:34","17:34","19:34"]
},
orla:{
weekday:["04:48","05:30","06:10","06:50","07:16","07:50","08:20","08:50","09:20","10:15","11:05","12:20","13:20","13:55","14:20","14:50","15:20","15:45","16:15","16:50","17:20","18:20","19:25","20:25","21:55","23:25"],
saturday:["05:20","07:15","08:50","10:20","11:50","13:20","14:50","16:30","17:50","19:20","21:25"],
sunday:["08:10","10:15","12:20","14:20","16:20","18:15","20:20"]
}
};

function buildData(){
const day=getDay();
let out=[];
if(routeSelect.value==="os_all"){
out=[...SKM_OS];
(day==="weekday"?KM_WEEK:KM_WEEKEND).forEach(t=>out.push({time:t,type:"km"}));
}
if(routeSelect.value==="so_skm") out=[...SKM_SO];
if(routeSelect.value==="a2_teklin") out=A2.teklin[day].map(t=>({time:t,type:"bus"}));
if(routeSelect.value==="a2_orla") out=A2.orla[day].map(t=>({time:t,type:"bus"}));
return out.sort((a,b)=>t2m(a.time)-t2m(b.time));
}

function renderSchedule(){
schedule.innerHTML="";
const data=buildData();
const now=new Date();
const nowSec=now.getHours()*3600+now.getMinutes()*60+now.getSeconds();
const next=data.map(e=>t2m(e.time)*60).find(t=>t>nowSec);

data.forEach(e=>{
const d=document.createElement("div");
d.className="tile "+e.type;
d.dataset.tip=e.type==="bus"?"⏱ 10 min":"⏱ 35 min";
if(t2m(e.time)*60<nowSec)d.classList.add("past");
if(next && t2m(e.time)*60===next && next-nowSec<720)d.classList.add("blink");
d.textContent=e.time;
schedule.appendChild(d);
});
updateCountdown(next);
}

function updateCountdown(next){
if(!next){countdown.textContent="Brak dalszych kursów";return;}
const now=new Date();
const diff=next-(now.getHours()*3600+now.getMinutes()*60+now.getSeconds());
countdown.textContent=`Najbliższy odjazd za ${Math.floor(diff/60)} min ${diff%60} s`;
}

function toggleDark(){
document.body.classList.toggle("dark");
localStorage.setItem("dark",document.body.classList.contains("dark"));
}
if(localStorage.getItem("dark")==="true")document.body.classList.add("dark");

renderSchedule();
setInterval(renderSchedule,1000);
</script>

</body>
</html>

<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0"/>
<title>两地时区互转 | 极简版</title>
<style>
:root{
  --bg:#fff;
  --fg:#000;
  --card:#f5f5f5;
  --accent:#1976d2;
  --border:#ddd;
}
body{
  margin:0;
  font-family:system-ui,-apple-system,BlinkMacSystemFont,"Segoe UI",Roboto,"Helvetica Neue",Arial,"Noto Sans",sans-serif;
  background:var(--bg);
  color:var(--fg);
  display:flex;
  flex-direction:column;
  align-items:center;
  justify-content:center;
  min-height:100vh;
  transition:background .3s,color .3s;
}
body.night{
  --bg:#000;
  --fg:#fff;
  --card:#111;
  --border:#333;
}
.card{
  background:var(--card);
  padding:2rem;
  border-radius:12px;
  width:90%;
  max-width:400px;
  box-shadow:0 8px 20px rgba(0,0,0,.15);
}
h2{margin:0 0 1rem;text-align:center}
.row{
  display:flex;
  gap:.5rem;
  margin-bottom:1rem;
}
.row select,.row input{
  flex:1;
  padding:.6rem .8rem;
  font-size:1rem;
  border:1px solid var(--border);
  border-radius:6px;
  background:var(--bg);
  color:var(--fg);
}
.btn{
  width:100%;
  padding:.7rem;
  font-size:1rem;
  border:none;
  border-radius:6px;
  background:var(--accent);
  color:#fff;
  cursor:pointer;
}
.btn:hover{opacity:.9}
.switch{
  text-align:center;
  margin:.5rem 0;
  cursor:pointer;
  font-size:1.2rem
}
.res{
  text-align:center;
  margin-top:1rem;
  font-size:1.1rem;
}
.theme-toggle{
  position:absolute;
  top:1rem;
  right:1rem;
  cursor:pointer;
  font-size:1.4rem
}
</style>
</head>
<body>
<div class="theme-toggle" id="themeToggle" title="切换主题">🌓</div>

<div class="card">
  <h2>两地时区互转</h2>

  <div class="row">
    <select id="fromTz"></select>
    <input id="fromDt" type="datetime-local"/>
  </div>

  <div class="switch" id="switchBtn">⇅ 互换</div>

  <div class="row">
    <select id="toTz"></select>
    <input id="toDt" type="datetime-local" readonly />
  </div>

  <button class="btn" id="convertBtn">转换</button>
  <div class="res" id="res"></div>
</div>

<script>
/* ===== 数据 ===== */
const cities=[
{label:"纽约",tz:"America/New_York"},
{label:"洛杉矶",tz:"America/Los_Angeles"},
{label:"伦敦",tz:"Europe/London"},
{label:"巴黎",tz:"Europe/Paris"},
{label:"迪拜",tz:"Asia/Dubai"},
{label:"莫斯科",tz:"Europe/Moscow"},
{label:"北京",tz:"Asia/Shanghai"},
{label:"东京",tz:"Asia/Tokyo"},
{label:"悉尼",tz:"Australia/Sydney"},
{label:"奥克兰",tz:"Pacific/Auckland"}
];

/* ===== 工具 ===== */
const $=s=>document.querySelector(s);
const fmt=(ts,tz)=>
  new Date(ts).toLocaleString('zh-CN',{timeZone:tz, hour12:false});
const localDtToTs=dtLocal=>new Date(dtLocal).getTime();

/* ===== 渲染城市列表 ===== */
function fillSelect(){
  const frag=document.createDocumentFragment();
  cities.forEach(c=>{
    const opt=document.createElement('option');
    opt.value=c.tz;
    opt.textContent=c.label;
    frag.appendChild(opt.cloneNode(true));
    frag.appendChild(opt);
  });
  $('#fromTz').appendChild(frag.cloneNode(true));
  $('#toTz').appendChild(frag.cloneNode(true));
  $('#fromTz').value=Intl.DateTimeFormat().resolvedOptions().timeZone;
  $('#toTz').value='America/New_York';
}
fillSelect();

/* ===== 主题 ===== */
const body=document.body;
const themeKey='tzTheme';
function applyTheme(){
  const saved=localStorage.getItem(themeKey);
  const auto=!(saved==='light'||saved==='dark');
  if(auto){
    const h=new Date().getHours();
    body.classList.toggle('night',h<6||h>=18);
  }else{
    body.classList.toggle('night',saved==='dark');
  }
}
applyTheme();
$('#themeToggle').onclick=()=>{
  const cur=body.classList.contains('night');
  body.classList.toggle('night',!cur);
  localStorage.setItem(themeKey,cur?'light':'dark');
};

/* ===== 互换 ===== */
$('#switchBtn').onclick=()=>{
  const a=$('#fromTz').value,b=$('#toTz').value;
  $('#fromTz').value=b; $('#toTz').value=a;
};

/* ===== 转换 ===== */
$('#convertBtn').onclick=()=>{
  const dt=$('#fromDt').value;
  if(!dt){$('#res').textContent='请先选择日期时间';return;}
  const fromTz=$('#fromTz').value,toTz=$('#toTz').value;
  const ts=localDtToTs(dt);                 // 用户输入视为「fromTz」的本地时间
  const inUTC=ts - new Date().getTimezoneOffset()*60_000; // 转 UTC
  const target=new Date(inUTC).toLocaleString('sv-SE',{timeZone:fromTz}).replace(' ','T');
  $('#toDt').value=target;                  // 回填目标控件（仅展示）
  $('#res').textContent=`${fmt(ts,fromTz)} ➜ ${fmt(ts,toTz)}`;
};

/* ===== 默认填入当前时间 ===== */
const now=new Date();
const pad=n=>n.toString().padStart(2,'0');
const localISO=
  now.getFullYear()+'-'+
  pad(now.getMonth()+1)+'-'+
  pad(now.getDate())+'T'+
  pad(now.getHours())+':'+
  pad(now.getMinutes());
$('#fromDt').value=localISO;
</script>
</body>
</html>

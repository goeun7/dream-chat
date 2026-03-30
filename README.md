<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>字词库聊天（拼词版）</title>
<link href="https://fonts.googleapis.com/css2?family=Noto+Serif+SC:wght@300;400;500&family=Noto+Sans+SC:wght@300;400&display=swap" rel="stylesheet">
<style>
*{margin:0;padding:0;box-sizing:border-box}
body{background:#0b0b15;display:flex;justify-content:center;align-items:center;height:100vh;font-family:'Noto Sans SC'}
#app{width:400px;height:820px;background:#111128;border-radius:30px;display:flex;flex-direction:column;overflow:hidden}
header{display:flex;align-items:center;padding:16px;border-bottom:1px solid #222}
#lib-btn{margin-left:auto}
#msgs{flex:1;overflow:auto;padding:12px}
.msg{margin:6px 0}
.mine{text-align:right}
.bbl{display:inline-block;padding:8px 12px;border-radius:12px;background:#1e1040;color:#fff}
.his .bbl{background:#f0e6d4;color:#000}
#inp-area{display:flex;padding:10px}
textarea{flex:1}
button{cursor:pointer}
.view{flex:1}
.gone{display:none}
#lib-view{padding:10px;overflow:auto}
.kw{display:inline-block;margin:3px;padding:3px 6px;background:#333;border-radius:6px}
</style>
</head>
<body>

<div id="app">
<header>
<div>陆沉</div>
<button id="lib-btn" onclick="toggleLib()">字词库</button>
</header>

<div id="chat-view" class="view">
<div id="msgs"></div>
<div id="inp-area">
<textarea id="inp" rows="1"></textarea>
<button onclick="send()">➤</button>
</div>
</div>

<div id="lib-view" class="view gone"></div>
</div>

<script>
/* ───────── 字词库（核心改造：全部为“词”而不是句） ───────── */
const LIB = {
pronoun:["我","你","他","我们","你们"],
logic:["但是","所以","因为","如果","可是"],
emotion:["喜欢","爱","在意","难过","开心","想"],
action:["去","做","说","陪","等","看看","睡"],
noun:["事情","工作","你","我","现在","问题"],
degree:["很","有点","特别","不太","真的"],
modal:["吧","呢","吗","啊","了"],
care:["早点","记得","可以","不要","先","慢慢"],
encourage:["可以","没关系","再试","别怕","加油"],
intimate:["宝贝","乖","别怕","在这"],
negation:["不","没","不会","不是"],
punct:["。","，","？","…"]
};

/* ───────── 生成逻辑（核心升级） ───────── */
function pick(arr){return arr[Math.floor(Math.random()*arr.length)]}

function genSentence(input){
  // 根据输入简单匹配情绪倾向
  let mood = "neutral";
  if(/难过|伤心|哭/.test(input)) mood="sad";
  if(/开心|高兴/.test(input)) mood="happy";
  if(/累|烦/.test(input)) mood="tired";

  // 基础结构模板（动态拼接）
  let s = "";

  if(mood==="sad"){
    s = `${pick(LIB.pronoun)}${pick(LIB.care)}${pick(LIB.action)}${pick(LIB.pronoun)}${pick(LIB.modal)}`;
  }else if(mood==="happy"){
    s = `${pick(LIB.pronoun)}${pick(LIB.emotion)}${pick(LIB.pronoun)}${pick(LIB.degree)}${pick(LIB.modal)}`;
  }else{
    s = `${pick(LIB.pronoun)}${pick(LIB.degree)}${pick(LIB.emotion)}${pick(LIB.pronoun)}${pick(LIB.logic)}${pick(LIB.action)}${pick(LIB.noun)}${pick(LIB.modal)}`;
  }

  // 随机加入亲密/鼓励
  if(Math.random()<0.5){
    s = `${pick(LIB.intimate)}，${s}`;
  }

  if(Math.random()<0.5){
    s += `，${pick(LIB.encourage)}`;
  }

  return s + pick(LIB.punct);
}

/* ───────── UI ───────── */
function addMsg(txt,isHis){
  const d=document.createElement('div');
  d.className='msg '+(isHis?'his':'mine');
  d.innerHTML=`<div class="bbl">${txt}</div>`;
  msgs.appendChild(d);
  msgs.scrollTop=msgs.scrollHeight;
}

function send(){
  const txt=inp.value.trim();
  if(!txt)return;
  addMsg(txt,false);
  inp.value='';
  setTimeout(()=>addMsg(genSentence(txt),true),600);
}

/* ───────── 字词库管理 ───────── */
let libOpen=false;
function toggleLib(){
  libOpen=!libOpen;
  chat-view.classList.toggle('gone',libOpen);
  lib-view.classList.toggle('gone',!libOpen);
  renderLib();
}

function renderLib(){
  let html="";
  for(const k in LIB){
    html+=`<h4>${k}</h4>`;
    LIB[k].forEach((w,i)=>{
      html+=`<span class="kw">${w} <button onclick="delWord('${k}',${i})">x</button></span>`;
    });
    html+=`<br><input placeholder="添加词" onkeydown="if(event.key==='Enter')addWord('${k}',this)"><hr>`;
  }
  lib-view.innerHTML=html;
}

function addWord(cat,el){
  const v=el.value.trim();
  if(!v)return;
  const arr=v.split(/[\s,，]+/);
  LIB[cat].push(...arr);
  el.value="";
  renderLib();
}

function delWord(cat,i){
  LIB[cat].splice(i,1);
  renderLib();
}

/* 初始 */
setTimeout(()=>addMsg("在。",true),500);
</script>
</body>
</html>

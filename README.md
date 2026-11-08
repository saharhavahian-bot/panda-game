# panda-game

<!DOCTYPE html>
<html lang="fa" dir="rtl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>پاندا و گنجینه دانایی</title>

<style>
*{box-sizing:border-box}
body{
 margin:0;overflow:hidden;font-family:Tahoma,Arial;
 background:#9be4e5;color:white
}
#game{position:fixed;inset:0}
canvas{display:block}
#hud{
 position:fixed;top:10px;left:10px;right:10px;
 display:flex;justify-content:space-between;gap:6px;
 z-index:5;pointer-events:none
}
.panel{
 background:#164c45dd;border:2px solid #ffffff66;
 border-radius:13px;padding:9px 12px;
 font-size:13px;box-shadow:0 3px 10px #0004
}
#message{
 position:fixed;top:85px;left:50%;
 transform:translateX(-50%);
 background:#164c45ee;border:2px solid white;
 border-radius:14px;padding:10px 16px;
 z-index:6;text-align:center;white-space:nowrap
}
#controls{position:fixed;bottom:20px;left:20px;z-index:8}
#joy{
 width:125px;height:125px;border-radius:50%;
 background:#ffffff55;border:3px solid #ffffffaa;
 position:relative;touch-action:none
}
#stick{
 position:absolute;width:48px;height:48px;
 left:35px;top:35px;border-radius:50%;
 background:#ffe66d;box-shadow:0 3px 10px #0006
}
#buttons{
 position:fixed;bottom:25px;right:15px;
 z-index:8;display:flex;flex-direction:column;gap:9px
}
button{
 border:0;border-radius:13px;padding:12px 16px;
 font-family:inherit;font-weight:bold;font-size:14px;
 color:#173c37;background:#ffe66d;
 box-shadow:0 4px 0 #b99c36;cursor:pointer;
 touch-action:manipulation;
 -webkit-tap-highlight-color:transparent
}
button:active{transform:translateY(2px);box-shadow:none}
button:disabled{opacity:.6;cursor:default}
#overlay{
 position:fixed;inset:0;z-index:20;
 display:flex;align-items:center;justify-content:center;
 background:#09262ddd;padding:15px
}
#dialog{
 width:100%;max-width:460px;max-height:90vh;overflow:auto;
 background:linear-gradient(145deg,#287a63,#17445a);
 border:3px solid #ffe66d;border-radius:24px;
 padding:23px;text-align:center;box-shadow:0 15px 45px #0008
}
#dialog h1{color:#ffe66d;font-size:24px}
#dialog p{line-height:2}
.answer{
 display:block;width:100%;margin:9px 0;
 background:#fff1b8;padding:14px;font-size:16px;
 min-height:48px
}
.hidden{display:none!important}
#question{font-size:18px;color:#ffe66d;font-weight:bold}
@media(max-width:600px){
 .panel{font-size:10px;padding:7px}
 #message{font-size:12px;top:78px}
 #dialog{padding:17px}
}
</style>

<script type="importmap">
{"imports":{"three":"https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js"}}
</script>
</head>

<body>
<div id="game"></div>

<div id="hud">
 <div class="panel">
  مرحله: <span id="stage">۱</span>/۱۰<br>
  امتیاز: <span id="score">۰</span>
 </div>
 <div class="panel">
  جان: <span id="hearts">❤️❤️❤️</span><br>
  ستاره: <span id="stars">⭐ ۰</span>
 </div>
 <div class="panel">
  سکه: <span id="coins">۰</span><br>
  سازنده: ماهان احمدی نژاد
 </div>
</div>

<div id="message">به جنگل دانایی خوش آمدی! 🐼</div>

<div id="controls">
 <div id="joy"><div id="stick"></div></div>
</div>

<div id="buttons">
 <button id="soundBtn">🔊 صدا روشن</button>
</div>

<div id="overlay">
 <div id="dialog">
  <h1>🐼 پاندا و گنجینه دانایی</h1>
  <p>
   به جنگل جادویی خوش آمدی!<br>
   با پاندا در جنگل حرکت کن، سکه جمع کن
   و به سؤال‌های جالب پاسخ بده.<br>
   با پاسخ درست، ستاره می‌گیری و به مرحله بعد می‌روی!
  </p>
  <p>🎮 حرکت با کلیدهای جهت‌دار یا WASD و دسته لمسی</p>
  <p>❤️ سه جان داری؛ مراقب قارچ‌های قرمز باش!</p>
  <button id="startBtn">شروع ماجراجویی 🚀</button>
  <p style="font-size:12px">طراحی بازی: ماهان احمدی نژاد</p>
 </div>
</div>

<script type="module">
import * as THREE from 'three';

/* ---------- سؤال‌های بازی ---------- */

const levels = [
 {category:'فارسی',q:'کدام کلمه نام یک حیوان است؟',
  a:['درخت','شیر','باران'],correct:1,info:'شیر یکی از حیوانات است.'},

 {category:'ریاضی',q:'۵ + ۷ چند می‌شود؟',
  a:['۱۰','۱۲','۱۳'],correct:1,info:'۵ به اضافه ۷ برابر ۱۲ است.'},

 {category:'علوم',q:'گیاهان برای رشد به چه چیزی نیاز دارند؟',
  a:['نور خورشید','تلویزیون','اسباب‌بازی'],correct:0,info:'نور خورشید به رشد گیاهان کمک می‌کند.'},

 {category:'هوش',q:'کدام عدد با بقیه فرق دارد؟',
  a:['۲','۴','۷'],correct:2,info:'۲ و ۴ زوج هستند؛ ۷ فرد است.'},

 {category:'فارسی',q:'متضاد کلمه بزرگ چیست؟',
  a:['کوچک','بلند','زیبا'],correct:0,info:'متضاد بزرگ، کوچک است.'},

 {category:'ریاضی',q:'۳ ضربدر ۴ چند می‌شود؟',
  a:['۷','۱۲','۱۵'],correct:1,info:'سه گروه چهار‌تایی برابر ۱۲ است.'},

 {category:'علوم',q:'کدام جانور پرواز می‌کند؟',
  a:['ماهی','گنجشک','گربه'],correct:1,info:'گنجشک یک پرنده است.'},

 {category:'هوش',q:'بعد از عدد ۱۵ چه عددی می‌آید؟',
  a:['۱۴','۱۶','۱۸'],correct:1,info:'بعد از ۱۵ عدد ۱۶ قرار دارد.'},

 {category:'اطلاعات عمومی',q:'کدام سیاره به سیاره سرخ معروف است؟',
  a:['مریخ','زمین','زهره'],correct:0,info:'مریخ به دلیل رنگ سرخ خود مشهور است.'},

 {category:'مرحله نهایی',q:'برای داشتن جنگلی سالم چه کاری بهتر است؟',
  a:['قطع درختان','ریختن زباله','حفاظت از طبیعت'],correct:2,
  info:'با مراقبت از طبیعت، جنگل‌ها سالم می‌مانند.'}
];

/* ---------- تنظیمات ---------- */

const fa = n => String(n).replace(/\d/g,d=>'۰۱۲۳۴۵۶۷۸۹'[d]);

let stage=0,score=0,stars=0,hearts=3,coins=0;
let playing=false,soundOn=true,questionOpen=false;
let answerLocked=false;
let coinsForStage=0;
let audioContext;

const $=id=>document.getElementById(id);
const overlay=$('overlay');

function updateHUD(){
 $('stage').textContent=fa(Math.min(stage+1,10));
 $('score').textContent=fa(score);
 $('stars').textContent=fa(stars);
 $('hearts').textContent='❤️'.repeat(hearts)+'🖤'.repeat(3-hearts);
 $('coins').textContent=fa(coins);
}

function say(text){
 $('message').textContent=text;
}

function sound(freq=550,duration=.13){
 if(!soundOn)return;
 try{
  audioContext??=new (window.AudioContext||window.webkitAudioContext)();
  if(audioContext.state==='suspended')audioContext.resume();

  const o=audioContext.createOscillator();
  const g=audioContext.createGain();

  o.frequency.value=freq;
  g.gain.setValueAtTime(.1,audioContext.currentTime);
  g.gain.exponentialRampToValueAtTime(
   .001,audioContext.currentTime+duration
  );

  o.connect(g);
  g.connect(audioContext.destination);
  o.start();
  o.stop(audioContext.currentTime+duration);
 }catch(e){}
}

$('soundBtn').onclick=()=>{
 soundOn=!soundOn;
 $('soundBtn').textContent=soundOn?'🔊 صدا روشن':'🔇 صدا خاموش';
};

/* ---------- صحنه سه‌بعدی ---------- */

const scene=new THREE.Scene();
scene.background=new THREE.Color(0x9be4e5);
scene.fog=new THREE.Fog(0x9be4e5,25,85);

const camera=new THREE.PerspectiveCamera(
 65,innerWidth/innerHeight,.1,150
);

const renderer=new THREE.WebGLRenderer({antialias:true});
renderer.setSize(innerWidth,innerHeight);
renderer.setPixelRatio(Math.min(devicePixelRatio,2));
renderer.shadowMap.enabled=true;
$('game').appendChild(renderer.domElement);

scene.add(new THREE.HemisphereLight(0xffffff,0x47734b,2));

const sun=new THREE.DirectionalLight(0xffffff,2);
sun.position.set(-8,15,8);
sun.castShadow=true;
scene.add(sun);

/* ---------- زمین ---------- */

const ground=new THREE.Mesh(
 new THREE.PlaneGeometry(16,120),
 new THREE.MeshStandardMaterial({color:0x55b85d})
);
ground.rotation.x=-Math.PI/2;
ground.position.set(0,-.5,-25);
ground.receiveShadow=true;
scene.add(ground);

const path=new THREE.Mesh(
 new THREE.PlaneGeometry(7,120),
 new THREE.MeshStandardMaterial({color:0xc9ad70})
);
path.rotation.x=-Math.PI/2;
path.position.set(0,-.47,-25);
scene.add(path);

/* ---------- درخت‌ها ---------- */

const trees=new THREE.Group();
scene.add(trees);

function makeTree(x,z,s=1){
 const trunk=new THREE.Mesh(
  new THREE.CylinderGeometry(.18,.3,2.1,8),
  new THREE.MeshStandardMaterial({color:0x81502c})
 );
 trunk.position.set(x,.5,z);
 trunk.scale.setScalar(s);
 trees.add(trunk);

 for(let i=0;i<3;i++){
  const crown=new THREE.Mesh(
   new THREE.SphereGeometry(.85-i*.1,12,10),
   new THREE.MeshStandardMaterial({
    color:[0x168c55,0x35a853,0x77bd42][i]
   })
  );
  crown.position.set(x,1.5+i*.55,z);
  crown.scale.setScalar(s);
  trees.add(crown);
 }
}

for(let z=15;z>-95;z-=5){
 makeTree(-5.2,z,.8+Math.random()*.4);
 makeTree(5.2,z-2,.8+Math.random()*.4);
}

/* ---------- ساخت پاندا ---------- */

const panda=new THREE.Group();
scene.add(panda);

const black=new THREE.MeshStandardMaterial({color:0x171717});
const white=new THREE.MeshStandardMaterial({color:0xffffff});
const pink=new THREE.MeshStandardMaterial({color:0xffa6a6});

function ball(parent,mat,x,y,z,sx,sy,sz){
 const m=new THREE.Mesh(
  new THREE.SphereGeometry(1,20,16),mat
 );
 m.position.set(x,y,z);
 m.scale.set(sx,sy,sz);
 m.castShadow=true;
 parent.add(m);
 return m;
}

ball(panda,white,0,.45,0,.7,.8,.55);
ball(panda,white,0,.35,-.45,.48,.53,.13);
ball(panda,white,0,1.25,-.05,.72,.68,.65);

for(const x of [-.53,.53]){
 ball(panda,black,x,1.78,-.02,.23,.25,.2);
}

for(const x of [-.28,.28]){
 const patch=ball(panda,black,x,1.34,-.61,.22,.29,.09);
 patch.rotation.z=x*.6;
 ball(panda,white,x,1.37,-.69,.105,.12,.045);
 ball(panda,black,x,1.37,-.73,.052,.065,.025);
}

ball(panda,black,0,1.08,-.72,.12,.08,.07);
ball(panda,pink,0,.91,-.68,.1,.045,.035);

const arms=[];
for(const x of [-.68,.68]){
 const arm=ball(panda,black,x,.55,-.02,.2,.38,.22);
 arms.push(arm);
}

for(const x of [-.4,.4]){
 ball(panda,black,x,-.08,.05,.28,.22,.3);
}

panda.position.set(0,0,5);

/* ---------- سکه‌ها و قارچ‌ها ---------- */

let coinItems=[],mushrooms=[],particles=[];

function makeCoin(x,z){
 const coin=new THREE.Mesh(
  new THREE.TorusGeometry(.35,.12,10,24),
  new THREE.MeshStandardMaterial({
   color:0xffd52a,metalness:.65,roughness:.25
  })
 );
 coin.position.set(x,1,z);
 coin.rotation.x=Math.PI/2;
 scene.add(coin);
 coinItems.push(coin);
}

function makeMushroom(x,z){
 const g=new THREE.Group();

 const stem=new THREE.Mesh(
  new THREE.CylinderGeometry(.15,.2,.5,10),
  new THREE.MeshStandardMaterial({color:0xffe4b5})
 );
 stem.position.y=.15;
 g.add(stem);

 const cap=new THREE.Mesh(
  new THREE.SphereGeometry(.45,14,10),
  new THREE.MeshStandardMaterial({color:0xe74747})
 );
 cap.scale.y=.5;
 cap.position.y=.43;
 g.add(cap);

 g.position.set(x,-.25,z);
 g.userData.hit=false;
 scene.add(g);
 mushrooms.push(g);
}

function clearList(list){
 for(const o of list){
  scene.remove(o);
  o.traverse?.(c=>{
   c.geometry?.dispose();
   if(c.material){
    if(Array.isArray(c.material)){
     c.material.forEach(m=>m.dispose());
    }else{
     c.material.dispose();
    }
   }
  });
 }
 list.length=0;
}

function burst(pos,color=0xffe66d){
 for(let i=0;i<12;i++){
  const p=new THREE.Mesh(
   new THREE.SphereGeometry(.07,6,6),
   new THREE.MeshBasicMaterial({color})
  );

  p.position.copy(pos);
  p.userData.v=new THREE.Vector3(
   (Math.random()-.5)*.2,
   Math.random()*.2,
   (Math.random()-.5)*.2
  );
  p.userData.life=1;
  scene.add(p);
  particles.push(p);
 }
}

/* ---------- مرحله ---------- */

function loadStage(){
 clearList(coinItems);
 clearList(mushrooms);

 coinsForStage=0;
 panda.position.set(0,0,5);
 questionOpen=false;
 answerLocked=false;

 for(let i=0;i<5;i++){
  const x=(i%2===0?-1:1)*(1+Math.random());
  makeCoin(x,-3-i*7);
 }

 for(let i=0;i<4;i++){
  const x=(i%2===0?-1:1)*(2.2+Math.random()*.4);
  makeMushroom(x,-7-i*10);
 }

 updateHUD();
 say('سکه‌ها را جمع کن و مراقب قارچ‌ها باش!');
}

/* ---------- نمایش سؤال ---------- */

function showQuestion(){
 if(questionOpen)return;

 questionOpen=true;
 playing=false;
 answerLocked=false;

 const q=levels[stage];

 $('dialog').innerHTML=`
  <h1>🧠 ${q.category}</h1>
  <p id="question">${q.q}</p>
  <p>یکی از پاسخ‌ها را انتخاب کن:</p>

  ${q.a.map((a,i)=>`
   <button type="button" class="answer" data-index="${i}">
    ${a}
   </button>
  `).join('')}

  <p style="font-size:12px">سازنده: ماهان احمدی نژاد</p>
 `;

 overlay.classList.remove('hidden');
}

/* ---------- بررسی پاسخ ---------- */

function checkAnswer(index){
 if(answerLocked || !questionOpen)return;

 answerLocked=true;

 const q=levels[stage];

 if(index===q.correct){

  score+=100;
  stars++;
  updateHUD();
  sound(900,.3);
  burst(panda.position.clone(),0x66ffcc);

  $('dialog').innerHTML=`
   <h1>آفرین! جواب درست بود! ⭐</h1>
   <p>${q.info}</p>
   <p>۱۰۰ امتیاز و یک ستاره گرفتی!</p>
   <button type="button" id="continueBtn">
    ادامه بازی 🚀
   </button>
  `;

  $('continueBtn').onclick=()=>{
   overlay.classList.add('hidden');
   questionOpen=false;
   stage++;

   if(stage>=10){
    finishGame();
   }else{
    loadStage();
    playing=true;
   }
  };

 }else{

  hearts--;
  updateHUD();
  sound(150,.3);

  if(hearts<=0){

   $('dialog').innerHTML=`
    <h1>فرصت‌ها تمام شد! 💔</h1>
    <p>اشکالی ندارد؛ دوباره تلاش کن.</p>
    <button type="button" id="retryBtn">
     شروع دوباره
    </button>
   `;

   $('retryBtn').onclick=restartGame;

  }else{

   $('dialog').innerHTML=`
    <h1>یک بار دیگر تلاش کن! 🌱</h1>
    <p>این پاسخ درست نبود.</p>
    <p>می‌توانی دوباره فکر کنی و پاسخ بدهی.</p>
    <button type="button" id="tryBtn">
     دوباره پاسخ می‌دهم
    </button>
   `;

   $('tryBtn').onclick=()=>{
    questionOpen=false;
    showQuestion();
   };
  }
 }
}

/* ---------- ثبت کلیک پاسخ‌ها ---------- */

$('dialog').addEventListener('click',function(e){
 const btn=e.target.closest('.answer');

 if(!btn)return;
 if(btn.disabled || answerLocked)return;

 e.preventDefault();
 btn.disabled=true;

 checkAnswer(Number(btn.dataset.index));
});

/* ---------- پایان بازی ---------- */

function finishGame(){
 playing=false;
 questionOpen=false;
 answerLocked=true;

 $('dialog').innerHTML=`
  <h1>🏆 قهرمان جنگل دانایی!</h1>
  <p>تبریک! هر ۱۰ مرحله را با موفقیت گذراندی.</p>
  <p>امتیاز نهایی: ${fa(score)}</p>
  <p>ستاره‌ها: ${fa(stars)} ⭐</p>
  <p>سکه‌ها: ${fa(coins)} 🪙</p>
  <button type="button" id="againBtn">بازی دوباره</button>
  <p style="font-size:12px">سازنده: ماهان احمدی نژاد</p>
 `;

 overlay.classList.remove('hidden');
 $('againBtn').onclick=restartGame;
}

/* ---------- شروع دوباره ---------- */

function restartGame(){
 stage=0;
 score=0;
 stars=0;
 hearts=3;
 coins=0;

 playing=false;
 questionOpen=false;
 answerLocked=false;

 updateHUD();
 loadStage();

 overlay.classList.add('hidden');
 playing=true;

 sound(650,.2);
}

/* ---------- کنترل‌ها ---------- */

const keys={};

window.addEventListener('keydown',e=>{
 keys[e.key.toLowerCase()]=true;

 if(['ArrowUp','ArrowDown','ArrowLeft','ArrowRight',' '].includes(e.key)){
  e.preventDefault();
 }
});

window.addEventListener('keyup',e=>{
 keys[e.key.toLowerCase()]=false;
});

let joystick={x:0,y:0};
const joy=$('joy'),stick=$('stick');
let pointer=null;

function moveJoy(e){
 const r=joy.getBoundingClientRect();

 let dx=e.clientX-(r.left+r.width/2);
 let dy=e.clientY-(r.top+r.height/2);

 const max=38;
 const len=Math.hypot(dx,dy);

 if(len>max){
  dx=dx/len*max;
  dy=dy/len*max;
 }

 joystick.x=dx/max;
 joystick.y=dy/max;

 stick.style.transform=`translate(${dx}px,${dy}px)`;
}

joy.addEventListener('pointerdown',e=>{
 pointer=e.pointerId;
 joy.setPointerCapture(e.pointerId);
 moveJoy(e);
});

joy.addEventListener('pointermove',e=>{
 if(e.pointerId===pointer)moveJoy(e);
});

function releaseJoy(){
 pointer=null;
 joystick={x:0,y:0};
 stick.style.transform='translate(0,0)';
}

joy.addEventListener('pointerup',releaseJoy);
joy.addEventListener('pointercancel',releaseJoy);

/* ---------- حرکت پاندا ---------- */

const clock=new THREE.Clock();

function updatePlayer(dt){
 if(!playing)return;

 let dx=joystick.x;
 let dz=-joystick.y;

 if(keys['arrowleft']||keys['a'])dx-=1;
 if(keys['arrowright']||keys['d'])dx+=1;
 if(keys['arrowup']||keys['w'])dz+=1;
 if(keys['arrowdown']||keys['s'])dz-=1;

 const len=Math.hypot(dx,dz);

 if(len>1){
  dx/=len;
  dz/=len;
 }

 panda.position.x+=dx*6*dt;
 panda.position.z-=dz*6*dt;

 panda.position.x=THREE.MathUtils.clamp(
  panda.position.x,-3,3
 );

 panda.position.z=THREE.MathUtils.clamp(
  panda.position.z,-55,7
 );

 panda.rotation.z=-dx*.12;

 const t=clock.elapsedTime;
 arms[0].rotation.z=Math.sin(t*8)*.25;
 arms[1].rotation.z=-Math.sin(t*8)*.25;
}

/* ---------- برخوردها ---------- */

function checkCollisions(){
 if(!playing)return;

 for(const coin of [...coinItems]){

  const dx=panda.position.x-coin.position.x;
  const dz=panda.position.z-coin.position.z;

  if(Math.hypot(dx,dz)<1.25){

   scene.remove(coin);
   coinItems.splice(coinItems.indexOf(coin),1);

   coins++;
   score+=10;

   updateHUD();
   sound(700,.1);
   burst(coin.position.clone());
   say('یک سکه پیدا کردی! 🪙');

   coinsForStage++;

   if(coinsForStage>=3){
    showQuestion();
    return;
   }
  }
 }

 for(const m of mushrooms){

  if(!m.userData.hit&&
    Math.abs(panda.position.x-m.position.x)<.85&&
    Math.abs(panda.position.z-m.position.z)<1){

   m.userData.hit=true;
   hearts--;

   updateHUD();
   sound(140,.25);
   say('مواظب باش! به قارچ برخورد کردی!');

   panda.position.x=0;

   if(hearts<=0){

    playing=false;
    questionOpen=false;

    $('dialog').innerHTML=`
     <h1>بازی تمام شد! 💔</h1>
     <p>می‌توانی دوباره تلاش کنی.</p>
     <button type="button" id="retryBtn">شروع دوباره</button>
    `;

    overlay.classList.remove('hidden');
    $('retryBtn').onclick=restartGame;
    return;
   }
  }
 }
}

/* ---------- انیمیشن ---------- */

function animate(){
 requestAnimationFrame(animate);

 const dt=Math.min(clock.getDelta(),.04);
 const t=clock.elapsedTime;

 updatePlayer(dt);

 for(const c of coinItems){
  c.rotation.y+=dt*2;
  c.position.y=1+Math.sin(t*3+c.position.z)*.15;
 }

 for(const p of [...particles]){
  p.position.add(p.userData.v);
  p.userData.life-=dt*1.5;
  p.scale.setScalar(Math.max(0,p.userData.life));

  if(p.userData.life<=0){
   scene.remove(p);
   particles.splice(particles.indexOf(p),1);
   p.geometry.dispose();
   p.material.dispose();
  }
 }

 panda.position.y=Math.sin(t*4)*.06;

 camera.position.lerp(
  new THREE.Vector3(
   panda.position.x*.5,
   4.5,
   panda.position.z+9
  ),
  .06
 );

 camera.lookAt(
  panda.position.x,.9,panda.position.z-3
 );

 checkCollisions();
 renderer.render(scene,camera);
}

animate();

/* ---------- شروع بازی ---------- */

$('startBtn').onclick=()=>{
 restartGame();
};

window.addEventListener('resize',()=>{
 camera.aspect=innerWidth/innerHeight;
 camera.updateProjectionMatrix();
 renderer.setSize(innerWidth,innerHeight);
});

</script>
</body>
</html>

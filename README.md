# idk
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
<title>APEX GP — Night Street Circuit</title>
<style>
  :root{
    --accent:#36e6ff;
    --accent2:#ff4fa3;
    --panel:rgba(7,11,20,0.78);
    --text:#eef3f9;
    --dim:#8b96a8;
    --mono:ui-monospace,'SF Mono','Cascadia Mono','Roboto Mono',Menlo,Consolas,monospace;
  }
  *{margin:0;padding:0;box-sizing:border-box;-webkit-tap-highlight-color:transparent;}
  html,body{width:100%;height:100%;overflow:hidden;background:#05070f;font-family:var(--mono);color:var(--text);}
  canvas#game{display:block;width:100%;height:100%;}
  #vignette{position:fixed;inset:0;z-index:4;pointer-events:none;
    background:radial-gradient(ellipse at center, transparent 52%, rgba(2,4,10,0.5) 100%);}

  /* ---------- HUD ---------- */
  .hud{position:fixed;pointer-events:none;z-index:5;}
  #lapPanel{top:12px;left:12px;background:var(--panel);border-left:3px solid var(--accent);
    padding:10px 14px;border-radius:6px;font-size:12px;line-height:1.7;min-width:176px;
    box-shadow:0 0 22px rgba(54,230,255,.10);}
  #lapPanel b{color:var(--accent);font-size:13px;letter-spacing:1px;}
  #lapPanel .val{float:right;color:var(--text);}
  #lapPanel .lbl{color:var(--dim);}
  #speedPanel{bottom:14px;left:50%;transform:translateX(-50%);text-align:center;
    background:var(--panel);padding:8px 22px 10px;border-radius:8px;border-bottom:3px solid var(--accent);
    box-shadow:0 0 22px rgba(54,230,255,.10);}
  #speedVal{font-size:38px;font-weight:700;letter-spacing:1px;line-height:1;
    text-shadow:0 0 14px rgba(54,230,255,.55);}
  #speedUnit{font-size:11px;color:var(--dim);letter-spacing:2px;}
  #gearBox{position:absolute;right:-46px;top:50%;transform:translateY(-50%);
    background:var(--panel);border:1px solid #233047;border-radius:6px;width:38px;height:46px;
    display:flex;align-items:center;justify-content:center;font-size:22px;font-weight:700;color:var(--accent);}
  #msg{top:18%;left:50%;transform:translateX(-50%);font-size:26px;font-weight:700;white-space:nowrap;
    letter-spacing:3px;color:var(--accent);text-shadow:0 0 18px rgba(54,230,255,.8),0 2px 12px rgba(0,0,0,.7);
    opacity:0;transition:opacity .25s;}
  #offTrack{top:26%;left:50%;transform:translateX(-50%);font-size:13px;color:#ffd34d;white-space:nowrap;
    background:var(--panel);padding:6px 12px;border-radius:6px;opacity:0;transition:opacity .2s;}
  #miniWrap{top:12px;right:12px;background:var(--panel);border-radius:8px;padding:6px;
    box-shadow:0 0 22px rgba(54,230,255,.10);}
  #minimap{display:block;width:140px;height:140px;}

  /* ---------- Buttons ---------- */
  #btnBar{position:fixed;bottom:14px;right:12px;z-index:6;display:flex;flex-direction:column;gap:8px;}
  .ctl{pointer-events:auto;background:var(--panel);color:var(--text);border:1px solid #233047;
    border-radius:8px;padding:9px 12px;font-family:var(--mono);font-size:12px;cursor:pointer;
    letter-spacing:1px;transition:border-color .15s,box-shadow .15s;}
  .ctl:hover{border-color:var(--accent);box-shadow:0 0 12px rgba(54,230,255,.25);}
  .ctl:focus-visible{outline:2px solid var(--accent);outline-offset:2px;}

  /* ---------- Touch controls ---------- */
  .pad{position:fixed;bottom:78px;z-index:6;display:none;gap:14px;}
  body.touch .pad{display:flex;}
  #padL{left:16px;} #padR{right:16px;flex-direction:column-reverse;}
  .padBtn{pointer-events:auto;width:72px;height:72px;border-radius:50%;border:2px solid #2a3a55;
    background:rgba(10,16,28,.65);color:var(--text);font-size:26px;display:flex;align-items:center;
    justify-content:center;user-select:none;touch-action:none;}
  .padBtn.active{border-color:var(--accent);background:rgba(54,230,255,.18);box-shadow:0 0 16px rgba(54,230,255,.35);}
  #padR .padBtn{width:84px;height:84px;font-size:15px;letter-spacing:1px;}

  /* ---------- Start overlay ---------- */
  #overlay{position:fixed;inset:0;z-index:10;
    background:radial-gradient(ellipse at 50% 22%,#1c2552 0%,#0c1228 45%,#05070f 100%);
    display:flex;flex-direction:column;align-items:center;justify-content:center;text-align:center;padding:24px;}
  #overlay h1{font-size:clamp(34px,7vw,60px);letter-spacing:9px;font-weight:800;
    text-shadow:0 0 26px rgba(54,230,255,.5);}
  #overlay h1 span{color:var(--accent);}
  #overlay .sub{color:var(--dim);font-size:13px;letter-spacing:4px;margin:10px 0 26px;}
  #overlay .sub i{font-style:normal;color:var(--accent2);}
  #ctrlList{display:grid;grid-template-columns:auto auto;gap:6px 18px;font-size:13px;text-align:left;
    background:var(--panel);padding:16px 22px;border-radius:10px;margin-bottom:28px;line-height:1.5;
    border:1px solid #1d2a45;}
  #ctrlList .k{color:var(--accent);font-weight:700;}
  #startBtn{background:linear-gradient(135deg,#36e6ff,#2f9bff);color:#04101c;border:none;border-radius:10px;
    font-family:var(--mono);font-size:17px;font-weight:800;letter-spacing:3px;padding:15px 42px;cursor:pointer;
    box-shadow:0 6px 30px rgba(54,230,255,.45);transition:transform .12s;}
  #startBtn:hover{transform:scale(1.04);}
  #startBtn:focus-visible{outline:3px solid #fff;outline-offset:3px;}
  #lights{display:none;gap:16px;margin-top:10px;}
  .light{width:clamp(34px,9vw,54px);height:clamp(34px,9vw,54px);border-radius:50%;background:#1b1e26;
    border:3px solid #10131c;box-shadow:inset 0 3px 8px rgba(0,0,0,.7);transition:background .1s,box-shadow .1s;}
  .light.on{background:#ff2018;box-shadow:0 0 28px #ff2018,inset 0 0 10px #ff9a90;}
  #overlay.lightsMode h1,#overlay.lightsMode .sub,#overlay.lightsMode #ctrlList,#overlay.lightsMode #startBtn{display:none;}
  #overlay.lightsMode{background:transparent;}
  #overlay.lightsMode #lights{display:flex;}
  @media (prefers-reduced-motion: reduce){ #startBtn{transition:none;} }
  @media (max-width:560px){ #ctrlList{grid-template-columns:auto;} #miniWrap{transform:scale(.78);transform-origin:top right;} }
</style>
</head>
<body>
<canvas id="game"></canvas>
<div id="vignette"></div>

<!-- HUD -->
<div class="hud" id="lapPanel">
  <b>APEX GP · NIGHT</b><br>
  <span class="lbl">LAP</span><span class="val" id="lapNum">1</span><br>
  <span class="lbl">TIME</span><span class="val" id="lapTime">--:--.-</span><br>
  <span class="lbl">LAST</span><span class="val" id="lastLap">--:--.-</span><br>
  <span class="lbl">BEST</span><span class="val" id="bestLap">--:--.-</span>
</div>
<div class="hud" id="speedPanel">
  <div id="speedVal">0</div>
  <div id="speedUnit">KM/H</div>
  <div id="gearBox">N</div>
</div>
<div class="hud" id="msg"></div>
<div class="hud" id="offTrack">ON THE GRASS — ease back onto the track (or press R)</div>
<div class="hud" id="miniWrap"><canvas id="minimap" width="280" height="280"></canvas></div>

<!-- Buttons -->
<div id="btnBar">
  <button class="ctl" id="camBtn" title="Toggle camera (C)">CAM: CHASE</button>
  <button class="ctl" id="resetBtn" title="Reset to track (R)">RESET (R)</button>
  <button class="ctl" id="sndBtn" title="Toggle engine sound">SOUND: ON</button>
</div>

<!-- Touch pads -->
<div class="pad" id="padL">
  <div class="padBtn" id="tLeft">◀</div>
  <div class="padBtn" id="tRight">▶</div>
</div>
<div class="pad" id="padR">
  <div class="padBtn" id="tGas">GAS</div>
  <div class="padBtn" id="tBrake">BRAKE</div>
</div>

<!-- Start overlay -->
<div id="overlay">
  <h1>APEX <span>GP</span></h1>
  <div class="sub">NIGHT STREET CIRCUIT · <i>16 CORNERS</i> · FLOODLIT</div>
  <div id="ctrlList">
    <span><span class="k">W / ↑</span> Accelerate</span>
    <span><span class="k">S / ↓</span> Brake / Reverse</span>
    <span><span class="k">A D / ← →</span> Steer</span>
    <span><span class="k">C</span> Switch camera view</span>
    <span><span class="k">R</span> Reset car to track</span>
    <span><span class="k">Tip</span> 300·200·100 boards = brake!</span>
  </div>
  <button id="startBtn">START RACE</button>
  <div id="lights">
    <div class="light"></div><div class="light"></div><div class="light"></div><div class="light"></div><div class="light"></div>
  </div>
</div>

<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
<script>
// GAME-JS-START
(function(){
'use strict';

/* ============================================================
   RENDERER / SCENE  — filmic PBR pipeline
============================================================ */
const canvas = document.getElementById('game');
const renderer = new THREE.WebGLRenderer({canvas, antialias:true});
renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.shadowMap.enabled = true;
renderer.shadowMap.type = THREE.PCFSoftShadowMap;
renderer.outputEncoding = THREE.sRGBEncoding;
renderer.toneMapping = THREE.ACESFilmicToneMapping;
renderer.toneMappingExposure = 1.25;

const scene = new THREE.Scene();
scene.background = new THREE.Color(0x070a18);
scene.fog = new THREE.Fog(0x10142e, 140, 680);

const camera = new THREE.PerspectiveCamera(70, window.innerWidth/window.innerHeight, 0.1, 1500);

window.addEventListener('resize', () => {
  camera.aspect = window.innerWidth/window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
});

/* ---------- lighting: floodlit night ---------- */
scene.add(new THREE.HemisphereLight(0x44598f, 0x0a0c14, 0.8));
const flood = new THREE.DirectionalLight(0xdcecff, 1.6);
flood.position.set(90, 160, 50);
flood.castShadow = true;
flood.shadow.mapSize.set(2048, 2048);
flood.shadow.camera.left = -40; flood.shadow.camera.right = 40;
flood.shadow.camera.top = 40;   flood.shadow.camera.bottom = -40;
flood.shadow.camera.far = 500;
scene.add(flood); scene.add(flood.target);
const carLight = new THREE.PointLight(0xbfe2ff, 1.2, 60, 2);
scene.add(carLight);

/* ---------- helper: canvas textures ---------- */
function makeCanvas(w,h,draw){
  const c = document.createElement('canvas'); c.width=w; c.height=h;
  draw(c.getContext('2d'), w, h);
  return new THREE.CanvasTexture(c);
}
const glowTex = makeCanvas(128,128,(g,w,h)=>{
  const r = g.createRadialGradient(64,64,2,64,64,62);
  r.addColorStop(0,'rgba(255,255,255,1)');
  r.addColorStop(0.35,'rgba(255,255,255,0.45)');
  r.addColorStop(1,'rgba(255,255,255,0)');
  g.fillStyle=r; g.fillRect(0,0,w,h);
});
function glowSprite(color, scale, opacity){
  const s = new THREE.Sprite(new THREE.SpriteMaterial({
    map:glowTex, color:color, transparent:true, opacity:opacity===undefined?0.9:opacity,
    blending:THREE.AdditiveBlending, depthWrite:false
  }));
  s.scale.set(scale,scale,1);
  return s;
}
function neonTextTex(text, color){
  return makeCanvas(512,128,(g)=>{
    g.fillStyle='rgba(7,10,20,0.94)'; g.fillRect(0,0,512,128);
    g.strokeStyle=color; g.lineWidth=4; g.strokeRect(6,6,500,116);
    g.font='800 64px ui-monospace, Menlo, monospace';
    g.textAlign='center'; g.textBaseline='middle';
    g.shadowColor=color; g.shadowBlur=26;
    g.fillStyle=color; g.fillText(text,256,68);
    g.shadowBlur=0; g.fillStyle='#fff'; g.globalAlpha=0.85; g.fillText(text,256,68);
  });
}

/* ---------- sky dome: dusk gradient + stars ---------- */
function drawSky(g,w,h,stars){
  const gr = g.createLinearGradient(0,0,0,h);
  gr.addColorStop(0.00,'#070b18');
  gr.addColorStop(0.45,'#101740');
  gr.addColorStop(0.72,'#2c2660');
  gr.addColorStop(0.88,'#583470');
  gr.addColorStop(1.00,'#221840');
  g.fillStyle=gr; g.fillRect(0,0,w,h);
  if (!stars) return;
  for (let i=0;i<340;i++){
    const y = Math.random()*h*0.62;
    g.globalAlpha = 0.3 + Math.random()*0.7;
    g.fillStyle = Math.random()<0.12 ? '#aee4ff' : '#ffffff';
    const s = Math.random()<0.1 ? 2 : 1;
    g.fillRect(Math.random()*w, y, s, s);
  }
  g.globalAlpha = 1;
}
const skyTex = makeCanvas(512,512,(g,w,h)=>drawSky(g,w,h,true));
const sky = new THREE.Mesh(
  new THREE.SphereGeometry(1000, 24, 16),
  new THREE.MeshBasicMaterial({map:skyTex, side:THREE.BackSide, fog:false})
);
scene.add(sky);
const moon = glowSprite(0xdfeaff, 110, 0.55); moon.position.set(-420, 330, -520); scene.add(moon);
const moonCore = glowSprite(0xffffff, 36, 0.95); moonCore.position.copy(moon.position); scene.add(moonCore);

/* ---------- environment map for PBR reflections ---------- */
{
  const envScene = new THREE.Scene();
  const envSky = new THREE.Mesh(new THREE.SphereGeometry(50, 16, 12),
    new THREE.MeshBasicMaterial({map:makeCanvas(256,256,(g,w,h)=>drawSky(g,w,h,false)), side:THREE.BackSide}));
  envScene.add(envSky);
  const blob = (color,x,y,z,s)=>{
    const m = new THREE.Mesh(new THREE.SphereGeometry(s,8,8),
      new THREE.MeshBasicMaterial({color:color}));
    m.position.set(x,y,z); envScene.add(m);
  };
  blob(0x9fe9ff, 20, 14, -22, 5);   // cool floodlight bounce
  blob(0xff4fa3, -24, 10, 14, 4);   // neon pink
  blob(0xffb45e, 14, 6, 24, 4);     // sodium city glow
  blob(0xdfeaff, -16, 30, -10, 3);  // moon
  const floor = new THREE.Mesh(new THREE.PlaneGeometry(120,120),
    new THREE.MeshBasicMaterial({color:0x0a0e16}));
  floor.rotation.x = -Math.PI/2; floor.position.y = -2; envScene.add(floor);
  const pmrem = new THREE.PMREMGenerator(renderer);
  scene.environment = pmrem.fromScene(envScene, 0.04).texture;
  pmrem.dispose();
}

/* ============================================================
   TRACK — 16-corner technical street circuit
============================================================ */
const controlPts = [
  [-230,-190],[ -60,-194],[  90,-190],
  [ 175,-188],[ 222,-170],[ 228,-120],[ 185,-105],
  [ 100,-108],[  55, -80],[  95, -45],[ 150, -40],
  [ 195,  -5],[ 155,  40],[ 200,  85],[ 165, 125],
  [ 120, 170],[  40, 192],
  [ -60, 196],[-115, 178],[-150, 200],
  [-205, 178],[-222, 120],
  [-180,  75],[-228,  20],[-185, -35],
  [-228, -90],[-232,-150]
].map(p => new THREE.Vector3(p[0], 0, p[1]));
const curve = new THREE.CatmullRomCurve3(controlPts, true, 'centripetal');

const N = 900;
const ROAD_W = 13, KERB_W = 1.3;
const centers = [], tangents = [], sides = [];
const UP = new THREE.Vector3(0,1,0);
for (let i=0;i<N;i++){
  const t = i/N;
  centers.push(curve.getPointAt(t));
  const tg = curve.getTangentAt(t).normalize();
  tangents.push(tg);
  sides.push(new THREE.Vector3().crossVectors(UP, tg).normalize()); // left of travel
}
const cumLen = [0];
for (let i=1;i<=N;i++) cumLen.push(cumLen[i-1] + centers[i-1].distanceTo(centers[i%N]));
const lapLen = cumLen[N];
const perM = N / lapLen;

const curvature = [], turnSign = [];
for (let i=0;i<N;i++){
  const a = tangents[i], b = tangents[(i+6)%N];
  curvature.push(a.angleTo(b));
  turnSign.push(Math.sign(a.z*b.x - a.x*b.z)); // >0 = left turn
}

function ribbon(halfIn, halfOut, y, colorFn, withUV){
  const pos=[], col=[], uv=[], idx=[];
  for (let i=0;i<N;i++){
    const c=centers[i], s=sides[i];
    pos.push(c.x+s.x*halfOut, y, c.z+s.z*halfOut,
             c.x+s.x*halfIn,  y, c.z+s.z*halfIn);
    const cc = colorFn ? colorFn(i) : [1,1,1];
    col.push(cc[0],cc[1],cc[2], cc[0],cc[1],cc[2]);
    if (withUV){ const v = cumLen[i]*0.10; uv.push(0,v, 1,v); }
    const a=i*2, b=i*2+1, c2=((i+1)%N)*2, d=((i+1)%N)*2+1;
    idx.push(a,b,c2, b,d,c2);
  }
  const g = new THREE.BufferGeometry();
  g.setAttribute('position', new THREE.Float32BufferAttribute(pos,3));
  g.setAttribute('color', new THREE.Float32BufferAttribute(col,3));
  if (withUV) g.setAttribute('uv', new THREE.Float32BufferAttribute(uv,2));
  g.setIndex(idx);
  g.computeVertexNormals();
  return g;
}

// asphalt: procedural grain + darker racing groove, slight wet sheen
const asphaltTex = makeCanvas(256,256,(g)=>{
  g.fillStyle='#2e323b'; g.fillRect(0,0,256,256);
  for (let i=0;i<2600;i++){
    const v = Math.random();
    g.fillStyle = v<0.5 ? 'rgba(255,255,255,0.045)' : 'rgba(0,0,0,0.10)';
    g.fillRect(Math.random()*256, Math.random()*256, 1.6, 1.6);
  }
  const groove = g.createLinearGradient(0,0,256,0);   // x = across the road
  groove.addColorStop(0.00,'rgba(0,0,0,0)');
  groove.addColorStop(0.32,'rgba(0,0,0,0.30)');
  groove.addColorStop(0.50,'rgba(0,0,0,0.42)');
  groove.addColorStop(0.68,'rgba(0,0,0,0.30)');
  groove.addColorStop(1.00,'rgba(0,0,0,0)');
  g.fillStyle=groove; g.fillRect(0,0,256,256);
});
asphaltTex.wrapS = asphaltTex.wrapT = THREE.RepeatWrapping;
const road = new THREE.Mesh(ribbon(-ROAD_W/2, ROAD_W/2, 0.05, null, true),
  new THREE.MeshStandardMaterial({map:asphaltTex, bumpMap:asphaltTex, bumpScale:0.015,
    roughness:0.5, metalness:0.0, envMapIntensity:0.8}));
road.receiveShadow = true;
scene.add(road);

// glowing edge lines
const lineMat = new THREE.MeshBasicMaterial({color:0xb9f1ff});
scene.add(new THREE.Mesh(ribbon(ROAD_W/2-0.32, ROAD_W/2-0.04, 0.08), lineMat));
scene.add(new THREE.Mesh(ribbon(-ROAD_W/2+0.04, -ROAD_W/2+0.32, 0.08), lineMat));

// kerbs
function kerbColor(i){
  const corner = curvature[i] > 0.028;
  const stripe = Math.floor(i/6)%2===0;
  if (!corner) return [0.13,0.15,0.19];
  return stripe ? [0.95,0.16,0.20] : [0.95,0.96,1.0];
}
const kerbMat = new THREE.MeshBasicMaterial({vertexColors:true});
scene.add(new THREE.Mesh(ribbon( ROAD_W/2,  ROAD_W/2+KERB_W, 0.07, kerbColor), kerbMat));
scene.add(new THREE.Mesh(ribbon(-ROAD_W/2-KERB_W, -ROAD_W/2, 0.07, kerbColor), kerbMat));

// surroundings: dark night turf
const grassTex = makeCanvas(256,256,(g)=>{
  g.fillStyle='#15241b'; g.fillRect(0,0,256,256);
  for (let i=0;i<900;i++){
    g.fillStyle = Math.random()<0.5 ? '#1a2c20' : '#101c15';
    g.fillRect(Math.random()*256, Math.random()*256, 3, 3);
  }
});
grassTex.wrapS = grassTex.wrapT = THREE.RepeatWrapping;
grassTex.repeat.set(110,110);
const ground = new THREE.Mesh(new THREE.PlaneGeometry(2000,2000),
  new THREE.MeshStandardMaterial({map:grassTex, roughness:1.0, metalness:0, envMapIntensity:0.15}));
ground.rotation.x = -Math.PI/2;
ground.receiveShadow = true;
scene.add(ground);

// start / finish checker strip
const chkTex = makeCanvas(128,32,(c)=>{
  for (let x=0;x<16;x++) for (let y=0;y<4;y++){
    c.fillStyle = (x+y)%2 ? '#0c0d10' : '#e8ecf2';
    c.fillRect(x*8, y*8, 8, 8);
  }
});
const startMesh = new THREE.Mesh(new THREE.PlaneGeometry(ROAD_W, 3.2),
  new THREE.MeshBasicMaterial({map:chkTex}));
startMesh.rotation.x = -Math.PI/2;
startMesh.position.copy(centers[0]).setY(0.09);
startMesh.rotation.z = Math.atan2(tangents[0].x, tangents[0].z);
scene.add(startMesh);

/* ============================================================
   VENUE
============================================================ */
const envSpin = [];

/* ---- city skyline ring ---- */
const windowTexes = [0,1,2].map(()=>makeCanvas(64,128,(g)=>{
  g.fillStyle='#070a16'; g.fillRect(0,0,64,128);
  for (let y=4;y<124;y+=8) for (let x=4;x<60;x+=8){
    const r = Math.random();
    if (r < 0.42){
      g.fillStyle = r<0.10 ? '#79e6ff' : (r<0.20 ? '#ffd9a0' : '#ffb45e');
      g.globalAlpha = 0.55+Math.random()*0.45;
      g.fillRect(x,y,5,5);
    }
  }
  g.globalAlpha=1;
}));
const roofMat = new THREE.MeshBasicMaterial({color:0x05070d});
for (let k=0;k<38;k++){
  const ang = (k/38)*Math.PI*2 + Math.random()*0.12;
  const rad = 330 + Math.random()*120;
  const w = 18+Math.random()*26, d = 18+Math.random()*26, h = 28+Math.random()*75;
  const winMat = new THREE.MeshBasicMaterial({map:windowTexes[k%3]});
  const b = new THREE.Mesh(new THREE.BoxGeometry(w,h,d),
    [winMat,winMat,roofMat,roofMat,winMat,winMat]);
  b.position.set(Math.cos(ang)*rad, h/2, Math.sin(ang)*rad);
  b.rotation.y = Math.random()*Math.PI;
  scene.add(b);
  if (Math.random()<0.4){
    const bc = glowSprite(0xff3b4a, 6, 0.8);
    bc.position.set(b.position.x, h+2, b.position.z);
    scene.add(bc);
  }
}

/* ---- trackside placement helpers ---- */
function trackPoint(i, lateral, y){
  i = ((i%N)+N)%N;
  return new THREE.Vector3(
    centers[i].x + sides[i].x*lateral, y||0, centers[i].z + sides[i].z*lateral);
}
function headingAt(i){ i=((i%N)+N)%N; return Math.atan2(tangents[i].x, tangents[i].z); }

/* ---- pit building along the main straight ---- */
{
  const i = Math.round(N*0.045);
  const pitWin = new THREE.MeshBasicMaterial({map:windowTexes[1]});
  const pit = new THREE.Mesh(new THREE.BoxGeometry(12, 9, 110),
    [pitWin,pitWin,roofMat,roofMat,roofMat,roofMat]);
  pit.position.copy(trackPoint(i, -(ROAD_W/2+16), 4.5));
  pit.rotation.y = headingAt(i);   // long axis along the straight
  scene.add(pit);
  const strip = new THREE.Mesh(new THREE.BoxGeometry(12.4, 0.5, 110.4),
    new THREE.MeshBasicMaterial({color:0x36e6ff}));
  strip.position.copy(pit.position); strip.position.y = 9.2;
  strip.rotation.y = pit.rotation.y;
  scene.add(strip);
}

/* ---- grandstands: built facing +Z, then lookAt the track
       (long side runs PARALLEL to the road, never across it) ---- */
const crowdTex = makeCanvas(128,64,(g)=>{
  g.fillStyle='#11141d'; g.fillRect(0,0,128,64);
  const cols=['#ff5964','#ffd34d','#4fc3ff','#9be564','#ff8bd1','#e8ecf2','#b89bff'];
  for (let i=0;i<560;i++){
    g.fillStyle = cols[(Math.random()*cols.length)|0];
    g.fillRect(Math.random()*128, Math.random()*64, 2, 2);
  }
});
function grandstand(i, side, len){
  const grp = new THREE.Group();
  const base = new THREE.Mesh(new THREE.BoxGeometry(len,1.2,10),
    new THREE.MeshLambertMaterial({color:0x1a2030}));
  base.position.y = 0.6;
  const crowd = new THREE.Mesh(new THREE.PlaneGeometry(len,7.5),
    new THREE.MeshBasicMaterial({map:crowdTex}));
  crowd.position.set(0, 3.9, 4.2);
  crowd.rotation.x = -0.42;
  const roof = new THREE.Mesh(new THREE.BoxGeometry(len+2,0.4,11),
    new THREE.MeshLambertMaterial({color:0x232a3a}));
  roof.position.set(0, 7.6, 0);
  const edge = new THREE.Mesh(new THREE.BoxGeometry(len+2,0.28,0.3),
    new THREE.MeshBasicMaterial({color:0x36e6ff}));
  edge.position.set(0, 7.6, 5.6);
  grp.add(base, crowd, roof, edge);
  grp.position.copy(trackPoint(i, side*(ROAD_W/2+19)));
  grp.lookAt(centers[i].x, 0, centers[i].z);  // front (+z) faces the track
  scene.add(grp);
}
grandstand(Math.round(N*0.96), 1, 46);   // main straight
grandstand(Math.round(N*0.155), 1, 34);  // hairpin exit
grandstand(Math.round(N*0.60), -1, 40);  // top straight

/* ---- floodlight towers ---- */
const poleMat = new THREE.MeshLambertMaterial({color:0x141821});
for (let k=0;k<9;k++){
  const i = Math.round(k*N/9 + 30);
  const grp = new THREE.Group();
  const pole = new THREE.Mesh(new THREE.CylinderGeometry(0.28,0.42,15,6), poleMat);
  pole.position.y = 7.5;
  const head = new THREE.Mesh(new THREE.BoxGeometry(3.2,1.1,0.5),
    new THREE.MeshBasicMaterial({color:0xeaf4ff}));
  head.position.y = 15;
  grp.add(pole, head);
  const gl = glowSprite(0xcfe8ff, 14, 0.55); gl.position.y = 15; grp.add(gl);
  grp.position.copy(trackPoint(i, (k%2?1:-1)*(ROAD_W/2+8)));
  grp.lookAt(centers[i].x, 0, centers[i].z);  // yaw only — pole stays vertical
  scene.add(grp);
}

/* ---- glowing barrier blocks on corner exteriors ---- */
{
  const red  = new THREE.MeshBasicMaterial({color:0xff3b4a});
  const wht  = new THREE.MeshBasicMaterial({color:0xdfe6f2});
  let n = 0, alt = 0;
  for (let i=0;i<N && n<120;i+=6){
    if (curvature[i] < 0.05) continue;
    const out = -turnSign[i] * (ROAD_W/2 + KERB_W + 1.5);
    const b = new THREE.Mesh(new THREE.BoxGeometry(2.6,0.95,0.8), (alt++%2)?red:wht);
    b.position.copy(trackPoint(i, out, 0.5));
    b.rotation.y = headingAt(i);
    scene.add(b); n++;
  }
}

/* ---- braking boards 300/200/100 before Turn 1 ---- */
{
  let entry = 0;
  for (let i=Math.round(N*0.03); i<N; i++){ if (curvature[i] > 0.07){ entry = i; break; } }
  ['100','200','300'].forEach((label, k) => {
    const i = Math.round(entry - (k+1)*100*perM);
    const tex = makeCanvas(128,128,(g)=>{
      g.fillStyle='#0a0e1a'; g.fillRect(0,0,128,128);
      g.strokeStyle='#36e6ff'; g.lineWidth=7; g.strokeRect(5,5,118,118);
      g.fillStyle='#fff'; g.font='800 52px ui-monospace,monospace';
      g.textAlign='center'; g.textBaseline='middle';
      g.shadowColor='#36e6ff'; g.shadowBlur=14;
      g.fillText(label,64,66);
    });
    const grp = new THREE.Group();
    const pole = new THREE.Mesh(new THREE.BoxGeometry(0.16,2.2,0.16), poleMat);
    pole.position.y = 1.1;
    const sign = new THREE.Mesh(new THREE.PlaneGeometry(2.0,2.0),
      new THREE.MeshBasicMaterial({map:tex, side:THREE.DoubleSide}));
    sign.position.y = 3.1;
    grp.add(pole, sign);
    grp.position.copy(trackPoint(i, ROAD_W/2 + 2.4));
    grp.rotation.y = headingAt(i) + Math.PI;
    scene.add(grp);
  });
}

/* ---- gantries: start banner + sector bridge ---- */
function gantry(i, text){
  const grp = new THREE.Group();
  const pillarG = new THREE.BoxGeometry(0.7,8,0.7);
  for (const s of [1,-1]){
    const p = new THREE.Mesh(pillarG, poleMat);
    p.position.set(s*(ROAD_W/2+2.6), 4, 0);
    grp.add(p);
  }
  const beam = new THREE.Mesh(new THREE.BoxGeometry(ROAD_W+6.5,1.5,1.0), poleMat);
  beam.position.y = 7.6;
  grp.add(beam);
  const banner = new THREE.Mesh(new THREE.PlaneGeometry(11,2.6),
    new THREE.MeshBasicMaterial({map:neonTextTex(text,'#36e6ff'), side:THREE.DoubleSide}));
  banner.position.set(0, 7.6, -0.62);
  banner.rotation.y = Math.PI;
  grp.add(banner);
  grp.position.copy(centers[i]).setY(0);
  grp.rotation.y = headingAt(i);
  scene.add(grp);
  return grp;
}
const startGantry = gantry(0, 'APEX GP');
for (let k=0;k<5;k++){
  const d = new THREE.Mesh(new THREE.SphereGeometry(0.22,8,8),
    new THREE.MeshBasicMaterial({color:0x3a0e10}));
  d.position.set((k-2)*1.1, 6.6, 0);
  startGantry.add(d);
}
gantry(Math.round(N*0.585), 'SECTOR 3');

/* ---- palm trees ---- */
const trunkMat = new THREE.MeshLambertMaterial({color:0x3a3148});
const frondMat = new THREE.MeshLambertMaterial({color:0x1d4632});
function palm(x,z,s){
  const grp = new THREE.Group();
  const trunk = new THREE.Mesh(new THREE.CylinderGeometry(0.18*s,0.34*s,6*s,5), trunkMat);
  trunk.position.y = 3*s;
  grp.add(trunk);
  for (let k=0;k<6;k++){
    const f = new THREE.Mesh(new THREE.BoxGeometry(3.4*s,0.07,0.7*s), frondMat);
    f.position.y = 6*s;
    f.rotation.y = k*Math.PI/3;
    f.rotation.z = 0.45;
    f.translateX(1.35*s);
    grp.add(f);
  }
  grp.position.set(x,0,z);
  scene.add(grp);
}
let placed=0, guard=0;
while (placed<26 && guard++<2500){
  const x=(Math.random()-0.5)*620, z=(Math.random()-0.5)*580;
  if (Math.hypot(x,z) > 300) continue;
  let near=false;
  for (let i=0;i<N;i+=10){
    const dx=centers[i].x-x, dz=centers[i].z-z;
    if (dx*dx+dz*dz < 23*23){ near=true; break; }
  }
  if (!near){ palm(x,z, 0.8+Math.random()*0.8); placed++; }
}

/* ---- giant observation wheel ---- */
{
  const wheel = new THREE.Group();
  const R = 42;
  const rim = new THREE.Mesh(new THREE.TorusGeometry(R, 0.9, 8, 48),
    new THREE.MeshBasicMaterial({color:0x36e6ff}));
  const rim2 = new THREE.Mesh(new THREE.TorusGeometry(R*0.82, 0.4, 6, 40),
    new THREE.MeshBasicMaterial({color:0x1a5e74}));
  const spinner = new THREE.Group();
  spinner.add(rim, rim2);
  for (let k=0;k<6;k++){
    const sp = new THREE.Mesh(new THREE.BoxGeometry(0.5, R*2, 0.5),
      new THREE.MeshLambertMaterial({color:0x2a3550}));
    sp.rotation.z = k*Math.PI/6;
    spinner.add(sp);
  }
  for (let k=0;k<16;k++){
    const a = k/16*Math.PI*2;
    const cab = glowSprite(k%2 ? 0xff4fa3 : 0x7ce8ff, 5.5, 0.95);
    cab.position.set(Math.cos(a)*R, Math.sin(a)*R, 0);
    spinner.add(cab);
  }
  spinner.position.y = R+8;
  wheel.add(spinner);
  for (const s of [1,-1]){
    const leg = new THREE.Mesh(new THREE.BoxGeometry(1.4, R+10, 1.4), poleMat);
    leg.position.set(s*12, (R+8)/2, 0);
    leg.rotation.z = s*0.24;
    wheel.add(leg);
  }
  const hub = glowSprite(0xffffff, 10, 0.9); hub.position.y = R+8; wheel.add(hub);
  wheel.position.set(330, 0, 160);
  wheel.rotation.y = Math.atan2(-330, -160);
  scene.add(wheel);
  envSpin.push(dt => { spinner.rotation.z += dt*0.07; });
}

/* ---- searchlight beams ---- */
function beacon(x,z,speed,phase){
  const grp = new THREE.Group();
  const cone = new THREE.Mesh(new THREE.ConeGeometry(20, 190, 12, 1, true),
    new THREE.MeshBasicMaterial({color:0x6fc8ff, transparent:true, opacity:0.06,
      blending:THREE.AdditiveBlending, side:THREE.DoubleSide, depthWrite:false}));
  cone.geometry.translate(0, 95, 0);
  cone.rotation.x = Math.PI;
  const tilt = new THREE.Group();
  tilt.add(cone);
  tilt.rotation.z = 0.42;
  grp.add(tilt);
  grp.position.set(x, 0, z);
  scene.add(grp);
  grp.rotation.y = phase;
  envSpin.push(dt => { grp.rotation.y += dt*speed; });
}
beacon( 270, -260, 0.30, 1.2);
beacon(-310,  240, -0.24, 3.8);
beacon(  40,  330, 0.20, 5.1);

/* ---- blimp ---- */
{
  const blimp = new THREE.Group();
  const body = new THREE.Mesh(new THREE.SphereGeometry(9, 12, 10),
    new THREE.MeshLambertMaterial({color:0xaeb6c6}));
  body.scale.set(2.4,1,1);
  const fin = new THREE.Mesh(new THREE.BoxGeometry(2.5,6,0.4),
    new THREE.MeshLambertMaterial({color:0x8d96a8}));
  fin.position.x = -20;
  const gondola = new THREE.Mesh(new THREE.BoxGeometry(6,2,2.4),
    new THREE.MeshBasicMaterial({map:windowTexes[2]}));
  gondola.position.y = -9;
  const blink = glowSprite(0xff3b4a, 7, 1);
  blink.position.set(0, 11, 0);
  blimp.add(body, fin, gondola, blink);
  scene.add(blimp);
  let T = 0;
  envSpin.push(dt => {
    T += dt;
    const a = T*0.045;
    blimp.position.set(Math.cos(a)*250, 135, Math.sin(a)*250);
    blimp.rotation.y = -a;
    blink.material.opacity = (Math.sin(T*4) > 0.2) ? 1 : 0.05;
  });
}

/* ============================================================
   CAR — glossy PBR open-wheel
============================================================ */
const car = new THREE.Group();
const livery  = new THREE.MeshStandardMaterial({color:0xe9eef6, metalness:0.7, roughness:0.22, envMapIntensity:1.2});
const liveryB = new THREE.MeshStandardMaterial({color:0x10141d, metalness:0.5, roughness:0.35, envMapIntensity:1.0});
const neonMat = new THREE.MeshBasicMaterial({color:0x36e6ff});
const wingMat = new THREE.MeshStandardMaterial({color:0x0c0f16, metalness:0.6, roughness:0.3, envMapIntensity:1.0});

function box(w,h,d, mat, x,y,z){
  const m = new THREE.Mesh(new THREE.BoxGeometry(w,h,d), mat);
  m.position.set(x,y,z); m.castShadow = true; car.add(m); return m;
}
box(0.55,0.30,2.2, livery, 0, 0.42,  1.55);
box(0.50,0.10,0.8, neonMat,0, 0.60,  1.70);
box(1.05,0.45,1.9, livery, 0, 0.48,  0.15);
box(0.95,0.55,1.6, livery, 0, 0.52, -1.25);
box(0.34,0.42,0.9, liveryB,0, 0.95, -1.30);
box(0.36,0.06,0.92,neonMat,0, 1.17, -1.30);
box(0.45,0.34,1.5, liveryB, 0.72, 0.40, -0.45);
box(0.45,0.34,1.5, liveryB,-0.72, 0.40, -0.45);
box(0.47,0.08,1.52,neonMat, 0.72, 0.60, -0.45);
box(0.47,0.08,1.52,neonMat,-0.72, 0.60, -0.45);
box(1.9,0.07,0.55, wingMat, 0, 0.26,  2.55);
box(0.09,0.30,0.4, neonMat, 0.92,0.42, 2.5);
box(0.09,0.30,0.4, neonMat,-0.92,0.42, 2.5);
box(1.45,0.08,0.50, wingMat, 0, 1.02, -2.15);
box(0.08,0.45,0.50, neonMat, 0.70,0.80,-2.15);
box(0.08,0.45,0.50, neonMat,-0.70,0.80,-2.15);
const rainMat = new THREE.MeshBasicMaterial({color:0x550a0e});
box(0.22,0.22,0.08, rainMat, 0, 0.55, -2.42);
const halo = new THREE.Mesh(new THREE.TorusGeometry(0.42,0.05,8,18,Math.PI),
  new THREE.MeshStandardMaterial({color:0x1c1f24, metalness:0.6, roughness:0.4}));
halo.rotation.x = Math.PI/2; halo.position.set(0,0.92,0.42); halo.castShadow = true;
car.add(halo);
box(0.07,0.34,0.07, liveryB, 0,0.78,0.80);
const helmet = new THREE.Mesh(new THREE.SphereGeometry(0.21,10,10),
  new THREE.MeshStandardMaterial({color:0xff4fa3, metalness:0.4, roughness:0.25, envMapIntensity:1.2}));
helmet.position.set(0,0.78,0.05); helmet.castShadow = true;
car.add(helmet);
const wheelRim = new THREE.Mesh(new THREE.TorusGeometry(0.16,0.03,6,14),
  new THREE.MeshStandardMaterial({color:0x222831, metalness:0.5, roughness:0.5}));
wheelRim.position.set(0,0.72,0.62);
car.add(wheelRim);
const dash = new THREE.Mesh(new THREE.PlaneGeometry(0.22,0.08),
  new THREE.MeshBasicMaterial({color:0x4ff0c8}));
dash.position.set(0,0.74,0.605);
dash.rotation.x = -0.35;
car.add(dash);

const tyreMat = new THREE.MeshStandardMaterial({color:0x0c0d11, roughness:0.92, metalness:0});
const rimMat  = new THREE.MeshBasicMaterial({color:0x36e6ff});
const wheels = [], brakeGlows = [];
function wheel(x,z,front){
  const grp = new THREE.Group();
  const tyre = new THREE.Mesh(new THREE.CylinderGeometry(0.45,0.45,0.42,14), tyreMat);
  tyre.rotation.z = Math.PI/2; tyre.castShadow = true;
  const rim = new THREE.Mesh(new THREE.CylinderGeometry(0.24,0.24,0.44,10), rimMat);
  rim.rotation.z = Math.PI/2;
  grp.add(tyre, rim);
  const bg = glowSprite(0xff8a2a, 1.5, 0);   // brake-disc glow under braking
  grp.add(bg);
  brakeGlows.push(bg);
  grp.position.set(x, 0.45, z);
  car.add(grp);
  wheels.push({grp, tyre, rim, front, spin:0});
}
wheel( 0.85, 1.55,true);  wheel(-0.85, 1.55,true);
wheel( 0.92,-1.55,false); wheel(-0.92,-1.55,false);
scene.add(car);

/* ============================================================
   STATE & PHYSICS
============================================================ */
const S = {
  pos: centers[0].clone(),
  heading: Math.atan2(tangents[0].x, tangents[0].z),
  speed: 0, steer: 0,
  nearIdx: 0, onTrack: true,
  cam: 'chase', running: false,
  lap: 1, lapStart: 0, last: null, best: null,
  cp1: false, cp2: false,
};
const input = {gas:false, brake:false, left:false, right:false};

const MAX_SPEED = 78;
const GRASS_MAX = 19;
const ACCEL = 16.5, BRAKE = 30, COAST = 1.6;

function nearestIndex(full){
  let bestI = S.nearIdx, bestD = Infinity;
  const span = full ? N : 35;
  for (let k=-span;k<=span;k++){
    const i = ((S.nearIdx + k) % N + N) % N;
    const dx = centers[i].x - S.pos.x, dz = centers[i].z - S.pos.z;
    const d = dx*dx + dz*dz;
    if (d < bestD){ bestD = d; bestI = i; }
  }
  S.nearIdx = bestI;
  return Math.sqrt(bestD);
}

function resetToTrack(){
  nearestIndex(true);
  const i = S.nearIdx;
  S.pos.copy(centers[i]); S.pos.y = 0;
  S.heading = Math.atan2(tangents[i].x, tangents[i].z);
  S.speed = 0;
}

function physics(dt){
  const target = (input.left?1:0) + (input.right?-1:0);
  S.steer += (target - S.steer) * Math.min(1, dt*8);

  if (S.running){
    if (input.gas){
      const a = ACCEL * (1 - Math.max(0,S.speed)/MAX_SPEED*0.75);
      S.speed += a*dt;
    }
    if (input.brake){
      if (S.speed > 0.4) S.speed -= BRAKE*dt;
      else S.speed = Math.max(S.speed - 7*dt, -8);
    }
  }
  const drag = COAST + S.speed*S.speed*0.0021;
  if (S.speed > 0) S.speed = Math.max(0, S.speed - drag*dt);
  else if (S.speed < 0) S.speed = Math.min(0, S.speed + (COAST+2)*dt);

  const lat = nearestIndex(false);
  S.onTrack = lat <= ROAD_W/2 + KERB_W + 0.4;
  if (!S.onTrack){
    if (S.speed > GRASS_MAX) S.speed -= 26*dt;
    S.speed -= S.speed*0.9*dt;
  }
  if (S.speed > MAX_SPEED) S.speed = MAX_SPEED;

  const grip = Math.min(1, Math.abs(S.speed)/4);
  const rate = 2.1 / (1 + Math.abs(S.speed)*0.022);
  S.heading += S.steer * rate * grip * dt * Math.sign(S.speed || 1);

  S.pos.x += Math.sin(S.heading) * S.speed * dt;
  S.pos.z += Math.cos(S.heading) * S.speed * dt;

  if (S.running){
    const i = S.nearIdx, prev = S.prevIdx ?? i;
    const cpA = Math.floor(N/3), cpB = Math.floor(2*N/3);
    if (!S.cp1 && i >= cpA && i < cpA+50) S.cp1 = true;
    if (S.cp1 && !S.cp2 && i >= cpB && i < cpB+50) S.cp2 = true;
    if (prev > N-50 && i < 50 && S.cp1 && S.cp2){
      const t = performance.now()/1000;
      S.last = t - S.lapStart;
      if (S.best === null || S.last < S.best){ S.best = S.last; flash('BEST LAP!'); }
      else flash('LAP ' + S.lap + ' DONE');
      S.lapStart = t; S.lap++; S.cp1 = S.cp2 = false;
    }
    S.prevIdx = i;
  }

  // apply to model
  car.position.copy(S.pos);
  car.position.y = 0.06;
  car.rotation.set(0, S.heading, 0);
  car.rotation.z = -S.steer * Math.min(1, Math.abs(S.speed)/30) * 0.06;
  const spin = S.speed*dt/0.45;
  for (const w of wheels){
    w.spin += spin;
    w.tyre.rotation.x = w.rim.rotation.x = w.spin;
    if (w.front) w.grp.rotation.y = S.steer*0.42;
  }
  wheelRim.rotation.z = S.steer * 1.2;
  const braking = input.brake && S.running && S.speed > 14;
  rainMat.color.setHex(input.brake && S.running ? 0xff2030 : 0x550a0e);
  for (const bg of brakeGlows){
    bg.material.opacity += ((braking ? 0.85 : 0) - bg.material.opacity)*Math.min(1, dt*8);
  }

  flood.position.set(S.pos.x+90, 160, S.pos.z+50);
  flood.target.position.copy(S.pos);
  carLight.position.set(S.pos.x, 7, S.pos.z);
}

/* ============================================================
   CAMERAS
============================================================ */
const fwd = new THREE.Vector3(), tmp = new THREE.Vector3();
let camInit = false;
function updateCamera(dt){
  fwd.set(Math.sin(S.heading), 0, Math.cos(S.heading));
  if (S.cam === 'chase'){
    tmp.copy(S.pos).addScaledVector(fwd,-8.5); tmp.y = 3.4;
    if (!camInit){ camera.position.copy(tmp); camInit = true; }
    camera.position.lerp(tmp, 1 - Math.exp(-6*dt));
    tmp.copy(S.pos).addScaledVector(fwd, 3); tmp.y = 1.1;
    camera.lookAt(tmp);
    camera.fov = 70 + Math.max(0,S.speed)*0.10;
  } else {
    // cockpit cam sits ABOVE the halo and tilts down so the road is visible
    tmp.copy(S.pos).addScaledVector(fwd, 0.28); tmp.y = 1.18;
    camera.position.copy(tmp);
    tmp.copy(S.pos).addScaledVector(fwd, 30); tmp.y = 0.45;
    camera.lookAt(tmp);
    camera.fov = 76 + Math.max(0,S.speed)*0.12;
  }
  camera.updateProjectionMatrix();
}
function setCam(mode){
  S.cam = mode;
  helmet.visible = (mode === 'chase');
  document.getElementById('camBtn').textContent = mode==='chase' ? 'CAM: CHASE' : 'CAM: COCKPIT';
  if (mode === 'chase') camInit = false;
}

/* ============================================================
   SOUND
============================================================ */
let audio = null, soundOn = true;
function initAudio(){
  if (audio) return;
  try{
    const ctx = new (window.AudioContext||window.webkitAudioContext)();
    const osc1 = ctx.createOscillator(); osc1.type='sawtooth';
    const osc2 = ctx.createOscillator(); osc2.type='square';
    const gain = ctx.createGain(); gain.gain.value = 0;
    const filt = ctx.createBiquadFilter(); filt.type='lowpass'; filt.frequency.value=900;
    osc1.connect(filt); osc2.connect(filt); filt.connect(gain); gain.connect(ctx.destination);
    osc1.start(); osc2.start();
    audio = {ctx, osc1, osc2, gain};
  }catch(e){ audio = null; }
}
function updateAudio(){
  if (!audio) return;
  const on = soundOn && S.running;
  const sp = Math.abs(S.speed);
  audio.gain.gain.setTargetAtTime(on ? 0.035 + sp*0.0006 : 0, audio.ctx.currentTime, 0.05);
  const f = 55 + sp*5.2 + (input.gas?22:0);
  audio.osc1.frequency.setTargetAtTime(f, audio.ctx.currentTime, 0.04);
  audio.osc2.frequency.setTargetAtTime(f*0.5, audio.ctx.currentTime, 0.04);
}

/* ============================================================
   HUD / MINIMAP
============================================================ */
const $ = id => document.getElementById(id);
function fmt(t){
  if (t == null) return '--:--.-';
  const m = Math.floor(t/60), s = t - m*60;
  return m + ':' + (s<10?'0':'') + s.toFixed(1);
}
let msgTimer = 0;
function flash(text){
  $('msg').textContent = text;
  $('msg').style.opacity = 1;
  msgTimer = 2.2;
}
function updateHUD(dt){
  $('speedVal').textContent = Math.round(Math.abs(S.speed)*3.6);
  const sp = Math.abs(S.speed);
  let gear = 'N';
  if (S.speed < -0.3) gear = 'R';
  else if (sp > 0.5) gear = String(Math.min(8, 1 + Math.floor(sp/9.5)));
  $('gearBox').textContent = gear;
  $('lapNum').textContent = S.lap;
  $('lapTime').textContent = S.running ? fmt(performance.now()/1000 - S.lapStart) : '--:--.-';
  $('lastLap').textContent = fmt(S.last);
  $('bestLap').textContent = fmt(S.best);
  $('offTrack').style.opacity = (!S.onTrack && S.running) ? 1 : 0;
  if (msgTimer > 0){ msgTimer -= dt; if (msgTimer <= 0) $('msg').style.opacity = 0; }
}

const mini = $('minimap').getContext('2d');
let mm;
{
  let minX=1e9,maxX=-1e9,minZ=1e9,maxZ=-1e9;
  for (const c of centers){
    minX=Math.min(minX,c.x); maxX=Math.max(maxX,c.x);
    minZ=Math.min(minZ,c.z); maxZ=Math.max(maxZ,c.z);
  }
  const pad=24, w=280, h=280;
  const sc = Math.min((w-pad*2)/(maxX-minX), (h-pad*2)/(maxZ-minZ));
  mm = {sc, ox: w/2 - (minX+maxX)/2*sc, oz: h/2 - (minZ+maxZ)/2*sc};
}
function mapPt(x,z){ return [x*mm.sc+mm.ox, z*mm.sc+mm.oz]; }
function drawMinimap(){
  mini.clearRect(0,0,280,280);
  mini.lineWidth = 8; mini.strokeStyle = '#39435f'; mini.lineCap='round'; mini.lineJoin='round';
  mini.beginPath();
  for (let i=0;i<=N;i+=5){
    const c = centers[i%N], p = mapPt(c.x,c.z);
    if (i===0) mini.moveTo(p[0],p[1]); else mini.lineTo(p[0],p[1]);
  }
  mini.stroke();
  const st = mapPt(centers[0].x, centers[0].z);
  mini.fillStyle = '#e8e8e8';
  mini.fillRect(st[0]-5, st[1]-5, 10, 10);
  const cp = mapPt(S.pos.x, S.pos.z);
  mini.fillStyle = '#36e6ff';
  mini.beginPath(); mini.arc(cp[0],cp[1],7,0,Math.PI*2); mini.fill();
  mini.lineWidth = 2; mini.strokeStyle = '#fff'; mini.stroke();
}

/* ============================================================
   INPUT
============================================================ */
const keyMap = {
  'KeyW':'gas','ArrowUp':'gas',
  'KeyS':'brake','ArrowDown':'brake',
  'KeyA':'left','ArrowLeft':'left',
  'KeyD':'right','ArrowRight':'right',
};
window.addEventListener('keydown', e => {
  if (keyMap[e.code]){ input[keyMap[e.code]] = true; e.preventDefault(); }
  if (e.code === 'KeyC') setCam(S.cam==='chase' ? 'cockpit' : 'chase');
  if (e.code === 'KeyR') resetToTrack();
});
window.addEventListener('keyup', e => {
  if (keyMap[e.code]) input[keyMap[e.code]] = false;
});

$('camBtn').addEventListener('click', () => setCam(S.cam==='chase'?'cockpit':'chase'));
$('resetBtn').addEventListener('click', resetToTrack);
$('sndBtn').addEventListener('click', () => {
  soundOn = !soundOn;
  $('sndBtn').textContent = soundOn ? 'SOUND: ON' : 'SOUND: OFF';
  if (soundOn) initAudio();
});

if ('ontouchstart' in window || navigator.maxTouchPoints > 0){
  document.body.classList.add('touch');
  const bind = (id, key) => {
    const el = $(id);
    const on  = ev => { ev.preventDefault(); input[key]=true;  el.classList.add('active'); };
    const off = ev => { ev.preventDefault(); input[key]=false; el.classList.remove('active'); };
    el.addEventListener('pointerdown', on);
    el.addEventListener('pointerup', off);
    el.addEventListener('pointercancel', off);
    el.addEventListener('pointerleave', off);
  };
  bind('tLeft','left'); bind('tRight','right'); bind('tGas','gas'); bind('tBrake','brake');
}

/* ============================================================
   START SEQUENCE
============================================================ */
$('startBtn').addEventListener('click', () => {
  initAudio();
  const ov = $('overlay');
  ov.classList.add('lightsMode');
  const lights = ov.querySelectorAll('.light');
  let i = 0;
  const lightUp = () => {
    if (i < lights.length){
      lights[i++].classList.add('on');
      setTimeout(lightUp, 650);
    } else {
      setTimeout(() => {
        lights.forEach(l => l.classList.remove('on'));
        setTimeout(() => {
          ov.style.display = 'none';
          S.running = true;
          S.lapStart = performance.now()/1000;
          flash("IT'S LIGHTS OUT — GO!");
        }, 320);
      }, 500 + Math.random()*900);
    }
  };
  setTimeout(lightUp, 450);
});

/* ============================================================
   MAIN LOOP
============================================================ */
resetToTrack();
setCam('chase');
let lastT = performance.now();
let mmTick = 0;
function loop(now){
  requestAnimationFrame(loop);
  let dt = (now - lastT)/1000;
  lastT = now;
  if (dt > 0.05) dt = 0.05;
  physics(dt);
  for (const fn of envSpin) fn(dt);
  updateCamera(dt);
  updateHUD(dt);
  updateAudio();
  mmTick += dt;
  if (mmTick > 0.08){ drawMinimap(); mmTick = 0; }
  renderer.render(scene, camera);
}
requestAnimationFrame(loop);
})();
// GAME-JS-END
</script>
</body>
</html>

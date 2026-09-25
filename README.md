# Index.html<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
<meta name="theme-color" content="#0a0a15">
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="mobile-web-app-capable" content="yes">
<title>Yi Jin Jing — 12 Movimentos</title>
<style>
  *{margin:0;padding:0;box-sizing:border-box;-webkit-tap-highlight-color:transparent;user-select:none}
  html,body{width:100%;height:100%;overflow:hidden;background:#0a0a15;
    font-family:-apple-system,BlinkMacSystemFont,"Segoe UI",sans-serif}
  #app{position:relative;width:100vw;height:100vh}
  canvas{display:block;width:100%;height:100%}
  #topbar{position:absolute;top:0;left:0;right:0;padding:14px 16px;
    padding-top:max(14px,env(safe-area-inset-top));
    background:linear-gradient(180deg,rgba(10,10,21,.95),transparent);
    pointer-events:none;z-index:10}
  #title{color:#d4af37;font-size:11px;letter-spacing:3px;text-transform:uppercase;opacity:.8}
  #current-move{color:#fff;font-size:17px;font-weight:600;margin-top:2px;
    text-shadow:0 2px 12px rgba(0,0,0,.9);transition:opacity .15s}
  #controls{position:absolute;bottom:0;left:0;right:0;padding:12px;
    padding-bottom:max(14px,env(safe-area-inset-bottom));
    background:linear-gradient(0deg,rgba(10,10,21,.98) 60%,transparent);
    z-index:10}
  #move-grid{display:grid;grid-template-columns:repeat(4,1fr);gap:6px;margin-bottom:10px}
  .move-btn{padding:8px 3px;background:rgba(25,25,45,.85);
    border:1px solid rgba(212,175,55,.25);border-radius:9px;color:#9a8f78;
    font-size:8.5px;line-height:1.2;text-align:center;cursor:pointer;
    transition:all .2s;backdrop-filter:blur(12px);-webkit-backdrop-filter:blur(12px);
    overflow:hidden;white-space:nowrap;text-overflow:ellipsis}
  .move-btn .num{display:block;font-size:13px;font-weight:700;color:#d4af37;margin-bottom:1px}
  .move-btn.active{background:rgba(212,175,55,.22);border-color:#d4af37;color:#fff;
    box-shadow:0 0 18px rgba(212,175,55,.35),inset 0 0 12px rgba(212,175,55,.1)}
  #speed-row{display:flex;gap:6px}
  .speed-btn{flex:1;padding:11px;background:rgba(25,25,45,.85);
    border:1px solid rgba(212,175,55,.25);border-radius:9px;color:#9a8f78;
    font-size:13px;font-weight:600;cursor:pointer;transition:all .2s;
    backdrop-filter:blur(12px);-webkit-backdrop-filter:blur(12px)}
  .speed-btn.active{background:#d4af37;color:#0a0a15;border-color:#d4af37;
    box-shadow:0 0 16px rgba(212,175,55,.4)}
  #loading{position:absolute;inset:0;display:flex;align-items:center;justify-content:center;
    background:#0a0a15;color:#d4af37;font-size:12px;letter-spacing:4px;z-index:100;
    transition:opacity .5s}
</style>
</head>
<body>
<div id="app">
  <div id="loading">CARREGANDO…</div>
  <div id="topbar">
    <div id="title">易筋經 · YI JIN JING</div>
    <div id="current-move">Wei Tuo Oferece o Bastão I</div>
  </div>
  <div id="controls">
    <div id="move-grid"></div>
    <div id="speed-row">
      <button class="speed-btn" data-speed="0.5">0.5×</button>
      <button class="speed-btn active" data-speed="1">1×</button>
      <button class="speed-btn" data-speed="1.5">1.5×</button>
      <button class="speed-btn" data-speed="2">2×</button>
    </div>
  </div>
</div>

<script type="importmap">
{"imports":{"three":"https://unpkg.com/three@0.160.0/build/three.module.js"}}
</script>

<script type="module">
import * as THREE from 'three';

/* =========================================================
   CENA
   ========================================================= */
const scene = new THREE.Scene();
scene.background = new THREE.Color(0x141428);
scene.fog = new THREE.Fog(0x141428, 12, 42);

const camera = new THREE.PerspectiveCamera(45, innerWidth/innerHeight, 0.1, 200);
camera.position.set(0, 1.6, 4.8);
camera.lookAt(0, 1.1, 0);

const renderer = new THREE.WebGLRenderer({antialias:true, powerPreference:'high-performance'});
renderer.setSize(innerWidth, innerHeight);
renderer.setPixelRatio(Math.min(devicePixelRatio, 2));
renderer.shadowMap.enabled = true;
renderer.shadowMap.type = THREE.PCFSoftShadowMap;
renderer.toneMapping = THREE.ACESFilmicToneMapping;
renderer.toneMappingExposure = 1.15;
document.getElementById('app').insertBefore(renderer.domElement, document.getElementById('controls'));

/* =========================================================
   LUZES
   ========================================================= */
scene.add(new THREE.HemisphereLight(0x9ab4ff, 0x2a1810, 0.5));

const sun = new THREE.DirectionalLight(0xffd9a0, 2.4);
sun.position.set(5, 9, 4);
sun.castShadow = true;
sun.shadow.mapSize.set(2048, 2048);
sun.shadow.camera.left = -6; sun.shadow.camera.right = 6;
sun.shadow.camera.top = 6;  sun.shadow.camera.bottom = -6;
sun.shadow.camera.near = 1; sun.shadow.camera.far = 25;
sun.shadow.bias = -0.0006;
scene.add(sun);

const rim = new THREE.DirectionalLight(0xff5a5a, 0.7);
rim.position.set(-5, 3, -6);
scene.add(rim);

const fill = new THREE.PointLight(0xffa050, 1.2, 12, 2);
fill.position.set(0, 2.5, -2);
scene.add(fill);

/* =========================================================
   AMBIENTE
   ========================================================= */
// Chão
const ground = new THREE.Mesh(
  new THREE.CircleGeometry(35, 64),
  new THREE.MeshStandardMaterial({color:0x241c14, roughness:0.95, metalness:0.05})
);
ground.rotation.x = -Math.PI/2;
ground.receiveShadow = true;
scene.add(ground);

// Anel sagrado no chão
const ring = new THREE.Mesh(
  new THREE.RingGeometry(1.6, 1.72, 96),
  new THREE.MeshBasicMaterial({color:0xd4af37, side:THREE.DoubleSide,
    transparent:true, opacity:0.45, depthWrite:false})
);
ring.rotation.x = -Math.PI/2;
ring.position.y = 0.012;
scene.add(ring);

// Segundo anel externo
const ring2 = new THREE.Mesh(
  new THREE.RingGeometry(2.4, 2.45, 96),
  new THREE.MeshBasicMaterial({color:0xd4af37, side:THREE.DoubleSide,
    transparent:true, opacity:0.15, depthWrite:false})
);
ring2.rotation.x = -Math.PI/2;
ring2.position.y = 0.012;
scene.add(ring2);

// Montanhas distantes
(function mountains(){
  for (let i = 0; i < 16; i++) {
    const h = 3 + Math.random()*7;
    const r = 2.5 + Math.random()*3.5;
    const m = new THREE.Mesh(
      new THREE.ConeGeometry(r, h, 4),
      new THREE.MeshStandardMaterial({
        color:new THREE.Color().setHSL(0.62, 0.35, 0.06 + Math.random()*0.08),
        roughness:1, flatShading:true
      })
    );
    const a = (i/16)*Math.PI*2 + Math.random()*0.3;
    const d = 20 + Math.random()*7;
    m.position.set(Math.cos(a)*d, h/2 - 0.6, Math.sin(a)*d);
    m.rotation.y = Math.random()*Math.PI;
    scene.add(m);
  }
})();

// Torii (portão)
(function torii(){
  const g = new THREE.Group();
  const red = new THREE.MeshStandardMaterial({color:0xb91c1c, roughness:0.65});
  const dark = new THREE.MeshStandardMaterial({color:0x120606, roughness:0.85});

  for (const x of [-1.6, 1.6]) {
    const p = new THREE.Mesh(new THREE.CylinderGeometry(0.13, 0.15, 3.2, 12), red);
    p.position.set(x, 1.6, 0); p.castShadow = true; g.add(p);
  }
  const top = new THREE.Mesh(new THREE.BoxGeometry(4.3, 0.28, 0.4), red);
  top.position.y = 3.3; top.castShadow = true; g.add(top);
  const mid = new THREE.Mesh(new THREE.BoxGeometry(3.6, 0.16, 0.28), red);
  mid.position.y = 2.75; mid.castShadow = true; g.add(mid);
  const roof = new THREE.Mesh(new THREE.BoxGeometry(5.0, 0.12, 0.7), dark);
  roof.position.y = 3.52; g.add(roof);

  g.position.set(0, 0, -3.8);
  scene.add(g);
})();

// Lanternas de pedra
function lantern(x, z){
  const g = new THREE.Group();
  const stone = new THREE.MeshStandardMaterial({color:0x4a4a4a, roughness:0.95});
  const base = new THREE.Mesh(new THREE.CylinderGeometry(0.22,0.28,0.16,8), stone);
  base.position.y = 0.08; g.add(base);
  const pillar = new THREE.Mesh(new THREE.CylinderGeometry(0.09,0.09,0.65,8), stone);
  pillar.position.y = 0.48; g.add(pillar);
  const lightBox = new THREE.Mesh(new THREE.BoxGeometry(0.26,0.22,0.26),
    new THREE.MeshStandardMaterial({color:0xffcc70, emissive:0xff8800, emissiveIntensity:1.8}));
  lightBox.position.y = 0.91; g.add(lightBox);
  const cap = new THREE.Mesh(new THREE.ConeGeometry(0.24,0.16,4), stone);
  cap.position.y = 1.09; g.add(cap);
  const pl = new THREE.PointLight(0xffa540, 3, 5, 2);
  pl.position.y = 0.91; g.add(pl);
  g.position.set(x, 0, z);
  g.traverse(o => {if(o.isMesh) o.castShadow = true});
  return g;
}
scene.add(lantern(-2.6, -1.8));
scene.add(lantern( 2.6, -1.8));

// Pétalas de cerejeira
const petals = (function(){
  const N = 220;
  const pos = new Float32Array(N*3);
  const vel = [];
  for (let i = 0; i < N; i++) {
    pos[i*3]   = (Math.random()-0.5)*30;
    pos[i*3+1] = Math.random()*14;
    pos[i*3+2] = (Math.random()-0.5)*30;
    vel.push({
      x:(Math.random()-0.5)*0.018,
      y:-0.012 - Math.random()*0.018,
      z:(Math.random()-0.5)*0.018,
      ph: Math.random()*Math.PI*2
    });
  }
  const geo = new THREE.BufferGeometry();
  geo.setAttribute('position', new THREE.BufferAttribute(pos,3));
  const mat = new THREE.PointsMaterial({
    color:0xffb7c5, size:0.075, transparent:true, opacity:0.85,
    sizeAttenuation:true, depthWrite:false
  });
  const pts = new THREE.Points(geo, mat);
  pts.userData.vel = vel;
  scene.add(pts);
  return pts;
})();

/* =========================================================
   SAMURAI — rig de ossos
   ========================================================= */
function limb(length, topR, botR, material){
  const pivot = new THREE.Group();
  const geo = new THREE.CylinderGeometry(topR, botR, length, 10);
  geo.translate(0, -length/2, 0);
  const mesh = new THREE.Mesh(geo, material);
  mesh.castShadow = true;
  pivot.add(mesh);
  return pivot;
}

function buildSamurai(){
  const root = new THREE.Group();

  const skin   = new THREE.MeshStandardMaterial({color:0xc98b54, roughness:0.75});
  const gi     = new THREE.MeshStandardMaterial({color:0x14142a, roughness:0.8});
  const armor  = new THREE.MeshStandardMaterial({color:0x8b1a1a, metalness:0.35, roughness:0.55});
  const gold   = new THREE.MeshStandardMaterial({color:0xd4af37, metalness:0.9, roughness:0.28});
  const hair   = new THREE.MeshStandardMaterial({color:0x080808, roughness:0.95});
  const band   = new THREE.MeshStandardMaterial({color:0xf2f2f2, roughness:0.9});

  // HIPS
  const hips = new THREE.Group();
  root.add(hips);
  const pelvis = new THREE.Mesh(new THREE.BoxGeometry(0.42,0.24,0.28), gi);
  pelvis.castShadow = true; hips.add(pelvis);
  const belt = new THREE.Mesh(new THREE.TorusGeometry(0.23,0.032,8,22), gold);
  belt.rotation.x = Math.PI/2; belt.position.y = 0.055; hips.add(belt);

  // SPINE
  const spine = new THREE.Group();
  spine.position.y = 0.12;
  hips.add(spine);

  const chest = new THREE.Mesh(new THREE.BoxGeometry(0.47,0.52,0.27), armor);
  chest.position.y = 0.31; chest.castShadow = true; spine.add(chest);

  for (let i = 0; i < 3; i++) {
    const plate = new THREE.Mesh(new THREE.BoxGeometry(0.38,0.055,0.02), gold);
    plate.position.set(0, 0.42 - i*0.13, 0.145);
    spine.add(plate);
  }

  // NECK + HEAD
  const neck = new THREE.Group();
  neck.position.y = 0.6;
  spine.add(neck);
  const neckMesh = new THREE.Mesh(new THREE.CylinderGeometry(0.06,0.07,0.1,10), skin);
  neckMesh.position.y = 0.05; neck.add(neckMesh);

  const head = new THREE.Group();
  head.position.y = 0.19;
  neck.add(head);
  const headMesh = new THREE.Mesh(new THREE.SphereGeometry(0.145,22,22), skin);
  headMesh.scale.set(1,1.14,1); headMesh.castShadow = true; head.add(headMesh);

  const hairTop = new THREE.Mesh(
    new THREE.SphereGeometry(0.152,18,18,0,Math.PI*2,0,Math.PI*0.62), hair);
  hairTop.position.y = 0.012; head.add(hairTop);
  const bun = new THREE.Mesh(new THREE.SphereGeometry(0.07,12,12), hair);
  bun.position.set(0,0.14,-0.09); head.add(bun);

  const hachimaki = new THREE.Mesh(new THREE.TorusGeometry(0.147,0.02,8,24), band);
  hachimaki.rotation.x = Math.PI/2; hachimaki.position.y = 0.055; head.add(hachimaki);
  const tail = new THREE.Mesh(new THREE.BoxGeometry(0.035,0.2,0.012), band);
  tail.position.set(0.05,0.02,-0.14); tail.rotation.z = 0.2; head.add(tail);

  // SHOULDERS / ARMS
  const SH_Y = 0.47, SH_X = 0.26;

  const sodeL = new THREE.Mesh(new THREE.BoxGeometry(0.22,0.2,0.26), armor);
  sodeL.position.set(SH_X + 0.05, SH_Y + 0.02, 0); sodeL.castShadow = true; spine.add(sodeL);
  const sodeR = new THREE.Mesh(new THREE.BoxGeometry(0.22,0.2,0.26), armor);
  sodeR.position.set(-SH_X - 0.05, SH_Y + 0.02, 0); sodeR.castShadow = true; spine.add(sodeR);

  // Braço esquerdo (lado +X = esquerda do personagem olhando para +Z)
  const shoulderL = new THREE.Group();
  shoulderL.position.set(SH_X, SH_Y, 0); spine.add(shoulderL);
  const upperArmL = limb(0.32, 0.078, 0.066, gi); shoulderL.add(upperArmL);
  const foreArmL  = limb(0.30, 0.062, 0.052, skin); foreArmL.position.y = -0.32; upperArmL.add(foreArmL);
  const handL = new THREE.Group(); handL.position.y = -0.30; foreArmL.add(handL);
  const handMeshL = new THREE.Mesh(new THREE.BoxGeometry(0.09,0.14,0.065), skin);
  handMeshL.position.y = -0.07; handMeshL.castShadow = true; handL.add(handMeshL);

  // Braço direito
  const shoulderR = new THREE.Group();
  shoulderR.position.set(-SH_X, SH_Y, 0); spine.add(shoulderR);
  const upperArmR = limb(0.32, 0.078, 0.066, gi); shoulderR.add(upperArmR);
  const foreArmR  = limb(0.30, 0.062, 0.052, skin); foreArmR.position.y = -0.32; upperArmR.add(foreArmR);
  const handR = new THREE.Group(); handR.position.y = -0.30; foreArmR.add(handR);
  const handMeshR = new THREE.Mesh(new THREE.BoxGeometry(0.09,0.14,0.065), skin);
  handMeshR.position.y = -0.07; handMeshR.castShadow = true; handR.add(handMeshR);

  // PERNAS
  const LX = 0.13;
  const thighL = limb(0.45, 0.10, 0.085, gi);
  thighL.position.set(LX, -0.05, 0); hips.add(thighL);
  const shinL = limb(0.45, 0.082, 0.068, gi);
  shinL.position.y = -0.45; thighL.add(shinL);
  const footL = new THREE.Mesh(new THREE.BoxGeometry(0.13,0.075,0.25), skin);
  footL.position.set(0,-0.45,0.06); footL.castShadow = true; shinL.add(footL);

  const thighR = limb(0.45, 0.10, 0.085, gi);
  thighR.position.set(-LX, -0.05, 0); hips.add(thighR);
  const shinR = limb(0.45, 0.082, 0.068, gi);
  shinR.position.y = -0.45; thighR.add(shinR);
  const footR = new THREE.Mesh(new THREE.BoxGeometry(0.13,0.075,0.25), skin);
  footR.position.set(0,-0.45,0.06); footR.castShadow = true; shinR.add(footR);

  return {root, hips, spine, chest, neck, head,
    shoulderL, upperArmL, foreArmL, handL,
    shoulderR, upperArmR, foreArmR, handR,
    thighL, shinL, footL, thighR, shinR, footR};
}

const rig = buildSamurai();
scene.add(rig.root);

const HIP_BASE = 0.98;
const D = Math.PI/180;
const ALL = [
  rig.root, rig.hips, rig.spine, rig.chest, rig.neck, rig.head,
  rig.shoulderL, rig.upperArmL, rig.foreArmL, rig.handL,
  rig.shoulderR, rig.upperArmR, rig.foreArmR, rig.handR,
  rig.thighL, rig.shinL, rig.footL, rig.thighR, rig.shinR, rig.footR
];

function resetPose(){
  for (const b of ALL) b.rotation.set(0,0,0);
  rig.root.position.set(0,0,0);
  rig.hips.position.set(0,HIP_BASE,0);
}

function rot(b, x, y, z){ b.rotation.set(x*D, y*D, z*D); }

/* =========================================================
   12 MOVIMENTOS
   ========================================================= */
const movements = [
  {
    id:1, namePt:'Wei Tuo Oferece o Bastão I',
    fn:(t)=>{
      const lift = (1 - Math.cos(t))/2;
      const br = Math.sin(t)*3;
      rot(rig.upperArmL, -70 - lift*8, 0, 18);
      rot(rig.foreArmL,  -48 + br,  0, 12);
      rot(rig.upperArmR, -70 - lift*8, 0, -18);
      rot(rig.foreArmR,  -48 + br,  0, -12);
      rot(rig.spine, -3, 0, 0);
      rot(rig.head, 0, 0, 0);
      const k = 9 + lift*3;
      rot(rig.thighL, -k/2, 0, 4); rot(rig.shinL, k, 0, 0);
      rot(rig.thighR, -k/2, 0, -4); rot(rig.shinR, k, 0, 0);
      rig.hips.position.y = HIP_BASE - lift*0.03;
    }
  },
  {
    id:2, namePt:'Wei Tuo Oferece o Bastão II',
    fn:(t)=>{
      const open = (1 - Math.cos(t))/2;
      rot(rig.upperArmL, 0, 0, 88 + open*8);
      rot(rig.foreArmL, 0, 0, 6);
      rot(rig.upperArmR, 0, 0, -(88 + open*8));
      rot(rig.foreArmR, 0, 0, -6);
      rot(rig.spine, -2, 0, 0);
      const k = 6;
      rot(rig.thighL, -k/2, 0, 4); rot(rig.shinL, k, 0, 0);
      rot(rig.thighR, -k/2, 0, -4); rot(rig.shinR, k, 0, 0);
    }
  },
  {
    id:3, namePt:'Palmas Sustentando o Céu',
    fn:(t)=>{
      const push = (1 - Math.cos(t))/2;
      rot(rig.upperArmL, 0, 0, 168 + push*6);
      rot(rig.foreArmL, -10, 0, 6);
      rot(rig.upperArmR, 0, 0, -(168 + push*6));
      rot(rig.foreArmR, -10, 0, -6);
      rot(rig.spine, 3, 0, 0);
      rot(rig.head, -8, 0, 0);
      const k = 4;
      rot(rig.thighL, -k/2, 0, 3); rot(rig.shinL, k, 0, 0);
      rot(rig.thighR, -k/2, 0, -3); rot(rig.shinR, k, 0, 0);
      rig.hips.position.y = HIP_BASE + push*0.02;
    }
  },
  {
    id:4, namePt:'Colher Estrelas, Trocar Constelação',
    fn:(t)=>{
      const s = Math.sin(t);
      rot(rig.upperArmL, 0, 0, 158 + s*8);
      rot(rig.foreArmL, -28, 0, 10);
      rot(rig.upperArmR, 38, 0, -28);
      rot(rig.foreArmR, -100, 0, 0);
      rot(rig.spine, 0, 14*s, 0);
      rot(rig.neck, 0, -4*s, 0);
      rot(rig.head, -5, 0, 0);
      const k = 8;
      rot(rig.thighL, -k/2, 0, 4); rot(rig.shinL, k, 0, 0);
      rot(rig.thighR, -k/2, 0, -4); rot(rig.shinR, k, 0, 0);
    }
  },
  {
    id:5, namePt:'Puxando a Cauda de Nove Bois',
    fn:(t)=>{
      const p = Math.sin(t);
      const s = (1 - Math.cos(t))/2;
      rot(rig.upperArmL, -78, 0, -12);
      rot(rig.foreArmL, -28 - p*18, 0, 0);
      rot(rig.upperArmR, 28, 0, 22);
      rot(rig.foreArmR, -108 - p*18, 0, 0);
      rot(rig.spine, 0, 8, 0);
      rot(rig.head, 0, -8, 0);
      rot(rig.thighL, -40, 0, 14); rot(rig.shinL, 75, 0, 0);
      rot(rig.thighR, 18, 0, -10); rot(rig.shinR, 25, 0, 0);
      rig.hips.position.y = HIP_BASE - 0.12*s;
      rig.hips.position.z = 0.05*s;
    }
  },
  {
    id:6, namePt:'Garras Sendo Lançadas',
    fn:(t)=>{
      const e = (1 - Math.cos(t))/2;
      rot(rig.upperArmL, -80 - e*16, 0, -10);
      rot(rig.foreArmL, -10 - e*10, 0, 0);
      rot(rig.upperArmR, -80 - e*16, 0, 10);
      rot(rig.foreArmR, -10 - e*10, 0, 0);
      rot(rig.spine, -5, 0, 0);
      rot(rig.head, -5, 0, 0);
      const k = 12 + e*10;
      rot(rig.thighL, -k/2, 0, 5); rot(rig.shinL, k, 0, 0);
      rot(rig.thighR, -k/2, 0, -5); rot(rig.shinR, k, 0, 0);
      rig.hips.position.y = HIP_BASE - e*0.06;
    }
  },
  {
    id:7, namePt:'Nove Fantasmas Puxando a Espada',
    fn:(t)=>{
      const tw = Math.sin(t);
      rot(rig.upperArmL, 28, 0, 40);
      rot(rig.foreArmL, -138, 0, 0);
      rot(rig.upperArmR, -28, 0, -100);
      rot(rig.foreArmR, -128, 0, 0);
      rot(rig.spine, 0, -20 - tw*8, 0);
      rot(rig.neck, 0, 10, 0);
      rot(rig.head, 0, 5, 0);
      const k = 10;
      rot(rig.thighL, -k/2, 0, 4); rot(rig.shinL, k, 0, 0);
      rot(rig.thighR, -k/2, 0, -4); rot(rig.shinR, k, 0, 0);
    }
  },
  {
    id:8, namePt:'Três Pratos Caindo ao Chão',
    fn:(t)=>{
      const sq = (1 - Math.cos(t))/2;
      const knee = 18 + sq*70;
      rot(rig.upperArmL, -30, 0, 70 + sq*15);
      rot(rig.foreArmL, -22, 0, 6);
      rot(rig.upperArmR, -30, 0, -(70 + sq*15));
      rot(rig.foreArmR, -22, 0, -6);
      rot(rig.spine, 8 + sq*10, 0, 0);
      rot(rig.head, -8, 0, 0);
      rot(rig.thighL, -knee/2, 0, 18 + sq*10);
      rot(rig.shinL, knee, 0, 0);
      rot(rig.thighR, -knee/2, 0, -(18 + sq*10));
      rot(rig.shinR, knee, 0, 0);
      rig.hips.position.y = HIP_BASE - sq*0.33;
    }
  },
  {
    id:9, namePt:'Dragão Azul Estendendo as Garras',
    fn:(t)=>{
      const r = Math.sin(t);
      const e = (1 - Math.cos(t))/2;
      rot(rig.upperArmL, -28, 0, 58 + e*22);
      rot(rig.foreArmL, -10, 0, 10);
      rot(rig.upperArmR, -58, 0
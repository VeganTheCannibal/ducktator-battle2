# ducktator-battle2
fight dickei 
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Ducktator Battle — Web</title>
<style>
  :root { --bg:#0c0f1a; --panel:#11162b; --text:#e8eef7; --muted:#a7b0c0; --hero:#35c0ff; --vill:#f87171; --ok:#22c55e; --warn:#f59e0b; }
  * { box-sizing: border-box; }
  body { margin:0; font-family: ui-sans-serif, system-ui, -apple-system, Segoe UI, Roboto, Helvetica, Arial; background: var(--bg); color: var(--text); }
  .wrap { max-width: 980px; margin: 24px auto; padding: 12px; }

  /* Loading overlay */
  .loading {
    position: fixed; inset: 0; display:flex; align-items:center; justify-content:center;
    background: radial-gradient(1200px 600px at 50% 40%, #192045 0%, #0c0f1a 60%);
    z-index: 9999; transition: opacity .4s ease;
  }
  .loader {
    width: min(520px, 90vw); padding: 18px; border-radius: 14px;
    background: #0e1430; border: 1px solid rgba(255,255,255,.08);
    box-shadow: 0 12px 40px rgba(0,0,0,.45);
  }
  .loader h2 { margin: 0 0 10px; font-size: 18px; letter-spacing:.3px; }
  .bar { height: 14px; border-radius: 999px; background: #0a1026; border: 1px solid rgba(255,255,255,.08); overflow: hidden; }
  .bar > span { display:block; height:100%; width:0%; background: linear-gradient(90deg,#35c0ff,#22c55e); transition: width .25s ease; }
  .pct { margin-top: 8px; font-size: 12px; color: var(--muted); }

  /* Card layout */
  .card { background: var(--panel); border:1px solid rgba(255,255,255,.08); border-radius: 16px; overflow: hidden; box-shadow: 0 14px 40px rgba(0,0,0,.35); }
  header { padding: 12px 16px; border-bottom: 1px solid rgba(255,255,255,.08); display:flex; justify-content:space-between; align-items:center; gap:12px; }
  header h1 { margin:0; font-size: clamp(18px, 2.5vw, 22px); }
  .content { display:grid; grid-template-columns: 1fr 320px; gap: 12px; padding: 12px; }
  @media (max-width: 900px){ .content { grid-template-columns: 1fr;} }

  /* Arena */
  .arena {
    position: relative; height: 380px; border-radius: 12px; overflow: hidden; border:1px solid rgba(255,255,255,.08);
    background: linear-gradient(180deg,#1c244a 0%, #0e1430 55%, #0a0e1f 100%);
  }
  /* Ducktator sunset gradient split */
  .arena::before {
    content:''; position:absolute; inset:0;
    background:
      radial-gradient(600px 260px at 78% 28%, rgba(255,92,0,.28), transparent 60%),
      radial-gradient(800px 320px at 20% 15%, rgba(53,192,255,.22), transparent 55%);
    pointer-events:none;
  }
  /* Ground */
  .ground {
    position:absolute; left:0; right:0; bottom:0; height: 38%;
    background:
      linear-gradient(180deg, rgba(0,0,0,0) 0%, rgba(0,0,0,.15) 20%, rgba(0,0,0,.35) 100%),
      linear-gradient(90deg, #1a2a32 0%, #0f171b 50%, #2b1212 100%);
  }

  /* Platforms (hero bright, boss dark) */
  .plat { position:absolute; bottom: 56px; width: 44%; height: 18%; border-radius: 50%;
    box-shadow: inset 0 6px 20px rgba(0,0,0,.45), 0 20px 40px rgba(0,0,0,.35); }
  .hero-plat { left: 6%; background: radial-gradient(65% 65% at 45% 45%, #1e3a5f, #0f2744); }
  .boss-plat { right: 6%; background: radial-gradient(65% 65% at 55% 45%, #3b0f0f, #200808); }

  /* Sprites */
  .sprite { position:absolute; bottom: 88px; width: 220px; height: 220px; display:flex; align-items:flex-end; justify-content:center; pointer-events:none; }
  .hero { left: 6%; }
  .boss { right: 6%; }

  /* Placeholder office worker (SVG data URI) */
  .hero img, .boss img { max-width: 210px; max-height: 210px; image-rendering: -webkit-optimize-contrast; filter: drop-shadow(0 8px 18px rgba(0,0,0,.45)); }

  /* HUD */
  .hud { position:absolute; left: 0; right:0; top:0; display:flex; justify-content:space-between; padding: 10px; gap: 10px; }
  .hud-box { min-width: 260px; background: rgba(8,12,28,.75); border:1px solid rgba(255,255,255,.1); border-radius: 10px; padding: 10px; }
  .name-row { display:flex; justify-content:space-between; align-items:center; }
  .hpbar { height: 12px; border-radius: 8px; background: #0b1228; border:1px solid rgba(255,255,255,.1); overflow: hidden; margin-top: 6px; }
  .hp { height:100%; width:100%; background: linear-gradient(90deg, #35c0ff, #22c55e); transition: width .55s ease; }
  .hp.danger { background: linear-gradient(90deg, #f59e0b, #ef4444); }

  /* Controls & Log */
  .panel { background:#0e1430; border:1px solid rgba(255,255,255,.08); border-radius:12px; padding: 10px; }
  .moves { display:grid; grid-template-columns: repeat(2, minmax(0,1fr)); gap: 8px; }
  .btn { border:1px solid rgba(255,255,255,.14); background:#12183a; color:var(--text); padding: 10px 8px; border-radius: 10px; font-weight:600; cursor:pointer; }
  .btn:disabled { opacity:.45; cursor:not-allowed; }
  .row { display:flex; align-items:center; gap: 10px; flex-wrap:wrap; }
  .log { height: 210px; overflow:auto; background:#0a0f24; border:1px solid rgba(255,255,255,.06); border-radius:10px; padding: 8px; font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, monospace; font-size: 12.5px; line-height: 1.35; }
  .log p { margin: 0 0 6px; color: #dbe6ff; }
  .hint { font-size: 12px; color: var(--muted); }

  /* Status chips */
  .chip { position:absolute; top: 58px; padding: 3px 8px; border-radius: 999px; font-size: 12px; border:1px solid rgba(255,255,255,.18); background: rgba(13,19,45,.8); }
  .chip.hero { left: 18px; }
  .chip.boss { right: 18px; }
  .chip.stun { background: rgba(255,188,0,.2); }
  .chip.silence { background: rgba(255,0,0,.18); }

  /* Animations */
  @keyframes lungeR { 0%{transform: translateX(0)} 50%{transform: translateX(24px)} 100%{transform: translateX(0)} }
  @keyframes lungeL { 0%{transform: translateX(0)} 50%{transform: translateX(-24px)} 100%{transform: translateX(0)} }
  @keyframes hitShake { 0%,100%{transform: translateX(0)} 25%{transform: translateX(-6px)} 75%{transform: translateX(6px)} }
  .lungeR { animation: lungeR .5s ease; }
  .lungeL { animation: lungeL .5s ease; }
  .hit   { animation: hitShake .5s ease; }

  /* File input */
  .file { color:#dbe6ff; }
</style>
</head>
<body>
<div class="loading" id="loading">
  <div class="loader">
    <h2>Loading Ducktator Battle…</h2>
    <div class="bar"><span id="bar"></span></div>
    <div class="pct"><span id="pct">0%</span> Preparing arena, sprites & sounds…</div>
  </div>
</div>

<div class="wrap">
  <div class="card">
    <header>
      <h1>Battle: Office Worker vs. Dickie</h1>
      <div class="row hint">
        <span>Hide (+5, 3 uses)</span> ·
        <span>DMH Complaint (15 dmg, 33% stun)</span> ·
        <span>Stapler (−5 self)</span> ·
        <span>Stab (50 dmg)</span>
      </div>
    </header>

    <div class="content">
      <section class="panel">
        <div class="arena" id="arena">
          <!-- HUD -->
          <div class="hud">
            <div class="hud-box" id="heroHud">
              <div class="name-row"><strong>You</strong><span id="pHpTxt">100 / 100</span></div>
              <div class="hpbar"><div class="hp" id="pHp"></div></div>
            </div>
            <div class="hud-box" id="bossHud">
              <div class="name-row"><strong>Dickie</strong><span id="bHpTxt">100 / 100</span></div>
              <div class="hpbar"><div class="hp" id="bHp"></div></div>
            </div>
          </div>

          <!-- Platforms & ground -->
          <div class="plat hero-plat"></div>
          <div class="plat boss-plat"></div>
          <div class="ground"></div>

          <!-- Status chips -->
          <div class="chip hero" id="chipHero" style="display:none;"></div>
          <div class="chip boss" id="chipBoss" style="display:none;"></div>

          <!-- Sprites -->
          <div class="sprite hero"><img id="heroImg" alt="office worker" /></div>
          <div class="sprite boss"><img id="bossImg" alt="Dickie" /></div>
        </div>
      </section>

      <aside class="panel">
        <div class="row" style="justify-content:space-between;align-items:center">
          <strong>Moves</strong>
          <button class="btn" id="reset">Reset</button>
        </div>
        <div class="moves" style="margin-top:8px">
          <button class="btn" id="mvHide">Hide in Mike’s Office</button>
          <button class="btn" id="mvComplaint">DMH Complaint</button>
          <button class="btn" id="mvStapler">Stapler (self-damage)</button>
          <button class="btn" id="mvStab">Stab</button>
        </div>

        <div class="row" style="margin-top:10px">
          <label class="hint">Use your Dickie image: <input id="file" class="file" type="file" accept="image/*"></label>
        </div>

        <div class="row" style="margin-top:10px; justify-content:space-between">
          <span class="hint" id="hidesLeft">Hides left: 3</span>
          <span class="hint">Advice doubles. Talk may silence. Phone hurts.</span>
        </div>

        <div class="log" id="log" style="margin-top:10px" aria-live="polite"></div>
      </aside>
    </div>
  </div>
</div>

<script>
(function(){
  // ===== Assets (hero SVG inline, sounds as simple bleeps) =====
  const heroSVG = `
  <svg xmlns="http://www.w3.org/2000/svg" width="220" height="220">
    <defs>
      <linearGradient id="g1" x1="0" x2="1"><stop offset="0" stop-color="#6aa9ff"/><stop offset="1" stop-color="#9ae6b4"/></linearGradient>
    </defs>
    <rect width="100%" height="100%" fill="none"/>
    <!-- cartoon office worker -->
    <g transform="translate(55,20)">
      <circle cx="55" cy="40" r="28" fill="#ffd7b1" stroke="#333" />
      <rect x="15" y="68" width="80" height="90" rx="14" fill="url(#g1)" stroke="#2e3a5a"/>
      <rect x="15" y="68" width="80" height="26" fill="rgba(0,0,0,.12)"/>
      <rect x="43" y="94" width="24" height="46" fill="#fff" stroke="#2e3a5a"/>
      <polygon points="55,94 43,140 67,140" fill="#3b82f6"/>
      <rect x="5" y="100" width="20" height="50" rx="6" fill="#8da2fb" stroke="#2e3a5a"/>
      <rect x="85" y="100" width="20" height="50" rx="6" fill="#8da2fb" stroke="#2e3a5a"/>
    </g>
  </svg>`;
  const heroURI = 'data:image/svg+xml;charset=utf-8,' + encodeURIComponent(heroSVG);

  // Tiny cartoony bleeps (WebAudio synthesized once; loading bar still shows progress)
  let audioCtx, quack, boing, bonk, stamp, smash, win, lose;
  function initAudio(){
    if(audioCtx) return;
    audioCtx = new (window.AudioContext || window.webkitAudioContext)();
    const makeBeep = (type='sine', freq=440, dur=0.15, gain=0.08) => () => {
      const o = audioCtx.createOscillator();
      const g = audioCtx.createGain();
      o.type = type; o.frequency.value = freq;
      g.gain.value = gain;
      o.connect(g); g.connect(audioCtx.destination);
      o.start();
      setTimeout(()=>{ o.stop(); }, dur*1000);
    };
    // Assign sounds
    quack = makeBeep('square', 320, .18, .12);      // advice
    boing = makeBeep('triangle', 540, .22, .12);    // your attack
    bonk  = makeBeep('square', 180, .10, .10);      // hit
    stamp = makeBeep('sawtooth', 260, .18, .12);    // complaint
    smash = makeBeep('sawtooth', 140, .25, .14);    // phone
    win   = makeBeep('triangle', 660, .35, .16);
    lose  = makeBeep('square', 120, .35, .16);
  }

  // ===== State =====
  const S = {
    pHP: 100, bHP: 100, hides: 3,
    pSkip: false, bSkip: false,
    ua: 5, winner: null,
    busy: true  // locked during animations
  };

  // ===== DOM =====
  const el = {
    loading: document.getElementById('loading'),
    bar: document.getElementById('bar'),
    pct: document.getElementById('pct'),
    pHp: document.getElementById('pHp'), bHp: document.getElementById('bHp'),
    pHpTxt: document.getElementById('pHpTxt'), bHpTxt: document.getElementById('bHpTxt'),
    hidesLeft: document.getElementById('hidesLeft'),
    heroImg: document.getElementById('heroImg'), bossImg: document.getElementById('bossImg'),
    chipHero: document.getElementById('chipHero'), chipBoss: document.getElementById('chipBoss'),
    mvHide: document.getElementById('mvHide'), mvComplaint: document.getElementById('mvComplaint'),
    mvStapler: document.getElementById('mvStapler'), mvStab: document.getElementById('mvStab'),
    reset: document.getElementById('reset'), log: document.getElementById('log'),
    file: document.getElementById('file'),
  };

  // ===== Loading bar logic =====
  let loadProgress = 0;
  function setProgress(p){
    loadProgress = Math.max(loadProgress, p);
    el.bar.style.width = Math.round(loadProgress*100) + '%';
    el.pct.textContent = Math.round(loadProgress*100) + '%';
  }

  // Simulate staged progress while assets decode
  setProgress(0.1);

  // Preload hero
  const heroImg = new Image();
  heroImg.onload = () => setProgress(0.35);
  heroImg.src = heroURI;

  // Boss defaults to emoji until user provides image
  const duckSVG = `<svg xmlns='http://www.w3.org/2000/svg' width='220' height='220'><rect width='100%' height='100%' fill='none'/><text x='50%' y='55%' dominant-baseline='middle' text-anchor='middle' font-size='88'>🦆</text></svg>`;
  const duckURI = 'data:image/svg+xml;charset=utf-8,' + encodeURIComponent(duckSVG);
  const bossImg = new Image();
  bossImg.onload = () => setProgress(0.5);
  bossImg.src = duckURI;

  // Small timer to “fill” the rest while WebAudio warms up
  const sim = setInterval(()=> {
    if(loadProgress < .85) setProgress(loadProgress + 0.05);
  }, 180);

  // Finish when both images assigned to DOM + audio primed
  function finishLoading(){
    clearInterval(sim);
    setProgress(1);
    el.loading.style.opacity = '0';
    setTimeout(()=> el.loading.style.display = 'none', 420);
    S.busy = false;
  }

  // Assign initial sprites
  el.heroImg.src = heroURI;
  el.bossImg.src = duckURI;

  // Initialize audio a moment later (user gesture will fully unlock)
  setTimeout(()=>{ initAudio(); setProgress(0.95); finishLoading(); }, 900);

  // ===== UI helpers =====
  const clamp = (n,min,max)=>Math.max(min,Math.min(max,n));
  function setHP(){
    el.pHp.style.width = clamp(S.pHP,0,100)+'%';
    el.bHp.style.width = clamp(S.bHP,0,100)+'%';
    el.pHp.classList.toggle('danger', S.pHP<=35);
    el.bHp.classList.toggle('danger', S.bHP<=35);
    el.pHpTxt.textContent = `${clamp(S.pHP,0,100)} / 100`;
    el.bHpTxt.textContent = `${clamp(S.bHP,0,100)} / 100`;
    el.hidesLeft.textContent = `Hides left: ${S.hides}`;
  }
  function log(t){ const p=document.createElement('p'); p.textContent=t; el.log.appendChild(p); el.log.scrollTop=el.log.scrollHeight; }

  function showChip(target, text, cls){
    const c = (target==='hero')? el.chipHero : el.chipBoss;
    c.textContent = text; c.classList.add(cls); c.style.display='block';
    setTimeout(()=>{ c.style.display='none'; c.classList.remove(cls); }, 900);
  }

  function disableMoves(d=true){
    [el.mvHide, el.mvComplaint, el.mvStapler, el.mvStab].forEach(b => b.disabled = d || !!S.winner);
    if(S.hides<=0) el.mvHide.disabled = true;
  }

  function lunge(attacker){
    const node = attacker==='hero' ? el.heroImg : el.bossImg;
    node.classList.remove(attacker==='hero'?'lungeR':'lungeL'); // reset
    void node.offsetWidth;
    node.classList.add(attacker==='hero'?'lungeR':'lungeL');
  }
  function hit(target){
    const node = target==='hero' ? el.heroImg : el.bossImg;
    node.classList.remove('hit'); void node.offsetWidth; node.classList.add('hit');
  }

  function endIfNeeded(){
    if(S.bHP<=0 && !S.winner){ S.winner='player'; log('Dickie is defeated! You win!'); if(win) win(); }
    if(S.pHP<=0 && !S.winner){ S.winner='dickie'; log('You have fallen. Dickie wins.'); if(lose) lose(); }
    disableMoves(false);
  }

  // ===== Mechanics =====
  function playerTurn(move){
    if(S.busy || S.winner) return;
    initAudio(); // ensure audio ready on first click
    disableMoves(true);

    // If silenced
    if(S.pSkip){ log('You are silenced and miss your turn.'); S.pSkip=false; showChip('hero','SILENCED','silence'); return setTimeout(bossTurn, 600); }

    // Resolve move
    if(move==='hide'){
      if(S.hides>0){ S.pHP = clamp(S.pHP+5,0,100); S.hides--; log('You hide in Mike’s office (+5 HP).'); }
      else { log('You try to hide, but Dickie already found you (no effect).'); }
    } else if(move==='complaint'){
      lunge('hero'); if(boing) boing();
      S.bHP -= 15; log('You file a DMH Complaint (−15 Dickie HP).'); if(stamp) stamp();
      if(Math.random()<0.33){ S.bSkip=true; showChip('boss','STUNNED','stun'); log('Dickie will miss the next turn!'); }
      setTimeout(()=> hit('boss'), 280);
    } else if(move==='stapler'){
      S.pHP -= 5; log('You staple yourself (−5 HP).'); if(bonk) bonk();
      setTimeout(()=> hit('hero'), 240);
    } else if(move==='stab'){
      lunge('hero'); if(boing) boing();
      S.bHP -= 50; log('You use STAB (−50 Dickie HP).');
      setTimeout(()=> hit('boss'), 280);
    }

    setHP();
    if(S.bHP<=0){ endIfNeeded(); return; }

    // Next: boss
    setTimeout(bossTurn, 650);
  }

  function bossTurn(){
    if(S.winner) return;
    // Stunned?
    if(S.bSkip){ S.bSkip=false; showChip('boss','STUNNED','stun'); log('Dickie misses the turn.'); disableMoves(false); return; }

    // Choose move: ~1/3 each
    const r = Math.random();
    let mv = (r<.333)?'advice':(r<.666)?'talk':'phone';

    if(mv==='advice'){
      lunge('boss'); if(quack) quack();
      const dmg = S.ua; S.pHP -= dmg; log(`Dickie uses Unsolicited Advice (−${dmg} HP).`);
      S.ua *= 2;
      setTimeout(()=> hit('hero'), 300);
    } else if(mv==='talk'){
      lunge('boss'); if(quack) quack();
      S.pHP -= 5; log('Dickie talks over you (−5 HP).');
      if(Math.random()<0.33){ S.pSkip=true; showChip('hero','SILENCED','silence'); log('You will miss your next turn!'); }
      setTimeout(()=> hit('hero'), 300);
    } else if(mv==='phone'){
      lunge('boss'); if(smash) smash();
      S.pHP -= 25; log('Dickie throws his phone (−25 HP).');
      setTimeout(()=> hit('hero'), 300);
    }

    setHP();
    endIfNeeded();
    disableMoves(false);
  }

  // ===== Wire up =====
  el.mvHide.onclick = ()=> playerTurn('hide');
  el.mvComplaint.onclick = ()=> playerTurn('complaint');
  el.mvStapler.onclick = ()=> playerTurn('stapler');
  el.mvStab.onclick = ()=> playerTurn('stab');

  el.reset.onclick = ()=>{
    S.pHP=100; S.bHP=100; S.hides=3; S.pSkip=false; S.bSkip=false; S.ua=5; S.winner=null;
    el.log.innerHTML=''; setHP(); disableMoves(false);
  };

  // Use actual Dickie image “as is”
  el.file.addEventListener('change', e=>{
    const f = e.target.files?.[0]; if(!f) return;
    const url = URL.createObjectURL(f);
    S.busy = true; disableMoves(true);
    setProgress(0.6);
    const img = new Image();
    img.onload = ()=>{
      el.bossImg.src = url;
      setTimeout(()=>{ setProgress(0.98); finishLoading(); setHP(); disableMoves(false); }, 250);
    };
    img.src = url;
  });

  // Initial UI state
  setHP(); disableMoves(false);
})();
</script>
</body>
</html>

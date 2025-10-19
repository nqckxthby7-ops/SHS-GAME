# Soft-safe-slice
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1,viewport-fit=cover" />
<title>Soft & Safe: Food Slice</title>
<style>
  :root{
    --bg: #fff6b3;           /* calm yellow background */
    --ui: #222;
    --safe: #2ecc71;         /* green */
    --caution: #f1c40f;      /* yellow */
    --avoid: #e74c3c;        /* red */
    --panel: #ffffffcc;
    --btn: #1f7aec;
    --btn-dark: #1452a6;
    --outline: #333;
    --shadow: 0 8px 24px rgba(0,0,0,0.15);
  }

  html, body {
    margin: 0;
    height: 100%;
    background: var(--bg);
    color: var(--ui);
    font-family: system-ui, -apple-system, Segoe UI, Roboto, Helvetica, Arial, "Apple Color Emoji", "Segoe UI Emoji", "Noto Color Emoji", sans-serif;
    overscroll-behavior: none;
    touch-action: none; /* help swipes inside canvas */
  }

  /* Layout */
  .app {
    display: grid;
    grid-template-rows: auto 1fr auto;
    min-height: 100dvh;
  }

  header {
    position: sticky;
    top: 0;
    z-index: 5;
    background: linear-gradient(180deg, rgba(255,255,255,0.9), rgba(255,255,255,0.6), transparent);
    backdrop-filter: blur(6px);
    padding: 10px 12px;
  }

  .topbar {
    display: flex;
    align-items: center;
    gap: 10px;
    flex-wrap: wrap;
    justify-content: space-between;
  }

  .group {
    display: inline-flex;
    align-items: center;
    gap: 12px;
    flex-wrap: wrap;
  }

  .pill {
    background: var(--panel);
    border: 2px solid #0002;
    border-radius: 999px;
    padding: 6px 12px;
    box-shadow: var(--shadow);
    display: inline-flex;
    align-items: center;
    gap: 8px;
    font-weight: 700;
  }

  .legend {
    display: inline-flex;
    align-items: center;
    gap: 10px;
    flex-wrap: wrap;
  }
  .legend .tag {
    display: inline-flex;
    align-items: center;
    gap: 6px;
    padding: 4px 8px;
    border-radius: 999px;
    border: 2px solid #0002;
    background: #fff;
    font-size: 13px;
    font-weight: 700;
  }
  .dot { width: 14px; height: 14px; border-radius: 50%; display: inline-block; border: 2px solid #0003; }
  .dot.safe { background: var(--safe); }
  .dot.caution { background: var(--caution); }
  .dot.avoid { background: var(--avoid); }

  .controls {
    display: inline-flex;
    align-items: center;
    gap: 8px;
    flex-wrap: wrap;
  }

  button, .btn {
    appearance: none;
    border: none;
    border-radius: 12px;
    padding: 10px 14px;
    font-size: 16px;
    font-weight: 800;
    color: #fff;
    background: var(--btn);
    box-shadow: var(--shadow);
    cursor: pointer;
    transition: transform .05s ease, background .2s ease;
  }
  button:active { transform: translateY(1px) scale(0.99); }
  .btn.secondary {
    background: #666;
  }
  .btn.ghost {
    background: #fff;
    color: #333;
    border: 2px solid #0002;
  }

  /* Canvas area */
  #stage-wrap {
    position: relative;
    width: 100%;
    height: calc(100dvh - 140px);
    min-height: 400px;
    max-height: 1000px;
    display: grid;
    grid-template-rows: 1fr;
    place-items: stretch;
    overflow: hidden;
  }

  canvas {
    width: 100%;
    height: 100%;
    background:
      radial-gradient(circle at 20% 10%, #fff9d4 0%, transparent 60%),
      radial-gradient(circle at 85% 15%, #fff4a3 0%, transparent 50%),
      linear-gradient(180deg, #fff9cc, #fff3a8 45%, #ffef8a 100%);
    border-top: 2px solid #0001;
    border-bottom: 2px solid #0001;
    touch-action: none;
  }

  footer {
    padding: 10px 12px 16px;
    display: flex;
    justify-content: center;
    gap: 10px;
    flex-wrap: wrap;
  }

  /* Modals */
  .modal-backdrop {
    position: fixed; inset: 0;
    background: rgba(0,0,0,0.35);
    display: none;
    align-items: center;
    justify-content: center;
    z-index: 50;
    padding: 16px;
  }
  .modal-backdrop.show { display: flex; }

  .modal {
    width: min(900px, 100%);
    max-height: 88dvh;
    overflow: auto;
    background: #fff;
    border-radius: 16px;
    box-shadow: 0 24px 60px rgba(0,0,0,0.35);
    padding: 18px;
  }
  .modal h1, .modal h2 { margin: 10px 0; }
  .modal .actions {
    display: flex; gap: 10px; flex-wrap: wrap; margin-top: 12px;
  }
  .grid {
    display: grid;
    grid-template-columns: repeat( auto-fit, minmax(230px, 1fr) );
    gap: 12px;
    align-items: start;
  }
  .card {
    background: #fffef7;
    border: 2px solid #0001;
    border-radius: 14px;
    padding: 12px;
  }

  .food-list {
    display: grid;
    grid-template-columns: repeat( auto-fit, minmax(220px, 1fr) );
    gap: 8px;
  }
  .food-item {
    display: grid;
    grid-template-columns: auto 1fr auto;
    align-items: center;
    gap: 8px;
    padding: 8px 10px;
    border-radius: 12px;
    border: 2px solid #0001;
    background: #fff;
  }
  .food-item .emoji {
    font-size: 28px;
    line-height: 1;
  }
  .badge {
    padding: 2px 8px;
    border-radius: 999px;
    font-weight: 800;
    font-size: 12px;
    border: 2px solid #0002;
  }
  .badge.safe { background: #e9f9f0; color: #126e3a; border-color: #c8eed9; }
  .badge.caution { background: #fff7d6; color: #856a00; border-color: #fde59a; }
  .badge.avoid { background: #ffe8e7; color: #7a1414; border-color: #f8c1bd; }

  .hr {
    height: 2px; background: #0001; border-radius: 2px; margin: 8px 0 12px;
  }

  .toast {
    position: absolute;
    left: 50%; top: 12px;
    transform: translateX(-50%);
    background: #000c;
    color: #fff;
    padding: 8px 12px;
    border-radius: 999px;
    font-weight: 700;
    font-size: 14px;
    opacity: 0; pointer-events: none;
    transition: opacity .25s ease, transform .25s ease;
    z-index: 4;
  }
  .toast.show { opacity: 1; transform: translateX(-50%) translateY(4px); }

  /* Floating text feedback */
  .float {
    position: absolute;
    font-weight: 900;
    pointer-events: none;
    text-shadow: 0 2px 4px rgba(0,0,0,0.25);
    z-index: 3;
  }

  /* Pause overlay */
  .overlay {
    position: absolute; inset: 0;
    display: none;
    align-items: center; justify-content: center;
    background: rgba(255,255,255,0.6);
    backdrop-filter: blur(3px);
    z-index: 3;
  }
  .overlay.show { display: flex; }

  /* Responsive tweaks */
  @media (max-width: 480px) {
    .pill { font-size: 14px; padding: 6px 10px; }
    .controls button { padding: 8px 10px; font-size: 14px; }
    .legend .tag { font-size: 12px; }
  }
</style>
</head>
<body>
<div class="app" role="application" aria-label="Soft and Safe Food Slice game">
  <header>
    <div class="topbar">
      <div class="group">
        <div class="pill" id="scorePill" aria-live="polite">⭐ Score: <span id="score">0</span></div>
        <div class="pill" id="livesPill" aria-live="polite">❤️ <span id="lives">3</span></div>
        <div class="pill" id="timePill" aria-live="polite">⏱ <span id="time">60</span>s</div>
      </div>
      <div class="legend" aria-label="Legend">
        <span class="tag"><span class="dot safe"></span> Safe</span>
        <span class="tag"><span class="dot caution"></span> Ask a grown‑up</span>
        <span class="tag"><span class="dot avoid"></span> Risky</span>
      </div>
      <div class="controls">
        <button id="btnPause" class="btn ghost" aria-pressed="false" title="Pause/Resume">⏸ Pause</button>
        <button id="btnGuide" class="btn ghost" title="Food guide">📚 Guide</button>
        <button id="btnSettings" class="btn ghost" title="Settings">⚙️ Settings</button>
      </div>
    </div>
  </header>

  <div id="stage-wrap">
    <canvas id="gameCanvas" aria-label="Game area"></canvas>
    <div id="pauseOverlay" class="overlay">
      <div class="card" style="max-width: 520px; text-align:center;">
        <h2>Game paused</h2>
        <p>Swipe the green foods. Avoid the red ones. Yellow means ask a grown‑up.</p>
        <div class="actions" style="justify-content:center;">
          <button id="btnResume">▶ Resume</button>
          <button id="btnRestart" class="btn secondary">🔁 Restart</button>
        </div>
      </div>
    </div>
    <div id="toast" class="toast" role="status"></div>
  </div>

  <footer>
    <button id="btnIntro" class="btn ghost">ℹ️ Intro</button>
    <button id="btnStart">🎮 Start New Game</button>
  </footer>
</div>

<!-- Intro Modal -->
<div id="introModal" class="modal-backdrop show" role="dialog" aria-modal="true" aria-labelledby="introTitle">
  <div class="modal">
    <h1 id="introTitle">Soft & Safe: Food Slice</h1>
    <p style="font-size: 1.05rem;">
      Learn to spot foods that are usually safer for kids with swallowing difficulties (dysphagia).
      Slice the green foods. Avoid the red ones. Yellow means “ask a grown‑up.”
    </p>
    <div class="grid">
      <div class="card">
        <h2>How to play</h2>
        <ul>
          <li>Swipe across foods to slice them.</li>
          <li>Green ring = soft and smooth → +points</li>
          <li>Red ring = hard, dry, sticky, or round → lose a heart</li>
          <li>Yellow ring = depends on how it’s prepared → no points (check with an adult)</li>
          <li>Game lasts 60 seconds. Try for a high score!</li>
        </ul>
      </div>
      <div class="card">
        <h2>Why it matters</h2>
        <ul>
          <li>Soft, smooth, and moist foods are easier to chew and swallow.</li>
          <li>Hard, crunchy, sticky, or mixed textures can be risky.</li>
          <li>Always follow the child’s care plan and therapist’s guidance.</li>
        </ul>
        <p style="font-size:.95rem;"><strong>Note:</strong> This is a learning game, not medical advice. Food safety depends on the individual child and preparation.</p>
      </div>
    </div>
    <div class="hr"></div>
    <div class="actions">
      <button id="introStart">🎮 Start</button>
      <button id="openSettings" class="btn ghost">⚙️ Settings</button>
      <button id="openGuide" class="btn ghost">📚 Food guide</button>
    </div>
  </div>
</div>

<!-- Settings Modal -->
<div id="settingsModal" class="modal-backdrop" role="dialog" aria-modal="true" aria-labelledby="settingsTitle">
  <div class="modal">
    <h2 id="settingsTitle">Settings</h2>
    <div class="grid">
      <div class="card">
        <h3>Game speed</h3>
        <label>Food speed: <strong id="speedVal">Slow</strong></label>
        <input id="speedSlider" type="range" min="0.5" max="1.5" step="0.05" value="0.7" />
        <label>Spawn rate: <strong id="spawnVal">1.3s</strong></label>
        <input id="spawnSlider" type="range" min="600" max="2000" step="50" value="1300" />
        <div style="margin-top:8px;">
          <label><input type="checkbox" id="assistMode" checked /> Assist mode (bigger targets, slower items)</label>
        </div>
      </div>
      <div class="card">
        <h3>Display, audio & accessibility</h3>
        <div><label><input type="checkbox" id="showLabels" checked /> Show food names on items</label></div>
        <div><label><input type="checkbox" id="soundOn" checked /> Sound effects</label></div>
        <div><label><input type="checkbox" id="voiceOn" /> Voice tips</label></div>
        <div><label><input type="checkbox" id="highContrast" /> High-contrast rings</label></div>
        <div><label><input type="checkbox" id="bigText" /> Bigger text</label></div>
      </div>
    </div>
    <div class="actions">
      <button id="settingsClose">Done</button>
      <button id="settingsStart" class="btn secondary">Start New Game</button>
    </div>
  </div>
</div>

<!-- Guide Modal -->
<div id="guideModal" class="modal-backdrop" role="dialog" aria-modal="true" aria-labelledby="guideTitle">
  <div class="modal">
    <h2 id="guideTitle">Food guide (examples)</h2>
    <p>Green = usually soft/smooth. Yellow = check preparation and the child’s plan. Red = often risky (hard, dry, sticky, or round).</p>
    <div class="food-list" id="foodList"></div>
    <div class="actions">
      <button id="guideClose">Close</button>
    </div>
  </div>
</div>

<!-- Game Over Modal -->
<div id="overModal" class="modal-backdrop" role="dialog" aria-modal="true" aria-labelledby="overTitle">
  <div class="modal">
    <h2 id="overTitle">Great job!</h2>
    <p>Your score: <strong id="finalScore">0</strong></p>
    <div class="grid">
      <div class="card">
        <h3>Summary</h3>
        <div>✅ Correct safe slices: <strong id="sumSafe">0</strong></div>
        <div>⚠️ Caution slices: <strong id="sumCaution">0</strong></div>
        <div>❌ Risky mistakes: <strong id="sumAvoid">0</strong></div>
      </div>
      <div class="card">
        <h3>Review misses</h3>
        <div id="reviewList" class="food-list"></div>
      </div>
    </div>
    <div class="actions">
      <button id="overRestart">🔁 Play Again</button>
      <button id="overClose" class="btn ghost">Close</button>
    </div>
  </div>
</div>

<script>
(function(){
  // -----------------------------
  // Data: Foods and categories
  // -----------------------------
  const FOODS = [
    // Safe (usually soft/smooth)
    {emoji:"🍌", name:"Banana (mashed)", cat:"safe", note:"Soft and smooth"},
    {emoji:"🥣", name:"Oatmeal / porridge", cat:"safe", note:"Soft, moist, cohesive"},
    {emoji:"🍮", name:"Pudding / custard", cat:"safe", note:"Smooth, no chunks"},
    {emoji:"🥑", name:"Avocado (mashed)", cat:"safe", note:"Soft and creamy"},
    {emoji:"🍠", name:"Mashed sweet potato", cat:"safe", note:"Soft and moist"},
    {emoji:"🍳", name:"Scrambled eggs (moist)", cat:"safe", note:"Soft, easy to chew"},
    {emoji:"🍎", name:"Applesauce", cat:"safe", note:"Smooth purée"},
    {emoji:"🥣", name:"Yogurt (smooth)", cat:"safe", note:"No chunks or seeds"},
    {emoji:"🥔", name:"Mashed potatoes", cat:"safe", note:"Smooth, with gravy"},
    {emoji:"🍲", name:"Pureed vegetable soup", cat:"safe", note:"Smooth, no chunks"},
    {emoji:"🍐", name:"Pear (ripe, mashed)", cat:"safe", note:"Very soft"},

    // Caution (depends on prep/plan)
    {emoji:"🍜", name:"Noodles (soft, cut small)", cat:"caution", note:"Can be long/slippery"},
    {emoji:"🍞", name:"Bread (small, moist bites)", cat:"caution", note:"Can be sticky/dry"},
    {emoji:"🍙", name:"Rice", cat:"caution", note:"Can scatter, be dry"},
    {emoji:"🍑", name:"Peach (peeled, diced)", cat:"caution", note:"Soft, but pieces"},
    {emoji:"🧀", name:"Cheese (small soft pieces)", cat:"caution", note:"Can be sticky if dry"},
    {emoji:"🍇", name:"Grapes (must be cut)", cat:"caution", note:"Whole grapes are risky"},
    {emoji:"🥞", name:"Pancakes (moist, cut small)", cat:"caution", note:"Add sauce/syrup"},
    {emoji:"🧇", name:"Waffles (moist, cut small)", cat:"caution", note:"Crispy bits can be dry"},
    {emoji:"🌯", name:"Soft wrap / tortilla (small pieces)", cat:"caution", note:"Can be chewy/sticky"},
    {emoji:"🍈", name:"Melon (peeled, diced)", cat:"caution", note:"Juicy but slippery"},
    {emoji:"🍓", name:"Strawberries (mashed)", cat:"caution", note:"Seeds and skins"},
    {emoji:"🍚", name:"Sticky rice with sauce", cat:"caution", note:"Can clump"},
    {emoji:"🍲", name:"Vegetables (very soft, diced)", cat:"caution", note:"Check softness"},

    // Avoid (often risky: hard/dry/sticky/round)
    {emoji:"🥕", name:"Carrot (raw)", cat:"avoid", note:"Hard and crunchy"},
    {emoji:"🍿", name:"Popcorn", cat:"avoid", note:"Dry bits can scatter"},
    {emoji:"🥜", name:"Nuts / peanuts", cat:"avoid", note:"Small and hard"},
    {emoji:"🍬", name:"Hard/gummy candy", cat:"avoid", note:"Sticky or hard"},
    {emoji:"🌭", name:"Hot dog (whole)", cat:"avoid", note:"Round and slippery"},
    {emoji:"🍏", name:"Apple (raw)", cat:"avoid", note:"Hard slices"},
    {emoji:"🥨", name:"Pretzel", cat:"avoid", note:"Dry and hard"},
    {emoji:"🍟", name:"Fries (dry)", cat:"avoid", note:"Dry pieces"},
    {emoji:"🍘", name:"Crackers (hard)", cat:"avoid", note:"Dry, crumbly"},
    {emoji:"🥬", name:"Celery (raw)", cat:"avoid", note:"Fibrous strings"},
    {emoji:"🍅", name:"Cherry tomato (whole)", cat:"avoid", note:"Round, can slip"},
    {emoji:"🍡", name:"Sticky rice cake", cat:"avoid", note:"Very sticky"},
    {emoji:"🥩", name:"Tough steak chunks", cat:"avoid", note:"Hard to chew"},
    {emoji:"🌮", name:"Hard taco shell", cat:"avoid", note:"Sharp, brittle"},
    {emoji:"🍭", name:"Lollipop", cat:"avoid", note:"Hard, choking risk"}
  ];

  // Build guide list
  const foodListEl = document.getElementById('foodList');
  function renderFoodGuide(list, intoEl){
    intoEl.innerHTML = "";
    list.forEach(f=>{
      const item = document.createElement('div');
      item.className = 'food-item';
      const badgeClass = f.cat;
      item.innerHTML = `
        <div class="emoji" aria-hidden="true">${f.emoji}</div>
        <div>
          <div style="font-weight:800">${f.name}</div>
          <div style="font-size:12px;opacity:.8">${f.note}</div>
        </div>
        <div class="badge ${badgeClass}">${f.cat === 'safe' ? 'SAFE' : f.cat === 'caution' ? 'ASK' : 'RISKY'}</div>
      `;
      intoEl.appendChild(item);
    });
  }
  renderFoodGuide(FOODS, foodListEl);

  // -----------------------------
  // State and settings
  // -----------------------------
  const state = {
    running: false,
    paused: false,
    timeLeft: 60,
    score: 0,
    lives: 3,
    stats: { safe:0, caution:0, avoid:0 },
    mistakes: [], // list of mistaken items
    dpr: Math.max(1, Math.min(2, window.devicePixelRatio || 1)),

    // Settings
    speedFactor: 0.7,   // slower by default
    spawnInterval: 1300,
    assistMode: true,
    showLabels: true,
    soundOn: true,
    voiceOn: false,
    highContrast: false,
    bigText: false,

    // Spawn probabilities
    spawnWeights: { safe: 0.55, caution: 0.2, avoid: 0.25 }
  };

  // -----------------------------
  // UI elements
  // -----------------------------
  const scoreEl = document.getElementById('score');
  const livesEl = document.getElementById('lives');
  const timeEl = document.getElementById('time');

  const btnStart = document.getElementById('btnStart');
  const btnPause = document.getElementById('btnPause');
  const btnResume = document.getElementById('btnResume');
  const btnRestart = document.getElementById('btnRestart');
  const btnIntro = document.getElementById('btnIntro');

  const introModal = document.getElementById('introModal');
  const introStart = document.getElementById('introStart');
  const openSettings = document.getElementById('openSettings');
  const openGuide = document.getElementById('openGuide');

  const settingsModal = document.getElementById('settingsModal');
  const settingsClose = document.getElementById('settingsClose');
  const settingsStart = document.getElementById('settingsStart');
  const speedSlider = document.getElementById('speedSlider');
  const spawnSlider = document.getElementById('spawnSlider');
  const speedVal = document.getElementById('speedVal');
  const spawnVal = document.getElementById('spawnVal');
  const assistMode = document.getElementById('assistMode');
  const showLabels = document.getElementById('showLabels');
  const soundOn = document.getElementById('soundOn');
  const voiceOn = document.getElementById('voiceOn');
  const highContrast = document.getElementById('highContrast');
  const bigText = document.getElementById('bigText');

  const guideModal = document.getElementById('guideModal');
  const guideClose = document.getElementById('guideClose');
  const btnGuide = document.getElementById('btnGuide');

  const overModal = document.getElementById('overModal');
  const overRestart = document.getElementById('overRestart');
  const overClose = document.getElementById('overClose');
  const finalScore = document.getElementById('finalScore');
  const sumSafe = document.getElementById('sumSafe');
  const sumCaution = document.getElementById('sumCaution');
  const sumAvoid = document.getElementById('sumAvoid');
  const reviewList = document.getElementById('reviewList');

  const pauseOverlay = document.getElementById('pauseOverlay');
  const toastEl = document.getElementById('toast');

  const canvas = document.getElementById('gameCanvas');
  const stageWrap = document.getElementById('stage-wrap');
  const ctx = canvas.getContext('2d');

  // -----------------------------
  // Audio (WebAudio beeps)
  // -----------------------------
  let audioCtx = null;
  function ensureAudio() {
    if (!audioCtx) {
      try { audioCtx = new (window.AudioContext || window.webkitAudioContext)(); }
      catch { audioCtx = null; }
    }
  }

  function playTone(freq=440, dur=0.12, type='sine', gain=0.12) {
    if (!state.soundOn || !audioCtx) return;
    const t = audioCtx.currentTime;
    const o = audioCtx.createOscillator();
    const g = audioCtx.createGain();
    o.type = type;
    o.frequency.value = freq;
    g.gain.setValueAtTime(0.0001, t);
    g.gain.exponentialRampToValueAtTime(gain, t + 0.01);
    g.gain.exponentialRampToValueAtTime(0.0001, t + dur);
    o.connect(g).connect(audioCtx.destination);
    o.start(t);
    o.stop(t + dur + 0.02);
  }

  function successSound() { playTone(660, 0.08, 'sine', 0.15); setTimeout(()=>playTone(880, 0.08, 'sine', 0.12), 60); }
  function cautionSound() { playTone(520, 0.09, 'triangle', 0.12); }
  function errorSound() { playTone(140, 0.13, 'sawtooth', 0.20); setTimeout(()=>playTone(110, 0.12, 'sawtooth', 0.18), 60); }
  function swooshSound() { playTone(420, 0.05, 'square', 0.06); }

  function speak(text) {
    if (!state.voiceOn || !('speechSynthesis' in window)) return;
    try {
      window.speechSynthesis.cancel();
      const u = new SpeechSynthesisUtterance(text);
      u.rate = 1; u.pitch = 1; u.volume = 0.9;
      window.speechSynthesis.speak(u);
    } catch {}
  }

  // -----------------------------
  // Canvas sizing
  // -----------------------------
  function resizeCanvas() {
    const rect = stageWrap.getBoundingClientRect();
    const dpr = state.dpr;
    canvas.width = Math.floor(rect.width * dpr);
    canvas.height = Math.floor(rect.height * dpr);
    ctx.setTransform(dpr,0,0,dpr,0,0); // 1 canvas unit = 1 CSS pixel
  }
  window.addEventListener('resize', resizeCanvas, {passive:true});
  resizeCanvas();

  // -----------------------------
  // Game entities
  // -----------------------------
  const items = [];
  const particles = [];
  const floaters = [];  // floating texts

  let spawnTimer = 0;
  let lastTime = performance.now();
  let pointer = {
    active: false,
    points: [],   // recent swipe points with timestamps [{x,y,t}]
    lastSwoosh: 0
  };

  function pickFood() {
    const r = Math.random();
    const w = state.spawnWeights;
    const cat = r < w.safe ? 'safe' : (r < w.safe + w.caution ? 'caution' : 'avoid');
    const candidates = FOODS.filter(f => f.cat === cat);
    return candidates[Math.floor(Math.random() * candidates.length)];
  }

  // Tunables: higher jump
  const LAUNCH_BASE = 560;   // increased for higher arcs
  const LAUNCH_VAR  = 260;

  function spawnItem() {
    const f = pickFood();
    const w = canvas.clientWidth;
    const h = canvas.clientHeight;
    const radiusBase = state.assistMode ? 42 : 36;
    const x = 30 + Math.random() * (w - 60);
    const y = h + radiusBase + 10;
    const dir = (Math.random() < 0.5 ? -1 : 1);
    const speedScale = state.assistMode ? state.speedFactor * 0.9 : state.speedFactor;

    // Higher initial upward speed for a taller arc
    const vy = - (LAUNCH_BASE + Math.random()*LAUNCH_VAR) * speedScale; // upward (more negative = higher)
    const vx = dir * (90 + Math.random()*120) * speedScale;

    const item = {
      id: Math.random().toString(36).slice(2),
      emoji: f.emoji,
      name: f.name,
      cat: f.cat, note: f.note,
      x, y, vx, vy,
      r: radiusBase,
      rot: (Math.random()*Math.PI*2),
      vr: (Math.random()*2 - 1) * 0.8 * speedScale,
      sliced: false,
      removed: false,
      born: performance.now()
    };
    items.push(item);
  }

  function spawnMaybe(dt) {
    spawnTimer += dt;
    const interval = state.spawnInterval * (state.assistMode ? 1.05 : 1.0);
    if (spawnTimer >= interval) {
      spawnTimer = 0;
      const count = Math.random() < 0.25 ? 2 : 1;
      for (let i=0;i<count;i++) spawnItem();
    }
  }

  // -----------------------------
  // Input handling
  // -----------------------------
  function canvasPos(e){
    const rect = canvas.getBoundingClientRect();
    return { x: e.clientX - rect.left, y: e.clientY - rect.top };
  }

  function startPointer(e){
    ensureAudio();
    if (!state.running || state.paused) return;
    pointer.active = true;
    pointer.points.length = 0;
    const p = canvasPos(e);
    const t = performance.now();
    pointer.points.push({x:p.x, y:p.y, t});
  }
  function movePointer(e){
    if (!state.running || state.paused || !pointer.active) return;
    const p = canvasPos(e);
    const t = performance.now();
    pointer.points.push({x:p.x, y:p.y, t});
    // trim old
    const horizon = t - 220; // keep last ~220ms
    while (pointer.points.length && pointer.points[0].t < horizon) pointer.points.shift();
    // occasional swoosh
    if (t - pointer.lastSwoosh > 120) {
      swooshSound();
      pointer.lastSwoosh = t;
    }
  }
  function endPointer(){
    pointer.active = false;
    pointer.points.length = 0;
  }

  canvas.addEventListener('pointerdown', (e)=>{ e.preventDefault(); startPointer(e); });
  canvas.addEventListener('pointermove', (e)=>{ e.preventDefault(); movePointer(e); });
  canvas.addEventListener('pointerup', (e)=>{ e.preventDefault(); endPointer(e); });
  canvas.addEventListener('pointercancel', endPointer);
  canvas.addEventListener('pointerleave', endPointer);

  // -----------------------------
  // Collision: segment-circle
  // -----------------------------
  function segCircleHit(p1, p2, cx, cy, r) {
    const d = { x: p2.x - p1.x, y: p2.y - p1.y };
    const f = { x: p1.x - cx, y: p1.y - cy };
    const a = d.x*d.x + d.y*d.y;
    const b = 2*(f.x*d.x + f.y*d.y);
    const c = (f.x*f.x + f.y*f.y) - r*r;
    if (a <= 1e-6) return false;
    const disc = b*b - 4*a*c;
    if (disc < 0) return false;
    const s = Math.sqrt(disc);
    const t1 = (-b - s)/(2*a);
    const t2 = (-b + s)/(2*a);
    if ((t1 >= 0 && t1 <= 1) || (t2 >= 0 && t2 <= 1)) return true;
    return false;
  }

  function checkSlices() {
    if (!pointer.active || pointer.points.length < 2) return;
    for (let it of items) {
      if (it.removed || it.sliced) continue;
      const r = it.r * (state.assistMode ? 1.1 : 1.0);
      for (let i = pointer.points.length - 2; i >= 0; i--) {
        const p1 = pointer.points[i], p2 = pointer.points[i+1];
        if (segCircleHit(p1, p2, it.x, it.y, r)) {
          sliceItem(it);
          break;
        }
      }
    }
  }

  // -----------------------------
  // Feedback helpers
  // -----------------------------
  function vibrate(ms=30){ try { if (navigator.vibrate) navigator.vibrate(ms); } catch{} }

  function floatingText(text, x, y, color, big=false) {
    const el = document.createElement('div');
    el.className = 'float';
    el.textContent = text;
    el.style.left = (x-10)+'px';
    el.style.top = (y-10)+'px';
    el.style.color = color;
    el.style.fontSize = big ? '24px' : '18px';
    stageWrap.appendChild(el);
    const start = performance.now();
    const dur = 800;
    const anim = () => {
      const t = performance.now() - start;
      const k = Math.min(1, t / dur);
      el.style.transform = `translateY(${-28*k}px)`;
      el.style.opacity = String(1 - k);
      if (k < 1) requestAnimationFrame(anim);
      else stageWrap.removeChild(el);
    };
    requestAnimationFrame(anim);
  }

  let toastTimer = null;
  function toast(msg) {
    toastEl.textContent = msg;
    toastEl.classList.add('show');
    clearTimeout(toastTimer);
    toastTimer = setTimeout(()=>toastEl.classList.remove('show'), 1200);
  }

  // Particles for juice
  function spawnParticles(x,y,color) {
    for (let i=0;i<10;i++) {
      particles.push({
        x, y,
        vx: (Math.random()*2-1) * 120,
        vy: (Math.random()*-1) * 120 - 40,
        r: 3 + Math.random()*3,
        life: 600 + Math.random()*400,
        color
      });
    }
  }

  // -----------------------------
  // Slice handling
  // -----------------------------
  function sliceItem(it) {
    it.sliced = true;
    spawnParticles(it.x, it.y, it.cat==='safe'? '#2ecc71' : it.cat==='avoid' ? '#e74c3c' : '#f1c40f');

    if (it.cat === 'safe') {
      const pts = 10;
      state.score += pts;
      state.stats.safe++;
      successSound();
      floatingText('+'+pts, it.x, it.y, '#0a7d3b', true);
      if (Math.random() < 0.25) speak('Soft and smooth!');
    } else if (it.cat === 'caution') {
      state.stats.caution++;
      cautionSound();
      toast('Ask a grown‑up!');
      floatingText('Ask a grown‑up', it.x, it.y, '#7a5d00');
      if (Math.random() < 0.3) speak('Ask a grown-up.');
    } else {
      state.stats.avoid++;
      state.lives = Math.max(0, state.lives - 1);
      errorSound(); vibrate(60);
      floatingText('Risky!', it.x, it.y, '#9a1414', true);
      state.mistakes.push({emoji: it.emoji, name: it.name, cat: it.cat, note: it.note});
      if (Math.random() < 0.3) speak('That one is risky.');
      screenFlash();
      if (state.lives <= 0) {
        endGame();
      }
    }

    it.removed = true;
  }

  function screenFlash() {
    stageWrap.style.transition = 'background-color .12s ease';
    stageWrap.style.backgroundColor = 'rgba(231,76,60,0.2)';
    setTimeout(()=>{ stageWrap.style.backgroundColor = ''; }, 120);
  }

  // -----------------------------
  // Drawing helpers: labels
  // -----------------------------
  function roundRectPath(ctx, x, y, w, h, r) {
    const rr = Math.min(r, w/2, h/2);
    ctx.beginPath();
    ctx.moveTo(x+rr, y);
    ctx.arcTo(x+w, y, x+w, y+h, rr);
    ctx.arcTo(x+w, y+h, x, y+h, rr);
    ctx.arcTo(x, y+h, x, y, rr);
    ctx.arcTo(x, y, x+w, y, rr);
    ctx.closePath();
  }

  function wrapLines(ctx, text, maxWidth, maxLines, fontSize) {
    const words = text.split(/\s+/);
    const lines = [];
    let cur = '';
    for (let i=0;i<words.length;i++) {
      const w = words[i];
      const test = cur ? cur + ' ' + w : w;
      if (ctx.measureText(test).width <= maxWidth) {
        cur = test;
      } else {
        if (cur) lines.push(cur);
        cur = w;
        if (lines.length === maxLines - 1) {
          // Ellipsize
          let ell = cur;
          while (ctx.measureText(ell + '…').width > maxWidth && ell.length > 1) {
            ell = ell.slice(0, -1);
          }
          lines.push(ell + '…');
          return lines;
        }
      }
    }
    if (cur) lines.push(cur);
    // If after wrapping still too many lines, trim and ellipsize last
    if (lines.length > maxLines) {
      const last = lines[maxLines-1];
      let ell = last;
      while (ctx.measureText(ell + '…').width > maxWidth && ell.length > 1) {
        ell = ell.slice(0, -1);
      }
      return lines.slice(0, maxLines-1).concat(ell + '…');
    }
    return lines;
  }

  function drawNameLabel(it) {
    if (!state.showLabels) return;
    const w = canvas.clientWidth;

    // Use main name without parentheses for brevity
    const mainName = it.name.replace(/\s*\(.*?\)\s*/,'').trim() || it.name;

    const ringColor = it.cat==='safe' ? '#2ecc71' : it.cat==='avoid' ? '#e74c3c' : '#f1c40f';
    const fontSize = state.bigText ? 16 : 14;
    const lineHeight = fontSize + 2;
    const maxWidth = Math.min(180, w * 0.5);

    ctx.save();
    ctx.font = `700 ${fontSize}px system-ui, -apple-system, Segoe UI, Roboto, "Apple Color Emoji", "Segoe UI Emoji"`;

    const lines = wrapLines(ctx, mainName, maxWidth, 2, fontSize);
    let textWidth = 0;
    for (const ln of lines) textWidth = Math.max(textWidth, ctx.measureText(ln).width);
    const padX = 8, padY = 4;
    const boxW = Math.ceil(textWidth) + padX*2;
    const boxH = lineHeight * lines.length + padY*2;

    // Place above the item if there's room, else below
    let boxX = Math.round(it.x - boxW/2);
    let boxY = Math.round(it.y - it.r - boxH - 8);
    if (boxY < 4) boxY = Math.round(it.y + it.r + 8);

    // Clamp within screen
    boxX = Math.max(4, Math.min(boxX, w - boxW - 4));

    // Draw bubble
    roundRectPath(ctx, boxX, boxY, boxW, boxH, 8);
    ctx.fillStyle = 'rgba(255,255,255,0.85)';
    ctx.fill();
    ctx.lineWidth = state.highContrast ? 2.5 : 2;
    ctx.strokeStyle = ringColor;
    ctx.stroke();

    // Text
    ctx.fillStyle = '#222';
    ctx.textBaseline = 'top';
    let ty = boxY + padY;
    const tx = boxX + padX;
    for (const ln of lines) {
      ctx.fillText(ln, tx, ty);
      ty += lineHeight;
    }
    ctx.restore();
  }

  // -----------------------------
  // Game loop
  // -----------------------------
  function update(dt) {
    // time
    state.timeLeft -= dt/1000;
    if (state.timeLeft < 0) state.timeLeft = 0;

    // spawn
    spawnMaybe(dt);

    // move items
    const g = 540 * state.speedFactor; // gravity
    for (let it of items) {
      if (it.removed) continue;
      it.vy += g * (dt/1000);
      it.x += it.vx * (dt/1000);
      it.y += it.vy * (dt/1000);
      it.rot += it.vr * (dt/1000);

      // off screen cleanup
      if (it.y > canvas.clientHeight + it.r + 40 || it.x < -60 || it.x > canvas.clientWidth + 60) {
        it.removed = true;
      }
    }

    // particles
    for (let p of particles) {
      p.vy += g * 0.6 * (dt/1000);
      p.x += p.vx * (dt/1000);
      p.y += p.vy * (dt/1000);
      p.life -= dt;
    }

    // cleanup arrays
    for (let i=items.length-1;i>=0;i--) if (items[i].removed) items.splice(i,1);
    for (let i=particles.length-1;i>=0;i--) if (particles[i].life <= 0) particles.splice(i,1);

    // slice checks
    checkSlices();

    // end check
    if (state.timeLeft <= 0) endGame();

    // UI
    scoreEl.textContent = state.score;
    livesEl.textContent = state.lives;
    timeEl.textContent = Math.ceil(state.timeLeft);
  }

  function draw() {
    const w = canvas.clientWidth;
    const h = canvas.clientHeight;
    ctx.clearRect(0,0,w,h);

    // Draw items
    for (let it of items) {
      if (it.removed) continue;
      const ringColor = it.cat==='safe' ? '#2ecc71' : it.cat==='avoid' ? '#e74c3c' : '#f1c40f';
      const ringStroke = state.highContrast ? '#000' : '#0006';

      // ring + emoji
      ctx.save();
      ctx.translate(it.x, it.y);
      ctx.rotate(it.rot);
      ctx.lineWidth = state.highContrast ? 5 : 4;
      ctx.beginPath();
      ctx.arc(0,0,it.r,0,Math.PI*2);
      ctx.fillStyle = it.cat==='safe' ? 'rgba(46,204,113,0.08)' : it.cat==='avoid' ? 'rgba(231,76,60,0.08)' : 'rgba(241,196,15,0.10)';
      ctx.fill();
      ctx.strokeStyle = ringColor;
      ctx.stroke();

      ctx.beginPath();
      ctx.arc(0,0,it.r-6,0,Math.PI*2);
      ctx.strokeStyle = ringStroke;
      ctx.setLineDash(state.highContrast && it.cat==='caution' ? [6,6] : []);
      ctx.stroke();
      ctx.setLineDash([]);

      ctx.font = (state.bigText ? 26 : 22) + 'px system-ui, "Apple Color Emoji", "Segoe UI Emoji"';
      ctx.textAlign = 'center';
      ctx.textBaseline = 'middle';
      ctx.fillStyle = ringColor;
      const sym = it.cat==='safe' ? '✓' : it.cat==='avoid' ? '✕' : '❓';
      ctx.fillText(sym, it.r*0.65, -it.r*0.65);

      ctx.font = (state.bigText ? 40 : 34) + 'px "Apple Color Emoji","Segoe UI Emoji",system-ui';
      ctx.fillText(it.emoji, 0, 2);

      ctx.restore();

      // name label (not rotated)
      drawNameLabel(it);
    }

    // Particles
    for (let p of particles) {
      ctx.beginPath();
      ctx.arc(p.x, p.y, p.r, 0, Math.PI*2);
      ctx.fillStyle = p.color;
      ctx.globalAlpha = Math.max(0, Math.min(1, p.life / 600));
      ctx.fill();
      ctx.globalAlpha = 1;
    }

    // Swipe trail
    if (pointer.points.length >= 2) {
      ctx.lineWidth = state.highContrast ? 6 : 5;
      const grad = ctx.createLinearGradient(pointer.points[0].x, pointer.points[0].y, pointer.points[pointer.points.length-1].x, pointer.points[pointer.points.length-1].y);
      grad.addColorStop(0,'rgba(255,255,255,0.3)');
      grad.addColorStop(1,'rgba(0,0,0,0.2)');
      ctx.strokeStyle = grad;
      ctx.lineCap = 'round';
      ctx.beginPath();
      for (let i=0;i<pointer.points.length;i++) {
        const p = pointer.points[i];
        if (i===0) ctx.moveTo(p.x, p.y); else ctx.lineTo(p.x, p.y);
      }
      ctx.stroke();
    }
  }

  function loop(ts) {
    const dt = Math.min(40, ts - lastTime);
    lastTime = ts;
    if (state.running && !state.paused) update(dt);
    draw();
    requestAnimationFrame(loop);
  }
  requestAnimationFrame(loop);

  // -----------------------------
  // Game control
  // -----------------------------
  function resetGame() {
    state.running = false;
    state.paused = false;
    state.timeLeft = 60;
    state.score = 0;
    state.lives = 3;
    state.stats = {safe:0, caution:0, avoid:0};
    state.mistakes = [];
    items.length = 0;
    particles.length = 0;
    floaters.length = 0;
    spawnTimer = 0;
    scoreEl.textContent = '0';
    livesEl.textContent = '3';
    timeEl.textContent = '60';
  }

  function startGame() {
    resetGame();
    state.running = true;
    pauseOverlay.classList.remove('show');
    hideModal(introModal);
    hideModal(settingsModal);
    hideModal(guideModal);
    hideModal(overModal);
    speak('Slice the green foods. Avoid the red ones. Yellow means ask a grown-up.');
  }

  function endGame() {
    if (!state.running) return;
    state.running = false;
    state.paused = false;
    showGameOver();
  }

  function togglePause() {
    if (!state.running) return;
    state.paused = !state.paused;
    btnPause.textContent = state.paused ? '▶ Resume' : '⏸ Pause';
    btnPause.setAttribute('aria-pressed', String(state.paused));
    pauseOverlay.classList.toggle('show', state.paused);
    if (state.paused) speak('Game paused.');
  }

  // -----------------------------
  // Modals helpers
  // -----------------------------
  function showModal(el){ el.classList.add('show'); }
  function hideModal(el){ el.classList.remove('show'); }
  function showGameOver() {
    finalScore.textContent = state.score;
    sumSafe.textContent = state.stats.safe;
    sumCaution.textContent = state.stats.caution;
    sumAvoid.textContent = state.stats.avoid;

    // Build review list from mistakes (unique)
    const uniq = [];
    const seen = new Set();
    for (const m of state.mistakes) {
      const key = m.emoji + '|' + m.name;
      if (!seen.has(key)) { seen.add(key); uniq.push(m); }
    }
    reviewList.innerHTML = '';
    if (uniq.length === 0) {
      const empty = document.createElement('div');
      empty.className = 'food-item';
      empty.innerHTML = '<div class="emoji">🎉</div><div>Nice! No risky slices.</div><div></div>';
      reviewList.appendChild(empty);
    } else {
      renderFoodGuide(uniq, reviewList);
    }

    showModal(overModal);
  }

  // -----------------------------
  // UI bindings
  // -----------------------------
  btnStart.addEventListener('click', startGame);
  introStart.addEventListener('click', startGame);
  settingsStart.addEventListener('click', startGame);

  btnPause.addEventListener('click', togglePause);
  btnResume.addEventListener('click', togglePause);
  btnRestart.addEventListener('click', startGame);
  overRestart.addEventListener('click', startGame);
  overClose.addEventListener('click', ()=>hideModal(overModal));

  btnIntro.addEventListener('click', ()=>showModal(introModal));
  openSettings.addEventListener('click', ()=>{ hideModal(introModal); showModal(settingsModal); });
  openGuide.addEventListener('click', ()=>{ hideModal(introModal); showModal(guideModal); });
  btnGuide.addEventListener('click', ()=>showModal(guideModal));
  guideClose.addEventListener('click', ()=>hideModal(guideModal));

  document.getElementById('btnSettings').addEventListener('click', ()=>showModal(settingsModal));
  settingsClose.addEventListener('click', ()=>hideModal(settingsModal));

  // Settings controls
  function updateSpeedLabel() {
    const v = parseFloat(speedSlider.value);
    speedVal.textContent = v < 0.75 ? 'Slow' : v > 1.25 ? 'Fast' : 'Normal';
  }
  function updateSpawnLabel() {
    const v = parseInt(spawnSlider.value,10);
    spawnVal.textContent = (v/1000).toFixed(1)+'s';
  }
  speedSlider.addEventListener('input', ()=>{
    state.speedFactor = parseFloat(speedSlider.value);
    updateSpeedLabel();
  });
  spawnSlider.addEventListener('input', ()=>{
    state.spawnInterval = parseInt(spawnSlider.value,10);
    updateSpawnLabel();
  });
  assistMode.addEventListener('change', ()=>state.assistMode = assistMode.checked);
  showLabels.addEventListener('change', ()=>state.showLabels = showLabels.checked);
  soundOn.addEventListener('change', ()=>state.soundOn = soundOn.checked);
  voiceOn.addEventListener('change', ()=>state.voiceOn = voiceOn.checked);
  highContrast.addEventListener('change', ()=>state.highContrast = highContrast.checked);
  bigText.addEventListener('change', ()=>{
    state.bigText = bigText.checked;
    document.body.style.fontSize = state.bigText ? '18px' : '';
  });
  updateSpeedLabel(); updateSpawnLabel();

  // Keyboard fallback (optional)
  window.addEventListener('keydown', (e)=>{
    if (e.key === ' '){ e.preventDefault(); togglePause(); }
    if (e.key === 'r') startGame();
    if (e.key === 'Escape') {
      if (introModal.classList.contains('show')) hideModal(introModal);
      else if (settingsModal.classList.contains('show')) hideModal(settingsModal);
      else if (guideModal.classList.contains('show')) hideModal(guideModal);
      else if (overModal.classList.contains('show')) hideModal(overModal);
      else togglePause();
    }
  }, {passive:false});

  // Start with intro shown (already visible)
})();
</script>
</body>
</html>

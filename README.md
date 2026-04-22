<!DOCTYPE html>
<html lang="zh-TW">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>塞車大逃亡</title>
<style>
  @import url('https://fonts.googleapis.com/css2?family=Noto+Sans+TC:wght@400;700;900&family=Share+Tech+Mono&display=swap');
  *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
  :root {
    --bg: #0d0d0f; --surface: #16161a; --border: #2a2a35;
    --grid-bg: #1a1a22; --cell: 72px; --gap: 4px;
    --accent: #ff3c3c; --accent2: #ffaa00; --text: #e8e8f0; --muted: #666680;
  }
  body {
    background: var(--bg); color: var(--text); font-family: 'Noto Sans TC', sans-serif;
    min-height: 100vh; display: flex; flex-direction: column;
    align-items: center; justify-content: center; padding: 20px; gap: 24px; user-select: none;
  }
  header { text-align: center; }
  header h1 { font-size: 2.2rem; font-weight: 900; letter-spacing: 0.08em; color: var(--accent); text-shadow: 0 0 30px rgba(255,60,60,0.4); }
  header p { color: var(--muted); font-family: 'Share Tech Mono', monospace; font-size: 0.8rem; margin-top: 4px; letter-spacing: 0.15em; }
  .game-area { display: flex; gap: 32px; align-items: flex-start; flex-wrap: wrap; justify-content: center; }
  .board-wrapper { position: relative; }
  .exit-arrow {
    position: absolute; right: -36px;
    top: calc(2 * (var(--cell) + var(--gap)) + var(--cell)/2 - 10px);
    width: 0; height: 0;
    border-top: 12px solid transparent; border-bottom: 12px solid transparent;
    border-left: 20px solid var(--accent); filter: drop-shadow(0 0 6px var(--accent));
    animation: pulse-arrow 1.2s ease-in-out infinite;
  }
  @keyframes pulse-arrow { 0%,100%{opacity:1;transform:translateX(0)} 50%{opacity:0.5;transform:translateX(4px)} }
  .board {
    display: grid; grid-template-columns: repeat(6, var(--cell)); grid-template-rows: repeat(6, var(--cell));
    gap: var(--gap); background: var(--grid-bg); border: 2px solid var(--border);
    padding: var(--gap); border-radius: 8px; position: relative;
    box-shadow: 0 0 40px rgba(0,0,0,0.6), inset 0 0 60px rgba(0,0,0,0.3);
  }
  .cell { background: var(--surface); border-radius: 3px; }
  .cell.exit-row { background: rgba(255,60,60,0.06); }
  .cars-layer { position: absolute; inset: var(--gap); pointer-events: none; }
  .car {
    position: absolute; border-radius: 6px; cursor: grab; pointer-events: all;
    display: flex; align-items: center; justify-content: center;
    font-family: 'Share Tech Mono', monospace; font-size: 0.65rem; font-weight: bold;
    letter-spacing: 0.05em; z-index: 2;
  }
  .car:active { cursor: grabbing; }
  .car.dragging { z-index: 10; filter: brightness(1.3); }
  .sidebar { display: flex; flex-direction: column; gap: 16px; min-width: 200px; }
  .stats-box { background: var(--surface); border: 1px solid var(--border); border-radius: 8px; padding: 16px 20px; }
  .stats-box h3 { font-size: 0.7rem; letter-spacing: 0.2em; text-transform: uppercase; color: var(--muted); margin-bottom: 12px; font-family: 'Share Tech Mono', monospace; }
  .stat-row { display: flex; justify-content: space-between; align-items: center; padding: 6px 0; border-bottom: 1px solid var(--border); }
  .stat-row:last-child { border-bottom: none; }
  .stat-label { color: var(--muted); font-size: 0.85rem; }
  .stat-value { font-family: 'Share Tech Mono', monospace; font-size: 1.1rem; color: var(--accent2); }
  .level-select { display: flex; flex-direction: column; gap: 8px; }
  .level-select h3 { font-size: 0.7rem; letter-spacing: 0.2em; text-transform: uppercase; color: var(--muted); font-family: 'Share Tech Mono', monospace; }
  .level-btn { background: var(--surface); border: 1px solid var(--border); color: var(--text); padding: 8px 14px; border-radius: 6px; cursor: pointer; font-family: 'Noto Sans TC', sans-serif; font-size: 0.85rem; text-align: left; transition: all 0.15s; display: flex; align-items: center; gap: 10px; }
  .level-btn:hover { border-color: var(--accent2); color: var(--accent2); }
  .level-btn.active { border-color: var(--accent); color: var(--accent); background: rgba(255,60,60,0.07); }
  .level-dot { width: 8px; height: 8px; border-radius: 50%; background: var(--muted); flex-shrink: 0; }
  .level-btn.active .level-dot { background: var(--accent); box-shadow: 0 0 6px var(--accent); }
  .btn-row { display: flex; gap: 8px; }
  .btn { flex: 1; padding: 10px; border: 1px solid var(--border); border-radius: 6px; background: var(--surface); color: var(--text); cursor: pointer; font-family: 'Share Tech Mono', monospace; font-size: 0.8rem; letter-spacing: 0.1em; transition: all 0.15s; }
  .btn:hover { border-color: var(--accent2); color: var(--accent2); }
  .btn.primary { background: var(--accent); border-color: var(--accent); color: #fff; font-weight: bold; }
  .btn.primary:hover { background: #ff5555; }
  .win-overlay { position: fixed; inset: 0; background: rgba(0,0,0,0.85); display: flex; flex-direction: column; align-items: center; justify-content: center; z-index: 100; opacity: 0; pointer-events: none; transition: opacity 0.4s; gap: 24px; }
  .win-overlay.visible { opacity: 1; pointer-events: all; }
  .win-overlay h2 { font-size: 3rem; font-weight: 900; color: var(--accent2); text-shadow: 0 0 40px rgba(255,170,0,0.5); animation: win-bounce 0.6s ease; }
  @keyframes win-bounce { 0%{transform:scale(0.5);opacity:0} 70%{transform:scale(1.1)} 100%{transform:scale(1);opacity:1} }
  .win-overlay p { color: var(--muted); font-family: 'Share Tech Mono', monospace; font-size: 1rem; }
  .win-overlay .btn { min-width: 160px; text-align: center; flex: none; }
  @media (max-width: 560px) {
    :root { --cell: 50px; --gap: 3px; }
    header h1 { font-size: 1.6rem; }
    .sidebar { min-width: 0; width: 100%; }
    .exit-arrow { right: -28px; }
  }
</style>
</head>
<body>

<header>
  <h1>塞車大逃亡</h1>
  <p>RUSH HOUR PUZZLE</p>
</header>

<div class="game-area">
  <div class="board-wrapper">
    <div class="board" id="board"></div>
    <div class="exit-arrow"></div>
  </div>
  <div class="sidebar">
    <div class="stats-box">
      <h3>統計</h3>
      <div class="stat-row"><span class="stat-label">移動次數</span><span class="stat-value" id="move-count">0</span></div>
      <div class="stat-row"><span class="stat-label">最佳紀錄</span><span class="stat-value" id="best-score">—</span></div>
    </div>
    <div class="level-select">
      <h3>關卡</h3>
      <button class="level-btn active" onclick="loadLevel(0)"><span class="level-dot"></span>簡單 — 入門</button>
      <button class="level-btn"        onclick="loadLevel(1)"><span class="level-dot"></span>普通 — 挑戰</button>
      <button class="level-btn"        onclick="loadLevel(2)"><span class="level-dot"></span>困難 — 地獄</button>
    </div>
    <div class="btn-row">
      <button class="btn primary" onclick="resetLevel()">重設</button>
      <button class="btn" onclick="undoMove()">上一步</button>
    </div>
    <div class="stats-box" style="font-family:'Share Tech Mono',monospace;font-size:0.75rem;color:var(--muted);line-height:1.8;">
      <h3>說明</h3>
      拖曳車輛沿方向滑動<br>
      將紅色車開到右側出口<br>
      挑戰最少移動次數！
    </div>
  </div>
</div>

<div class="win-overlay" id="win-overlay">
  <h2>🎉 成功逃脫！</h2>
  <p id="win-msg">共移動 0 次</p>
  <button class="btn primary" onclick="nextLevel()">下一關</button>
  <button class="btn" onclick="resetLevel()">再玩一次</button>
</div>

<script>
// ── 關卡 0：簡單（約 6 步）
//   Col:  0   1   2   3   4   5
// Row 0: [ .   .   B   C   C   . ]
// Row 1: [ .   .   B   .   D   . ]
// Row 2: [ R   R   B   .   D   . ]  ← exit →
// Row 3: [ .   .   .   E   E   . ]
// Row 4: [ F   F   F   .   .   . ]
// Row 5: [ .   .   .   .   .   . ]
const LEVEL0 = [
  { id:'red_car', length:2, orientation:'h', x:0, y:2, color:'#ff3c3c' },
  { id:'B',       length:3, orientation:'v', x:2, y:0, color:'#3c8cff' },
  { id:'C',       length:2, orientation:'h', x:3, y:0, color:'#44cc88' },
  { id:'D',       length:2, orientation:'v', x:4, y:1, color:'#ffaa00' },
  { id:'E',       length:2, orientation:'h', x:3, y:3, color:'#cc44ff' },
  { id:'F',       length:3, orientation:'h', x:0, y:4, color:'#44ccff' },
];

// ── 關卡 1：普通（約 10 步）
//   Col:  0   1   2   3   4   5
// Row 0: [ .   A   A   .   B   . ]
// Row 1: [ .   .   C   .   B   . ]
// Row 2: [ R   R   C   .   .   . ]  ← exit →
// Row 3: [ D   .   C   .   E   E ]
// Row 4: [ D   .   .   F   F   . ]
// Row 5: [ .   G   G   G   .   . ]
const LEVEL1 = [
  { id:'red_car', length:2, orientation:'h', x:0, y:2, color:'#ff3c3c' },
  { id:'A',       length:2, orientation:'h', x:1, y:0, color:'#3c8cff' },
  { id:'B',       length:2, orientation:'v', x:4, y:0, color:'#44cc88' },
  { id:'C',       length:3, orientation:'v', x:2, y:1, color:'#ffaa00' },
  { id:'D',       length:2, orientation:'v', x:0, y:3, color:'#ff88cc' },
  { id:'E',       length:2, orientation:'h', x:4, y:3, color:'#cc44ff' },
  { id:'F',       length:2, orientation:'h', x:3, y:4, color:'#44ccff' },
  { id:'G',       length:3, orientation:'h', x:1, y:5, color:'#ffcc44' },
];

// ── 關卡 2：困難（約 16 步）
//   Col:  0   1   2   3   4   5
// Row 0: [ A   A   .   B   B   . ]
// Row 1: [ .   .   C   .   D   . ]
// Row 2: [ R   R   C   .   D   . ]  ← exit →
// Row 3: [ .   E   C   .   .   F ]
// Row 4: [ .   E   .   G   G   F ]
// Row 5: [ H   H   H   .   I   I ]
const LEVEL2 = [
  { id:'red_car', length:2, orientation:'h', x:0, y:2, color:'#ff3c3c' },
  { id:'A',       length:2, orientation:'h', x:0, y:0, color:'#3c8cff' },
  { id:'B',       length:2, orientation:'h', x:3, y:0, color:'#44cc88' },
  { id:'C',       length:3, orientation:'v', x:2, y:1, color:'#ffaa00' },
  { id:'D',       length:2, orientation:'v', x:4, y:1, color:'#ff88cc' },
  { id:'E',       length:2, orientation:'v', x:1, y:3, color:'#cc44ff' },
  { id:'F',       length:2, orientation:'v', x:5, y:3, color:'#44ccff' },
  { id:'G',       length:2, orientation:'h', x:3, y:4, color:'#ffcc44' },
  { id:'H',       length:3, orientation:'h', x:0, y:5, color:'#88ff44' },
  { id:'I',       length:2, orientation:'h', x:4, y:5, color:'#ff6644' },
];

const LEVELS = [LEVEL0, LEVEL1, LEVEL2];

// ── 執行期驗證
function getCells(car) {
  const out = [];
  for (let i = 0; i < car.length; i++)
    out.push({ x: car.orientation==='h' ? car.x+i : car.x,
               y: car.orientation==='v' ? car.y+i : car.y });
  return out;
}

(function validateAll() {
  LEVELS.forEach((lvl, li) => {
    const seen = new Map();
    lvl.forEach(car => {
      getCells(car).forEach(({x,y}) => {
        const k = `${x},${y}`;
        if (seen.has(k)) console.error(`L${li}: ${car.id} overlaps ${seen.get(k)} at (${k})`);
        else seen.set(k, car.id);
        if (x<0||x>5||y<0||y>5) console.error(`L${li}: ${car.id} out of bounds at (${x},${y})`);
      });
    });
  });
})();

// ── STATE
let currentLevel = 0, cars = [], moves = 0, history = [];
const bestScores = [null, null, null];
const GRID = 6;
let activeDragCarId = null;

const cs = () => parseFloat(getComputedStyle(document.documentElement).getPropertyValue('--cell')) || 72;
const gs = () => parseFloat(getComputedStyle(document.documentElement).getPropertyValue('--gap'))  || 4;

// ── LEVEL MANAGEMENT
function loadLevel(idx) {
  currentLevel = idx;
  document.querySelectorAll('.level-btn').forEach((b,i) => b.classList.toggle('active', i===idx));
  cars = LEVELS[idx].map(c => ({...c}));
  moves = 0; history = [];
  activeDragCarId = null;
  updateMoveCount();
  render();
  document.getElementById('win-overlay').classList.remove('visible');
}
function resetLevel() { loadLevel(currentLevel); }
function nextLevel()  { loadLevel((currentLevel+1) % LEVELS.length); }
function undoMove() {
  if (!history.length) return;
  const snap = history.pop();
  cars = snap.cars.map(c => ({...c}));
  moves = snap.moves;
  updateMoveCount();
  render();
}

// ── RENDER
function render() {
  const board = document.getElementById('board');
  board.innerHTML = '';
  for (let r=0; r<GRID; r++)
    for (let c=0; c<GRID; c++) {
      const d = document.createElement('div');
      d.className = 'cell' + (r===2 ? ' exit-row' : '');
      board.appendChild(d);
    }

  const layer = document.createElement('div');
  layer.className = 'cars-layer';
  board.appendChild(layer);

  const C = cs(), G = gs();
  cars.forEach(car => {
    const el = document.createElement('div');
    el.className = 'car';
    el.dataset.id = car.id;
    const isH = car.orientation==='h';
    const w = isH ? car.length*C + (car.length-1)*G : C;
    const h = isH ? C : car.length*C + (car.length-1)*G;
    el.style.cssText = `
      width:${w}px; height:${h}px;
      left:${car.x*(C+G)}px; top:${car.y*(C+G)}px;
      background:${car.color};
      box-shadow:0 4px 16px ${car.color}55,inset 0 1px 0 rgba(255,255,255,.2);
      opacity:.93;
    `;
    if (car.id==='red_car')
      el.innerHTML = `<span style="color:#fff;font-size:.8rem;font-weight:900;text-shadow:0 1px 3px rgba(0,0,0,.5)">EXIT →</span>`;
    attachDrag(el, car);
    if (car.id === activeDragCarId) el.classList.add('dragging');
    layer.appendChild(el);
  });
}

// ── COLLISION: 逐格路徑碰撞檢查
function furthestPos(movingCar, targetPos) {
  const isH = movingCar.orientation === 'h';
  const cur = isH ? movingCar.x : movingCar.y;
  if (targetPos === cur) return cur;
  const dir = targetPos > cur ? 1 : -1;
  let pos = cur;
  while (pos !== targetPos) {
    const next = pos + dir;
    if (next < 0 || next > GRID - movingCar.length) break;
    const testCells = getCells({ ...movingCar, [isH?'x':'y']: next });
    let blocked = false;
    for (const other of cars) {
      if (other.id === movingCar.id) continue;
      for (const oc of getCells(other)) {
        if (testCells.some(tc => tc.x===oc.x && tc.y===oc.y)) { blocked=true; break; }
      }
      if (blocked) break;
    }
    if (blocked) break;
    pos = next;
  }
  return pos;
}

// ── DRAG
function attachDrag(el, car) {
  let startMouse, startGridPos, moved = false;

  const onStart = e => {
    e.preventDefault();
    const pt = e.touches ? e.touches[0] : e;
    startMouse   = car.orientation==='h' ? pt.clientX : pt.clientY;
    startGridPos = car.orientation==='h' ? car.x : car.y;
    moved = false;
    activeDragCarId = car.id;
    document.addEventListener('mousemove', onMove);
    document.addEventListener('touchmove', onMove, {passive:false});
    document.addEventListener('mouseup',   onEnd);
    document.addEventListener('touchend',  onEnd);
  };

  const onMove = e => {
    if (e.cancelable) e.preventDefault();
    const pt  = e.touches ? e.touches[0] : e;
    const cur = car.orientation==='h' ? pt.clientX : pt.clientY;
    const C = cs(), G = gs();
    const raw  = startGridPos + (cur - startMouse) / (C + G);
    const want = Math.max(0, Math.min(GRID - car.length, Math.round(raw)));
    const reach = furthestPos(car, want);
    const axis = car.orientation==='h' ? 'x' : 'y';
    if (reach !== car[axis]) {
      if (!moved) {
        history.push({ cars: cars.map(c=>({...c})), moves });
        moved = true;
      }
      car[axis] = reach;
      render();
    }
  };

  const onEnd = () => {
    if (moved) { moves++; updateMoveCount(); checkWin(); }
    activeDragCarId = null;
    document.removeEventListener('mousemove', onMove);
    document.removeEventListener('touchmove', onMove);
    document.removeEventListener('mouseup',   onEnd);
    document.removeEventListener('touchend',  onEnd);
  };

  el.addEventListener('mousedown',  onStart);
  el.addEventListener('touchstart', onStart, {passive:false});
}

// ── WIN
function checkWin() {
  const red = cars.find(c => c.id==='red_car');
  if (red && red.orientation==='h' && red.x + red.length >= GRID)
    setTimeout(showWin, 200);
}
function showWin() {
  if (!bestScores[currentLevel] || moves < bestScores[currentLevel])
    bestScores[currentLevel] = moves;
  document.getElementById('win-msg').textContent = `共移動 ${moves} 次`;
  document.getElementById('win-overlay').classList.add('visible');
  updateMoveCount();
}
function updateMoveCount() {
  document.getElementById('move-count').textContent = moves;
  document.getElementById('best-score').textContent = bestScores[currentLevel] ?? '—';
}

loadLevel(0);
window.addEventListener('resize', render);
</script>
</body>
</html>

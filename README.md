<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>Idle FarmVille Classic</title>
<style>
  :root {
    --bg-grass: #7cb342;
    --soil: #6d4c41;
    --soil-wet: #4e342e;
    --soil-ready: #8d6e63;
    --panel-bg: #fffde7;
    --accent: #fbc02d;
    --text: #2e3440;
    --btn-border: #4e342e;
  }

  * { box-sizing: border-box; margin: 0; padding: 0; user-select: none; }
  body {
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    background: var(--bg-grass);
    color: var(--text);
    padding: 16px;
    display: flex;
    justify-content: center;
  }

  #game-container {
    width: 100%;
    max-width: 1200px;
    background: #aed581;
    border: 6px solid var(--soil);
    border-radius: 12px;
    padding: 16px;
    display: flex;
    flex-direction: column;
    gap: 16px;
    box-shadow: 0 10px 30px rgba(0,0,0,0.2);
  }

  /* Header / Stats */
  header {
    background: var(--panel-bg);
    border: 3px solid var(--soil);
    border-radius: 8px;
    padding: 12px 20px;
    display: flex;
    flex-wrap: wrap;
    justify-content: space-between;
    align-items: center;
    font-size: 1.1rem;
    font-weight: bold;
  }

  .stat-group { display: flex; gap: 20px; align-items: center; }
  .stat { display: flex; align-items: center; gap: 6px; }
  .xp-bar-container {
    width: 140px;
    height: 16px;
    background: #ccc;
    border-radius: 8px;
    overflow: hidden;
    border: 1px solid #777;
  }
  .xp-bar-fill {
    height: 100%;
    width: 0%;
    background: #00bcd4;
    transition: width 0.3s ease;
  }

  /* Main Layout */
  .main-layout {
    display: grid;
    grid-template-columns: 2fr 1fr;
    gap: 16px;
  }

  @media (max-width: 900px) {
    .main-layout { grid-template-columns: 1fr; }
  }

  /* Farm Grid */
  .section-title {
    font-size: 1.2rem;
    font-weight: bold;
    margin-bottom: 8px;
    text-transform: uppercase;
    color: #3e2723;
  }

  .farm-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(100px, 1fr));
    gap: 10px;
    background: #81c784;
    padding: 16px;
    border-radius: 8px;
    border: 3px solid #558b2f;
    min-height: 400px;
  }

  .tile {
    aspect-ratio: 1;
    background: var(--soil);
    border-radius: 8px;
    border: 3px solid #4e342e;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: space-between;
    padding: 6px;
    cursor: pointer;
    position: relative;
    box-shadow: inset 0 0 10px rgba(0,0,0,0.3);
    transition: transform 0.1s;
  }

  .tile:active { transform: scale(0.96); }
  .tile.empty { background: #5d4037; opacity: 0.9; }
  .tile.growing { background: var(--soil-wet); }
  .tile.ready {
    background: var(--soil-ready);
    border-color: #ffd600;
    box-shadow: 0 0 12px #ffd600;
  }

  .tile-icon { font-size: 2rem; margin-top: 4px; }
  .tile-timer {
    font-size: 0.75rem;
    background: rgba(0,0,0,0.6);
    color: #fff;
    padding: 2px 6px;
    border-radius: 4px;
  }
  .tile-name { font-size: 0.75rem; font-weight: bold; color: #fff; text-shadow: 1px 1px 2px #000; }

  /* Side Panels */
  .side-panels {
    display: flex;
    flex-direction: column;
    gap: 16px;
  }

  .panel {
    background: var(--panel-bg);
    border: 3px solid var(--soil);
    border-radius: 8px;
    padding: 12px;
  }

  .panel-tabs {
    display: flex;
    gap: 6px;
    margin-bottom: 12px;
  }

  .tab-btn {
    flex: 1;
    padding: 8px 4px;
    border: 2px solid var(--btn-border);
    background: #ffecb3;
    cursor: pointer;
    font-weight: bold;
    border-radius: 4px;
    font-size: 0.85rem;
  }

  .tab-btn.active {
    background: var(--accent);
  }

  .tab-content { display: none; }
  .tab-content.active { display: flex; flex-direction: column; gap: 8px; max-height: 480px; overflow-y: auto; }

  /* Item Cards */
  .card {
    background: #fff;
    border: 2px solid #ccc;
    border-radius: 6px;
    padding: 8px 12px;
    display: flex;
    justify-content: space-between;
    align-items: center;
  }

  .card-info h4 { font-size: 0.95rem; }
  .card-info p { font-size: 0.75rem; color: #555; }
  .card-btn {
    padding: 6px 12px;
    border: 2px solid var(--btn-border);
    background: #8bc34a;
    font-weight: bold;
    cursor: pointer;
    border-radius: 4px;
    font-size: 0.8rem;
    transition: background 0.2s;
  }

  .card-btn:disabled {
    background: #bdbdbd;
    cursor: not-allowed;
  }

  .card-btn:not(:disabled):hover {
    background: #7cb342;
  }

  /* Selection Indicator */
  .selected-tool {
    display: flex;
    align-items: center;
    justify-content: space-between;
    background: #ffe082;
    padding: 8px 12px;
    border-radius: 6px;
    border: 2px dashed #ff8f00;
    margin-bottom: 8px;
    font-weight: bold;
    font-size: 0.9rem;
  }

  /* Log output */
  .log-box {
    background: #37474f;
    color: #eceff1;
    font-family: monospace;
    font-size: 0.75rem;
    padding: 8px;
    border-radius: 4px;
    height: 80px;
    overflow-y: auto;
  }

  /* Notifications */
  .offline-modal {
    position: fixed;
    top: 0; left: 0; right: 0; bottom: 0;
    background: rgba(0,0,0,0.6);
    display: none;
    align-items: center;
    justify-content: center;
    z-index: 1000;
  }
  .modal-box {
    background: var(--panel-bg);
    border: 4px solid var(--soil);
    padding: 24px;
    border-radius: 12px;
    max-width: 400px;
    text-align: center;
  }
  .modal-btn {
    margin-top: 16px;
    padding: 8px 24px;
    background: var(--accent);
    border: 2px solid var(--soil);
    font-weight: bold;
    cursor: pointer;
    border-radius: 6px;
  }
</style>
</head>
<body>

<div id="game-container">
  <header>
    <div class="stat-group">
      <div class="stat">💰 <span id="stat-coins">100</span></div>
      <div class="stat">⭐ Lvl <span id="stat-level">1</span></div>
      <div class="stat">
        <div class="xp-bar-container" title="Experience">
          <div class="xp-bar-fill" id="xp-bar"></div>
        </div>
        <span id="stat-xp" style="font-size: 0.8rem; margin-left: 4px;">0/100</span>
      </div>
    </div>
    <div class="stat-group">
      <button class="tab-btn" onclick="Game.saveGame()" style="padding: 4px 10px;">💾 Save</button>
      <button class="tab-btn" onclick="Game.resetGame()" style="padding: 4px 10px; background: #ffab91;">🔄 Reset</button>
    </div>
  </header>

  <div class="main-layout">
    <div style="display:flex; flex-direction: column; gap: 8px;">
      <div class="selected-tool">
        <span>Active Seed: <span id="current-selected-crop">Wheat</span></span>
        <button class="card-btn" onclick="Game.harvestAll()" style="background:#ffca28;">🌾 Harvest All Ready</button>
      </div>

      <div class="farm-grid" id="farm-grid">
        </div>

      <div class="log-box" id="game-log"></div>
    </div>

    <div class="side-panels">
      <div class="panel">
        <div class="panel-tabs">
          <button class="tab-btn active" onclick="Game.switchTab('crops')">🌱 Crops</button>
          <button class="tab-btn" onclick="Game.switchTab('animals')">🐔 Animals</button>
          <button class="tab-btn" onclick="Game.switchTab('upgrades')">⚙️ Upgrades</button>
        </div>

        <div id="tab-crops" class="tab-content active"></div>

        <div id="tab-animals" class="tab-content"></div>

        <div id="tab-upgrades" class="tab-content"></div>
      </div>
    </div>
  </div>
</div>

<div id="offline-modal" class="offline-modal">
  <div class="modal-box">
    <h2>Welcome Back, Farmer!</h2>
    <p style="margin-top: 10px;" id="offline-summary">While you were away, your farm produced goods.</p>
    <button class="modal-btn" onclick="document.getElementById('offline-modal').style.display='none'">Collect & Continue</button>
  </div>
</div>

<script>
/**
 * Idle FarmVille Engine
 */

const CROPS = [
  { id: 'wheat', name: 'Wheat', icon: '🌾', cost: 5, sell: 12, growTime: 3, xp: 4, unlockLevel: 1 },
  { id: 'carrot', name: 'Carrot', icon: '🥕', cost: 15, sell: 35, growTime: 7, xp: 10, unlockLevel: 2 },
  { id: 'tomato', name: 'Tomato', icon: '🍅', cost: 50, sell: 110, growTime: 15, xp: 25, unlockLevel: 3 },
  { id: 'corn', name: 'Corn', icon: '🌽', cost: 120, sell: 280, growTime: 30, xp: 55, unlockLevel: 4 },
  { id: 'strawberry', name: 'Strawberry', icon: '🍓', cost: 350, sell: 850, growTime: 60, xp: 130, unlockLevel: 5 },
  { id: 'pumpkin', name: 'Pumpkin', icon: '🎃', cost: 1000, sell: 2600, growTime: 120, xp: 300, unlockLevel: 7 }
];

const ANIMALS = [
  { id: 'chicken', name: 'Chicken', icon: '🐔', cost: 100, baseCost: 100, costMult: 1.25, produceRate: 2, item: 'Egg 🥚', itemVal: 5, count: 0, unlockLevel: 1 },
  { id: 'sheep', name: 'Sheep', icon: '🐑', cost: 500, baseCost: 500, costMult: 1.3, produceRate: 15, item: 'Wool 🧶', itemVal: 40, count: 0, unlockLevel: 3 },
  { id: 'cow', name: 'Cow', icon: '🐄', cost: 2500, baseCost: 2500, costMult: 1.35, produceRate: 120, item: 'Milk 🥛', itemVal: 350, count: 0, unlockLevel: 5 }
];

const UPGRADES = [
  { id: 'expand_plot', name: 'Buy Farm Plot', desc: 'Adds +1 plot to your field', cost: 50, baseCost: 50, costMult: 1.8, count: 0, max: 24, unlockLevel: 1 },
  { id: 'auto_harvester', name: 'Auto Harvester', desc: 'Automatically harvests ready crops every 2s', cost: 300, purchased: false, unlockLevel: 2 },
  { id: 'fertilizer', name: 'Golden Fertilizer', desc: 'Increases crop sell values by 25%', cost: 500, level: 0, baseCost: 500, costMult: 2.0, unlockLevel: 3 },
  { id: 'sprinkler', name: 'Sprinkler System', desc: 'Speeds up crop growth time by 20%', cost: 1000, purchased: false, unlockLevel: 4 }
];

class FarmGame {
  constructor() {
    this.coins = 100;
    this.level = 1;
    this.xp = 0;
    this.selectedCrop = CROPS[0].id;
    this.plots = [];
    this.animals = JSON.parse(JSON.stringify(ANIMALS));
    this.upgrades = JSON.parse(JSON.stringify(UPGRADES));
    this.lastTick = Date.now();

    this.initPlots(6);
    this.loadGame();
    this.setupUI();
    this.startLoop();
  }

  initPlots(count) {
    this.plots = [];
    for (let i = 0; i < count; i++) {
      this.plots.push({ crop: null, progress: 0, ready: false });
    }
  }

  getXpNeeded(level) {
    return Math.floor(60 * Math.pow(1.5, level - 1));
  }

  addXp(amount) {
    this.xp += amount;
    const needed = this.getXpNeeded(this.level);
    if (this.xp >= needed) {
      this.xp -= needed;
      this.level++;
      this.log(`🎉 Level Up! You reached Level ${this.level}!`);
      this.renderTabs();
    }
    this.updateStatsUI();
  }

  log(msg) {
    const box = document.getElementById('game-log');
    if (!box) return;
    const time = new Date().toLocaleTimeString().split(' ')[0];
    box.innerHTML = `[${time}] ${msg}<br>` + box.innerHTML;
  }

  // --- Farm Tile Actions ---
  clickPlot(idx) {
    const plot = this.plots[idx];
    if (plot.ready) {
      this.harvestPlot(idx);
    } else if (!plot.crop) {
      this.plantPlot(idx, this.selectedCrop);
    }
  }

  plantPlot(idx, cropId) {
    const crop = CROPS.find(c => c.id === cropId);
    if (!crop) return;
    if (this.level < crop.unlockLevel) {
      this.log(`⚠️ You need level ${crop.unlockLevel} to plant ${crop.name}.`);
      return;
    }
    if (this.coins < crop.cost) {
      this.log(`⚠️ Not enough coins for ${crop.name} (Requires 💰${crop.cost}).`);
      return;
    }

    this.coins -= crop.cost;
    this.plots[idx] = {
      crop: crop.id,
      progress: 0,
      ready: false
    };
    this.updateStatsUI();
    this.renderPlots();
  }

  harvestPlot(idx) {
    const plot = this.plots[idx];
    if (!plot || !plot.ready) return;

    const crop = CROPS.find(c => c.id === plot.crop);
    const fertUpgrade = this.upgrades.find(u => u.id === 'fertilizer');
    const multiplier = 1 + (fertUpgrade.level * 0.25);
    const reward = Math.round(crop.sell * multiplier);

    this.coins += reward;
    this.addXp(crop.xp);
    this.plots[idx] = { crop: null, progress: 0, ready: false };

    this.updateStatsUI();
    this.renderPlots();
  }

  harvestAll() {
    this.plots.forEach((p, idx) => {
      if (p.ready) this.harvestPlot(idx);
    });
  }

  // --- Purchases ---
  buyAnimal(id) {
    const an = this.animals.find(a => a.id === id);
    if (!an || this.coins < an.cost) return;

    this.coins -= an.cost;
    an.count++;
    an.cost = Math.round(an.baseCost * Math.pow(an.costMult, an.count));
    this.log(`Bought a ${an.name}!`);
    this.updateStatsUI();
    this.renderTabs();
  }

  buyUpgrade(id) {
    const up = this.upgrades.find(u => u.id === id);
    if (!up || this.coins < up.cost) return;

    if (id === 'expand_plot') {
      if (this.plots.length >= up.max) return;
      this.coins -= up.cost;
      up.count++;
      up.cost = Math.round(up.baseCost * Math.pow(up.costMult, up.count));
      this.plots.push({ crop: null, progress: 0, ready: false });
      this.renderPlots();
    } else if (id === 'auto_harvester' || id === 'sprinkler') {
      this.coins -= up.cost;
      up.purchased = true;
    } else if (id === 'fertilizer') {
      this.coins -= up.cost;
      up.level++;
      up.cost = Math.round(up.baseCost * Math.pow(up.costMult, up.level));
    }

    this.log(`Purchased upgrade: ${up.name}!`);
    this.updateStatsUI();
    this.renderTabs();
  }

  // --- Loop & Engine ---
  startLoop() {
    setInterval(() => {
      const now = Date.now();
      const dt = (now - this.lastTick) / 1000;
      this.lastTick = now;
      this.update(dt);
    }, 100);

    // Auto-save every 10s
    setInterval(() => this.saveGame(true), 10000);
  }

  update(dt) {
    const sprinkler = this.upgrades.find(u => u.id === 'sprinkler')?.purchased;
    const speedMult = sprinkler ? 1.25 : 1.0;

    // Grow Plots
    let plotChanged = false;
    this.plots.forEach(plot => {
      if (plot.crop && !plot.ready) {
        const crop = CROPS.find(c => c.id === plot.crop);
        plot.progress += (dt * speedMult);
        if (plot.progress >= crop.growTime) {
          plot.progress = crop.growTime;
          plot.ready = true;
          plotChanged = true;
        }
      }
    });

    // Passive Animals
    this.animals.forEach(an => {
      if (an.count > 0) {
        const incomeRate = (an.itemVal / an.produceRate) * an.count;
        this.coins += incomeRate * dt;
      }
    });

    // Auto-Harvester Check
    const autoHarvest = this.upgrades.find(u => u.id === 'auto_harvester')?.purchased;
    if (autoHarvest) {
      this.autoTimer = (this.autoTimer || 0) + dt;
      if (this.autoTimer >= 1.5) {
        this.autoTimer = 0;
        this.harvestAll();
      }
    }

    this.updateStatsUI();
    this.updateGridVisuals();
  }

  // --- Rendering UI ---
  setupUI() {
    this.renderPlots();
    this.renderTabs();
    this.updateStatsUI();
  }

  updateStatsUI() {
    document.getElementById('stat-coins').innerText = Math.floor(this.coins).toLocaleString();
    document.getElementById('stat-level').innerText = this.level;
    const needed = this.getXpNeeded(this.level);
    document.getElementById('stat-xp').innerText = `${Math.floor(this.xp)}/${needed}`;
    document.getElementById('xp-bar').style.width = `${Math.min(100, (this.xp / needed) * 100)}%`;
  }

  renderPlots() {
    const grid = document.getElementById('farm-grid');
    grid.innerHTML = '';

    this.plots.forEach((plot, idx) => {
      const tile = document.createElement('div');
      tile.className = 'tile';
      tile.id = `plot-${idx}`;

      if (!plot.crop) {
        tile.classList.add('empty');
        tile.innerHTML = `
          <div class="tile-name">Empty</div>
          <div class="tile-icon">🟫</div>
          <div class="tile-timer">Click to Plant</div>
        `;
      } else {
        const crop = CROPS.find(c => c.id === plot.crop);
        if (plot.ready) {
          tile.classList.add('ready');
          tile.innerHTML = `
            <div class="tile-name">${crop.name}</div>
            <div class="tile-icon">${crop.icon}</div>
            <div class="tile-timer" style="background:#2e7d32;">READY</div>
          `;
        } else {
          tile.classList.add('growing');
          const remaining = Math.max(0, Math.ceil(crop.growTime - plot.progress));
          tile.innerHTML = `
            <div class="tile-name">${crop.name}</div>
            <div class="tile-icon">🌱</div>
            <div class="tile-timer">${remaining}s</div>
          `;
        }
      }

      tile.onclick = () => this.clickPlot(idx);
      grid.appendChild(tile);
    });
  }

  updateGridVisuals() {
    this.plots.forEach((plot, idx) => {
      const tile = document.getElementById(`plot-${idx}`);
      if (!tile || !plot.crop) return;

      const crop = CROPS.find(c => c.id === plot.crop);
      const timerEl = tile.querySelector('.tile-timer');

      if (plot.ready) {
        if (!tile.classList.contains('ready')) {
          tile.className = 'tile ready';
          tile.querySelector('.tile-icon').innerText = crop.icon;
          timerEl.innerText = 'READY';
          timerEl.style.background = '#2e7d32';
        }
      } else {
        const remaining = Math.max(0, Math.ceil(crop.growTime - plot.progress));
        if (timerEl) timerEl.innerText = `${remaining}s`;
      }
    });
  }

  switchTab(tabKey) {
    document.querySelectorAll('.panel-tabs .tab-btn').forEach(btn => btn.classList.remove('active'));
    document.querySelectorAll('.tab-content').forEach(c => c.classList.remove('active'));

    event.target.classList.add('active');
    document.getElementById(`tab-${tabKey}`).classList.add('active');
  }

  selectCrop(cropId) {
    this.selectedCrop = cropId;
    const crop = CROPS.find(c => c.id === cropId);
    document.getElementById('current-selected-crop').innerText = `${crop.name} (${crop.icon})`;
  }

  renderTabs() {
    // Crops tab
    const cropsTab = document.getElementById('tab-crops');
    cropsTab.innerHTML = CROPS.map(c => {
      const locked = this.level < c.unlockLevel;
      return `
        <div class="card">
          <div class="card-info">
            <h4>${c.icon} ${c.name} ${locked ? `(Lvl ${c.unlockLevel})` : ''}</h4>
            <p>Cost: 💰${c.cost} | Profit: 💰${c.sell} | Time: ${c.growTime}s</p>
          </div>
          <button class="card-btn" 
            ${locked ? 'disabled' : ''} 
            onclick="Game.selectCrop('${c.id}')">
            ${this.selectedCrop === c.id ? 'Equipped' : 'Select'}
          </button>
        </div>
      `;
    }).join('');

    // Animals tab
    const animalsTab = document.getElementById('tab-animals');
    animalsTab.innerHTML = this.animals.map(a => {
      const locked = this.level < a.unlockLevel;
      const rate = (a.itemVal / a.produceRate).toFixed(1);
      return `
        <div class="card">
          <div class="card-info">
            <h4>${a.icon} ${a.name} [x${a.count}]</h4>
            <p>Produces ${a.item} (+💰${rate}/s total)</p>
          </div>
          <button class="card-btn" 
            ${locked || this.coins < a.cost ? 'disabled' : ''} 
            onclick="Game.buyAnimal('${a.id}')">
            Buy (💰${a.cost})
          </button>
        </div>
      `;
    }).join('');

    // Upgrades tab
    const upTab = document.getElementById('tab-upgrades');
    upTab.innerHTML = this.upgrades.map(u => {
      const locked = this.level < u.unlockLevel;
      let btnLabel = `Buy (💰${u.cost})`;
      let isMax = false;

      if (u.id === 'expand_plot') {
        btnLabel = `Expand (💰${u.cost})`;
        if (this.plots.length >= u.max) isMax = true;
      } else if (u.purchased) {
        btnLabel = 'Owned';
        isMax = true;
      } else if (u.id === 'fertilizer') {
        btnLabel = `Lvl ${u.level + 1} (💰${u.cost})`;
      }

      return `
        <div class="card">
          <div class="card-info">
            <h4>${u.name} ${locked ? `(Lvl ${u.unlockLevel})` : ''}</h4>
            <p>${u.desc}</p>
          </div>
          <button class="card-btn" 
            ${locked || isMax || this.coins < u.cost ? 'disabled' : ''} 
            onclick="Game.buyUpgrade('${u.id}')">
            ${isMax ? 'MAXED' : btnLabel}
          </button>
        </div>
      `;
    }).join('');
  }

  // --- Save / Load & Offline Gains ---
  saveGame(silent = false) {
    const saveData = {
      coins: this.coins,
      xp: this.xp,
      level: this.level,
      plots: this.plots,
      animals: this.animals,
      upgrades: this.upgrades,
      selectedCrop: this.selectedCrop,
      lastSaved: Date.now()
    };
    localStorage.setItem('idle_farm_save', JSON.stringify(saveData));
    if (!silent) this.log('💾 Game Saved!');
  }

  loadGame() {
    const dataStr = localStorage.getItem('idle_farm_save');
    if (!dataStr) return;

    try {
      const data = JSON.parse(dataStr);
      this.coins = data.coins || 0;
      this.xp = data.xp || 0;
      this.level = data.level || 1;
      this.plots = data.plots || this.plots;
      this.animals = data.animals || this.animals;
      this.upgrades = data.upgrades || this.upgrades;
      this.selectedCrop = data.selectedCrop || CROPS[0].id;

      // Process Offline Growth
      if (data.lastSaved) {
        const elapsedSec = (Date.now() - data.lastSaved) / 1000;
        if (elapsedSec > 5) {
          this.calculateOffline(elapsedSec);
        }
      }
    } catch (e) {
      console.error('Save file corrupt', e);
    }
  }

  calculateOffline(seconds) {
    let animalEarnings = 0;
    this.animals.forEach(an => {
      if (an.count > 0) {
        animalEarnings += (an.itemVal / an.produceRate) * an.count * seconds;
      }
    });

    let cropsMatured = 0;
    this.plots.forEach(plot => {
      if (plot.crop && !plot.ready) {
        const crop = CROPS.find(c => c.id === plot.crop);
        plot.progress += seconds;
        if (plot.progress >= crop.growTime) {
          plot.progress = crop.growTime;
          plot.ready = true;
          cropsMatured++;
        }
      }
    });

    this.coins += animalEarnings;
    const summary = document.getElementById('offline-summary');
    summary.innerHTML = `
      You were away for <strong>${Math.floor(seconds)}s</strong>.<br><br>
      🐔 Animals produced: <strong>💰${Math.floor(animalEarnings).toLocaleString()}</strong><br>
      🌾 Crops matured: <strong>${cropsMatured}</strong>
    `;
    document.getElementById('offline-modal').style.display = 'flex';
  }

  resetGame() {
    if (confirm('Are you sure you want to reset your entire farm?')) {
      localStorage.removeItem('idle_farm_save');
      location.reload();
    }
  }
}

// Start Game
let Game;
window.onload = () => {
  Game = new FarmGame();
};
</script>
</body>
</html>

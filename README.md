
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <title>Realm of Mastery</title>
  <style>
    :root {
      --bg-main: #0c0e14;
      --bg-panel: #161922;
      --bg-card: #1f2330;
      --bg-hover: #2a2f42;
      --border: #2d3345;
      --text-main: #e2e8f0;
      --text-muted: #94a3b8;
      --accent-gold: #f59e0b;
      --accent-green: #10b981;
      --accent-blue: #3b82f6;
      --accent-red: #ef4444;
      --font-pixel: 'Courier New', Courier, monospace;
    }

    * { box-sizing: border-box; margin: 0; padding: 0; user-select: none; -webkit-tap-highlight-color: transparent; }
    body {
      background-color: var(--bg-main);
      color: var(--text-main);
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
      height: 100vh;
      display: flex;
      flex-direction: column;
      overflow: hidden;
    }

    /* Top Navigation Header */
    header {
      background: var(--bg-panel);
      border-bottom: 1px solid var(--border);
      padding: 10px 16px;
      display: flex;
      justify-content: space-between;
      align-items: center;
      flex-shrink: 0;
      z-index: 10;
    }
    .brand-title { font-size: 1rem; color: var(--accent-gold); font-weight: 800; display: flex; align-items: center; gap: 6px; }
    .gold-counter { font-size: 0.95rem; font-weight: bold; color: var(--accent-gold); }

    /* Horizontal Nav Skill Ribbon */
    .skill-ribbon {
      background: #11141c;
      border-bottom: 1px solid var(--border);
      display: flex;
      overflow-x: auto;
      white-space: nowrap;
      padding: 8px;
      gap: 8px;
      flex-shrink: 0;
      scrollbar-width: none;
    }
    .skill-ribbon::-webkit-scrollbar { display: none; }
    
    .nav-btn {
      background: var(--bg-card);
      border: 1px solid var(--border);
      color: var(--text-muted);
      padding: 8px 12px;
      border-radius: 20px;
      cursor: pointer;
      font-size: 0.8rem;
      font-weight: 600;
      display: inline-flex;
      align-items: center;
      gap: 6px;
      flex-shrink: 0;
      transition: all 0.15s;
    }
    .nav-btn.active {
      background: var(--accent-blue);
      border-color: var(--accent-blue);
      color: #fff;
    }
    .skill-lvl {
      background: rgba(0,0,0,0.35);
      padding: 1px 6px;
      border-radius: 10px;
      font-size: 0.7rem;
    }

    /* Main Scrollable View Area */
    #main-content {
      flex: 1;
      overflow-y: auto;
      padding: 16px;
      padding-bottom: 85px; /* Room for action bar */
    }

    .view-panel { display: none; }
    .view-panel.active { display: block; }
    .grid-cards { display: grid; grid-template-columns: repeat(auto-fill, minmax(160px, 1fr)); gap: 12px; margin-top: 14px; }

    .card {
      background: var(--bg-card);
      border: 1px solid var(--border);
      border-radius: 8px;
      padding: 12px;
      display: flex;
      flex-direction: column;
      justify-content: space-between;
      gap: 8px;
    }
    .card-header { display: flex; justify-content: space-between; align-items: center; font-size: 0.9rem; font-weight: bold; }
    .card-req { font-size: 0.75rem; color: var(--accent-red); font-weight: normal; }
    .card-req.met { color: var(--accent-green); }

    .btn {
      background: var(--accent-blue);
      color: white;
      border: none;
      padding: 8px;
      border-radius: 6px;
      cursor: pointer;
      font-weight: 600;
      font-size: 0.8rem;
      width: 100%;
    }
    .btn:disabled { background: #334155; color: #64748b; }
    .btn-danger { background: var(--accent-red); }

    /* Bottom Fixed Action Status */
    #current-action-bar {
      position: fixed;
      bottom: 0;
      left: 0;
      right: 0;
      background: var(--bg-panel);
      border-top: 1px solid var(--border);
      padding: 10px 16px;
      z-index: 20;
    }
    .action-info { display: flex; justify-content: space-between; font-size: 0.8rem; margin-bottom: 6px; }
    .progress-track { width: 100%; height: 8px; background: #0f1118; border-radius: 4px; overflow: hidden; }
    .progress-fill { height: 100%; width: 0%; background: var(--accent-green); transition: width 0.05s linear; }

    /* Inventory & Gear */
    .inv-grid {
      display: grid;
      grid-template-columns: repeat(auto-fill, minmax(52px, 1fr));
      gap: 6px;
      background: var(--bg-card);
      padding: 10px;
      border-radius: 8px;
      margin-top: 10px;
      max-height: 260px;
      overflow-y: auto;
    }
    .inv-slot {
      aspect-ratio: 1;
      background: var(--bg-panel);
      border: 1px solid var(--border);
      border-radius: 6px;
      display: flex;
      align-items: center;
      justify-content: center;
      position: relative;
      font-size: 1.3rem;
      cursor: pointer;
    }
    .inv-count {
      position: absolute;
      bottom: 2px;
      right: 3px;
      font-size: 0.65rem;
      font-weight: bold;
      color: #fff;
      font-family: var(--font-pixel);
      text-shadow: 1px 1px 2px #000;
    }

    .gear-grid {
      display: grid;
      grid-template-columns: repeat(4, 1fr);
      gap: 8px;
      background: var(--bg-card);
      padding: 10px;
      border-radius: 8px;
      margin-top: 8px;
    }
    .gear-slot {
      aspect-ratio: 1;
      background: var(--bg-panel);
      border: 1px dashed var(--border);
      border-radius: 6px;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 1.2rem;
    }
    .gear-slot.filled { border-style: solid; border-color: var(--accent-blue); }

    /* Combat */
    .combat-arena {
      display: grid;
      grid-template-columns: 1fr auto 1fr;
      gap: 10px;
      align-items: center;
      background: var(--bg-card);
      border: 1px solid var(--border);
      border-radius: 8px;
      padding: 14px;
      margin-top: 10px;
    }
    .fighter-box { text-align: center; }
    .fighter-sprite { font-size: 2.2rem; margin-bottom: 4px; }
    .health-bar-bg { width: 100%; height: 10px; background: #0f1118; border-radius: 5px; overflow: hidden; }
    .health-fill { height: 100%; width: 100%; background: var(--accent-red); }
    .combat-log {
      height: 110px;
      background: #0f1118;
      border: 1px solid var(--border);
      border-radius: 6px;
      padding: 8px;
      overflow-y: auto;
      font-family: var(--font-pixel);
      font-size: 0.75rem;
      margin-top: 10px;
      display: flex;
      flex-direction: column;
      gap: 3px;
    }
    .log-hit-player { color: #f87171; }
    .log-hit-enemy { color: #4ade80; }
    .log-loot { color: var(--accent-gold); font-weight: bold; }

    /* Offline Modal */
    #offline-modal {
      position: fixed;
      inset: 0;
      background: rgba(0,0,0,0.8);
      display: none;
      align-items: center;
      justify-content: center;
      z-index: 100;
      padding: 20px;
    }
    .modal-box {
      background: var(--bg-panel);
      border: 1px solid var(--accent-gold);
      border-radius: 12px;
      padding: 20px;
      width: 100%;
      max-width: 360px;
      text-align: center;
      display: flex;
      flex-direction: column;
      gap: 12px;
    }
  </style>
</head>
<body>

  <!-- Top Header -->
  <header>
    <div class="brand-title">⚔️ REALM OF MASTERY</div>
    <div class="gold-counter">🪙 <span id="gold-display">0</span></div>
  </header>

  <!-- Horizontal Skill Selector Ribbon -->
  <nav class="skill-ribbon">
    <button class="nav-btn active" onclick="switchTab(this, 'woodcutting')">🪓 Wood <span class="skill-lvl" id="lvl-woodcutting">1</span></button>
    <button class="nav-btn" onclick="switchTab(this, 'mining')">⛏️ Mining <span class="skill-lvl" id="lvl-mining">1</span></button>
    <button class="nav-btn" onclick="switchTab(this, 'fishing')">🎣 Fishing <span class="skill-lvl" id="lvl-fishing">1</span></button>
    <button class="nav-btn" onclick="switchTab(this, 'smelting')">🔥 Smithing <span class="skill-lvl" id="lvl-smithing">1</span></button>
    <button class="nav-btn" onclick="switchTab(this, 'cooking')">🍳 Cooking <span class="skill-lvl" id="lvl-cooking">1</span></button>
    <button class="nav-btn" onclick="switchTab(this, 'combat')">⚔️ Combat <span class="skill-lvl" id="lvl-combat">3</span></button>
    <button class="nav-btn" onclick="switchTab(this, 'inventory')">🎒 Gear & Bag</button>
    <button class="nav-btn" onclick="switchTab(this, 'settings')">⚙️ Settings</button>
  </nav>

  <!-- Main Viewports -->
  <main id="main-content">
    
    <!-- Woodcutting -->
    <section id="view-woodcutting" class="view-panel active">
      <h2>🪓 Woodcutting</h2>
      <div class="grid-cards" id="wc-grid"></div>
    </section>

    <!-- Mining -->
    <section id="view-mining" class="view-panel">
      <h2>⛏️ Mining</h2>
      <div class="grid-cards" id="mining-grid"></div>
    </section>

    <!-- Fishing -->
    <section id="view-fishing" class="view-panel">
      <h2>🎣 Fishing</h2>
      <div class="grid-cards" id="fishing-grid"></div>
    </section>

    <!-- Smelting & Forging -->
    <section id="view-smelting" class="view-panel">
      <h2>🔥 Smelting & Forging</h2>
      <div class="grid-cards" id="smithing-grid"></div>
    </section>

    <!-- Cooking -->
    <section id="view-cooking" class="view-panel">
      <h2>🍳 Cooking Fire</h2>
      <div class="grid-cards" id="cooking-grid"></div>
    </section>

    <!-- Combat -->
    <section id="view-combat" class="view-panel">
      <h2>⚔️ Dungeon Combat</h2>
      
      <div class="combat-arena">
        <div class="fighter-box">
          <div class="fighter-sprite">🧙‍♂️</div>
          <div style="font-size:0.8rem; font-weight:bold;">Player</div>
          <div class="health-bar-bg"><div id="player-hp-bar" class="health-fill"></div></div>
          <div style="font-size:0.7rem; margin-top:2px;" id="player-hp-text">100 HP</div>
        </div>

        <div style="font-weight:800; color:var(--accent-red); font-size:1rem;">VS</div>

        <div class="fighter-box">
          <div class="fighter-sprite" id="enemy-sprite">👾</div>
          <div style="font-size:0.8rem; font-weight:bold;" id="enemy-name">None</div>
          <div class="health-bar-bg"><div id="enemy-hp-bar" class="health-fill"></div></div>
          <div style="font-size:0.7rem; margin-top:2px;" id="enemy-hp-text">0 HP</div>
        </div>
      </div>

      <div class="combat-log" id="combat-log">
        <div>Choose an enemy below to start combat.</div>
      </div>

      <h3 style="margin-top:14px; font-size:0.95rem;">Select Monster</h3>
      <div class="grid-cards" id="enemy-grid"></div>
    </section>

    <!-- Inventory & Equipment -->
    <section id="view-inventory" class="view-panel">
      <h2>🎒 Equipment & Inventory</h2>
      
      <div style="margin-top:10px;">
        <h4 style="font-size:0.85rem; color:var(--text-muted);">Equipped Gear (Tap to unequip)</h4>
        <div class="gear-grid">
          <div class="gear-slot" id="slot-helm" onclick="unequipSlot('helm')">🪖</div>
          <div class="gear-slot" id="slot-amulet" onclick="unequipSlot('amulet')">📿</div>
          <div class="gear-slot" id="slot-weapon" onclick="unequipSlot('weapon')">⚔️</div>
          <div class="gear-slot" id="slot-chest" onclick="unequipSlot('chest')">🦺</div>
          <div class="gear-slot" id="slot-shield" onclick="unequipSlot('shield')">🛡️</div>
          <div class="gear-slot" id="slot-legs" onclick="unequipSlot('legs')">👖</div>
          <div class="gear-slot" id="slot-boots" onclick="unequipSlot('boots')">🥾</div>
          <div class="gear-slot" id="slot-ring" onclick="unequipSlot('ring')">💍</div>
        </div>

        <div style="background:var(--bg-card); padding:10px; border-radius:6px; margin-top:8px; font-size:0.75rem; display:flex; justify-content:space-around;">
          <div>⚔️ Att: +<span id="stat-att">0</span></div>
          <div>💪 Str: +<span id="stat-str">0</span></div>
          <div>🛡️ Def: +<span id="stat-def">0</span></div>
        </div>
      </div>

      <div style="margin-top:14px;">
        <div style="display:flex; justify-content:space-between; align-items:center;">
          <h4 style="font-size:0.85rem; color:var(--text-muted);">Backpack (<span id="inv-capacity">0</span> items)</h4>
          <button class="btn btn-danger" style="width:auto; padding:4px 8px; font-size:0.75rem;" onclick="sellAllJunk()">Sell Raw Items</button>
        </div>
        <div class="inv-grid" id="inventory-container"></div>
        <div id="item-details" style="margin-top:8px; min-height:20px; font-size:0.8rem; color:var(--accent-gold);">Tap items to equip/consume.</div>
      </div>
    </section>

    <!-- Settings -->
    <section id="view-settings" class="view-panel">
      <h2>⚙️ Settings & Save</h2>
      <div class="card" style="margin-top:14px; gap:10px;">
        <button class="btn" onclick="saveGame()">💾 Save Game</button>
        <button class="btn" onclick="exportSave()">📤 Export Save Code</button>
        <button class="btn" onclick="importSave()">📥 Import Save Code</button>
        <button class="btn btn-danger" onclick="hardReset()">⚠️ Reset Game</button>
        <button class="btn" onclick="toggleAudio()" id="audio-toggle-btn">🔊 Audio: ON</button>
      </div>
    </section>

  </main>

  <!-- Sticky Bottom Active Progress Bar -->
  <div id="current-action-bar">
    <div class="action-info">
      <span id="active-action-name" style="font-weight:600;">Idle</span>
      <span id="active-action-timer" style="color:var(--text-muted);">0.0s</span>
    </div>
    <div class="progress-track">
      <div class="progress-fill" id="action-progress"></div>
    </div>
  </div>

  <!-- Offline Gains Modal -->
  <div id="offline-modal">
    <div class="modal-box">
      <h3 style="color:var(--accent-gold);">🌙 Welcome Back!</h3>
      <p id="offline-summary" style="font-size:0.85rem;"></p>
      <button class="btn" onclick="document.getElementById('offline-modal').style.display='none'">Claim Gains</button>
    </div>
  </div>

  <script>
    /* AUDIO SYNTHESIZER */
    const AudioEngine = {
      ctx: null,
      enabled: true,
      init() {
        if (!this.ctx) {
          const AudioContext = window.AudioContext || window.webkitAudioContext;
          this.ctx = new AudioContext();
        }
      },
      play(type) {
        if (!this.enabled) return;
        this.init();
        if (this.ctx.state === 'suspended') this.ctx.resume();
        const osc = this.ctx.createOscillator();
        const gain = this.ctx.createGain();
        osc.connect(gain);
        gain.connect(this.ctx.destination);
        const t = this.ctx.currentTime;

        if (type === 'chop' || type === 'mine') {
          osc.type = 'triangle';
          osc.frequency.setValueAtTime(140, t);
          osc.frequency.exponentialRampToValueAtTime(40, t + 0.08);
          gain.gain.setValueAtTime(0.3, t);
          gain.gain.linearRampToValueAtTime(0.01, t + 0.08);
          osc.start(t); osc.stop(t + 0.08);
        } else if (type === 'fish' || type === 'craft') {
          osc.type = 'sine';
          osc.frequency.setValueAtTime(300, t);
          osc.frequency.exponentialRampToValueAtTime(600, t + 0.1);
          gain.gain.setValueAtTime(0.2, t);
          gain.gain.linearRampToValueAtTime(0.01, t + 0.1);
          osc.start(t); osc.stop(t + 0.1);
        } else if (type === 'hit') {
          osc.type = 'sawtooth';
          osc.frequency.setValueAtTime(200, t);
          osc.frequency.linearRampToValueAtTime(70, t + 0.08);
          gain.gain.setValueAtTime(0.2, t);
          gain.gain.linearRampToValueAtTime(0.01, t + 0.08);
          osc.start(t); osc.stop(t + 0.08);
        } else if (type === 'levelup') {
          osc.type = 'square';
          osc.frequency.setValueAtTime(330, t);
          osc.frequency.setValueAtTime(440, t + 0.1);
          osc.frequency.setValueAtTime(660, t + 0.2);
          gain.gain.setValueAtTime(0.2, t);
          gain.gain.linearRampToValueAtTime(0.01, t + 0.4);
          osc.start(t); osc.stop(t + 0.4);
        }
      }
    };

    function toggleAudio() {
      AudioEngine.enabled = !AudioEngine.enabled;
      document.getElementById('audio-toggle-btn').innerText = AudioEngine.enabled ? '🔊 Audio: ON' : '🔇 Audio: OFF';
    }

    /* GAME DATA */
    const ITEMS = {
      normal_logs: { id: 'normal_logs', name: 'Logs', icon: '🪵', value: 4, type: 'resource' },
      oak_logs: { id: 'oak_logs', name: 'Oak Logs', icon: '🪵', value: 12, type: 'resource' },
      willow_logs: { id: 'willow_logs', name: 'Willow Logs', icon: '🪵', value: 28, type: 'resource' },
      copper_ore: { id: 'copper_ore', name: 'Copper Ore', icon: '🪨', value: 3, type: 'resource' },
      tin_ore: { id: 'tin_ore', name: 'Tin Ore', icon: '🪨', value: 3, type: 'resource' },
      iron_ore: { id: 'iron_ore', name: 'Iron Ore', icon: '🪨', value: 10, type: 'resource' },
      coal: { id: 'coal', name: 'Coal', icon: '⬛', value: 15, type: 'resource' },
      bronze_bar: { id: 'bronze_bar', name: 'Bronze Bar', icon: '🧱', value: 12, type: 'resource' },
      iron_bar: { id: 'iron_bar', name: 'Iron Bar', icon: '🧱', value: 28, type: 'resource' },
      steel_bar: { id: 'steel_bar', name: 'Steel Bar', icon: '🧱', value: 65, type: 'resource' },
      raw_shrimp: { id: 'raw_shrimp', name: 'Raw Shrimp', icon: '🦐', value: 3, type: 'raw_food' },
      cooked_shrimp: { id: 'cooked_shrimp', name: 'Cooked Shrimp', icon: '🍤', value: 6, type: 'food', heal: 10 },
      raw_trout: { id: 'raw_trout', name: 'Raw Trout', icon: '🐟', value: 12, type: 'raw_food' },
      cooked_trout: { id: 'cooked_trout', name: 'Cooked Trout', icon: '🍣', value: 24, type: 'food', heal: 25 },
      bronze_sword: { id: 'bronze_sword', name: 'Bronze Sword', icon: '🗡️', value: 30, slot: 'weapon', att: 4, str: 3 },
      iron_sword: { id: 'iron_sword', name: 'Iron Sword', icon: '⚔️', value: 100, slot: 'weapon', att: 10, str: 8 },
      bronze_chest: { id: 'bronze_chest', name: 'Bronze Plate', icon: '🦺', value: 50, slot: 'chest', def: 6 },
      iron_shield: { id: 'iron_shield', name: 'Iron Shield', icon: '🛡️', value: 90, slot: 'shield', def: 10 }
    };

    const GATHER_NODES = {
      woodcutting: [
        { id: 'normal_tree', name: 'Normal Tree', lvl: 1, xp: 15, time: 2.0, drop: 'normal_logs' },
        { id: 'oak_tree', name: 'Oak Tree', lvl: 15, xp: 38, time: 3.2, drop: 'oak_logs' },
        { id: 'willow_tree', name: 'Willow Tree', lvl: 30, xp: 68, time: 4.5, drop: 'willow_logs' }
      ],
      mining: [
        { id: 'copper_rock', name: 'Copper Rock', lvl: 1, xp: 18, time: 2.2, drop: 'copper_ore' },
        { id: 'tin_rock', name: 'Tin Rock', lvl: 1, xp: 18, time: 2.2, drop: 'tin_ore' },
        { id: 'iron_rock', name: 'Iron Rock', lvl: 15, xp: 35, time: 3.5, drop: 'iron_ore' },
        { id: 'coal_rock', name: 'Coal Deposit', lvl: 30, xp: 55, time: 4.0, drop: 'coal' }
      ],
      fishing: [
        { id: 'shrimp_spot', name: 'Pond (Shrimp)', lvl: 1, xp: 12, time: 2.0, drop: 'raw_shrimp' },
        { id: 'trout_spot', name: 'River (Trout)', lvl: 20, xp: 45, time: 3.4, drop: 'raw_trout' }
      ]
    };

    const RECIPES = {
      smithing: [
        { id: 'bronze_bar', name: 'Smelt Bronze Bar', lvl: 1, xp: 16, time: 2.0, req: { copper_ore: 1, tin_ore: 1 }, result: 'bronze_bar' },
        { id: 'iron_bar', name: 'Smelt Iron Bar', lvl: 15, xp: 28, time: 2.8, req: { iron_ore: 1 }, result: 'iron_bar' },
        { id: 'bronze_sword', name: 'Bronze Sword', lvl: 3, xp: 30, time: 3.0, req: { bronze_bar: 2 }, result: 'bronze_sword' },
        { id: 'bronze_chest', name: 'Bronze Plate', lvl: 8, xp: 60, time: 4.0, req: { bronze_bar: 4 }, result: 'bronze_chest' },
        { id: 'iron_sword', name: 'Iron Sword', lvl: 18, xp: 75, time: 3.5, req: { iron_bar: 3 }, result: 'iron_sword' }
      ],
      cooking: [
        { id: 'cooked_shrimp', name: 'Cook Shrimp', lvl: 1, xp: 12, time: 1.8, req: { raw_shrimp: 1 }, result: 'cooked_shrimp' },
        { id: 'cooked_trout', name: 'Cook Trout', lvl: 20, xp: 38, time: 2.6, req: { raw_trout: 1 }, result: 'cooked_trout' }
      ]
    };

    const ENEMIES = [
      { id: 'goblin', name: 'Goblin Scout', lvl: 3, hp: 35, maxHp: 35, att: 4, def: 2, speed: 2.4, sprite: '👺', drops: [{ id: 'bronze_sword', chance: 0.1 }, { id: 'copper_ore', chance: 0.5, qty: [1,3] }], gold: [3, 8] },
      { id: 'skeleton', name: 'Skeleton Guard', lvl: 12, hp: 80, maxHp: 80, att: 12, def: 8, speed: 2.2, sprite: '💀', drops: [{ id: 'iron_ore', chance: 0.6, qty: [2,4] }, { id: 'iron_sword', chance: 0.08 }], gold: [10, 25] },
      { id: 'orc', name: 'Orc Berserker', lvl: 28, hp: 190, maxHp: 190, att: 26, def: 18, speed: 2.6, sprite: '👹', drops: [{ id: 'coal', chance: 0.7, qty: [3,6] }], gold: [30, 75] }
    ];

    /* STATE */
    let State = {
      gold: 0,
      skills: {
        woodcutting: { xp: 0, lvl: 1 },
        mining: { xp: 0, lvl: 1 },
        fishing: { xp: 0, lvl: 1 },
        smithing: { xp: 0, lvl: 1 },
        cooking: { xp: 0, lvl: 1 },
        attack: { xp: 0, lvl: 1 },
        strength: { xp: 0, lvl: 1 },
        defense: { xp: 0, lvl: 1 },
        hitpoints: { xp: 1154, lvl: 10 }
      },
      inventory: {},
      equipment: { weapon: null, shield: null, helm: null, chest: null, legs: null, boots: null, amulet: null, ring: null },
      currentAction: null,
      combat: { active: false, enemy: null, playerHp: 100, playerTimer: 0, enemyTimer: 0 },
      lastTimestamp: Date.now()
    };

    function getLevelForXp(xp) {
      for (let lvl = 1; lvl <= 99; lvl++) {
        if (xp < Math.floor(50 * Math.pow(lvl, 2.1))) return lvl;
      }
      return 99;
    }

    function addXp(skillName, amount) {
      const s = State.skills[skillName];
      if (!s) return;
      s.xp += amount;
      const newLvl = getLevelForXp(s.xp);
      if (newLvl > s.lvl) {
        s.lvl = newLvl;
        AudioEngine.play('levelup');
        updateUI();
      }
    }

    function addItem(id, count = 1) {
      if (!ITEMS[id]) return;
      State.inventory[id] = (State.inventory[id] || 0) + count;
      renderInventory();
    }

    function removeItem(id, count = 1) {
      if (!State.inventory[id] || State.inventory[id] < count) return false;
      State.inventory[id] -= count;
      if (State.inventory[id] <= 0) delete State.inventory[id];
      renderInventory();
      return true;
    }

    function equipItem(id) {
      const item = ITEMS[id];
      if (!item || !item.slot) return;
      if (State.equipment[item.slot]) addItem(State.equipment[item.slot].id, 1);
      removeItem(id, 1);
      State.equipment[item.slot] = item;
      AudioEngine.play('craft');
      updateUI();
    }

    function unequipSlot(slot) {
      const item = State.equipment[slot];
      if (!item) return;
      addItem(item.id, 1);
      State.equipment[slot] = null;
      AudioEngine.play('craft');
      updateUI();
    }

    function getPlayerStats() {
      let att = State.skills.attack.lvl, str = State.skills.strength.lvl, def = State.skills.defense.lvl;
      for (let slot in State.equipment) {
        const itm = State.equipment[slot];
        if (itm) {
          if (itm.att) att += itm.att;
          if (itm.str) str += itm.str;
          if (itm.def) def += itm.def;
        }
      }
      return { att, str, def, maxHp: 100 + (State.skills.hitpoints.lvl - 10) * 8 };
    }

    function eatFood(id) {
      const item = ITEMS[id];
      if (!item || item.type !== 'food') return;
      const stats = getPlayerStats();
      if (State.combat.playerHp >= stats.maxHp) return;
      State.combat.playerHp = Math.min(stats.maxHp, State.combat.playerHp + item.heal);
      removeItem(id, 1);
      AudioEngine.play('fish');
      updateCombatUI();
    }

    function sellAllJunk() {
      let earned = 0;
      for (let id in State.inventory) {
        const itm = ITEMS[id];
        if (itm && (itm.type === 'resource' || itm.type === 'raw_food')) {
          earned += itm.value * State.inventory[id];
          delete State.inventory[id];
        }
      }
      State.gold += earned;
      AudioEngine.play('craft');
      updateUI();
    }

    function startAction(type, data) {
      if (State.combat.active) stopCombat();
      State.currentAction = { type, data, progress: 0, duration: data.time };
      document.getElementById('active-action-name').innerText = `${data.name}`;
    }

    function stopAction() {
      State.currentAction = null;
      document.getElementById('active-action-name').innerText = 'Idle';
      document.getElementById('active-action-timer').innerText = '0.0s';
      document.getElementById('action-progress').style.width = '0%';
    }

    function handleActionTick(dt) {
      if (!State.currentAction) return;
      const act = State.currentAction;
      act.progress += dt;
      const pct = Math.min(100, (act.progress / act.duration) * 100);
      document.getElementById('action-progress').style.width = pct + '%';
      document.getElementById('active-action-timer').innerText = (act.duration - act.progress).toFixed(1) + 's';

      if (act.progress >= act.duration) {
        act.progress = 0;
        completeAction(act);
      }
    }

    function completeAction(act) {
      if (['woodcutting', 'mining', 'fishing'].includes(act.type)) {
        addXp(act.type, act.data.xp);
        addItem(act.data.drop, 1);
        AudioEngine.play(act.type === 'fishing' ? 'fish' : 'chop');
      } else if (['smithing', 'cooking'].includes(act.type)) {
        let canCraft = true;
        for (let mat in act.data.req) {
          if (!State.inventory[mat] || State.inventory[mat] < act.data.req[mat]) { canCraft = false; break; }
        }
        if (canCraft) {
          for (let mat in act.data.req) removeItem(mat, act.data.req[mat]);
          addItem(act.data.result, 1);
          addXp(act.type, act.data.xp);
          AudioEngine.play('craft');
        } else {
          stopAction();
        }
      }
    }

    /* COMBAT TICK */
    function startCombat(enemyId) {
      stopAction();
      const template = ENEMIES.find(e => e.id === enemyId);
      if (!template) return;
      State.combat = {
        active: true,
        enemy: JSON.parse(JSON.stringify(template)),
        playerHp: State.combat.playerHp || getPlayerStats().maxHp,
        playerTimer: 0,
        enemyTimer: 0
      };
      document.getElementById('active-action-name').innerText = `Fighting ${template.name}`;
      logCombat(`⚔️ Attacking <strong>${template.name}</strong>!`);
      updateCombatUI();
    }

    function stopCombat() {
      State.combat.active = false;
      State.combat.enemy = null;
      document.getElementById('active-action-name').innerText = 'Idle';
      updateCombatUI();
    }

    function handleCombatTick(dt) {
      if (!State.combat.active || !State.combat.enemy) return;
      const c = State.combat;
      const stats = getPlayerStats();

      // Auto eat food
      if (c.playerHp < stats.maxHp * 0.35) {
        for (let id in State.inventory) {
          if (ITEMS[id] && ITEMS[id].type === 'food') { eatFood(id); break; }
        }
      }

      // Player Attack
      c.playerTimer += dt;
      if (c.playerTimer >= 2.4) {
        c.playerTimer = 0;
        const dmg = Math.floor(Math.random() * Math.max(2, stats.str * 0.8)) + 1;
        c.enemy.hp = Math.max(0, c.enemy.hp - dmg);
        addXp('attack', dmg * 2);
        addXp('strength', dmg * 2);
        addXp('hitpoints', dmg);
        logCombat(`🗡️ Hit ${c.enemy.name} for <strong class="log-hit-enemy">${dmg}</strong>!`);
        AudioEngine.play('hit');
      }

      // Enemy Defeated
      if (c.enemy.hp <= 0) {
        const goldLoot = Math.floor(Math.random() * (c.enemy.gold[1] - c.enemy.gold[0] + 1)) + c.enemy.gold[0];
        State.gold += goldLoot;
        c.enemy.drops.forEach(d => {
          if (Math.random() <= d.chance) {
            const count = d.qty ? Math.floor(Math.random() * (d.qty[1] - d.qty[0] + 1)) + d.qty[0] : 1;
            addItem(d.id, count);
            logCombat(`🎁 Got: ${ITEMS[d.id].name} x${count}`, 'log-loot');
          }
        });
        const template = ENEMIES.find(e => e.id === c.enemy.id);
        c.enemy.hp = template.maxHp;
      }

      // Enemy Attack
      c.enemyTimer += dt;
      if (c.enemyTimer >= c.enemy.speed) {
        c.enemyTimer = 0;
        const dmg = Math.max(1, Math.floor(Math.random() * (c.enemy.att * 0.5)));
        c.playerHp = Math.max(0, c.playerHp - dmg);
        addXp('defense', dmg * 2);
        logCombat(`🩸 ${c.enemy.name} hits you for <strong class="log-hit-player">${dmg}</strong>!`);
      }

      if (c.playerHp <= 0) {
        logCombat(`☠️ You were defeated!`, 'log-hit-player');
        c.playerHp = stats.maxHp;
        stopCombat();
      }

      updateCombatUI();
    }

    function logCombat(msg, className = '') {
      const log = document.getElementById('combat-log');
      const row = document.createElement('div');
      if (className) row.className = className;
      row.innerHTML = msg;
      log.appendChild(row);
      log.scrollTop = log.scrollHeight;
    }

    /* UI SWITCHING */
    function switchTab(btn, tabId) {
      document.querySelectorAll('.view-panel').forEach(el => el.classList.remove('active'));
      document.querySelectorAll('.nav-btn').forEach(el => el.classList.remove('active'));
      const target = document.getElementById('view-' + tabId);
      if (target) target.classList.add('active');
      if (btn) btn.classList.add('active');
    }

    function renderCards() {
      ['woodcutting', 'mining', 'fishing'].forEach(skill => {
        const container = document.getElementById(skill === 'woodcutting' ? 'wc-grid' : skill === 'mining' ? 'mining-grid' : 'fishing-grid');
        container.innerHTML = GATHER_NODES[skill].map(node => `
          <div class="card">
            <div class="card-header">
              <span>${ITEMS[node.drop].icon} ${node.name}</span>
              <span class="card-req ${State.skills[skill].lvl >= node.lvl ? 'met' : ''}">Lv ${node.lvl}</span>
            </div>
            <div style="font-size:0.75rem; color:var(--text-muted);">+${node.xp} XP | ${node.time}s</div>
            <button class="btn" ${State.skills[skill].lvl < node.lvl ? 'disabled' : ''} onclick='startAction("${skill}", ${JSON.stringify(node)})'>Gather</button>
          </div>
        `).join('');
      });

      ['smithing', 'cooking'].forEach(skill => {
        const container = document.getElementById(skill === 'smithing' ? 'smithing-grid' : 'cooking-grid');
        container.innerHTML = RECIPES[skill].map(rec => {
          const reqText = Object.entries(rec.req).map(([m, c]) => `${c}x ${ITEMS[m].name}`).join(', ');
          return `
            <div class="card">
              <div class="card-header">
                <span>${ITEMS[rec.result].icon} ${rec.name}</span>
                <span class="card-req ${State.skills[skill].lvl >= rec.lvl ? 'met' : ''}">Lv ${rec.lvl}</span>
              </div>
              <div style="font-size:0.75rem; color:var(--text-muted);">${reqText}</div>
              <button class="btn" ${State.skills[skill].lvl < rec.lvl ? 'disabled' : ''} onclick='startAction("${skill}", ${JSON.stringify(rec)})'>Craft</button>
            </div>
          `;
        }).join('');
      });

      document.getElementById('enemy-grid').innerHTML = ENEMIES.map(e => `
        <div class="card">
          <div class="card-header">
            <span>${e.sprite} ${e.name}</span>
            <span class="card-req met">Lv ${e.lvl}</span>
          </div>
          <div style="font-size:0.75rem; color:var(--text-muted);">HP: ${e.hp} | Atk: ${e.att}</div>
          <button class="btn" onclick="startCombat('${e.id}')">Fight</button>
        </div>
      `).join('');
    }

    function renderInventory() {
      const container = document.getElementById('inventory-container');
      const entries = Object.entries(State.inventory);
      document.getElementById('inv-capacity').innerText = entries.length;

      container.innerHTML = entries.map(([id, count]) => {
        const itm = ITEMS[id];
        return `
          <div class="inv-slot" onclick="handleItemClick('${id}')" title="${itm.name}">
            ${itm.icon}
            <span class="inv-count">${count}</span>
          </div>
        `;
      }).join('');
    }

    function handleItemClick(id) {
      const itm = ITEMS[id];
      const details = document.getElementById('item-details');
      if (itm.type === 'food') {
        eatFood(id);
        details.innerHTML = `Healed with <strong>${itm.name}</strong>!`;
      } else if (itm.slot) {
        equipItem(id);
        details.innerHTML = `Equipped <strong>${itm.name}</strong>!`;
      } else {
        details.innerHTML = `<strong>${itm.name}</strong>: ${itm.value} gold each.`;
      }
    }

    function renderEquipment() {
      ['weapon', 'shield', 'helm', 'chest', 'legs', 'boots', 'amulet', 'ring'].forEach(s => {
        const el = document.getElementById(`slot-${s}`);
        const itm = State.equipment[s];
        if (itm) {
          el.innerHTML = itm.icon;
          el.classList.add('filled');
        } else {
          el.innerHTML = s === 'weapon' ? '⚔️' : s === 'shield' ? '🛡️' : s === 'helm' ? '🪖' : s === 'chest' ? '🦺' : '💍';
          el.classList.remove('filled');
        }
      });
      const stats = getPlayerStats();
      document.getElementById('stat-att').innerText = stats.att;
      document.getElementById('stat-str').innerText = stats.str;
      document.getElementById('stat-def').innerText = stats.def;
    }

    function updateCombatUI() {
      const stats = getPlayerStats();
      document.getElementById('player-hp-bar').style.width = `${(State.combat.playerHp / stats.maxHp) * 100}%`;
      document.getElementById('player-hp-text').innerText = `${Math.floor(State.combat.playerHp)} / ${stats.maxHp} HP`;

      if (State.combat.enemy) {
        document.getElementById('enemy-name').innerText = State.combat.enemy.name;
        document.getElementById('enemy-sprite').innerText = State.combat.enemy.sprite;
        document.getElementById('enemy-hp-bar').style.width = `${(State.combat.enemy.hp / State.combat.enemy.maxHp) * 100}%`;
        document.getElementById('enemy-hp-text').innerText = `${Math.floor(State.combat.enemy.hp)} HP`;
      } else {
        document.getElementById('enemy-name').innerText = 'None';
        document.getElementById('enemy-sprite').innerText = '👾';
        document.getElementById('enemy-hp-bar').style.width = '0%';
        document.getElementById('enemy-hp-text').innerText = '0 HP';
      }
    }

    function updateUI() {
      document.getElementById('gold-display').innerText = State.gold.toLocaleString();
      for (let s in State.skills) {
        const el = document.getElementById(`lvl-${s}`);
        if (el) el.innerText = State.skills[s].lvl;
      }
      const combatLvl = Math.floor((State.skills.attack.lvl + State.skills.strength.lvl + State.skills.defense.lvl + State.skills.hitpoints.lvl) / 4);
      document.getElementById('lvl-combat').innerText = combatLvl;
      renderCards();
      renderEquipment();
      renderInventory();
    }

    /* SAVE / LOAD */
    function saveGame() {
      State.lastTimestamp = Date.now();
      localStorage.setItem('RoM_SaveData', JSON.stringify(State));
    }

    function loadGame() {
      const data = localStorage.getItem('RoM_SaveData');
      if (data) {
        try {
          const parsed = JSON.parse(data);
          State = Object.assign(State, parsed);
          const now = Date.now();
          const delta = (now - (State.lastTimestamp || now)) / 1000;
          if (delta > 10 && State.currentAction) {
            const cycles = Math.min(Math.floor(delta / State.currentAction.duration), 5000);
            for (let i = 0; i < cycles; i++) completeAction(State.currentAction);
            document.getElementById('offline-summary').innerHTML = `While away for ${Math.floor(delta/60)} mins, completed ${cycles}x actions!`;
            document.getElementById('offline-modal').style.display = 'flex';
          }
        } catch (e) {}
      }
      updateUI();
    }

    function exportSave() { prompt("Copy save code:", btoa(JSON.stringify(State))); }
    function importSave() {
      const code = prompt("Paste save code:");
      if (code) {
        try { State = JSON.parse(atob(code)); saveGame(); location.reload(); } catch (e) { alert("Invalid code!"); }
      }
    }
    function hardReset() {
      if (confirm("Erase all data?")) { localStorage.removeItem('RoM_SaveData'); location.reload(); }
    }

    /* ENGINE LOOP */
    let lastTickTime = performance.now();
    function gameLoop(now) {
      const dt = (now - lastTickTime) / 1000;
      lastTickTime = now;
      handleActionTick(dt);
      handleCombatTick(dt);
      requestAnimationFrame(gameLoop);
    }

    window.addEventListener('DOMContentLoaded', () => {
      loadGame();
      setInterval(saveGame, 10000);
      requestAnimationFrame(gameLoop);
    });
  </script>
</body>
</html>


<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Realm of Mastery - Idle RPG</title>
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
      --accent-purple: #8b5cf6;
      --font-pixel: 'Courier New', Courier, monospace;
    }

    * { box-sizing: border-box; margin: 0; padding: 0; user-select: none; }
    body {
      background-color: var(--bg-main);
      color: var(--text-main);
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
      height: 100vh;
      display: flex;
      overflow: hidden;
    }

    /* Layout */
    #sidebar {
      width: 280px;
      background: var(--bg-panel);
      border-right: 1px solid var(--border);
      display: flex;
      flex-direction: column;
      flex-shrink: 0;
    }

    #main-content {
      flex: 1;
      display: flex;
      flex-direction: column;
      overflow-y: auto;
      background: radial-gradient(circle at top right, #1a2035, var(--bg-main));
    }

    /* Sidebar Header */
    .brand {
      padding: 16px;
      border-bottom: 1px solid var(--border);
      display: flex;
      align-items: center;
      justify-content: space-between;
    }
    .brand h1 { font-size: 1.1rem; color: var(--accent-gold); font-weight: 800; letter-spacing: 0.5px; }
    .gold-counter { font-size: 0.95rem; font-weight: bold; color: var(--accent-gold); }

    /* Nav Buttons */
    .nav-tabs {
      flex: 1;
      overflow-y: auto;
      padding: 10px;
      display: flex;
      flex-direction: column;
      gap: 4px;
    }
    .nav-btn {
      background: transparent;
      border: 1px solid transparent;
      color: var(--text-muted);
      padding: 10px 14px;
      border-radius: 8px;
      cursor: pointer;
      display: flex;
      align-items: center;
      justify-content: space-between;
      font-size: 0.9rem;
      transition: all 0.2s;
    }
    .nav-btn:hover { background: var(--bg-hover); color: var(--text-main); }
    .nav-btn.active {
      background: var(--bg-card);
      border-color: var(--border);
      color: var(--accent-blue);
      font-weight: bold;
    }
    .nav-btn .skill-lvl {
      font-size: 0.75rem;
      background: rgba(0,0,0,0.3);
      padding: 2px 6px;
      border-radius: 4px;
      color: var(--text-main);
    }

    /* Active Task Bar */
    #current-action-bar {
      padding: 14px;
      background: var(--bg-card);
      border-top: 1px solid var(--border);
    }
    .action-info { display: flex; justify-content: space-between; font-size: 0.85rem; margin-bottom: 6px; }
    .progress-track {
      width: 100%;
      height: 8px;
      background: #0f1118;
      border-radius: 4px;
      overflow: hidden;
    }
    .progress-fill { height: 100%; width: 0%; background: var(--accent-green); transition: width 0.05s linear; }

    /* Content Views */
    .view-panel { padding: 24px; display: none; }
    .view-panel.active { display: block; }
    .grid-cards { display: grid; grid-template-columns: repeat(auto-fill, minmax(220px, 1fr)); gap: 16px; margin-top: 16px; }

    .card {
      background: var(--bg-card);
      border: 1px solid var(--border);
      border-radius: 8px;
      padding: 16px;
      display: flex;
      flex-direction: column;
      justify-content: space-between;
      gap: 12px;
      transition: transform 0.15s, border-color 0.15s;
    }
    .card:hover { transform: translateY(-2px); border-color: var(--accent-blue); }
    .card-header { display: flex; justify-content: space-between; align-items: center; }
    .card-title { font-weight: bold; font-size: 1rem; }
    .card-req { font-size: 0.75rem; color: var(--accent-red); }
    .card-req.met { color: var(--accent-green); }

    .btn {
      background: var(--accent-blue);
      color: white;
      border: none;
      padding: 8px 12px;
      border-radius: 6px;
      cursor: pointer;
      font-weight: 600;
      font-size: 0.85rem;
      transition: opacity 0.2s;
    }
    .btn:hover:not(:disabled) { opacity: 0.9; }
    .btn:disabled { background: #334155; color: #64748b; cursor: not-allowed; }
    .btn-danger { background: var(--accent-red); }

    /* Inventory Grid */
    .inv-grid {
      display: grid;
      grid-template-columns: repeat(auto-fill, minmax(64px, 1fr));
      gap: 8px;
      background: var(--bg-card);
      padding: 16px;
      border-radius: 8px;
      border: 1px solid var(--border);
      max-height: 480px;
      overflow-y: auto;
    }
    .inv-slot {
      aspect-ratio: 1;
      background: var(--bg-panel);
      border: 1px solid var(--border);
      border-radius: 6px;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      position: relative;
      cursor: pointer;
      font-size: 1.5rem;
    }
    .inv-slot:hover { border-color: var(--accent-gold); background: var(--bg-hover); }
    .inv-count {
      position: absolute;
      bottom: 2px;
      right: 4px;
      font-size: 0.7rem;
      font-weight: bold;
      color: #fff;
      text-shadow: 1px 1px 2px #000;
      font-family: var(--font-pixel);
    }

    /* Combat View */
    .combat-arena {
      display: grid;
      grid-template-columns: 1fr auto 1fr;
      gap: 20px;
      align-items: center;
      background: var(--bg-card);
      border: 1px solid var(--border);
      border-radius: 12px;
      padding: 24px;
      margin-top: 16px;
    }
    .fighter-box { text-align: center; display: flex; flex-direction: column; gap: 8px; }
    .fighter-sprite { font-size: 4rem; margin-bottom: 8px; }
    .health-bar-bg { width: 100%; height: 14px; background: #0f1118; border-radius: 7px; overflow: hidden; }
    .health-fill { height: 100%; width: 100%; background: var(--accent-red); transition: width 0.15s ease-out; }
    .combat-log {
      height: 140px;
      background: #0f1118;
      border: 1px solid var(--border);
      border-radius: 8px;
      padding: 12px;
      overflow-y: auto;
      font-family: var(--font-pixel);
      font-size: 0.8rem;
      color: var(--text-muted);
      margin-top: 16px;
      display: flex;
      flex-direction: column;
      gap: 4px;
    }
    .log-hit-player { color: #f87171; }
    .log-hit-enemy { color: #4ade80; }
    .log-loot { color: var(--accent-gold); font-weight: bold; }

    /* Equipment Screen */
    .equipment-view {
      display: grid;
      grid-template-columns: 240px 1fr;
      gap: 20px;
    }
    .gear-grid {
      display: grid;
      grid-template-columns: repeat(3, 1fr);
      gap: 10px;
      background: var(--bg-card);
      padding: 16px;
      border-radius: 8px;
      border: 1px solid var(--border);
    }
    .gear-slot {
      aspect-ratio: 1;
      background: var(--bg-panel);
      border: 1px dashed var(--border);
      border-radius: 6px;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      cursor: pointer;
      font-size: 1.4rem;
    }
    .gear-slot.filled { border-style: solid; border-color: var(--accent-blue); }

    /* Notification Modal */
    #offline-modal {
      position: fixed;
      inset: 0;
      background: rgba(0,0,0,0.7);
      display: none;
      align-items: center;
      justify-content: center;
      z-index: 100;
    }
    .modal-box {
      background: var(--bg-panel);
      border: 1px solid var(--accent-gold);
      border-radius: 12px;
      padding: 24px;
      max-width: 440px;
      width: 90%;
      text-align: center;
      display: flex;
      flex-direction: column;
      gap: 14px;
    }

    /* Responsive */
    @media (max-width: 768px) {
      body { flex-direction: column; }
      #sidebar { width: 100%; height: auto; max-height: 200px; }
      .equipment-view { grid-template-columns: 1fr; }
    }
  </style>
</head>
<body>

  <div id="sidebar">
    <div class="brand">
      <h1>⚔️ REALM OF MASTERY</h1>
      <div class="gold-counter">🪙 <span id="gold-display">0</span></div>
    </div>
    
    <div class="nav-tabs" id="nav-container">
      <button class="nav-btn active" onclick="switchTab('woodcutting')">🪓 Woodcutting <span class="skill-lvl" id="lvl-woodcutting">Lv 1</span></button>
      <button class="nav-btn" onclick="switchTab('mining')">⛏️ Mining <span class="skill-lvl" id="lvl-mining">Lv 1</span></button>
      <button class="nav-btn" onclick="switchTab('fishing')">🎣 Fishing <span class="skill-lvl" id="lvl-fishing">Lv 1</span></button>
      <button class="nav-btn" onclick="switchTab('smelting')">🔥 Smelting & Smithing <span class="skill-lvl" id="lvl-smithing">Lv 1</span></button>
      <button class="nav-btn" onclick="switchTab('cooking')">🍳 Cooking <span class="skill-lvl" id="lvl-cooking">Lv 1</span></button>
      <button class="nav-btn" onclick="switchTab('combat')">⚔️ Combat Dungeon <span class="skill-lvl" id="lvl-combat">Lv 3</span></button>
      <button class="nav-btn" onclick="switchTab('inventory')">🎒 Inventory & Gear</button>
      <button class="nav-btn" onclick="switchTab('settings')">⚙️ Settings & Save</button>
    </div>

    <div id="current-action-bar">
      <div class="action-info">
        <span id="active-action-name" style="font-weight: 600;">Idle</span>
        <span id="active-action-timer" style="color: var(--text-muted);">0.0s</span>
      </div>
      <div class="progress-track">
        <div class="progress-fill" id="action-progress"></div>
      </div>
    </div>
  </div>

  <div id="main-content">
    
    <div id="view-woodcutting" class="view-panel active">
      <h2>🪓 Woodcutting</h2>
      <p style="color:var(--text-muted); font-size:0.9rem;">Chop trees to gather wood for fuel, construction, and upgrades.</p>
      <div class="grid-cards" id="wc-grid"></div>
    </div>

    <div id="view-mining" class="view-panel">
      <h2>⛏️ Mining</h2>
      <p style="color:var(--text-muted); font-size:0.9rem;">Extract ores and gems from the deep earth to smelt into metal bars.</p>
      <div class="grid-cards" id="mining-grid"></div>
    </div>

    <div id="view-fishing" class="view-panel">
      <h2>🎣 Fishing</h2>
      <p style="color:var(--text-muted); font-size:0.9rem;">Catch raw fish from rivers and oceans to sustain your combat adventures.</p>
      <div class="grid-cards" id="fishing-grid"></div>
    </div>

    <div id="view-smelting" class="view-panel">
      <h2>🔥 Smelting & Forging</h2>
      <p style="color:var(--text-muted); font-size:0.9rem;">Refine raw ore into bars and forge weapons & armor.</p>
      <div class="grid-cards" id="smithing-grid"></div>
    </div>

    <div id="view-cooking" class="view-panel">
      <h2>🍳 Cooking Fire</h2>
      <p style="color:var(--text-muted); font-size:0.9rem;">Cook raw meat and fish to restore HP in combat.</p>
      <div class="grid-cards" id="cooking-grid"></div>
    </div>

    <div id="view-combat" class="view-panel">
      <h2>⚔️ Dungeon Exploration</h2>
      <p style="color:var(--text-muted); font-size:0.9rem;">Fight monsters for rare items, gems, and combat mastery.</p>
      
      <div class="combat-arena">
        <div class="fighter-box">
          <div class="fighter-sprite">🧙‍♂️</div>
          <h3 id="player-name">Player</h3>
          <div class="health-bar-bg"><div id="player-hp-bar" class="health-fill"></div></div>
          <div style="font-size:0.85rem;" id="player-hp-text">100 / 100 HP</div>
        </div>

        <div style="text-align:center; font-weight:800; color:var(--accent-red); font-size:1.4rem;">VS</div>

        <div class="fighter-box">
          <div class="fighter-sprite" id="enemy-sprite">👾</div>
          <h3 id="enemy-name">None Selected</h3>
          <div class="health-bar-bg"><div id="enemy-hp-bar" class="health-fill"></div></div>
          <div style="font-size:0.85rem;" id="enemy-hp-text">0 / 0 HP</div>
        </div>
      </div>

      <div class="combat-log" id="combat-log">
        <div>Welcome to the Combat Dungeon. Select a monster below to attack.</div>
      </div>

      <h3 style="margin-top:20px;">Dungeon Enemies</h3>
      <div class="grid-cards" id="enemy-grid"></div>
    </div>

    <div id="view-inventory" class="view-panel">
      <h2>🎒 Inventory & Equipment</h2>
      <div class="equipment-view" style="margin-top: 16px;">
        
        <div>
          <h3>Equipped Gear</h3>
          <div class="gear-grid" style="margin-top:8px;">
            <div class="gear-slot" id="slot-helm" title="Helmet" onclick="unequipSlot('helm')">🪖</div>
            <div class="gear-slot" id="slot-amulet" title="Amulet" onclick="unequipSlot('amulet')">📿</div>
            <div class="gear-slot" id="slot-weapon" title="Main-Hand" onclick="unequipSlot('weapon')">⚔️</div>
            <div class="gear-slot" id="slot-chest" title="Chestplate" onclick="unequipSlot('chest')">🦺</div>
            <div class="gear-slot" id="slot-shield" title="Shield" onclick="unequipSlot('shield')">🛡️</div>
            <div class="gear-slot" id="slot-legs" title="Leggings" onclick="unequipSlot('legs')">👖</div>
            <div class="gear-slot" id="slot-boots" title="Boots" onclick="unequipSlot('boots')">🥾</div>
            <div class="gear-slot" id="slot-ring" title="Ring" onclick="unequipSlot('ring')">💍</div>
          </div>
          
          <div style="background:var(--bg-card); padding:14px; border-radius:8px; border:1px solid var(--border); margin-top:12px; font-size:0.85rem;">
            <div><strong>Total Attack:</strong> +<span id="stat-att">0</span></div>
            <div><strong>Total Strength:</strong> +<span id="stat-str">0</span></div>
            <div><strong>Total Armor:</strong> +<span id="stat-def">0</span></div>
          </div>
        </div>

        <div>
          <div style="display:flex; justify-content:space-between; align-items:center; margin-bottom:8px;">
            <h3>Bank Storage (<span id="inv-capacity">0</span> items)</h3>
            <button class="btn btn-danger" onclick="sellAllJunk()">Sell Extra Resources</button>
          </div>
          <div class="inv-grid" id="inventory-container"></div>
          <div id="item-details" style="margin-top: 12px; min-height: 40px; font-size:0.9rem; color:var(--accent-gold);"></div>
        </div>
      </div>
    </div>

    <div id="view-settings" class="view-panel">
      <h2>⚙️ Settings & Save Game</h2>
      <div class="card" style="margin-top:16px; max-width: 480px;">
        <button class="btn" onclick="saveGame()">💾 Save Game Now</button>
        <button class="btn" onclick="exportSave()">📤 Export Save Code</button>
        <button class="btn" onclick="importSave()">📥 Import Save Code</button>
        <button class="btn btn-danger" onclick="hardReset()">⚠️ Wipe & Reset Profile</button>
        <button class="btn" onclick="toggleAudio()" id="audio-toggle-btn">🔊 Audio: ON</button>
      </div>
    </div>

  </div>

  <div id="offline-modal">
    <div class="modal-box">
      <h2 style="color:var(--accent-gold);">🌙 Welcome Back!</h2>
      <p id="offline-summary">While you were resting, your master continued training...</p>
      <button class="btn" onclick="document.getElementById('offline-modal').style.display='none'">Claim Gains</button>
    </div>
  </div>

  <script>
    /* =========================================================================
       AUDIO SYNTHESIZER (Web Audio API - Zero External Files)
       ========================================================================= */
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
          osc.frequency.exponentialRampToValueAtTime(600, t + 0.12);
          gain.gain.setValueAtTime(0.2, t);
          gain.gain.linearRampToValueAtTime(0.01, t + 0.12);
          osc.start(t); osc.stop(t + 0.12);
        } else if (type === 'hit') {
          osc.type = 'sawtooth';
          osc.frequency.setValueAtTime(220, t);
          osc.frequency.linearRampToValueAtTime(80, t + 0.1);
          gain.gain.setValueAtTime(0.25, t);
          gain.gain.linearRampToValueAtTime(0.01, t + 0.1);
          osc.start(t); osc.stop(t + 0.1);
        } else if (type === 'levelup') {
          osc.type = 'square';
          osc.frequency.setValueAtTime(330, t);
          osc.frequency.setValueAtTime(440, t + 0.1);
          osc.frequency.setValueAtTime(554.37, t + 0.2);
          osc.frequency.setValueAtTime(659.25, t + 0.3);
          gain.gain.setValueAtTime(0.2, t);
          gain.gain.linearRampToValueAtTime(0.01, t + 0.5);
          osc.start(t); osc.stop(t + 0.5);
        }
      }
    };

    function toggleAudio() {
      AudioEngine.enabled = !AudioEngine.enabled;
      document.getElementById('audio-toggle-btn').innerText = AudioEngine.enabled ? '🔊 Audio: ON' : '🔇 Audio: OFF';
    }

    /* =========================================================================
       GAME DATABASE (Items, Monsters, Recipes)
       ========================================================================= */
    const ITEMS = {
      // Wood
      normal_logs: { id: 'normal_logs', name: 'Normal Logs', icon: '🪵', value: 4, type: 'resource' },
      oak_logs: { id: 'oak_logs', name: 'Oak Logs', icon: '🪵', value: 12, type: 'resource' },
      willow_logs: { id: 'willow_logs', name: 'Willow Logs', icon: '🪵', value: 28, type: 'resource' },
      yew_logs: { id: 'yew_logs', name: 'Yew Logs', icon: '🪵', value: 80, type: 'resource' },
      // Ores
      copper_ore: { id: 'copper_ore', name: 'Copper Ore', icon: '🪨', value: 3, type: 'resource' },
      tin_ore: { id: 'tin_ore', name: 'Tin Ore', icon: '🪨', value: 3, type: 'resource' },
      iron_ore: { id: 'iron_ore', name: 'Iron Ore', icon: '🪨', value: 10, type: 'resource' },
      coal: { id: 'coal', name: 'Coal', icon: '⬛', value: 15, type: 'resource' },
      mithril_ore: { id: 'mithril_ore', name: 'Mithril Ore', icon: '💎', value: 45, type: 'resource' },
      // Bars
      bronze_bar: { id: 'bronze_bar', name: 'Bronze Bar', icon: '🧱', value: 12, type: 'resource' },
      iron_bar: { id: 'iron_bar', name: 'Iron Bar', icon: '🧱', value: 28, type: 'resource' },
      steel_bar: { id: 'steel_bar', name: 'Steel Bar', icon: '🧱', value: 65, type: 'resource' },
      mithril_bar: { id: 'mithril_bar', name: 'Mithril Bar', icon: '🧱', value: 160, type: 'resource' },
      // Raw Fish & Cooked
      raw_shrimp: { id: 'raw_shrimp', name: 'Raw Shrimp', icon: '🦐', value: 3, type: 'raw_food' },
      cooked_shrimp: { id: 'cooked_shrimp', name: 'Cooked Shrimp', icon: '🍤', value: 6, type: 'food', heal: 8 },
      raw_trout: { id: 'raw_trout', name: 'Raw Trout', icon: '🐟', value: 12, type: 'raw_food' },
      cooked_trout: { id: 'cooked_trout', name: 'Cooked Trout', icon: '🍣', value: 24, type: 'food', heal: 25 },
      raw_swordfish: { id: 'raw_swordfish', name: 'Raw Swordfish', icon: '🐬', value: 45, type: 'raw_food' },
      cooked_swordfish: { id: 'cooked_swordfish', name: 'Cooked Swordfish', icon: '🍱', value: 90, type: 'food', heal: 70 },
      // Gear / Weapons
      bronze_sword: { id: 'bronze_sword', name: 'Bronze Sword', icon: '🗡️', value: 30, type: 'weapon', slot: 'weapon', att: 4, str: 3 },
      iron_sword: { id: 'iron_sword', name: 'Iron Sword', icon: '⚔️', value: 100, type: 'weapon', slot: 'weapon', att: 9, str: 8 },
      steel_sword: { id: 'steel_sword', name: 'Steel Longsword', icon: '⚔️', value: 300, type: 'weapon', slot: 'weapon', att: 18, str: 16 },
      mithril_sword: { id: 'mithril_sword', name: 'Mithril Blade', icon: '🗡️', value: 900, type: 'weapon', slot: 'weapon', att: 32, str: 28 },
      // Armor
      bronze_chest: { id: 'bronze_chest', name: 'Bronze Platebody', icon: '🦺', value: 50, type: 'armor', slot: 'chest', def: 6 },
      iron_chest: { id: 'iron_chest', name: 'Iron Platebody', icon: '🦺', value: 180, type: 'armor', slot: 'chest', def: 14 },
      steel_chest: { id: 'steel_chest', name: 'Steel Platebody', icon: '🦺', value: 500, type: 'armor', slot: 'chest', def: 26 },
      iron_shield: { id: 'iron_shield', name: 'Iron Kite Shield', icon: '🛡️', value: 90, type: 'armor', slot: 'shield', def: 8 },
      amulet_str: { id: 'amulet_str', name: 'Amulet of Power', icon: '📿', value: 500, type: 'armor', slot: 'amulet', str: 10, att: 5 },
      ring_warrior: { id: 'ring_warrior', name: 'Berserker Ring', icon: '💍', value: 1200, type: 'armor', slot: 'ring', str: 15 }
    };

    const GATHER_NODES = {
      woodcutting: [
        { id: 'normal_tree', name: 'Normal Tree', lvl: 1, xp: 15, time: 2.0, drop: 'normal_logs' },
        { id: 'oak_tree', name: 'Oak Tree', lvl: 15, xp: 38, time: 3.2, drop: 'oak_logs' },
        { id: 'willow_tree', name: 'Willow Tree', lvl: 30, xp: 68, time: 4.5, drop: 'willow_logs' },
        { id: 'yew_tree', name: 'Yew Tree', lvl: 60, xp: 175, time: 6.5, drop: 'yew_logs' }
      ],
      mining: [
        { id: 'copper_rock', name: 'Copper Rock', lvl: 1, xp: 18, time: 2.2, drop: 'copper_ore' },
        { id: 'tin_rock', name: 'Tin Rock', lvl: 1, xp: 18, time: 2.2, drop: 'tin_ore' },
        { id: 'iron_rock', name: 'Iron Rock', lvl: 15, xp: 35, time: 3.5, drop: 'iron_ore' },
        { id: 'coal_rock', name: 'Coal Deposit', lvl: 30, xp: 55, time: 4.0, drop: 'coal' },
        { id: 'mithril_rock', name: 'Mithril Vein', lvl: 55, xp: 140, time: 6.0, drop: 'mithril_ore' }
      ],
      fishing: [
        { id: 'shrimp_spot', name: 'Small Pond (Shrimp)', lvl: 1, xp: 12, time: 2.0, drop: 'raw_shrimp' },
        { id: 'trout_spot', name: 'River (Trout)', lvl: 20, xp: 45, time: 3.4, drop: 'raw_trout' },
        { id: 'swordfish_spot', name: 'Ocean (Swordfish)', lvl: 50, xp: 110, time: 5.5, drop: 'raw_swordfish' }
      ]
    };

    const RECIPES = {
      smithing: [
        { id: 'bronze_bar', name: 'Smelt Bronze Bar', lvl: 1, xp: 16, time: 2.0, req: { copper_ore: 1, tin_ore: 1 }, result: 'bronze_bar' },
        { id: 'iron_bar', name: 'Smelt Iron Bar', lvl: 15, xp: 28, time: 2.8, req: { iron_ore: 1 }, result: 'iron_bar' },
        { id: 'steel_bar', name: 'Smelt Steel Bar', lvl: 30, xp: 52, time: 3.5, req: { iron_ore: 1, coal: 2 }, result: 'steel_bar' },
        { id: 'mithril_bar', name: 'Smelt Mithril Bar', lvl: 50, xp: 120, time: 4.5, req: { mithril_ore: 1, coal: 4 }, result: 'mithril_bar' },
        { id: 'bronze_sword', name: 'Forge Bronze Sword', lvl: 3, xp: 30, time: 3.0, req: { bronze_bar: 2 }, result: 'bronze_sword' },
        { id: 'bronze_chest', name: 'Forge Bronze Plate', lvl: 8, xp: 60, time: 4.0, req: { bronze_bar: 4 }, result: 'bronze_chest' },
        { id: 'iron_sword', name: 'Forge Iron Sword', lvl: 18, xp: 75, time: 3.5, req: { iron_bar: 3 }, result: 'iron_sword' },
        { id: 'iron_shield', name: 'Forge Iron Shield', lvl: 22, xp: 90, time: 4.0, req: { iron_bar: 3 }, result: 'iron_shield' },
        { id: 'iron_chest', name: 'Forge Iron Plate', lvl: 26, xp: 140, time: 5.0, req: { iron_bar: 5 }, result: 'iron_chest' },
        { id: 'steel_sword', name: 'Forge Steel Sword', lvl: 35, xp: 220, time: 4.5, req: { steel_bar: 4 }, result: 'steel_sword' },
        { id: 'steel_chest', name: 'Forge Steel Plate', lvl: 42, xp: 360, time: 6.0, req: { steel_bar: 6 }, result: 'steel_chest' },
        { id: 'mithril_sword', name: 'Forge Mithril Blade', lvl: 55, xp: 550, time: 6.0, req: { mithril_bar: 5 }, result: 'mithril_sword' }
      ],
      cooking: [
        { id: 'cooked_shrimp', name: 'Cook Shrimp', lvl: 1, xp: 12, time: 1.8, req: { raw_shrimp: 1 }, result: 'cooked_shrimp' },
        { id: 'cooked_trout', name: 'Cook Trout', lvl: 20, xp: 38, time: 2.6, req: { raw_trout: 1 }, result: 'cooked_trout' },
        { id: 'cooked_swordfish', name: 'Cook Swordfish', lvl: 50, xp: 100, time: 4.0, req: { raw_swordfish: 1 }, result: 'cooked_swordfish' }
      ]
    };

    const ENEMIES = [
      { id: 'goblin', name: 'Goblin Scout', lvl: 3, hp: 35, maxHp: 35, att: 4, def: 2, speed: 2.4, sprite: '👺', drops: [{ id: 'bronze_sword', chance: 0.1 }, { id: 'copper_ore', chance: 0.5, qty: [1,3] }], gold: [3, 8] },
      { id: 'skeleton', name: 'Skeleton Warrior', lvl: 12, hp: 80, maxHp: 80, att: 12, def: 8, speed: 2.2, sprite: '💀', drops: [{ id: 'iron_ore', chance: 0.6, qty: [2,5] }, { id: 'iron_sword', chance: 0.08 }], gold: [10, 25] },
      { id: 'orc', name: 'Orc Berserker', lvl: 28, hp: 190, maxHp: 190, att: 26, def: 18, speed: 2.6, sprite: '👹', drops: [{ id: 'coal', chance: 0.7, qty: [3,8] }, { id: 'amulet_str', chance: 0.05 }], gold: [30, 75] },
      { id: 'dragon', name: 'Elder Fire Dragon', lvl: 65, hp: 650, maxHp: 650, att: 65, def: 45, speed: 3.0, sprite: '🐉', drops: [{ id: 'mithril_bar', chance: 0.8, qty: [2,6] }, { id: 'ring_warrior', chance: 0.08 }], gold: [150, 350] }
    ];

    /* =========================================================================
       GAME STATE
       ========================================================================= */
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
      combat: {
        active: false,
        enemy: null,
        playerHp: 100,
        playerTimer: 0,
        enemyTimer: 0
      },
      lastTimestamp: Date.now()
    };

    /* =========================================================================
       PROGRESSION & LEVEL FORMULAS
       ========================================================================= */
    function getLevelForXp(xp) {
      for (let lvl = 1; lvl <= 99; lvl++) {
        let req = Math.floor(50 * Math.pow(lvl, 2.1));
        if (xp < req) return lvl;
      }
      return 99;
    }

    function addXp(skillName, amount) {
      const skill = State.skills[skillName];
      if (!skill) return;
      skill.xp += amount;
      const newLvl = getLevelForXp(skill.xp);
      if (newLvl > skill.lvl) {
        skill.lvl = newLvl;
        AudioEngine.play('levelup');
        logCombat(`🎉 Level Up! Your <strong>${skillName.toUpperCase()}</strong> is now level <strong>${newLvl}</strong>!`, 'log-loot');
        updateUI();
      }
    }

    /* =========================================================================
       INVENTORY & EQUIPMENT SYSTEM
       ========================================================================= */
    function addItem(itemId, count = 1) {
      if (!ITEMS[itemId]) return;
      State.inventory[itemId] = (State.inventory[itemId] || 0) + count;
      renderInventory();
    }

    function removeItem(itemId, count = 1) {
      if (!State.inventory[itemId] || State.inventory[itemId] < count) return false;
      State.inventory[itemId] -= count;
      if (State.inventory[itemId] <= 0) delete State.inventory[itemId];
      renderInventory();
      return true;
    }

    function equipItem(itemId) {
      const item = ITEMS[itemId];
      if (!item || !item.slot) return;
      
      // Swap gear
      if (State.equipment[item.slot]) {
        addItem(State.equipment[item.slot].id, 1);
      }
      removeItem(itemId, 1);
      State.equipment[item.slot] = item;
      AudioEngine.play('craft');
      renderEquipment();
      renderInventory();
    }

    function unequipSlot(slotName) {
      const item = State.equipment[slotName];
      if (!item) return;
      addItem(item.id, 1);
      State.equipment[slotName] = null;
      AudioEngine.play('craft');
      renderEquipment();
      renderInventory();
    }

    function getPlayerStats() {
      let att = State.skills.attack.lvl;
      let str = State.skills.strength.lvl;
      let def = State.skills.defense.lvl;

      for (let slot in State.equipment) {
        const item = State.equipment[slot];
        if (item) {
          if (item.att) att += item.att;
          if (item.str) str += item.str;
          if (item.def) def += item.def;
        }
      }
      return { att, str, def, maxHp: 100 + (State.skills.hitpoints.lvl - 10) * 8 };
    }

    function eatFood(itemId) {
      const item = ITEMS[itemId];
      if (!item || item.type !== 'food') return;
      const stats = getPlayerStats();
      if (State.combat.playerHp >= stats.maxHp) return;

      State.combat.playerHp = Math.min(stats.maxHp, State.combat.playerHp + item.heal);
      removeItem(itemId, 1);
      AudioEngine.play('fish');
      updateCombatUI();
    }

    function sellAllJunk() {
      let earned = 0;
      for (let id in State.inventory) {
        const itm = ITEMS[id];
        if (itm && itm.type === 'resource') {
          const goldValue = itm.value * State.inventory[id];
          earned += goldValue;
          delete State.inventory[id];
        }
      }
      State.gold += earned;
      AudioEngine.play('craft');
      updateUI();
    }

    /* =========================================================================
       ACTION TICK ENGINE (Gathering / Crafting)
       ========================================================================= */
    function startAction(type, data) {
      if (State.combat.active) {
        stopCombat();
      }
      State.currentAction = {
        type: type,
        data: data,
        progress: 0,
        duration: data.time
      };
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
        // Check recipe requirements
        let canCraft = true;
        for (let mat in act.data.req) {
          if (!State.inventory[mat] || State.inventory[mat] < act.data.req[mat]) {
            canCraft = false;
            break;
          }
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

    /* =========================================================================
       REAL-TIME COMBAT ENGINE
       ========================================================================= */
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
      logCombat(`⚔️ Encountered a wild <strong>${template.name}</strong>!`);
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

      // Auto-eat if player is critical (< 30% HP)
      if (c.playerHp < stats.maxHp * 0.35) {
        for (let id in State.inventory) {
          if (ITEMS[id] && ITEMS[id].type === 'food') {
            eatFood(id);
            break;
          }
        }
      }

      // Player Attack Cycle
      c.playerTimer += dt;
      if (c.playerTimer >= 2.4) {
        c.playerTimer = 0;
        const hitChance = stats.att / (stats.att + c.enemy.def);
        if (Math.random() <= Math.max(0.2, hitChance)) {
          const maxHit = Math.max(2, Math.floor(stats.str * 0.8));
          const dmg = Math.floor(Math.random() * maxHit) + 1;
          c.enemy.hp = Math.max(0, c.enemy.hp - dmg);
          
          addXp('attack', dmg * 2);
          addXp('strength', dmg * 2);
          addXp('hitpoints', dmg * 1.5);
          
          logCombat(`🗡️ You struck ${c.enemy.name} for <strong class="log-hit-enemy">${dmg}</strong> damage!`);
          AudioEngine.play('hit');
        } else {
          logCombat(`💨 You swung at ${c.enemy.name} and missed!`);
        }
      }

      // Check Enemy Death
      if (c.enemy.hp <= 0) {
        logCombat(`💀 You defeated the <strong>${c.enemy.name}</strong>!`, 'log-loot');
        // Loot Drop
        const goldLoot = Math.floor(Math.random() * (c.enemy.gold[1] - c.enemy.gold[0] + 1)) + c.enemy.gold[0];
        State.gold += goldLoot;
        
        c.enemy.drops.forEach(drop => {
          if (Math.random() <= drop.chance) {
            const count = drop.qty ? Math.floor(Math.random() * (drop.qty[1] - drop.qty[0] + 1)) + drop.qty[0] : 1;
            addItem(drop.id, count);
            logCombat(`🎁 Looted: ${ITEMS[drop.id].icon} ${ITEMS[drop.id].name} x${count}`, 'log-loot');
          }
        });

        // Respawn
        const template = ENEMIES.find(e => e.id === c.enemy.id);
        c.enemy.hp = template.maxHp;
      }

      // Enemy Attack Cycle
      c.enemyTimer += dt;
      if (c.enemyTimer >= c.enemy.speed) {
        c.enemyTimer = 0;
        const enemyHitChance = c.enemy.att / (c.enemy.att + stats.def);
        if (Math.random() <= Math.max(0.15, enemyHitChance)) {
          const maxHit = Math.max(2, Math.floor(c.enemy.att * 0.6));
          const dmg = Math.floor(Math.random() * maxHit) + 1;
          c.playerHp = Math.max(0, c.playerHp - dmg);
          addXp('defense', dmg * 3);
          logCombat(`🩸 ${c.enemy.name} hits you for <strong class="log-hit-player">${dmg}</strong> damage!`);
        } else {
          logCombat(`🛡️ You blocked ${c.enemy.name}'s attack!`);
        }
      }

      // Player Death Check
      if (c.playerHp <= 0) {
        logCombat(`☠️ You were slain by ${c.enemy.name}! Respawning at town...`, 'log-hit-player');
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

    /* =========================================================================
       UI RENDERING & TABS
       ========================================================================= */
    function switchTab(tabId) {
      document.querySelectorAll('.view-panel').forEach(el => el.classList.remove('active'));
      document.querySelectorAll('.nav-btn').forEach(el => el.classList.remove('active'));
      
      const targetView = document.getElementById('view-' + tabId);
      if (targetView) targetView.classList.add('active');
      event.currentTarget.classList.add('active');
    }

    function renderCards() {
      // Gathering
      ['woodcutting', 'mining', 'fishing'].forEach(skill => {
        const container = document.getElementById(skill === 'woodcutting' ? 'wc-grid' : skill === 'mining' ? 'mining-grid' : 'fishing-grid');
        container.innerHTML = GATHER_NODES[skill].map(node => `
          <div class="card">
            <div class="card-header">
              <div class="card-title">${ITEMS[node.drop].icon} ${node.name}</div>
              <div class="card-req ${State.skills[skill].lvl >= node.lvl ? 'met' : ''}">Lv ${node.lvl}</div>
            </div>
            <div style="font-size:0.8rem; color:var(--text-muted);">Yields: ${ITEMS[node.drop].name}<br>Speed: ${node.time}s | +${node.xp} XP</div>
            <button class="btn" ${State.skills[skill].lvl < node.lvl ? 'disabled' : ''} onclick='startAction("${skill}", ${JSON.stringify(node)})'>Start Gathering</button>
          </div>
        `).join('');
      });

      // Crafting
      ['smithing', 'cooking'].forEach(skill => {
        const container = document.getElementById(skill === 'smithing' ? 'smithing-grid' : 'cooking-grid');
        container.innerHTML = RECIPES[skill].map(rec => {
          const reqText = Object.entries(rec.req).map(([m, c]) => `${c}x ${ITEMS[m].name}`).join(', ');
          return `
            <div class="card">
              <div class="card-header">
                <div class="card-title">${ITEMS[rec.result].icon} ${rec.name}</div>
                <div class="card-req ${State.skills[skill].lvl >= rec.lvl ? 'met' : ''}">Lv ${rec.lvl}</div>
              </div>
              <div style="font-size:0.8rem; color:var(--text-muted);">Requires: ${reqText}<br>Speed: ${rec.time}s | +${rec.xp} XP</div>
              <button class="btn" ${State.skills[skill].lvl < rec.lvl ? 'disabled' : ''} onclick='startAction("${skill}", ${JSON.stringify(rec)})'>Forge / Cook</button>
            </div>
          `;
        }).join('');
      });

      // Enemies
      document.getElementById('enemy-grid').innerHTML = ENEMIES.map(e => `
        <div class="card">
          <div class="card-header">
            <div class="card-title">${e.sprite} ${e.name}</div>
            <div class="card-req met">Combat Lv ${e.lvl}</div>
          </div>
          <div style="font-size:0.8rem; color:var(--text-muted);">HP: ${e.hp} | Atk: ${e.att} | Def: ${e.def}</div>
          <button class="btn" onclick="startCombat('${e.id}')">Attack Monster</button>
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
          <div class="inv-slot" onclick="handleItemClick('${id}')" title="${itm.name} (Value: ${itm.value}g)">
            ${itm.icon}
            <span class="inv-count">${count}</span>
          </div>
        `;
      }).join('');
    }

    function handleItemClick(itemId) {
      const itm = ITEMS[itemId];
      const details = document.getElementById('item-details');
      
      if (itm.type === 'food') {
        eatFood(itemId);
        details.innerHTML = `Consumed <strong>${itm.name}</strong> (+${itm.heal} HP).`;
      } else if (itm.slot) {
        equipItem(itemId);
        details.innerHTML = `Equipped <strong>${itm.name}</strong>.`;
      } else {
        details.innerHTML = `<strong>${itm.name}</strong>: ${itm.value} Gold each.`;
      }
    }

    function renderEquipment() {
      const slots = ['weapon', 'shield', 'helm', 'chest', 'legs', 'boots', 'amulet', 'ring'];
      slots.forEach(s => {
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
        document.getElementById('enemy-hp-text').innerText = `${Math.floor(State.combat.enemy.hp)} / ${State.combat.enemy.maxHp} HP`;
      } else {
        document.getElementById('enemy-name').innerText = 'No Target';
        document.getElementById('enemy-sprite').innerText = '👾';
        document.getElementById('enemy-hp-bar').style.width = '0%';
        document.getElementById('enemy-hp-text').innerText = '0 / 0 HP';
      }
    }

    function updateUI() {
      document.getElementById('gold-display').innerText = State.gold.toLocaleString();
      for (let s in State.skills) {
        const el = document.getElementById(`lvl-${s}`);
        if (el) el.innerText = `Lv ${State.skills[s].lvl}`;
      }
      const combatLvl = Math.floor((State.skills.attack.lvl + State.skills.strength.lvl + State.skills.defense.lvl + State.skills.hitpoints.lvl) / 4);
      document.getElementById('lvl-combat').innerText = `Lv ${combatLvl}`;
      renderCards();
      renderEquipment();
      renderInventory();
    }

    /* =========================================================================
       SAVE / LOAD & OFFLINE ENGINE
       ========================================================================= */
    function saveGame() {
      State.lastTimestamp = Date.now();
      localStorage.setItem('RoM_SaveData', JSON.stringify(State));
    }

    function loadGame() {
      const data = localStorage.getItem('RoM_SaveData');
      if (!data) {
        updateUI();
        return;
      }
      try {
        const parsed = JSON.parse(data);
        State = Object.assign(State, parsed);
        
        // Calculate Offline Gains
        const now = Date.now();
        const delta = (now - (State.lastTimestamp || now)) / 1000;
        if (delta > 10 && State.currentAction) {
          const cycles = Math.floor(delta / State.currentAction.duration);
          if (cycles > 0) {
            const cappedCycles = Math.min(cycles, 5000); // 12h cap
            for (let i = 0; i < cappedCycles; i++) completeAction(State.currentAction);
            document.getElementById('offline-summary').innerHTML = `While away for <strong>${Math.floor(delta / 60)} minutes</strong>, you completed <strong>${cappedCycles}x</strong> actions!`;
            document.getElementById('offline-modal').style.display = 'flex';
          }
        }
      } catch (e) {
        console.error("Save load error", e);
      }
      updateUI();
    }

    function exportSave() {
      const code = btoa(JSON.stringify(State));
      prompt("Copy your export save string:", code);
    }

    function importSave() {
      const code = prompt("Paste your export string here:");
      if (code) {
        try {
          State = JSON.parse(atob(code));
          saveGame();
          location.reload();
        } catch (e) { alert("Invalid save string!"); }
      }
    }

    function hardReset() {
      if (confirm("Are you sure you want to permanently erase all mastery data?")) {
        localStorage.removeItem('RoM_SaveData');
        location.reload();
      }
    }

    /* =========================================================================
       MAIN TICK ENGINE
       ========================================================================= */
    let lastTickTime = performance.now();
    function gameLoop(now) {
      const dt = (now - lastTickTime) / 1000;
      lastTickTime = now;

      handleActionTick(dt);
      handleCombatTick(dt);

      requestAnimationFrame(gameLoop);
    }

    // Initialize
    window.addEventListener('DOMContentLoaded', () => {
      loadGame();
      setInterval(saveGame, 10000); // Autosave every 10s
      requestAnimationFrame(gameLoop);
    });
  </script>
</body>
</html>

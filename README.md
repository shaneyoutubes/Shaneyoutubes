
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

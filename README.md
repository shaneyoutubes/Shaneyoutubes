<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <title>Annelid Assault - Artillery Tactics</title>
  <!-- PeerJS for WebRTC Multiplayer -->
  <script src="https://unpkg.com/peerjs@1.5.4/dist/peerjs.min.js"></script>
  <style>
    :root {
      --panel-bg: rgba(15, 23, 42, 0.9);
      --border-color: rgba(255, 255, 255, 0.15);
      --team-red: #f43f5e;
      --team-blue: #0ea5e9;
      --accent-gold: #fbbf24;
      --font-code: 'Courier New', Courier, monospace;
    }

    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
      user-select: none;
      -webkit-tap-highlight-color: transparent;
    }

    body {
      background-color: #0b0f19;
      color: #fff;
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
      overflow: hidden;
      height: 100vh;
      width: 100vw;
      display: flex;
      flex-direction: column;
    }

    /* Top HUD */
    #hud-top {
      height: 48px;
      background: var(--panel-bg);
      border-bottom: 1px solid var(--border-color);
      display: flex;
      align-items: center;
      justify-content: space-between;
      padding: 0 12px;
      z-index: 30;
      flex-shrink: 0;
    }

    .team-badge {
      display: flex;
      align-items: center;
      gap: 6px;
      font-weight: 800;
      font-size: 0.85rem;
    }
    .red-team { color: var(--team-red); }
    .blue-team { color: var(--team-blue); }

    .center-hud {
      display: flex;
      flex-direction: column;
      align-items: center;
    }
    #turn-timer {
      font-family: var(--font-code);
      font-size: 1.3rem;
      font-weight: 900;
      color: var(--accent-gold);
    }
    #wind-indicator {
      font-size: 0.7rem;
      color: #cbd5e1;
    }

    .top-actions {
      display: flex;
      align-items: center;
      gap: 6px;
    }

    .btn-icon {
      background: #1e293b;
      border: 1px solid var(--border-color);
      color: #fff;
      padding: 4px 8px;
      border-radius: 6px;
      cursor: pointer;
      font-size: 0.75rem;
      font-weight: 600;
    }
    .btn-donate {
      background: var(--accent-gold);
      color: #000;
      font-weight: 800;
      border: none;
    }

    /* Game Viewport */
    #viewport-container {
      flex: 1;
      position: relative;
      overflow: hidden;
      background: #020617;
    }

    #game-canvas {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      display: block;
    }

    /* Power Bar */
    #power-meter-container {
      position: absolute;
      bottom: 110px;
      left: 50%;
      transform: translateX(-50%);
      width: 200px;
      background: rgba(0, 0, 0, 0.85);
      padding: 4px;
      border-radius: 8px;
      border: 1px solid rgba(255,255,255,0.3);
      display: none;
      z-index: 25;
    }
    #power-meter-fill {
      height: 10px;
      width: 0%;
      background: linear-gradient(90deg, #10b981, #f59e0b, #ef4444);
      border-radius: 4px;
    }

    /* Weapon Selection Bar */
    #weapon-bar {
      position: absolute;
      bottom: 58px;
      left: 50%;
      transform: translateX(-50%);
      display: flex;
      gap: 4px;
      background: var(--panel-bg);
      padding: 4px 8px;
      border-radius: 12px;
      border: 1px solid var(--border-color);
      z-index: 25;
      max-width: 95vw;
      overflow-x: auto;
    }
    .wpn-btn {
      background: #1e293b;
      border: 1px solid rgba(255,255,255,0.15);
      color: #fff;
      padding: 5px 8px;
      border-radius: 6px;
      font-size: 0.72rem;
      font-weight: 600;
      cursor: pointer;
      white-space: nowrap;
    }
    .wpn-btn.active {
      background: var(--accent-gold);
      border-color: #fff;
      color: #000;
    }

    /* Mobile Touch Controls */
    #touch-controls {
      position: absolute;
      bottom: 6px;
      left: 0;
      right: 0;
      height: 46px;
      display: flex;
      justify-content: space-between;
      padding: 0 10px;
      z-index: 25;
      pointer-events: none;
    }
    .touch-btn-group { display: flex; gap: 6px; pointer-events: auto; }
    .t-btn {
      width: 44px;
      height: 44px;
      background: rgba(15, 23, 42, 0.9);
      border: 1px solid var(--border-color);
      border-radius: 8px;
      color: #fff;
      font-size: 1.1rem;
      display: flex;
      align-items: center;
      justify-content: center;
      touch-action: manipulation;
    }
    .t-btn:active { background: var(--accent-gold); color: #000; }
    .t-btn-fire { width: 75px; background: var(--team-red); font-weight: bold; font-size: 0.85rem; }

    /* Modals */
    .modal-overlay {
      position: absolute;
      inset: 0;
      background: rgba(8, 11, 17, 0.94);
      display: flex;
      align-items: center;
      justify-content: center;
      z-index: 100;
    }
    .modal-card {
      background: #0f172a;
      border: 1px solid var(--border-color);
      border-radius: 14px;
      padding: 20px;
      text-align: center;
      max-width: 360px;
      width: 90%;
      display: flex;
      flex-direction: column;
      gap: 10px;
    }
    .modal-card h2 { font-size: 1.4rem; color: var(--accent-gold); font-weight: 900; }
    .btn-main {
      background: var(--team-blue);
      color: #fff;
      border: none;
      padding: 10px 14px;
      border-radius: 8px;
      font-weight: bold;
      font-size: 0.9rem;
      cursor: pointer;
    }
    .btn-danger { background: var(--team-red); }
    .select-input {
      background: #1e293b;
      border: 1px solid #475569;
      color: #fff;
      padding: 8px;
      border-radius: 6px;
      font-size: 0.85rem;
      outline: none;
    }
  </style>
</head>
<body>

  <!-- Top Status Bar -->
  <header id="hud-top">
    <div class="team-badge red-team">
      <span>🔴 <span id="red-health-summary">200 HP</span></span>
    </div>
    
    <div class="center-hud">
      <div id="turn-timer">30</div>
      <div id="wind-indicator">💨 Wind: 0</div>
    </div>

    <div class="team-badge blue-team">
      <span>🔵 <span id="blue-health-summary">200 HP</span></span>
    </div>

    <div class="top-actions">
      <button class="btn-icon btn-donate" onclick="openDonateModal()">☕ Tip</button>
      <button class="btn-icon" onclick="openLobbyModal()">⚙️ Menu</button>
    </div>
  </header>

  <!-- Viewport -->
  <div id="viewport-container">
    <canvas id="game-canvas"></canvas>

    <!-- Power Bar -->
    <div id="power-meter-container">
      <div id="power-meter-fill"></div>
    </div>

    <!-- Weapon Radial Dock -->
    <div id="weapon-bar">
      <button class="wpn-btn active" onclick="selectWeapon('bazooka')">🚀 Bazooka</button>
      <button class="wpn-btn" onclick="selectWeapon('grenade')">💣 Grenade</button>
      <button class="wpn-btn" onclick="selectWeapon('cluster')">💥 Cluster</button>
      <button class="wpn-btn" onclick="selectWeapon('shotgun')">🔫 Shotgun</button>
      <button class="wpn-btn" onclick="selectWeapon('airstrike')">✈️ Air Strike</button>
      <button class="wpn-btn" onclick="selectWeapon('dynamite')">🧨 Dynamite</button>
    </div>

    <!-- Touch Controls for Mobile -->
    <div id="touch-controls">
      <div class="touch-btn-group">
        <button class="t-btn" id="btn-left">◀</button>
        <button class="t-btn" id="btn-right">▶</button>
        <button class="t-btn" id="btn-jump">⤴</button>
      </div>
      <div class="touch-btn-group">
        <button class="t-btn" id="btn-up">▲</button>
        <button class="t-btn" id="btn-down">▼</button>
        <button class="t-btn t-btn-fire" id="btn-fire">FIRE</button>
      </div>
    </div>
  </div>

  <!-- Menu / Map Lobby -->
  <div id="lobby-modal" class="modal-overlay">
    <div class="modal-card">
      <h2>ANNELID ASSAULT</h2>
      <p style="font-size:0.8rem; color:#94a3b8;">Choose Map & Play</p>
      
      <div style="display:flex; flex-direction:column; gap:4px; text-align:left;">
        <label style="font-size:0.75rem; color:#94a3b8; font-weight:bold;">BIOME:</label>
        <select id="biome-selector" class="select-input">
          <option value="classic" selected>🌿 Emerald Highlands</option>
          <option value="volcano">🌋 Volcanic Hellscape (Lava)</option>
          <option value="moon">🌙 Lunar Outpost (Low Gravity)</option>
          <option value="toxic">☣️ Toxic Wasteland</option>
          <option value="snow">❄️ Arctic Tundra (Ice)</option>
        </select>
      </div>

      <button class="btn-main" onclick="startLocalGame()">🎮 Pass & Play (Local)</button>
      <button class="btn-main" style="background:var(--accent-gold); color:#000;" onclick="startVsBot()">🤖 Play vs Computer AI</button>
    </div>
  </div>

  <!-- Tip Modal -->
  <div id="donate-modal" class="modal-overlay" style="display:none;">
    <div class="modal-card">
      <h2 style="color:var(--accent-gold);">☕ Support Dev</h2>
      <p style="font-size:0.8rem; color:#cbd5e1;">Enjoying the game? Tips help keep the game updated!</p>
      <div style="display:grid; grid-template-columns:1fr 1fr; gap:8px; margin:8px 0;">
        <a href="https://ko-fi.com" target="_blank" style="background:#1e293b; padding:10px; border-radius:6px; color:#fff; text-decoration:none; font-weight:bold; font-size:0.8rem;">☕ Ko-fi</a>
        <a href="https://paypal.com" target="_blank" style="background:#1e293b; padding:10px; border-radius:6px; color:#fff; text-decoration:none; font-weight:bold; font-size:0.8rem;">💳 PayPal</a>
      </div>
      <button class="btn-main" onclick="closeDonateModal()">Close</button>
    </div>
  </div>

  <!-- Game Over Modal -->
  <div id="gameover-modal" class="modal-overlay" style="display:none;">
    <div class="modal-card">
      <h2 id="winner-title">TEAM RED WINS!</h2>
      <p style="color:#cbd5e1; font-size:0.8rem;" id="winner-desc">All enemy worms eliminated.</p>
      <button class="btn-main" onclick="restartMatch()">Play Again</button>
    </div>
  </div>

  <script>
    /* AUDIO SYNTHESIZER */
    const AudioEngine = {
      ctx: null,
      init() {
        if (!this.ctx) {
          const AudioContext = window.AudioContext || window.webkitAudioContext;
          this.ctx = new AudioContext();
        }
      },
      play(type) {
        try {
          this.init();
          if (this.ctx.state === 'suspended') this.ctx.resume();
          const t = this.ctx.currentTime;
          const osc = this.ctx.createOscillator();
          const gain = this.ctx.createGain();
          osc.connect(gain);
          gain.connect(this.ctx.destination);

          if (type === 'launch') {
            osc.type = 'sawtooth';
            osc.frequency.setValueAtTime(140, t);
            osc.frequency.exponentialRampToValueAtTime(500, t + 0.15);
            gain.gain.setValueAtTime(0.3, t);
            gain.gain.linearRampToValueAtTime(0.01, t + 0.15);
            osc.start(t); osc.stop(t + 0.15);
          } else if (type === 'explode') {
            osc.type = 'triangle';
            osc.frequency.setValueAtTime(120, t);
            osc.frequency.linearRampToValueAtTime(30, t + 0.3);
            gain.gain.setValueAtTime(0.6, t);
            gain.gain.linearRampToValueAtTime(0.01, t + 0.3);
            osc.start(t); osc.stop(t + 0.3);
          } else if (type === 'jump') {
            osc.type = 'sine';
            osc.frequency.setValueAtTime(180, t);
            osc.frequency.exponentialRampToValueAtTime(360, t + 0.1);
            gain.gain.setValueAtTime(0.2, t);
            gain.gain.linearRampToValueAtTime(0.01, t + 0.1);
            osc.start(t); osc.stop(t + 0.1);
          }
        } catch(e) {}
      }
    };

    /* MAP BIOMES */
    const BIOMES = {
      classic: { name: 'Emerald Highlands', gravity: 0.28, soil: '#573318', grass: '#22c55e', water: 'rgba(14, 165, 233, 0.65)' },
      volcano: { name: 'Volcanic Hellscape', gravity: 0.30, soil: '#261c1a', grass: '#ea580c', water: 'rgba(239, 68, 68, 0.85)' },
      moon: { name: 'Lunar Outpost', gravity: 0.14, soil: '#334155', grass: '#94a3b8', water: 'rgba(168, 85, 247, 0.5)' },
      toxic: { name: 'Toxic Wasteland', gravity: 0.28, soil: '#1c1917', grass: '#84cc16', water: 'rgba(132, 204, 22, 0.85)' },
      snow: { name: 'Arctic Tundra', gravity: 0.28, soil: '#475569', grass: '#f8fafc', water: 'rgba(56, 189, 248, 0.7)' }
    };

    let currentBiome = BIOMES.classic;
    const WORLD_WIDTH = 2000;
    const WORLD_HEIGHT = 1000;
    const WATER_LEVEL = 920;

    const gameCanvas = document.getElementById('game-canvas');
    const gameCtx = gameCanvas.getContext('2d');

    let viewWidth = window.innerWidth;
    let viewHeight = window.innerHeight;
    let cameraX = 0, cameraY = 0;

    /* ZERO-CRASH HEIGHTMAP TERRAIN */
    let terrainHeights = new Float32Array(WORLD_WIDTH);

    function generateTerrain(biomeKey = 'classic') {
      currentBiome = BIOMES[biomeKey] || BIOMES.classic;
      const seed = Math.random() * 10;
      
      for (let x = 0; x < WORLD_WIDTH; x++) {
        const nx = x / WORLD_WIDTH;
        let h = Math.sin(nx * 5.5 + seed) * 140 +
                Math.sin(nx * 12 + seed * 2) * 70 +
                Math.sin(nx * 24) * 30 + 580;
        
        if (x < 140) h += (140 - x) * 3.5;
        if (x > WORLD_WIDTH - 140) h += (x - (WORLD_WIDTH - 140)) * 3.5;
        terrainHeights[x] = Math.min(WATER_LEVEL - 50, h);
      }
    }

    function isTerrainSolid(x, y) {
      const ix = Math.floor(x);
      if (ix < 0 || ix >= WORLD_WIDTH) return false;
      return y >= terrainHeights[ix];
    }

    function destroyTerrainCircle(cx, cy, radius) {
      const minX = Math.max(0, Math.floor(cx - radius));
      const maxX = Math.min(WORLD_WIDTH - 1, Math.ceil(cx + radius));

      for (let x = minX; x <= maxX; x++) {
        const dx = x - cx;
        const dyDist = Math.sqrt(Math.max(0, radius * radius - dx * dx));
        const craterBottom = cy + dyDist;
        if (terrainHeights[x] < craterBottom) {
          terrainHeights[x] = Math.min(WATER_LEVEL + 50, Math.max(terrainHeights[x], craterBottom));
        }
      }
    }

    /* WORM CLASS */
    class Worm {
      constructor(id, name, team, x) {
        this.id = id; this.name = name; this.team = team;
        this.x = x; this.y = 100;
        this.vx = 0; this.vy = 0;
        this.hp = 100; this.maxHp = 100;
        this.radius = 8;
        this.facingRight = team === 'red';
        this.aimAngle = team === 'red' ? -0.5 : -2.6;
        this.isGrounded = false;
        this.isDead = false;
      }

      update() {
        if (this.isDead) return;

        this.vy += currentBiome.gravity;
        this.vx *= 0.85;

        this.x += this.vx;
        this.y += this.vy;

        // Water death
        if (this.y >= WATER_LEVEL) {
          this.hp = 0;
          this.isDead = true;
          AudioEngine.play('explode');
          addFloatingText(this.x, this.y - 15, 'DROWNED!', '#f43f5e');
          return;
        }

        // Terrain Collision
        const groundY = terrainHeights[Math.floor(this.x)] || WATER_LEVEL;
        if (this.y + this.radius >= groundY) {
          this.y = groundY - this.radius;
          this.vy = 0;
          this.isGrounded = true;
        } else {
          this.isGrounded = false;
        }

        this.x = Math.max(this.radius, Math.min(WORLD_WIDTH - this.radius, this.x));

        if (this.hp <= 0 && !this.isDead) {
          this.isDead = true;
          createExplosion(this.x, this.y, 25, 0);
        }
      }

      walk(dir) {
        if (!this.isGrounded || (turnPhase !== 'ACTION' && turnPhase !== 'RETREAT')) return;
        this.vx = dir * 1.6;
        this.facingRight = dir > 0;
      }

      jump() {
        if (!this.isGrounded || (turnPhase !== 'ACTION' && turnPhase !== 'RETREAT')) return;
        this.vy = -5.8;
        this.vx = (this.facingRight ? 1 : -1) * 2.5;
        this.isGrounded = false;
        AudioEngine.play('jump');
      }

      draw(ctx) {
        if (this.isDead) return;

        ctx.save();
        ctx.translate(this.x, this.y);

        // Name & Health
        ctx.font = 'bold 10px sans-serif';
        ctx.textAlign = 'center';
        ctx.fillStyle = '#fff';
        ctx.fillText(this.name, 0, -20);

        ctx.fillStyle = 'rgba(0,0,0,0.7)';
        ctx.fillRect(-15, -16, 30, 4);
        ctx.fillStyle = this.team === 'red' ? '#f43f5e' : '#0ea5e9';
        ctx.fillRect(-15, -16, 30 * (this.hp / this.maxHp), 4);

        // Turn Pointer Arrow
        if (this === activeWorm && (turnPhase === 'ACTION' || turnPhase === 'RETREAT')) {
          const bob = Math.sin(Date.now() * 0.008) * 4;
          ctx.fillStyle = this.team === 'red' ? '#f43f5e' : '#0ea5e9';
          ctx.beginPath();
          ctx.moveTo(0, -26 + bob);
          ctx.lineTo(-5, -34 + bob);
          ctx.lineTo(5, -34 + bob);
          ctx.fill();
        }

        // Worm Body
        ctx.fillStyle = this.team === 'red' ? '#fda4af' : '#93c5fd';
        ctx.beginPath();
        ctx.arc(0, 0, 8, 0, Math.PI * 2);
        ctx.arc(this.facingRight ? -4 : 4, 3, 6, 0, Math.PI * 2);
        ctx.fill();

        // Team Headband
        ctx.fillStyle = this.team === 'red' ? '#f43f5e' : '#0ea5e9';
        ctx.fillRect(-6, -7, 12, 3);

        // Aim Line
        if (this === activeWorm && turnPhase === 'ACTION' && selectedWeapon !== 'airstrike') {
          const aimX = Math.cos(this.aimAngle) * 22;
          const aimY = Math.sin(this.aimAngle) * 22;
          ctx.strokeStyle = '#fbbf24';
          ctx.lineWidth = 2;
          ctx.beginPath(); ctx.moveTo(0, 0); ctx.lineTo(aimX, aimY); ctx.stroke();
        }

        ctx.restore();
      }
    }

    /* PROJECTILE */
    class Projectile {
      constructor(type, x, y, vx, vy) {
        this.type = type;
        this.x = x; this.y = y; this.vx = vx; this.vy = vy;
        this.timer = type === 'grenade' ? 3.0 : type === 'dynamite' ? 3.5 : 99;
        this.alive = true;
      }

      update(dt) {
        if (!this.alive) return;
        this.vx += (this.type === 'bazooka' ? wind * 0.015 : 0);
        this.vy += currentBiome.gravity * 0.9;

        if (this.timer < 90) {
          this.timer -= dt;
          if (this.timer <= 0) { this.detonate(); return; }
        }

        this.x += this.vx;
        this.y += this.vy;

        if (this.y >= WATER_LEVEL) {
          this.alive = false;
          AudioEngine.play('explode');
          return;
        }

        if (isTerrainSolid(this.x, this.y)) {
          if (this.type === 'grenade' || this.type === 'dynamite') {
            this.vx *= -0.5; this.vy *= -0.4;
            this.y = terrainHeights[Math.floor(this.x)] - 2;
          } else {
            this.detonate();
          }
        }
      }

      detonate() {
        this.alive = false;
        if (this.type === 'bazooka') createExplosion(this.x, this.y, 45, 50);
        else if (this.type === 'grenade') createExplosion(this.x, this.y, 55, 60);
        else if (this.type === 'dynamite') createExplosion(this.x, this.y, 75, 85);
        else if (this.type === 'shotgun') createExplosion(this.x, this.y, 20, 30);
        else if (this.type === 'cluster') {
          createExplosion(this.x, this.y, 35, 30);
          for (let i = 0; i < 4; i++) {
            const ang = -Math.PI * 0.8 + i * 0.5;
            projectiles.push(new Projectile('shotgun', this.x, this.y - 4, Math.cos(ang)*4, Math.sin(ang)*4));
          }
        }
      }

      draw(ctx) {
        if (!this.alive) return;
        ctx.save();
        ctx.translate(this.x, this.y);
        ctx.fillStyle = this.type === 'bazooka' ? '#ef4444' : '#fbbf24';
        ctx.beginPath(); ctx.arc(0, 0, 4, 0, Math.PI * 2); ctx.fill();
        ctx.restore();
      }
    }

    /* GAME ENGINE & PARTICLES */
    let worms = [], projectiles = [], floatingTexts = [];
    let currentTeam = 'red', activeWorm = null, activeWormIndex = { red: 0, blue: 0 };
    let turnTimer = 30, turnPhase = 'ACTION', wind = 0, selectedWeapon = 'bazooka';
    let isVsBot = false;

    let isChargingPower = false, currentPower = 0, chargeDirection = 1;
    const keys = { left: false, right: false, up: false, down: false, fire: false };

    function initSquads() {
      worms = [
        new Worm('r1', 'Bogdan', 'red', 400),
        new Worm('r2', 'Viper', 'red', 700),
        new Worm('b1', 'Glitch', 'blue', WORLD_WIDTH - 700),
        new Worm('b2', 'Shadow', 'blue', WORLD_WIDTH - 400)
      ];
      activeWormIndex = { red: 0, blue: 0 };
      currentTeam = 'red';
      pickNextActiveWorm();
    }

    function pickNextActiveWorm() {
      const teamWorms = worms.filter(w => w.team === currentTeam && !w.isDead);
      if (teamWorms.length === 0) return;
      activeWormIndex[currentTeam] = activeWormIndex[currentTeam] % teamWorms.length;
      activeWorm = teamWorms[activeWormIndex[currentTeam]];
      activeWormIndex[currentTeam] = (activeWormIndex[currentTeam] + 1) % teamWorms.length;

      cameraX = activeWorm.x - viewWidth / 2;
      cameraY = activeWorm.y - viewHeight / 2;

      // Bot AI Turn
      if (isVsBot && currentTeam === 'blue') {
        setTimeout(runBotAI, 1200);
      }
    }

    function runBotAI() {
      if (!activeWorm || activeWorm.isDead || turnPhase !== 'ACTION') return;
      const target = worms.find(w => w.team === 'red' && !w.isDead);
      if (!target) return;

      const dx = target.x - activeWorm.x;
      const dy = target.y - activeWorm.y;
      activeWorm.aimAngle = Math.atan2(dy, dx);
      activeWorm.facingRight = dx > 0;

      setTimeout(() => {
        const dist = Math.hypot(dx, dy);
        const power = Math.min(100, Math.max(30, (dist / 800) * 100));
        fireWeapon(power, activeWorm.aimAngle, 'bazooka');
      }, 800);
    }

    function createExplosion(x, y, radius, maxDamage) {
      AudioEngine.play('explode');
      destroyTerrainCircle(x, y, radius);

      worms.forEach(w => {
        if (w.isDead) return;
        const dist = Math.hypot(w.x - x, w.y - y);
        if (dist < radius + 15) {
          const factor = Math.max(0, 1 - (dist / (radius + 15)));
          const dmg = Math.floor(factor * maxDamage);
          if (dmg > 0) {
            w.hp = Math.max(0, w.hp - dmg);
            addFloatingText(w.x, w.y - 15, `-${dmg}`, '#ef4444');
          }
          const angle = Math.atan2(w.y - y, w.x - x);
          w.vx += Math.cos(angle) * factor * 10;
          w.vy += Math.sin(angle) * factor * 10 - 2;
          w.isGrounded = false;
        }
      });
    }

    function addFloatingText(x, y, text, color) { floatingTexts.push({ x, y, text, color, life: 35 }); }

    function startLocalGame() {
      isVsBot = false;
      document.getElementById('lobby-modal').style.display = 'none';
      startMatch();
    }

    function startVsBot() {
      isVsBot = true;
      document.getElementById('lobby-modal').style.display = 'none';
      startMatch();
    }

    function startMatch() {
      const biomeKey = document.getElementById('biome-selector').value;
      generateTerrain(biomeKey);
      initSquads();
      turnPhase = 'ACTION';
      turnTimer = 30;
    }

    function nextTurn() {
      const redAlive = worms.some(w => w.team === 'red' && !w.isDead);
      const blueAlive = worms.some(w => w.team === 'blue' && !w.isDead);

      if (!redAlive || !blueAlive) {
        turnPhase = 'GAME_OVER';
        const winTeam = redAlive ? 'RED' : 'BLUE';
        document.getElementById('winner-title').innerText = `TEAM ${winTeam} WINS!`;
        document.getElementById('gameover-modal').style.display = 'flex';
        return;
      }

      currentTeam = currentTeam === 'red' ? 'blue' : 'red';
      wind = Math.floor(Math.random() * 9) - 4;
      document.getElementById('wind-indicator').innerText = `💨 Wind: ${wind}`;
      turnTimer = 30;
      turnPhase = 'ACTION';
      pickNextActiveWorm();
    }

    function fireWeapon(power, aimAngle, weaponType) {
      if (!activeWorm) return;

      const p = (power / 100) * 15 + 2;
      const vx = Math.cos(aimAngle) * p;
      const vy = Math.sin(aimAngle) * p;
      const sx = activeWorm.x + Math.cos(aimAngle) * 14;
      const sy = activeWorm.y + Math.sin(aimAngle) * 14;

      projectiles.push(new Projectile(weaponType, sx, sy, vx, vy));
      AudioEngine.play('launch');
      turnPhase = 'PROJECTILE';
      document.getElementById('power-meter-container').style.display = 'none';
    }

    function selectWeapon(wpn) {
      selectedWeapon = wpn;
      document.querySelectorAll('.wpn-btn').forEach(b => b.classList.remove('active'));
      event.currentTarget.classList.add('active');
    }

    function openDonateModal() { document.getElementById('donate-modal').style.display = 'flex'; }
    function closeDonateModal() { document.getElementById('donate-modal').style.display = 'none'; }
    function openLobbyModal() { document.getElementById('lobby-modal').style.display = 'flex'; }
    function restartMatch() {
      document.getElementById('gameover-modal').style.display = 'none';
      startMatch();
    }

    /* INPUTS */
    window.addEventListener('keydown', e => {
      if (turnPhase !== 'ACTION') return;
      if (e.key === 'a' || e.key === 'ArrowLeft') keys.left = true;
      if (e.key === 'd' || e.key === 'ArrowRight') keys.right = true;
      if (e.key === 'w' || e.key === 'ArrowUp') keys.up = true;
      if (e.key === 's' || e.key === 'ArrowDown') keys.down = true;
      if (e.key === 'Shift') activeWorm?.jump();
      if (e.key === ' ' && !keys.fire) { keys.fire = true; startCharging(); }
    });

    window.addEventListener('keyup', e => {
      if (e.key === 'a' || e.key === 'ArrowLeft') keys.left = false;
      if (e.key === 'd' || e.key === 'ArrowRight') keys.right = false;
      if (e.key === 'w' || e.key === 'ArrowUp') keys.up = false;
      if (e.key === 's' || e.key === 'ArrowDown') keys.down = false;
      if (e.key === ' ' && keys.fire) { keys.fire = false; releaseCharge(); }
    });

    function bindTouch(id, onStart, onEnd) {
      const el = document.getElementById(id);
      el.addEventListener('touchstart', e => { e.preventDefault(); onStart(); });
      el.addEventListener('touchend', e => { e.preventDefault(); onEnd(); });
    }

    bindTouch('btn-left', () => keys.left = true, () => keys.left = false);
    bindTouch('btn-right', () => keys.right = true, () => keys.right = false);
    bindTouch('btn-up', () => keys.up = true, () => keys.up = false);
    bindTouch('btn-down', () => keys.down = true, () => keys.down = false);
    bindTouch('btn-jump', () => activeWorm?.jump(), () => {});
    bindTouch('btn-fire', () => startCharging(), () => releaseCharge());

    function startCharging() {
      if (turnPhase !== 'ACTION') return;
      isChargingPower = true; currentPower = 0; chargeDirection = 1;
      document.getElementById('power-meter-container').style.display = 'block';
    }

    function releaseCharge() {
      if (!isChargingPower) return;
      isChargingPower = false;
      fireWeapon(currentPower, activeWorm.aimAngle, selectedWeapon);
    }

    function resizeCanvas() {
      viewWidth = window.innerWidth;
      viewHeight = window.innerHeight;
      gameCanvas.width = viewWidth;
      gameCanvas.height = viewHeight;
    }

    /* MAIN GAME LOOP */
    let lastTime = performance.now();
    let secTimer = 0;

    function gameLoop(now) {
      const dt = Math.min(0.1, (now - lastTime) / 1000);
      lastTime = now;

      if (activeWorm && !activeWorm.isDead) {
        if (keys.left) activeWorm.walk(-1);
        if (keys.right) activeWorm.walk(1);
        if (keys.up) activeWorm.aimAngle = Math.max(-Math.PI * 0.95, activeWorm.aimAngle - 0.04);
        if (keys.down) activeWorm.aimAngle = Math.min(0.2, activeWorm.aimAngle + 0.04);
      }

      if (isChargingPower) {
        currentPower += chargeDirection * 90 * dt;
        if (currentPower >= 100) { currentPower = 100; chargeDirection = -1; }
        if (currentPower <= 0) { currentPower = 0; chargeDirection = 1; }
        document.getElementById('power-meter-fill').style.width = `${currentPower}%`;
      }

      if (turnPhase === 'ACTION' || turnPhase === 'RETREAT') {
        secTimer += dt;
        if (secTimer >= 1.0) {
          secTimer = 0;
          turnTimer--;
          document.getElementById('turn-timer').innerText = turnTimer;
          if (turnTimer <= 0) {
            turnPhase = 'RETREAT';
            nextTurn();
          }
        }
      }

      worms.forEach(w => w.update());
      projectiles.forEach(p => p.update(dt));
      projectiles = projectiles.filter(p => p.alive);

      if (turnPhase === 'PROJECTILE' && projectiles.length === 0) {
        if (!worms.some(w => Math.hypot(w.vx, w.vy) > 0.4)) {
          turnPhase = 'RETREAT';
          turnTimer = 3;
        }
      }

      // Camera Tracking
      if (projectiles.length > 0) {
        cameraX += (projectiles[0].x - viewWidth / 2 - cameraX) * 0.1;
        cameraY += (projectiles[0].y - viewHeight / 2 - cameraY) * 0.1;
      } else if (activeWorm && !activeWorm.isDead) {
        cameraX += (activeWorm.x - viewWidth / 2 - cameraX) * 0.08;
        cameraY += (activeWorm.y - viewHeight / 2 - cameraY) * 0.08;
      }

      cameraX = Math.max(0, Math.min(WORLD_WIDTH - viewWidth, cameraX));
      cameraY = Math.max(0, Math.min(WORLD_HEIGHT - viewHeight, cameraY));

      /* RENDER */
      gameCtx.clearRect(0, 0, viewWidth, viewHeight);

      // Sky Background
      gameCtx.fillStyle = '#0f172a';
      gameCtx.fillRect(0, 0, viewWidth, viewHeight);

      gameCtx.save();
      gameCtx.translate(-cameraX, -cameraY);

      // Draw Heightmap Polygon Terrain
      gameCtx.fillStyle = currentBiome.soil;
      gameCtx.beginPath();
      gameCtx.moveTo(0, WORLD_HEIGHT);
      gameCtx.lineTo(0, terrainHeights[0]);
      for (let x = 1; x < WORLD_WIDTH; x++) {
        gameCtx.lineTo(x, terrainHeights[x]);
      }
      gameCtx.lineTo(WORLD_WIDTH, WORLD_HEIGHT);
      gameCtx.closePath();
      gameCtx.fill();

      // Grass Top Line
      gameCtx.strokeStyle = currentBiome.grass;
      gameCtx.lineWidth = 6;
      gameCtx.beginPath();
      gameCtx.moveTo(0, terrainHeights[0]);
      for (let x = 1; x < WORLD_WIDTH; x++) {
        gameCtx.lineTo(x, terrainHeights[x]);
      }
      gameCtx.stroke();

      // Water Layer
      gameCtx.fillStyle = currentBiome.water;
      gameCtx.fillRect(0, WATER_LEVEL + Math.sin(now * 0.003) * 4, WORLD_WIDTH, WORLD_HEIGHT - WATER_LEVEL);

      worms.forEach(w => w.draw(gameCtx));
      projectiles.forEach(p => p.draw(gameCtx));

      floatingTexts.forEach(ft => {
        gameCtx.font = 'bold 12px sans-serif';
        gameCtx.fillStyle = ft.color;
        gameCtx.fillText(ft.text, ft.x - 10, ft.y);
        ft.y -= 0.6;
        ft.life--;
      });
      floatingTexts = floatingTexts.filter(ft => ft.life > 0);

      gameCtx.restore();

      const redHp = worms.filter(w => w.team === 'red').reduce((acc, w) => acc + w.hp, 0);
      const blueHp = worms.filter(w => w.team === 'blue').reduce((acc, w) => acc + w.hp, 0);
      document.getElementById('red-health-summary').innerText = `${redHp} HP`;
      document.getElementById('blue-health-summary').innerText = `${blueHp} HP`;

      requestAnimationFrame(gameLoop);
    }

    window.addEventListener('resize', resizeCanvas);

    window.addEventListener('DOMContentLoaded', () => {
      resizeCanvas();
      generateTerrain('classic');
      initSquads();
      requestAnimationFrame(gameLoop);
    });
  </script>
</body>
</html>

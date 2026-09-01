<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <title>CyberStrike 1v1 - Online Arena Shooter</title>
  <!-- PeerJS for WebRTC P2P Multiplayer -->
  <script src="https://unpkg.com/peerjs@1.5.4/dist/peerjs.min.js"></script>
  <style>
    :root {
      --bg: #050811;
      --panel: rgba(15, 23, 42, 0.92);
      --red: #f43f5e;
      --blue: #0ea5e9;
      --gold: #fbbf24;
      --border: rgba(255, 255, 255, 0.15);
      --font-code: 'Courier New', Courier, monospace;
    }

    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
      user-select: none;
      -webkit-tap-highlight-color: transparent;
    }

    html, body {
      width: 100%;
      height: 100%;
      overflow: hidden;
      background-color: var(--bg);
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
      color: #fff;
    }

    #game-container {
      position: relative;
      width: 100vw;
      height: 100vh;
      display: flex;
      flex-direction: column;
    }

    /* Top HUD */
    #hud-top {
      height: 54px;
      background: var(--panel);
      border-bottom: 1px solid var(--border);
      display: flex;
      align-items: center;
      justify-content: space-between;
      padding: 0 16px;
      z-index: 20;
      flex-shrink: 0;
      backdrop-filter: blur(8px);
    }

    .player-stat {
      display: flex;
      flex-direction: column;
      gap: 3px;
      min-width: 120px;
    }
    .stat-header {
      font-size: 0.8rem;
      font-weight: 800;
      display: flex;
      justify-content: space-between;
    }
    .bar-wrap {
      width: 100%;
      height: 8px;
      background: rgba(0,0,0,0.6);
      border-radius: 4px;
      overflow: hidden;
      display: flex;
    }
    .hp-fill { height: 100%; width: 100%; transition: width 0.1s linear; }
    .hp-red { background: var(--red); }
    .hp-blue { background: var(--blue); }
    .shield-fill { height: 4px; background: #38bdf8; width: 100%; border-radius: 2px; margin-top: 2px; }

    .center-hud {
      display: flex;
      flex-direction: column;
      align-items: center;
    }
    #score-board {
      font-family: var(--font-code);
      font-size: 1.4rem;
      font-weight: 900;
      color: var(--gold);
      letter-spacing: 2px;
    }
    #net-badge {
      font-size: 0.65rem;
      background: rgba(255,255,255,0.1);
      padding: 2px 8px;
      border-radius: 10px;
      color: #94a3b8;
    }

    /* Canvas */
    #canvas-wrap {
      flex: 1;
      position: relative;
      width: 100%;
      height: 100%;
    }
    canvas {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      display: block;
      cursor: crosshair;
    }

    /* Mobile On-Screen Controls */
    #touch-controls {
      position: absolute;
      bottom: 12px;
      left: 0;
      right: 0;
      height: 120px;
      display: flex;
      justify-content: space-between;
      align-items: center;
      padding: 0 20px;
      z-index: 25;
      pointer-events: none;
    }
    .joystick-zone {
      width: 110px;
      height: 110px;
      background: rgba(15, 23, 42, 0.6);
      border: 2px dashed rgba(255,255,255,0.2);
      border-radius: 50%;
      position: relative;
      pointer-events: auto;
      touch-action: none;
    }
    .joystick-knob {
      width: 44px;
      height: 44px;
      background: var(--blue);
      border-radius: 50%;
      position: absolute;
      top: 33px;
      left: 33px;
      box-shadow: 0 0 10px var(--blue);
    }
    .action-group {
      display: flex;
      flex-direction: column;
      gap: 10px;
      pointer-events: auto;
    }
    .act-btn {
      width: 58px;
      height: 58px;
      background: var(--panel);
      border: 2px solid var(--border);
      border-radius: 50%;
      color: #fff;
      font-weight: 800;
      font-size: 0.85rem;
      display: flex;
      align-items: center;
      justify-content: center;
      touch-action: manipulation;
    }
    .act-btn-fire { background: var(--red); border-color: #fff; width: 68px; height: 68px; font-size: 1rem; }
    .act-btn:active { transform: scale(0.92); }

    /* Modals */
    .modal-overlay {
      position: absolute;
      inset: 0;
      background: rgba(5, 8, 17, 0.94);
      display: flex;
      align-items: center;
      justify-content: center;
      z-index: 100;
      backdrop-filter: blur(10px);
    }
    .modal-card {
      background: #0f172a;
      border: 1px solid var(--border);
      border-radius: 16px;
      padding: 24px;
      text-align: center;
      max-width: 360px;
      width: 90%;
      display: flex;
      flex-direction: column;
      gap: 12px;
      box-shadow: 0 20px 50px rgba(0,0,0,0.8);
    }
    .modal-card h2 {
      font-size: 1.5rem;
      color: var(--gold);
      font-weight: 900;
      letter-spacing: 1px;
    }
    .btn-main {
      background: var(--blue);
      color: #fff;
      border: none;
      padding: 12px;
      border-radius: 8px;
      font-weight: 800;
      font-size: 0.95rem;
      cursor: pointer;
      display: flex;
      align-items: center;
      justify-content: center;
      gap: 8px;
      transition: opacity 0.2s;
    }
    .btn-main:hover { opacity: 0.9; }
    .btn-danger { background: var(--red); }
    .input-code {
      background: #020617;
      border: 1px solid #475569;
      color: #fff;
      padding: 10px;
      border-radius: 8px;
      font-size: 1.4rem;
      text-align: center;
      letter-spacing: 4px;
      font-family: var(--font-code);
      outline: none;
    }
    .input-code:focus { border-color: var(--gold); }
  </style>
</head>
<body>

  <div id="game-container">
    <!-- Top HUD -->
    <header id="hud-top">
      <div class="player-stat">
        <div class="stat-header" style="color:var(--red);">
          <span id="p1-name">P1 (RED)</span>
          <span id="p1-hp-text">100 HP</span>
        </div>
        <div class="bar-wrap"><div class="hp-fill hp-red" id="p1-hp-bar"></div></div>
        <div class="shield-fill" id="p1-shield-bar"></div>
      </div>

      <div class="center-hud">
        <div id="score-board">0 - 0</div>
        <div id="net-badge">OFFLINE / BOT</div>
      </div>

      <div class="player-stat">
        <div class="stat-header" style="color:var(--blue);">
          <span id="p2-name">P2 (BLUE)</span>
          <span id="p2-hp-text">100 HP</span>
        </div>
        <div class="bar-wrap"><div class="hp-fill hp-blue" id="p2-hp-bar"></div></div>
        <div class="shield-fill" id="p2-shield-bar"></div>
      </div>
    </header>

    <!-- Canvas Area -->
    <div id="canvas-wrap">
      <canvas id="game-canvas"></canvas>

      <!-- Touch Controls for Mobile -->
      <div id="touch-controls">
        <div class="joystick-zone" id="move-joystick">
          <div class="joystick-knob" id="move-knob"></div>
        </div>
        <div class="action-group">
          <button class="act-btn" id="btn-dash">⚡ DASH</button>
          <button class="act-btn act-btn-fire" id="btn-fire">🔥 FIRE</button>
        </div>
      </div>
    </div>
  </div>

  <!-- Main Menu / Matchmaking Modal -->
  <div id="lobby-modal" class="modal-overlay">
    <div class="modal-card">
      <h2>CYBERSTRIKE 1V1</h2>
      <p style="font-size:0.85rem; color:#94a3b8;">Real-Time 1v1 Cyberpunk Arena Shooter</p>

      <button class="btn-main" onclick="startVsBot()">🤖 Practice vs AI Bot</button>
      <button class="btn-main" style="background:var(--gold); color:#000;" onclick="showHostUI()">🌐 Host Online 1v1</button>
      <button class="btn-main" style="background:#8b5cf6;" onclick="showJoinUI()">🔗 Join with Room Code</button>

      <!-- Host Subpanel -->
      <div id="host-panel" style="display:none; flex-direction:column; gap:8px;">
        <div style="font-size:0.8rem; color:#94a3b8;">Share this 4-Digit Room Code:</div>
        <div id="host-code-display" style="font-size:2rem; font-weight:900; color:var(--gold); font-family:var(--font-code);">----</div>
        <div style="font-size:0.75rem; color:#cbd5e1;" id="host-status">Generating room...</div>
        <button class="btn-main btn-danger" onclick="cancelLobby()">Back</button>
      </div>

      <!-- Join Subpanel -->
      <div id="join-panel" style="display:none; flex-direction:column; gap:8px;">
        <div style="font-size:0.8rem; color:#94a3b8;">Enter Host's 4-Digit Code:</div>
        <input type="text" id="join-code-input" class="input-code" maxlength="4" placeholder="1234">
        <button class="btn-main" onclick="connectToHost()">Connect & Duel</button>
        <button class="btn-main btn-danger" onclick="cancelLobby()">Back</button>
      </div>
    </div>
  </div>

  <!-- Round Over Modal -->
  <div id="gameover-modal" class="modal-overlay" style="display:none;">
    <div class="modal-card">
      <h2 id="winner-title">RED WINS ROUND!</h2>
      <p id="winner-desc" style="font-size:0.85rem; color:#cbd5e1;">First to 3 round wins takes the match.</p>
      <button class="btn-main" onclick="nextRound()">Next Round</button>
    </div>
  </div>

  <script>
    /* =========================================================================
       1. SYNTHETIC AUDIO ENGINE (Web Audio API)
       ========================================================================= */
    const Audio = {
      ctx: null,
      init() {
        if (!this.ctx) {
          const AC = window.AudioContext || window.webkitAudioContext;
          this.ctx = new AC();
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

          if (type === 'rifle') {
            osc.type = 'sawtooth';
            osc.frequency.setValueAtTime(320, t);
            osc.frequency.exponentialRampToValueAtTime(60, t + 0.08);
            gain.gain.setValueAtTime(0.2, t);
            gain.gain.linearRampToValueAtTime(0.01, t + 0.08);
            osc.start(t); osc.stop(t + 0.08);
          } else if (type === 'shotgun') {
            osc.type = 'triangle';
            osc.frequency.setValueAtTime(180, t);
            osc.frequency.linearRampToValueAtTime(30, t + 0.16);
            gain.gain.setValueAtTime(0.4, t);
            gain.gain.linearRampToValueAtTime(0.01, t + 0.16);
            osc.start(t); osc.stop(t + 0.16);
          } else if (type === 'railgun') {
            osc.type = 'sawtooth';
            osc.frequency.setValueAtTime(880, t);
            osc.frequency.exponentialRampToValueAtTime(110, t + 0.25);
            gain.gain.setValueAtTime(0.3, t);
            gain.gain.linearRampToValueAtTime(0.01, t + 0.25);
            osc.start(t); osc.stop(t + 0.25);
          } else if (type === 'dash') {
            osc.type = 'sine';
            osc.frequency.setValueAtTime(200, t);
            osc.frequency.exponentialRampToValueAtTime(500, t + 0.12);
            gain.gain.setValueAtTime(0.2, t);
            gain.gain.linearRampToValueAtTime(0.01, t + 0.12);
            osc.start(t); osc.stop(t + 0.12);
          } else if (type === 'hit') {
            osc.type = 'square';
            osc.frequency.setValueAtTime(150, t);
            osc.frequency.linearRampToValueAtTime(40, t + 0.08);
            gain.gain.setValueAtTime(0.25, t);
            gain.gain.linearRampToValueAtTime(0.01, t + 0.08);
            osc.start(t); osc.stop(t + 0.08);
          } else if (type === 'pickup') {
            osc.type = 'sine';
            osc.frequency.setValueAtTime(440, t);
            osc.frequency.setValueAtTime(880, t + 0.08);
            gain.gain.setValueAtTime(0.2, t);
            gain.gain.linearRampToValueAtTime(0.01, t + 0.16);
            osc.start(t); osc.stop(t + 0.16);
          }
        } catch(e) {}
      }
    };

    /* =========================================================================
       2. WEAPONS CONFIGURATION
       ========================================================================= */
    const WEAPONS = {
      pistol: { name: 'Pistol', dmg: 14, speed: 18, rate: 0.28, count: 1, spread: 0.04, color: '#fbbf24', range: 50 },
      rifle: { name: 'Assault Rifle', dmg: 11, speed: 22, rate: 0.12, count: 1, spread: 0.08, color: '#38bdf8', range: 70 },
      shotgun: { name: 'Shotgun', dmg: 9, speed: 16, rate: 0.55, count: 6, spread: 0.35, color: '#f43f5e', range: 35 },
      railgun: { name: 'Railgun', dmg: 48, speed: 35, rate: 0.85, count: 1, spread: 0.0, color: '#a855f7', range: 100 }
    };

    /* =========================================================================
       3. ARENA & MAP LAYOUT
       ========================================================================= */
    const ARENA = {
      width: 1400,
      height: 900,
      walls: [
        // Outer Bounds
        { x: 0, y: 0, w: 1400, h: 20 },
        { x: 0, y: 880, w: 1400, h: 20 },
        { x: 0, y: 0, w: 20, h: 900 },
        { x: 1380, y: 0, w: 20, h: 900 },
        // Center Blockades
        { x: 650, y: 350, w: 100, h: 200 },
        { x: 350, y: 220, w: 120, h: 40 },
        { x: 350, y: 640, w: 120, h: 40 },
        { x: 930, y: 220, w: 120, h: 40 },
        { x: 930, y: 640, w: 120, h: 40 },
        // Corner Cover
        { x: 200, y: 400, w: 40, h: 100 },
        { x: 1160, y: 400, w: 40, h: 100 }
      ],
      spawns: {
        red: { x: 120, y: 450, angle: 0 },
        blue: { x: 1280, y: 450, angle: Math.PI }
      },
      weaponPads: [
        { x: 700, y: 180, weapon: 'railgun', timer: 0 },
        { x: 700, y: 720, weapon: 'shotgun', timer: 0 },
        { x: 410, y: 450, weapon: 'rifle', timer: 0 },
        { x: 990, y: 450, weapon: 'rifle', timer: 0 }
      ]
    };

    /* =========================================================================
       4. PLAYER ENTITY
       ========================================================================= */
    class Player {
      constructor(id, team, x, y, angle) {
        this.id = id;
        this.team = team;
        this.x = x;
        this.y = y;
        this.angle = angle;
        this.vx = 0;
        this.vy = 0;
        this.radius = 16;
        this.speed = 4.2;

        this.hp = 100;
        this.maxHp = 100;
        this.shield = 50;
        this.maxShield = 50;
        this.shieldRegenTimer = 0;

        this.weapon = 'pistol';
        this.fireTimer = 0;
        this.dashTimer = 0;
        this.isDashing = false;
        this.alive = true;
      }

      update(dt) {
        if (!this.alive) return;

        // Shield Recharge
        this.shieldRegenTimer += dt;
        if (this.shieldRegenTimer > 3.0 && this.shield < this.maxShield) {
          this.shield = Math.min(this.maxShield, this.shield + 20 * dt);
        }

        // Fire Cooldown
        if (this.fireTimer > 0) this.fireTimer -= dt;

        // Dash Cooldown / Decay
        if (this.dashTimer > 0) {
          this.dashTimer -= dt;
          if (this.dashTimer <= 0.25) this.isDashing = false;
        }

        // Movement application with wall collision
        const moveSpeed = this.isDashing ? this.speed * 2.6 : this.speed;
        let nextX = this.x + this.vx * moveSpeed;
        let nextY = this.y + this.vy * moveSpeed;

        // Wall collisions
        for (let wall of ARENA.walls) {
          if (circleRectCollide(nextX, this.y, this.radius, wall)) {
            nextX = this.x;
          }
          if (circleRectCollide(this.x, nextY, this.radius, wall)) {
            nextY = this.y;
          }
        }

        this.x = nextX;
        this.y = nextY;

        // Weapon Pad Pickups
        for (let pad of ARENA.weaponPads) {
          if (pad.timer <= 0 && Math.hypot(this.x - pad.x, this.y - pad.y) < 30) {
            this.weapon = pad.weapon;
            pad.timer = 15; // 15 sec respawn
            Audio.play('pickup');
            addFloatingText(this.x, this.y - 20, `+${WEAPONS[pad.weapon].name.toUpperCase()}!`, '#fbbf24');
          }
        }
      }

      dash() {
        if (this.dashTimer > 0 || (this.vx === 0 && this.vy === 0)) return;
        this.dashTimer = 1.0; // 1s cooldown
        this.isDashing = true;
        Audio.play('dash');
        for (let i = 0; i < 8; i++) {
          particles.push(new Particle(this.x, this.y, (Math.random()-0.5)*3, (Math.random()-0.5)*3, this.team === 'red' ? '#f43f5e' : '#0ea5e9', 15));
        }
      }

      takeDamage(dmg) {
        if (!this.alive || this.isDashing) return;
        this.shieldRegenTimer = 0;
        Audio.play('hit');
        addScreenShake(dmg * 0.25);

        // Shield absorb first
        if (this.shield > 0) {
          const absorbed = Math.min(this.shield, dmg);
          this.shield -= absorbed;
          dmg -= absorbed;
          addFloatingText(this.x, this.y - 15, `-${Math.round(absorbed)}`, '#38bdf8');
        }

        if (dmg > 0) {
          this.hp = Math.max(0, this.hp - dmg);
          addFloatingText(this.x, this.y - 20, `-${Math.round(dmg)}`, '#f43f5e');
        }

        if (this.hp <= 0) {
          this.alive = false;
          handleRoundEnd(this.team === 'red' ? 'blue' : 'red');
        }
      }

      draw(ctx) {
        if (!this.alive) return;

        ctx.save();
        ctx.translate(this.x, this.y);

        // Shield Aura
        if (this.shield > 0) {
          ctx.strokeStyle = `rgba(56, 189, 248, ${0.4 + (this.shield/50)*0.4})`;
          ctx.lineWidth = 3;
          ctx.beginPath();
          ctx.arc(0, 0, this.radius + 4, 0, Math.PI * 2);
          ctx.stroke();
        }

        // Body with Team Glow
        ctx.fillStyle = this.team === 'red' ? '#f43f5e' : '#0ea5e9';
        ctx.shadowColor = ctx.fillStyle;
        ctx.shadowBlur = this.isDashing ? 20 : 10;
        ctx.beginPath();
        ctx.arc(0, 0, this.radius, 0, Math.PI * 2);
        ctx.fill();
        ctx.shadowBlur = 0;

        // Gun Barrel Aiming Line
        ctx.rotate(this.angle);
        ctx.fillStyle = '#cbd5e1';
        ctx.fillRect(8, -3, 16, 6);

        ctx.restore();
      }
    }

    function circleRectCollide(cx, cy, r, rect) {
      const closestX = Math.max(rect.x, Math.min(cx, rect.x + rect.w));
      const closestY = Math.max(rect.y, Math.min(cy, rect.y + rect.h));
      const dx = cx - closestX;
      const dy = cy - closestY;
      return (dx * dx + dy * dy) < (r * r);
    }

    /* =========================================================================
       5. BULLET / PROJECTILE
       ========================================================================= */
    class Bullet {
      constructor(owner, x, y, angle, wpnKey) {
        this.owner = owner;
        this.wpn = WEAPONS[wpnKey];
        this.x = x;
        this.y = y;
        this.vx = Math.cos(angle) * this.wpn.speed;
        this.vy = Math.sin(angle) * this.wpn.speed;
        this.life = this.wpn.range;
        this.color = this.wpn.color;
        this.alive = true;
      }

      update() {
        this.x += this.vx;
        this.y += this.vy;
        this.life--;
        if (this.life <= 0) this.alive = false;

        // Wall collision
        for (let wall of ARENA.walls) {
          if (this.x >= wall.x && this.x <= wall.x + wall.w && this.y >= wall.y && this.y <= wall.y + wall.h) {
            this.alive = false;
            for (let i = 0; i < 4; i++) {
              particles.push(new Particle(this.x, this.y, (Math.random()-0.5)*3, (Math.random()-0.5)*3, '#fbbf24', 10));
            }
            break;
          }
        }

        // Hit Opponent
        const target = this.owner.team === 'red' ? p2 : p1;
        if (target && target.alive && Math.hypot(this.x - target.x, this.y - target.y) < target.radius + 4) {
          this.alive = false;
          target.takeDamage(this.wpn.dmg);
          for (let i = 0; i < 6; i++) {
            particles.push(new Particle(this.x, this.y, (Math.random()-0.5)*4, (Math.random()-0.5)*4, this.color, 14));
          }
        }
      }

      draw(ctx) {
        ctx.save();
        ctx.fillStyle = this.color;
        ctx.shadowColor = this.color;
        ctx.shadowBlur = 8;
        ctx.beginPath();
        ctx.arc(this.x, this.y, 4, 0, Math.PI * 2);
        ctx.fill();
        ctx.restore();
      }
    }

    /* =========================================================================
       6. PARTICLES & SCREEN SHAKE
       ========================================================================= */
    let particles = [];
    let floatingTexts = [];
    let screenShake = 0;

    class Particle {
      constructor(x, y, vx, vy, color, life) {
        this.x = x; this.y = y; this.vx = vx; this.vy = vy;
        this.color = color; this.life = life; this.maxLife = life;
      }
      update() { this.x += this.vx; this.y += this.vy; this.life--; }
      draw(ctx) {
        ctx.fillStyle = this.color;
        ctx.globalAlpha = Math.max(0, this.life / this.maxLife);
        ctx.beginPath(); ctx.arc(this.x, this.y, 2.5, 0, Math.PI * 2); ctx.fill();
        ctx.globalAlpha = 1.0;
      }
    }

    function addFloatingText(x, y, text, color) { floatingTexts.push({ x, y, text, color, life: 30 }); }
    function addScreenShake(val) { screenShake = Math.min(18, screenShake + val); }

    /* =========================================================================
       7. MULTIPLAYER ENGINE (PeerJS WebRTC)
       ========================================================================= */
    let peer = null, netConn = null;
    let isMultiplayer = false, isHost = false, myTeam = 'red';
    let isVsBot = true;

    function showHostUI() {
      document.querySelectorAll('#lobby-modal .btn-main').forEach(b => b.style.display = 'none');
      document.getElementById('host-panel').style.display = 'flex';
      const roomCode = Math.floor(1000 + Math.random() * 9000).toString();
      document.getElementById('host-code-display').innerText = roomCode;

      try {
        peer = new Peer('cyberstrike-' + roomCode, {
          config: { iceServers: [{ urls: 'stun:stun.l.google.com:19302' }] }
        });
        isHost = true;
        myTeam = 'red';

        peer.on('open', () => document.getElementById('host-status').innerText = 'Room Open! Waiting for opponent...');
        peer.on('connection', conn => {
          netConn = conn;
          setupNet();
          document.getElementById('host-status').innerText = 'Connected! Starting...';
          setTimeout(() => {
            document.getElementById('lobby-modal').style.display = 'none';
            document.getElementById('net-badge').innerText = 'ONLINE: HOST (🔴)';
            startMatch(false);
          }, 800);
        });
      } catch(e) {
        alert('Could not start online host. Playing vs Bot instead.');
        startVsBot();
      }
    }

    function showJoinUI() {
      document.querySelectorAll('#lobby-modal .btn-main').forEach(b => b.style.display = 'none');
      document.getElementById('join-panel').style.display = 'flex';
      isHost = false;
      myTeam = 'blue';
    }

    function connectToHost() {
      const code = document.getElementById('join-code-input').value.trim();
      if (code.length !== 4) return alert('Enter valid 4-digit code!');
      try {
        peer = new Peer({ config: { iceServers: [{ urls: 'stun:stun.l.google.com:19302' }] } });
        peer.on('open', () => {
          netConn = peer.connect('cyberstrike-' + code, { reliable: true });
          setupNet();
          netConn.on('open', () => {
            document.getElementById('lobby-modal').style.display = 'none';
            document.getElementById('net-badge').innerText = 'ONLINE: GUEST (🔵)';
            startMatch(false);
          });
        });
        peer.on('error', () => alert('Could not connect to room ' + code));
      } catch(e) {
        alert('Connection failed.');
      }
    }

    function setupNet() {
      isMultiplayer = true;
      isVsBot = false;
      netConn.on('data', data => {
        if (data.type === 'POS') {
          const remote = myTeam === 'red' ? p2 : p1;
          if (remote) {
            remote.x = data.x;
            remote.y = data.y;
            remote.angle = data.angle;
            remote.isDashing = data.isDashing;
          }
        } else if (data.type === 'FIRE') {
          spawnBullet(myTeam === 'red' ? p2 : p1, data.x, data.y, data.angle, data.weapon);
        } else if (data.type === 'DASH') {
          const remote = myTeam === 'red' ? p2 : p1;
          if (remote) remote.dash();
        } else if (data.type === 'RESTART') {
          resetRoundState();
          document.getElementById('gameover-modal').style.display = 'none';
        }
      });
      netConn.on('close', () => {
        alert('Opponent disconnected.');
        location.reload();
      });
    }

    function sendNet(pkg) {
      if (isMultiplayer && netConn && netConn.open) netConn.send(pkg);
    }

    function cancelLobby() {
      if (peer) peer.destroy();
      document.querySelectorAll('#lobby-modal .btn-main').forEach(b => b.style.display = 'flex');
      document.getElementById('host-panel').style.display = 'none';
      document.getElementById('join-panel').style.display = 'none';
    }

    function startVsBot() {
      isVsBot = true;
      isMultiplayer = false;
      document.getElementById('lobby-modal').style.display = 'none';
      document.getElementById('net-badge').innerText = 'SOLO VS BOT';
      startMatch(true);
    }

    /* =========================================================================
       8. GAME STATE, MATCH CONTROLLER & BOT AI
       ========================================================================= */
    const canvas = document.getElementById('game-canvas');
    const ctx = canvas.getContext('2d');

    let p1 = null, p2 = null;
    let bullets = [];
    let score = { red: 0, blue: 0 };
    let keys = { w: false, a: false, s: false, d: false, fire: false };
    let mouse = { x: 0, y: 0 };
    let cameraX = 0, cameraY = 0;

    function startMatch(vsBot) {
      isVsBot = vsBot;
      score = { red: 0, blue: 0 };
      updateScoreHUD();
      resetRoundState();
    }

    function resetRoundState() {
      p1 = new Player('p1', 'red', ARENA.spawns.red.x, ARENA.spawns.red.y, ARENA.spawns.red.angle);
      p2 = new Player('p2', 'blue', ARENA.spawns.blue.x, ARENA.spawns.blue.y, ARENA.spawns.blue.angle);
      bullets = [];
      particles = [];
      floatingTexts = [];
      ARENA.weaponPads.forEach(p => p.timer = 0);
    }

    function spawnBullet(player, x, y, angle, weaponKey) {
      const wpn = WEAPONS[weaponKey];
      Audio.play(weaponKey === 'pistol' ? 'rifle' : weaponKey);
      addScreenShake(weaponKey === 'shotgun' ? 4 : 2);

      for (let i = 0; i < wpn.count; i++) {
        const spread = (Math.random() - 0.5) * wpn.spread;
        bullets.push(new Bullet(player, x, y, angle + spread, weaponKey));
      }
    }

    function fireLocalWeapon() {
      const localPlayer = myTeam === 'red' ? p1 : p2;
      if (!localPlayer || !localPlayer.alive || localPlayer.fireTimer > 0) return;

      const wpn = WEAPONS[localPlayer.weapon];
      localPlayer.fireTimer = wpn.rate;

      const spawnX = localPlayer.x + Math.cos(localPlayer.angle) * 20;
      const spawnY = localPlayer.y + Math.sin(localPlayer.angle) * 20;
      spawnBullet(localPlayer, spawnX, spawnY, localPlayer.angle, localPlayer.weapon);

      sendNet({
        type: 'FIRE',
        x: spawnX,
        y: spawnY,
        angle: localPlayer.angle,
        weapon: localPlayer.weapon
      });
    }

    function handleRoundEnd(winnerTeam) {
      score[winnerTeam]++;
      updateScoreHUD();
      document.getElementById('winner-title').innerText = `${winnerTeam.toUpperCase()} WINS ROUND!`;
      document.getElementById('winner-title').style.color = winnerTeam === 'red' ? 'var(--red)' : 'var(--blue)';
      document.getElementById('gameover-modal').style.display = 'flex';
    }

    function nextRound() {
      document.getElementById('gameover-modal').style.display = 'none';
      resetRoundState();
      sendNet({ type: 'RESTART' });
    }

    function updateScoreHUD() {
      document.getElementById('score-board').innerText = `${score.red} - ${score.blue}`;
    }

    function runBotAI(dt) {
      if (!p2 || !p2.alive || !p1 || !p1.alive) return;

      const dx = p1.x - p2.x;
      const dy = p1.y - p2.y;
      const dist = Math.hypot(dx, dy);

      p2.angle = Math.atan2(dy, dx);

      // Seek weapon crate or player
      if (p2.weapon === 'pistol' && ARENA.weaponPads[1].timer <= 0) {
        const targetPad = ARENA.weaponPads[1];
        const px = targetPad.x - p2.x;
        const py = targetPad.y - p2.y;
        const padDist = Math.hypot(px, py);
        p2.vx = (px / padDist) * 0.9;
        p2.vy = (py / padDist) * 0.9;
      } else {
        // Keep mid distance
        if (dist > 280) {
          p2.vx = (dx / dist) * 0.9;
          p2.vy = (dy / dist) * 0.9;
        } else if (dist < 140) {
          p2.vx = -(dx / dist) * 0.9;
          p2.vy = -(dy / dist) * 0.9;
        } else {
          p2.vx = -dy / dist * 0.8;
          p2.vy = dx / dist * 0.8;
        }
      }

      // Shoot when line of sight is clear
      if (dist < 450 && p2.fireTimer <= 0) {
        const spawnX = p2.x + Math.cos(p2.angle) * 20;
        const spawnY = p2.y + Math.sin(p2.angle) * 20;
        spawnBullet(p2, spawnX, spawnY, p2.angle, p2.weapon);
        p2.fireTimer = WEAPONS[p2.weapon].rate;
      }
    }

    /* =========================================================================
       9. INPUT BINDINGS (DESKTOP & MOBILE TOUCH)
       ========================================================================= */
    window.addEventListener('keydown', e => {
      if (e.key === 'w' || e.key === 'ArrowUp') keys.w = true;
      if (e.key === 'a' || e.key === 'ArrowLeft') keys.a = true;
      if (e.key === 's' || e.key === 'ArrowDown') keys.s = true;
      if (e.key === 'd' || e.key === 'ArrowRight') keys.d = true;
      if (e.key === ' ' || e.key === 'Shift') {
        const lp = myTeam === 'red' ? p1 : p2;
        if (lp) { lp.dash(); sendNet({ type: 'DASH' }); }
      }
    });

    window.addEventListener('keyup', e => {
      if (e.key === 'w' || e.key === 'ArrowUp') keys.w = false;
      if (e.key === 'a' || e.key === 'ArrowLeft') keys.a = false;
      if (e.key === 's' || e.key === 'ArrowDown') keys.s = false;
      if (e.key === 'd' || e.key === 'ArrowRight') keys.d = false;
    });

    window.addEventListener('mousemove', e => {
      mouse.x = e.clientX;
      mouse.y = e.clientY;
    });

    window.addEventListener('mousedown', e => {
      if (e.button === 0) fireLocalWeapon();
    });

    // Touch Virtual Joystick
    const moveStick = document.getElementById('move-joystick');
    const moveKnob = document.getElementById('move-knob');
    let touchMoveId = null, stickCenter = { x: 0, y: 0 };

    moveStick.addEventListener('touchstart', e => {
      e.preventDefault();
      const t = e.changedTouches[0];
      touchMoveId = t.identifier;
      const rect = moveStick.getBoundingClientRect();
      stickCenter = { x: rect.left + rect.width/2, y: rect.top + rect.height/2 };
      handleStickMove(t.clientX, t.clientY);
    });

    window.addEventListener('touchmove', e => {
      for (let i = 0; i < e.changedTouches.length; i++) {
        const t = e.changedTouches[i];
        if (t.identifier === touchMoveId) handleStickMove(t.clientX, t.clientY);
      }
    });

    function handleStickMove(cx, cy) {
      const dx = cx - stickCenter.x;
      const dy = cy - stickCenter.y;
      const dist = Math.min(38, Math.hypot(dx, dy));
      const angle = Math.atan2(dy, dx);

      moveKnob.style.transform = `translate(${Math.cos(angle)*dist}px, ${Math.sin(angle)*dist}px)`;

      const lp = myTeam === 'red' ? p1 : p2;
      if (lp) {
        lp.vx = (Math.cos(angle) * (dist / 38));
        lp.vy = (Math.sin(angle) * (dist / 38));
      }
    }

    window.addEventListener('touchend', e => {
      for (let i = 0; i < e.changedTouches.length; i++) {
        if (e.changedTouches[i].identifier === touchMoveId) {
          touchMoveId = null;
          moveKnob.style.transform = 'translate(0px, 0px)';
          const lp = myTeam === 'red' ? p1 : p2;
          if (lp) { lp.vx = 0; lp.vy = 0; }
        }
      }
    });

    document.getElementById('btn-fire').addEventListener('touchstart', e => {
      e.preventDefault();
      fireLocalWeapon();
    });

    document.getElementById('btn-dash').addEventListener('touchstart', e => {
      e.preventDefault();
      const lp = myTeam === 'red' ? p1 : p2;
      if (lp) { lp.dash(); sendNet({ type: 'DASH' }); }
    });

    function resize() {
      canvas.width = window.innerWidth;
      canvas.height = window.innerHeight;
    }
    window.addEventListener('resize', resize);

    /* =========================================================================
       10. MAIN ENGINE LOOP
       ========================================================================= */
    let lastTime = performance.now();
    let netSyncTimer = 0;

    function gameLoop(now) {
      const dt = Math.min(0.1, (now - lastTime) / 1000);
      lastTime = now;

      const localPlayer = myTeam === 'red' ? p1 : p2;

      // Handle Keyboard Local Movement
      if (localPlayer && touchMoveId === null) {
        let mx = 0, my = 0;
        if (keys.w) my -= 1;
        if (keys.s) my += 1;
        if (keys.a) mx -= 1;
        if (keys.d) mx += 1;
        const len = Math.hypot(mx, my);
        localPlayer.vx = len > 0 ? mx / len : 0;
        localPlayer.vy = len > 0 ? my / len : 0;

        // Mouse Aim
        const screenX = localPlayer.x - cameraX;
        const screenY = localPlayer.y - cameraY;
        localPlayer.angle = Math.atan2(mouse.y - screenY, mouse.x - screenX);
      }

      // Update Players & Pads
      if (p1) p1.update(dt);
      if (p2) p2.update(dt);
      if (isVsBot) runBotAI(dt);

      ARENA.weaponPads.forEach(p => { if (p.timer > 0) p.timer -= dt; });

      // Bullets
      bullets.forEach(b => b.update());
      bullets = bullets.filter(b => b.alive);

      // Particles
      particles.forEach(p => p.update());
      particles = particles.filter(p => p.life > 0);

      // WebRTC Real-Time Sync (25 packets/sec)
      if (isMultiplayer && localPlayer) {
        netSyncTimer += dt;
        if (netSyncTimer > 0.04) {
          netSyncTimer = 0;
          sendNet({
            type: 'POS',
            x: localPlayer.x,
            y: localPlayer.y,
            angle: localPlayer.angle,
            isDashing: localPlayer.isDashing
          });
        }
      }

      // Camera Tracking
      if (localPlayer) {
        cameraX += (localPlayer.x - canvas.width / 2 - cameraX) * 0.1;
        cameraY += (localPlayer.y - canvas.height / 2 - cameraY) * 0.1;
      }
      cameraX = Math.max(0, Math.min(ARENA.width - canvas.width, cameraX));
      cameraY = Math.max(0, Math.min(ARENA.height - canvas.height, cameraY));

      // Screen Shake Decay
      if (screenShake > 0) screenShake = Math.max(0, screenShake - 0.4);
      const shakeX = (Math.random() - 0.5) * screenShake;
      const shakeY = (Math.random() - 0.5) * screenShake;

      /* RENDER SCENE */
      ctx.clearRect(0, 0, canvas.width, canvas.height);

      ctx.save();
      ctx.translate(-cameraX + shakeX, -cameraY + shakeY);

      // Grid Arena Floor
      ctx.strokeStyle = 'rgba(255, 255, 255, 0.04)';
      ctx.lineWidth = 1;
      for (let x = 0; x < ARENA.width; x += 40) {
        ctx.beginPath(); ctx.moveTo(x, 0); ctx.lineTo(x, ARENA.height); ctx.stroke();
      }
      for (let y = 0; y < ARENA.height; y += 40) {
        ctx.beginPath(); ctx.moveTo(0, y); ctx.lineTo(ARENA.width, y); ctx.stroke();
      }

      // Weapon Pads
      for (let pad of ARENA.weaponPads) {
        ctx.fillStyle = pad.timer <= 0 ? 'rgba(251, 191, 36, 0.2)' : 'rgba(255,255,255,0.05)';
        ctx.strokeStyle = pad.timer <= 0 ? '#fbbf24' : '#475569';
        ctx.beginPath(); ctx.arc(pad.x, pad.y, 22, 0, Math.PI*2); ctx.fill(); ctx.stroke();
        if (pad.timer <= 0) {
          ctx.fillStyle = '#fbbf24';
          ctx.font = 'bold 9px monospace';
          ctx.textAlign = 'center';
          ctx.fillText(WEAPONS[pad.weapon].name.toUpperCase(), pad.x, pad.y + 3);
        }
      }

      // Arena Walls
      ctx.fillStyle = '#0f172a';
      ctx.strokeStyle = '#38bdf8';
      ctx.lineWidth = 2;
      for (let wall of ARENA.walls) {
        ctx.fillRect(wall.x, wall.y, wall.w, wall.h);
        ctx.strokeRect(wall.x, wall.y, wall.w, wall.h);
      }

      // Entities
      if (p1) p1.draw(ctx);
      if (p2) p2.draw(ctx);
      bullets.forEach(b => b.draw(ctx));
      particles.forEach(p => p.draw(ctx));

      // Damage Numbers
      floatingTexts.forEach(ft => {
        ctx.font = 'bold 12px sans-serif';
        ctx.fillStyle = ft.color;
        ctx.fillText(ft.text, ft.x - 10, ft.y);
        ft.y -= 0.6;
        ft.life--;
      });
      floatingTexts = floatingTexts.filter(ft => ft.life > 0);

      ctx.restore();

      // Update HUD Bars
      if (p1) {
        document.getElementById('p1-hp-bar').style.width = `${(p1.hp / p1.maxHp)*100}%`;
        document.getElementById('p1-shield-bar').style.width = `${(p1.shield / p1.maxShield)*100}%`;
        document.getElementById('p1-hp-text').innerText = `${Math.ceil(p1.hp)} HP`;
      }
      if (p2) {
        document.getElementById('p2-hp-bar').style.width = `${(p2.hp / p2.maxHp)*100}%`;
        document.getElementById('p2-shield-bar').style.width = `${(p2.shield / p2.maxShield)*100}%`;
        document.getElementById('p2-hp-text').innerText = `${Math.ceil(p2.hp)} HP`;
      }

      requestAnimationFrame(gameLoop);
    }

    // Startup
    resize();
    resetRoundState();
    requestAnimationFrame(gameLoop);
  </script>
</body>
</html>

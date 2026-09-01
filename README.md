
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
  <title>CyberStrike 1v1 - Mobile Scaled</title>
  <!-- PeerJS for WebRTC P2P Multiplayer -->
  <script src="https://unpkg.com/peerjs@1.5.4/dist/peerjs.min.js"></script>
  <style>
    :root {
      --bg: #050811;
      --panel: rgba(15, 23, 42, 0.95);
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
      touch-action: none;
    }

    html, body {
      width: 100%;
      height: 100%;
      height: 100dvh;
      overflow: hidden;
      background-color: var(--bg);
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
      color: #fff;
    }

    #game-container {
      position: relative;
      width: 100vw;
      height: 100vh;
      height: 100dvh;
      display: flex;
      flex-direction: column;
    }

    /* Ultra-Compact Top HUD */
    #hud-top {
      height: 40px;
      background: var(--panel);
      border-bottom: 1px solid var(--border);
      display: flex;
      align-items: center;
      justify-content: space-between;
      padding: 0 10px;
      z-index: 20;
      flex-shrink: 0;
    }

    .player-stat {
      display: flex;
      flex-direction: column;
      gap: 2px;
      width: 80px;
      max-width: 25vw;
    }
    .stat-header {
      font-size: 0.65rem;
      font-weight: 800;
      display: flex;
      justify-content: space-between;
      line-height: 1;
    }
    .bar-wrap {
      width: 100%;
      height: 6px;
      background: rgba(0,0,0,0.6);
      border-radius: 3px;
      overflow: hidden;
    }
    .hp-fill { height: 100%; width: 100%; transition: width 0.1s linear; }
    .hp-red { background: var(--red); }
    .hp-blue { background: var(--blue); }
    .shield-fill { height: 3px; background: #38bdf8; width: 100%; border-radius: 2px; margin-top: 1px; }

    .center-hud {
      display: flex;
      flex-direction: column;
      align-items: center;
    }
    #score-board {
      font-family: var(--font-code);
      font-size: 1.05rem;
      font-weight: 900;
      color: var(--gold);
      letter-spacing: 1px;
      line-height: 1;
    }
    #net-badge {
      font-size: 0.55rem;
      background: rgba(255,255,255,0.1);
      padding: 1px 5px;
      border-radius: 6px;
      color: #94a3b8;
      margin-top: 2px;
    }

    /* Scaled Canvas Area */
    #canvas-wrap {
      flex: 1;
      position: relative;
      width: 100%;
      height: 100%;
      overflow: hidden;
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

    /* Mobile Touch Zones */
    #touch-layer {
      position: absolute;
      inset: 0;
      z-index: 25;
      display: flex;
      pointer-events: auto;
    }
    .touch-half {
      flex: 1;
      height: 100%;
      position: relative;
    }

    #joystick-base {
      position: absolute;
      width: 80px;
      height: 80px;
      border: 2px dashed rgba(255,255,255,0.3);
      border-radius: 50%;
      transform: translate(-50%, -50%);
      display: none;
      pointer-events: none;
      background: rgba(15, 23, 42, 0.4);
    }
    #joystick-knob {
      position: absolute;
      width: 36px;
      height: 36px;
      background: var(--blue);
      border-radius: 50%;
      transform: translate(-50%, -50%);
      top: 50%;
      left: 50%;
      box-shadow: 0 0 8px var(--blue);
    }

    #dash-btn {
      position: absolute;
      bottom: 16px;
      right: 16px;
      width: 52px;
      height: 52px;
      background: var(--panel);
      border: 2px solid var(--gold);
      border-radius: 50%;
      color: var(--gold);
      font-weight: 900;
      font-size: 0.65rem;
      display: flex;
      align-items: center;
      justify-content: center;
      pointer-events: auto;
    }
    #dash-btn:active { transform: scale(0.9); }

    /* Modals */
    .modal-overlay {
      position: absolute;
      inset: 0;
      background: rgba(5, 8, 17, 0.94);
      display: flex;
      align-items: center;
      justify-content: center;
      z-index: 100;
      padding: 14px;
    }
    .modal-card {
      background: #0f172a;
      border: 1px solid var(--border);
      border-radius: 14px;
      padding: 18px;
      text-align: center;
      max-width: 320px;
      width: 100%;
      display: flex;
      flex-direction: column;
      gap: 10px;
    }
    .modal-card h2 { font-size: 1.25rem; color: var(--gold); font-weight: 900; }
    .btn-main {
      background: var(--blue);
      color: #fff;
      border: none;
      padding: 10px;
      border-radius: 8px;
      font-weight: 800;
      font-size: 0.85rem;
      cursor: pointer;
    }
    .btn-danger { background: var(--red); }
    .input-code {
      background: #020617;
      border: 1px solid #475569;
      color: #fff;
      padding: 8px;
      border-radius: 6px;
      font-size: 1.2rem;
      text-align: center;
      letter-spacing: 3px;
      font-family: var(--font-code);
      outline: none;
    }
  </style>
</head>
<body>

  <div id="game-container">
    <!-- Compact Top HUD -->
    <header id="hud-top">
      <div class="player-stat">
        <div class="stat-header" style="color:var(--red);">
          <span>RED</span>
          <span id="p1-hp-text">100</span>
        </div>
        <div class="bar-wrap"><div class="hp-fill hp-red" id="p1-hp-bar"></div></div>
        <div class="shield-fill" id="p1-shield-bar"></div>
      </div>

      <div class="center-hud">
        <div id="score-board">0 - 0</div>
        <div id="net-badge">PRACTICE AI</div>
      </div>

      <div class="player-stat">
        <div class="stat-header" style="color:var(--blue);">
          <span>BLUE</span>
          <span id="p2-hp-text">100</span>
        </div>
        <div class="bar-wrap"><div class="hp-fill hp-blue" id="p2-hp-bar"></div></div>
        <div class="shield-fill" id="p2-shield-bar"></div>
      </div>
    </header>

    <!-- Canvas -->
    <div id="canvas-wrap">
      <canvas id="game-canvas"></canvas>

      <div id="touch-layer">
        <div class="touch-half" id="touch-move-area"></div>
        <div class="touch-half" id="touch-shoot-area">
          <button id="dash-btn" onclick="triggerDash()">⚡ DASH</button>
        </div>
      </div>

      <div id="joystick-base">
        <div id="joystick-knob"></div>
      </div>
    </div>
  </div>

  <!-- Start Modal -->
  <div id="lobby-modal" class="modal-overlay">
    <div class="modal-card">
      <h2>CYBERSTRIKE 1V1</h2>
      <p style="font-size:0.75rem; color:#94a3b8;">Scaled Mobile Arena Shooter</p>

      <button class="btn-main" onclick="startVsBot()">🤖 Practice vs AI Bot</button>
      <button class="btn-main" style="background:var(--gold); color:#000;" onclick="showHostUI()">🌐 Host Online Match</button>
      <button class="btn-main" style="background:#8b5cf6;" onclick="showJoinUI()">🔗 Join with Room Code</button>

      <!-- Host Subpanel -->
      <div id="host-panel" style="display:none; flex-direction:column; gap:6px;">
        <div style="font-size:0.75rem; color:#94a3b8;">Share Room Code:</div>
        <div id="host-code-display" style="font-size:1.6rem; font-weight:900; color:var(--gold); font-family:var(--font-code);">----</div>
        <div style="font-size:0.7rem; color:#cbd5e1;" id="host-status">Generating room...</div>
        <button class="btn-main btn-danger" onclick="cancelLobby()">Back</button>
      </div>

      <!-- Join Subpanel -->
      <div id="join-panel" style="display:none; flex-direction:column; gap:6px;">
        <div style="font-size:0.75rem; color:#94a3b8;">Enter 4-Digit Code:</div>
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
      <button class="btn-main" onclick="nextRound()">Next Round</button>
    </div>
  </div>

  <script>
    /* AUDIO SYNTHESIZER */
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
            osc.frequency.linearRampToValueAtTime(30, t + 0.14);
            gain.gain.setValueAtTime(0.3, t);
            gain.gain.linearRampToValueAtTime(0.01, t + 0.14);
            osc.start(t); osc.stop(t + 0.14);
          } else if (type === 'railgun') {
            osc.type = 'sawtooth';
            osc.frequency.setValueAtTime(880, t);
            osc.frequency.exponentialRampToValueAtTime(110, t + 0.22);
            gain.gain.setValueAtTime(0.3, t);
            gain.gain.linearRampToValueAtTime(0.01, t + 0.22);
            osc.start(t); osc.stop(t + 0.22);
          } else if (type === 'dash') {
            osc.type = 'sine';
            osc.frequency.setValueAtTime(200, t);
            osc.frequency.exponentialRampToValueAtTime(500, t + 0.1);
            gain.gain.setValueAtTime(0.2, t);
            gain.gain.linearRampToValueAtTime(0.01, t + 0.1);
            osc.start(t); osc.stop(t + 0.1);
          } else if (type === 'hit') {
            osc.type = 'square';
            osc.frequency.setValueAtTime(150, t);
            osc.frequency.linearRampToValueAtTime(40, t + 0.06);
            gain.gain.setValueAtTime(0.2, t);
            gain.gain.linearRampToValueAtTime(0.01, t + 0.06);
            osc.start(t); osc.stop(t + 0.06);
          } else if (type === 'pickup') {
            osc.type = 'sine';
            osc.frequency.setValueAtTime(440, t);
            osc.frequency.setValueAtTime(880, t + 0.08);
            gain.gain.setValueAtTime(0.18, t);
            gain.gain.linearRampToValueAtTime(0.01, t + 0.14);
            osc.start(t); osc.stop(t + 0.14);
          }
        } catch(e) {}
      }
    };

    /* WEAPONS & ARENA */
    const WEAPONS = {
      pistol: { name: 'Pistol', dmg: 14, speed: 18, rate: 0.25, count: 1, spread: 0.04, color: '#fbbf24', range: 45 },
      rifle: { name: 'Rifle', dmg: 11, speed: 22, rate: 0.12, count: 1, spread: 0.08, color: '#38bdf8', range: 65 },
      shotgun: { name: 'Shotgun', dmg: 9, speed: 16, rate: 0.55, count: 6, spread: 0.35, color: '#f43f5e', range: 30 },
      railgun: { name: 'Railgun', dmg: 48, speed: 35, rate: 0.85, count: 1, spread: 0.0, color: '#a855f7', range: 90 }
    };

    const ARENA = {
      width: 1000,
      height: 650,
      walls: [
        { x: 0, y: 0, w: 1000, h: 16 },
        { x: 0, y: 634, w: 1000, h: 16 },
        { x: 0, y: 0, w: 16, h: 650 },
        { x: 984, y: 0, w: 16, h: 650 },
        { x: 460, y: 240, w: 80, h: 170 },
        { x: 240, y: 150, w: 90, h: 30 },
        { x: 240, y: 470, w: 90, h: 30 },
        { x: 670, y: 150, w: 90, h: 30 },
        { x: 670, y: 470, w: 90, h: 30 },
        { x: 130, y: 280, w: 30, h: 90 },
        { x: 840, y: 280, w: 30, h: 90 }
      ],
      spawns: {
        red: { x: 80, y: 325, angle: 0 },
        blue: { x: 920, y: 325, angle: Math.PI }
      },
      weaponPads: [
        { x: 500, y: 120, weapon: 'railgun', timer: 0 },
        { x: 500, y: 530, weapon: 'shotgun', timer: 0 },
        { x: 285, y: 325, weapon: 'rifle', timer: 0 },
        { x: 715, y: 325, weapon: 'rifle', timer: 0 }
      ]
    };

    /* PLAYER CLASS */
    class Player {
      constructor(id, team, x, y, angle) {
        this.id = id;
        this.team = team;
        this.x = x;
        this.y = y;
        this.angle = angle;
        this.vx = 0;
        this.vy = 0;
        this.radius = 14;
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

        this.shieldRegenTimer += dt;
        if (this.shieldRegenTimer > 3.0 && this.shield < this.maxShield) {
          this.shield = Math.min(this.maxShield, this.shield + 20 * dt);
        }

        if (this.fireTimer > 0) this.fireTimer -= dt;

        if (this.dashTimer > 0) {
          this.dashTimer -= dt;
          if (this.dashTimer <= 0.25) this.isDashing = false;
        }

        const moveSpeed = this.isDashing ? this.speed * 2.5 : this.speed;
        let nextX = this.x + this.vx * moveSpeed;
        let nextY = this.y + this.vy * moveSpeed;

        for (let wall of ARENA.walls) {
          if (circleRectCollide(nextX, this.y, this.radius, wall)) nextX = this.x;
          if (circleRectCollide(this.x, nextY, this.radius, wall)) nextY = this.y;
        }

        this.x = nextX;
        this.y = nextY;

        for (let pad of ARENA.weaponPads) {
          if (pad.timer <= 0 && Math.hypot(this.x - pad.x, this.y - pad.y) < 26) {
            this.weapon = pad.weapon;
            pad.timer = 14;
            Audio.play('pickup');
            addFloatingText(this.x, this.y - 18, `+${WEAPONS[pad.weapon].name.toUpperCase()}!`, '#fbbf24');
          }
        }
      }

      dash() {
        if (this.dashTimer > 0 || (this.vx === 0 && this.vy === 0)) return;
        this.dashTimer = 1.0;
        this.isDashing = true;
        Audio.play('dash');
      }

      takeDamage(dmg) {
        if (!this.alive || this.isDashing) return;
        this.shieldRegenTimer = 0;
        Audio.play('hit');

        if (this.shield > 0) {
          const absorbed = Math.min(this.shield, dmg);
          this.shield -= absorbed;
          dmg -= absorbed;
          addFloatingText(this.x, this.y - 14, `-${Math.round(absorbed)}`, '#38bdf8');
        }

        if (dmg > 0) {
          this.hp = Math.max(0, this.hp - dmg);
          addFloatingText(this.x, this.y - 18, `-${Math.round(dmg)}`, '#f43f5e');
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

        if (this.shield > 0) {
          ctx.strokeStyle = `rgba(56, 189, 248, ${0.4 + (this.shield/50)*0.4})`;
          ctx.lineWidth = 2.5;
          ctx.beginPath();
          ctx.arc(0, 0, this.radius + 3, 0, Math.PI * 2);
          ctx.stroke();
        }

        ctx.fillStyle = this.team === 'red' ? '#f43f5e' : '#0ea5e9';
        ctx.beginPath();
        ctx.arc(0, 0, this.radius, 0, Math.PI * 2);
        ctx.fill();

        ctx.rotate(this.angle);
        ctx.fillStyle = '#cbd5e1';
        ctx.fillRect(6, -2.5, 14, 5);

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

    /* BULLET CLASS */
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

        for (let wall of ARENA.walls) {
          if (this.x >= wall.x && this.x <= wall.x + wall.w && this.y >= wall.y && this.y <= wall.y + wall.h) {
            this.alive = false;
            break;
          }
        }

        const target = this.owner.team === 'red' ? p2 : p1;
        if (target && target.alive && Math.hypot(this.x - target.x, this.y - target.y) < target.radius + 3) {
          this.alive = false;
          target.takeDamage(this.wpn.dmg);
        }
      }

      draw(ctx) {
        ctx.save();
        ctx.fillStyle = this.color;
        ctx.beginPath();
        ctx.arc(this.x, this.y, 3.5, 0, Math.PI * 2);
        ctx.fill();
        ctx.restore();
      }
    }

    let floatingTexts = [];
    function addFloatingText(x, y, text, color) { floatingTexts.push({ x, y, text, color, life: 28 }); }

    /* MULTIPLAYER LOGIC */
    let peer = null, netConn = null;
    let isMultiplayer = false, isHost = false, myTeam = 'red';
    let isVsBot = true;

    function showHostUI() {
      document.querySelectorAll('#lobby-modal .btn-main').forEach(b => b.style.display = 'none');
      document.getElementById('host-panel').style.display = 'flex';
      const roomCode = Math.floor(1000 + Math.random() * 9000).toString();
      document.getElementById('host-code-display').innerText = roomCode;

      try {
        peer = new Peer('cyberstrike-' + roomCode);
        isHost = true;
        myTeam = 'red';

        peer.on('open', () => document.getElementById('host-status').innerText = 'Waiting for opponent...');
        peer.on('connection', conn => {
          netConn = conn;
          setupNet();
          setTimeout(() => {
            document.getElementById('lobby-modal').style.display = 'none';
            document.getElementById('net-badge').innerText = 'ONLINE: HOST (🔴)';
            startMatch(false);
          }, 600);
        });
      } catch(e) { startVsBot(); }
    }

    function showJoinUI() {
      document.querySelectorAll('#lobby-modal .btn-main').forEach(b => b.style.display = 'none');
      document.getElementById('join-panel').style.display = 'flex';
      isHost = false;
      myTeam = 'blue';
    }

    function connectToHost() {
      const code = document.getElementById('join-code-input').value.trim();
      if (code.length !== 4) return alert('Enter 4-digit code!');
      try {
        peer = new Peer();
        peer.on('open', () => {
          netConn = peer.connect('cyberstrike-' + code, { reliable: true });
          setupNet();
          netConn.on('open', () => {
            document.getElementById('lobby-modal').style.display = 'none';
            document.getElementById('net-badge').innerText = 'ONLINE: GUEST (🔵)';
            startMatch(false);
          });
        });
        peer.on('error', () => alert('Room not found!'));
      } catch(e) { alert('Connection error'); }
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
      netConn.on('close', () => { alert('Opponent disconnected.'); location.reload(); });
    }

    function sendNet(pkg) {
      if (isMultiplayer && netConn && netConn.open) netConn.send(pkg);
    }

    function cancelLobby() {
      if (peer) peer.destroy();
      document.querySelectorAll('#lobby-modal .btn-main').forEach(b => b.style.display = 'block');
      document.getElementById('host-panel').style.display = 'none';
      document.getElementById('join-panel').style.display = 'none';
    }

    function startVsBot() {
      isVsBot = true;
      isMultiplayer = false;
      document.getElementById('lobby-modal').style.display = 'none';
      document.getElementById('net-badge').innerText = 'PRACTICE AI';
      startMatch(true);
    }

    /* ENGINE & CANVAS SCALING */
    const canvas = document.getElementById('game-canvas');
    const ctx = canvas.getContext('2d');

    let p1 = null, p2 = null;
    let bullets = [];
    let score = { red: 0, blue: 0 };
    let cameraX = 0, cameraY = 0;
    let gameScale = 1.0;

    const keys = { w: false, a: false, s: false, d: false };
    let mouse = { x: 0, y: 0 };
    let isTouchActive = false;

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
      floatingTexts = [];
      ARENA.weaponPads.forEach(p => p.timer = 0);
    }

    function spawnBullet(player, x, y, angle, weaponKey) {
      const wpn = WEAPONS[weaponKey];
      Audio.play(weaponKey === 'pistol' ? 'rifle' : weaponKey);

      for (let i = 0; i < wpn.count; i++) {
        const spread = (Math.random() - 0.5) * wpn.spread;
        bullets.push(new Bullet(player, x, y, angle + spread, weaponKey));
      }
    }

    function fireLocalWeapon() {
      const lp = myTeam === 'red' ? p1 : p2;
      if (!lp || !lp.alive || lp.fireTimer > 0) return;

      const wpn = WEAPONS[lp.weapon];
      lp.fireTimer = wpn.rate;

      const spawnX = lp.x + Math.cos(lp.angle) * 18;
      const spawnY = lp.y + Math.sin(lp.angle) * 18;
      spawnBullet(lp, spawnX, spawnY, lp.angle, lp.weapon);

      sendNet({
        type: 'FIRE',
        x: spawnX,
        y: spawnY,
        angle: lp.angle,
        weapon: lp.weapon
      });
    }

    function triggerDash() {
      const lp = myTeam === 'red' ? p1 : p2;
      if (lp) {
        lp.dash();
        sendNet({ type: 'DASH' });
      }
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

      if (dist > 220) {
        p2.vx = (dx / dist) * 0.9;
        p2.vy = (dy / dist) * 0.9;
      } else if (dist < 110) {
        p2.vx = -(dx / dist) * 0.9;
        p2.vy = -(dy / dist) * 0.9;
      } else {
        p2.vx = -dy / dist * 0.8;
        p2.vy = dx / dist * 0.8;
      }

      if (dist < 400 && p2.fireTimer <= 0) {
        const spawnX = p2.x + Math.cos(p2.angle) * 18;
        const spawnY = p2.y + Math.sin(p2.angle) * 18;
        spawnBullet(p2, spawnX, spawnY, p2.angle, p2.weapon);
        p2.fireTimer = WEAPONS[p2.weapon].rate;
      }
    }

    /* INPUTS */
    window.addEventListener('keydown', e => {
      const key = e.key.toLowerCase();
      if (key === 'w' || key === 'arrowup') keys.w = true;
      if (key === 'a' || key === 'arrowleft') keys.a = true;
      if (key === 's' || key === 'arrowdown') keys.s = true;
      if (key === 'd' || key === 'arrowright') keys.d = true;
      if (key === ' ' || key === 'shift') triggerDash();
    });

    window.addEventListener('keyup', e => {
      const key = e.key.toLowerCase();
      if (key === 'w' || key === 'arrowup') keys.w = false;
      if (key === 'a' || key === 'arrowleft') keys.a = false;
      if (key === 's' || key === 'arrowdown') keys.s = false;
      if (key === 'd' || key === 'arrowright') keys.d = false;
    });

    window.addEventListener('mousemove', e => {
      mouse.x = e.clientX;
      mouse.y = e.clientY;
    });

    window.addEventListener('mousedown', e => {
      if (e.target.tagName !== 'BUTTON' && e.target.tagName !== 'INPUT') {
        fireLocalWeapon();
      }
    });

    // Mobile Virtual Touch Joystick
    const moveArea = document.getElementById('touch-move-area');
    const shootArea = document.getElementById('touch-shoot-area');
    const joyBase = document.getElementById('joystick-base');
    const joyKnob = document.getElementById('joystick-knob');

    let moveTouchId = null;
    let joyCenter = { x: 0, y: 0 };

    moveArea.addEventListener('touchstart', e => {
      e.preventDefault();
      const t = e.changedTouches[0];
      moveTouchId = t.identifier;
      isTouchActive = true;

      joyCenter = { x: t.clientX, y: t.clientY };
      joyBase.style.left = `${t.clientX}px`;
      joyBase.style.top = `${t.clientY}px`;
      joyBase.style.display = 'block';
      joyKnob.style.transform = 'translate(-50%, -50%)';
    });

    window.addEventListener('touchmove', e => {
      for (let i = 0; i < e.changedTouches.length; i++) {
        const t = e.changedTouches[i];
        if (t.identifier === moveTouchId) {
          const dx = t.clientX - joyCenter.x;
          const dy = t.clientY - joyCenter.y;
          const dist = Math.min(35, Math.hypot(dx, dy));
          const angle = Math.atan2(dy, dx);

          joyKnob.style.transform = `translate(calc(-50% + ${Math.cos(angle)*dist}px), calc(-50% + ${Math.sin(angle)*dist}px))`;

          const lp = myTeam === 'red' ? p1 : p2;
          if (lp) {
            lp.vx = (Math.cos(angle) * (dist / 35));
            lp.vy = (Math.sin(angle) * (dist / 35));
            lp.angle = angle;
          }
        }
      }
    });

    window.addEventListener('touchend', e => {
      for (let i = 0; i < e.changedTouches.length; i++) {
        if (e.changedTouches[i].identifier === moveTouchId) {
          moveTouchId = null;
          joyBase.style.display = 'none';
          const lp = myTeam === 'red' ? p1 : p2;
          if (lp) { lp.vx = 0; lp.vy = 0; }
        }
      }
    });

    shootArea.addEventListener('touchstart', e => {
      if (e.target.id === 'dash-btn') return;
      e.preventDefault();
      const t = e.changedTouches[0];
      const lp = myTeam === 'red' ? p1 : p2;
      if (lp) {
        const screenX = (lp.x - cameraX) * gameScale;
        const screenY = (lp.y - cameraY) * gameScale;
        lp.angle = Math.atan2(t.clientY - screenY, t.clientX - screenX);
        fireLocalWeapon();
      }
    });

    // Resize and Scale Factor Calculation
    function resize() {
      const wrap = document.getElementById('canvas-wrap');
      const w = wrap.clientWidth || window.innerWidth;
      const h = wrap.clientHeight || (window.innerHeight - 40);
      canvas.width = w;
      canvas.height = h;

      // Auto-fit zoom: scale arena down to fit phone screens comfortably
      gameScale = Math.min(w / 480, h / 320, 1.0);
      if (w < 600) gameScale = Math.min(w / 440, h / 300);
    }
    window.addEventListener('resize', resize);

    /* MAIN GAME LOOP */
    let lastTime = performance.now();
    let netSyncTimer = 0;

    function gameLoop(now) {
      const dt = Math.min(0.1, (now - lastTime) / 1000);
      lastTime = now;

      const lp = myTeam === 'red' ? p1 : p2;

      if (lp && moveTouchId === null) {
        let mx = 0, my = 0;
        if (keys.w) my -= 1;
        if (keys.s) my += 1;
        if (keys.a) mx -= 1;
        if (keys.d) mx += 1;
        const len = Math.hypot(mx, my);
        lp.vx = len > 0 ? mx / len : 0;
        lp.vy = len > 0 ? my / len : 0;

        if (!isTouchActive) {
          const screenX = (lp.x - cameraX) * gameScale;
          const screenY = (lp.y - cameraY) * gameScale;
          lp.angle = Math.atan2(mouse.y - screenY, mouse.x - screenX);
        }
      }

      if (p1) p1.update(dt);
      if (p2) p2.update(dt);
      if (isVsBot) runBotAI(dt);

      ARENA.weaponPads.forEach(p => { if (p.timer > 0) p.timer -= dt; });

      bullets.forEach(b => b.update());
      bullets = bullets.filter(b => b.alive);

      if (isMultiplayer && lp) {
        netSyncTimer += dt;
        if (netSyncTimer > 0.04) {
          netSyncTimer = 0;
          sendNet({
            type: 'POS',
            x: lp.x,
            y: lp.y,
            angle: lp.angle,
            isDashing: lp.isDashing
          });
        }
      }

      // Camera Tracking adjusted for dynamic scale
      const viewW = canvas.width / gameScale;
      const viewH = canvas.height / gameScale;

      if (lp) {
        cameraX += (lp.x - viewW / 2 - cameraX) * 0.1;
        cameraY += (lp.y - viewH / 2 - cameraY) * 0.1;
      }
      cameraX = Math.max(0, Math.min(ARENA.width - viewW, cameraX));
      cameraY = Math.max(0, Math.min(ARENA.height - viewH, cameraY));

      /* RENDER SCENE */
      ctx.clearRect(0, 0, canvas.width, canvas.height);

      ctx.save();
      ctx.scale(gameScale, gameScale);
      ctx.translate(-cameraX, -cameraY);

      // Arena Grid
      ctx.strokeStyle = 'rgba(255, 255, 255, 0.04)';
      ctx.lineWidth = 1;
      for (let x = 0; x < ARENA.width; x += 40) {
        ctx.beginPath(); ctx.moveTo(x, 0); ctx.lineTo(x, ARENA.height); ctx.stroke();
      }
      for (let y = 0; y < ARENA.height; y += 40) {
        ctx.beginPath(); ctx.moveTo(0, y); ctx.lineTo(ARENA.width, y); ctx.stroke();
      }

      // Weapon Spawners
      for (let pad of ARENA.weaponPads) {
        ctx.fillStyle = pad.timer <= 0 ? 'rgba(251, 191, 36, 0.2)' : 'rgba(255,255,255,0.05)';
        ctx.strokeStyle = pad.timer <= 0 ? '#fbbf24' : '#475569';
        ctx.beginPath(); ctx.arc(pad.x, pad.y, 20, 0, Math.PI*2); ctx.fill(); ctx.stroke();
        if (pad.timer <= 0) {
          ctx.fillStyle = '#fbbf24';
          ctx.font = 'bold 8px monospace';
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

      if (p1) p1.draw(ctx);
      if (p2) p2.draw(ctx);
      bullets.forEach(b => b.draw(ctx));

      floatingTexts.forEach(ft => {
        ctx.font = 'bold 11px sans-serif';
        ctx.fillStyle = ft.color;
        ctx.fillText(ft.text, ft.x - 10, ft.y);
        ft.y -= 0.5;
        ft.life--;
      });
      floatingTexts = floatingTexts.filter(ft => ft.life > 0);

      ctx.restore();

      // HUD Update
      if (p1) {
        document.getElementById('p1-hp-bar').style.width = `${(p1.hp / p1.maxHp)*100}%`;
        document.getElementById('p1-shield-bar').style.width = `${(p1.shield / p1.maxShield)*100}%`;
        document.getElementById('p1-hp-text').innerText = `${Math.ceil(p1.hp)}`;
      }
      if (p2) {
        document.getElementById('p2-hp-bar').style.width = `${(p2.hp / p2.maxHp)*100}%`;
        document.getElementById('p2-shield-bar').style.width = `${(p2.shield / p2.maxShield)*100}%`;
        document.getElementById('p2-hp-text').innerText = `${Math.ceil(p2.hp)}`;
      }

      requestAnimationFrame(gameLoop);
    }

    // Initialize
    window.addEventListener('load', () => {
      resize();
      resetRoundState();
      requestAnimationFrame(gameLoop);
    });
  </script>
</body>
</html>

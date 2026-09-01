
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <title>Annelid Assault: Unreal Edition</title>
  <!-- PeerJS WebRTC P2P Networking -->
  <script src="https://unpkg.com/peerjs@1.5.4/dist/peerjs.min.js"></script>
  <!-- MQTT.js for Serverless Global Live Chat -->
  <script src="https://unpkg.com/mqtt@5.3.5/dist/mqtt.min.js"></script>
  <style>
    :root {
      --bg-dark: #080b11;
      --panel-bg: rgba(15, 23, 42, 0.85);
      --border-color: rgba(255, 255, 255, 0.15);
      --team-red: #f43f5e;
      --team-blue: #0ea5e9;
      --accent-gold: #fbbf24;
      --accent-purple: #a855f7;
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
      background-color: #000;
      color: #fff;
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
      overflow: hidden;
      height: 100vh;
      display: flex;
      flex-direction: column;
    }

    /* Top HUD */
    #hud-top {
      position: absolute;
      top: 0;
      left: 0;
      right: 0;
      height: 52px;
      background: linear-gradient(180deg, rgba(0,0,0,0.85) 0%, rgba(0,0,0,0.4) 100%);
      border-bottom: 1px solid var(--border-color);
      display: flex;
      align-items: center;
      justify-content: space-between;
      padding: 0 16px;
      z-index: 30;
      backdrop-filter: blur(8px);
    }

    .team-badge {
      display: flex;
      align-items: center;
      gap: 8px;
      font-weight: 800;
      font-size: 0.9rem;
      text-shadow: 0 0 10px rgba(0,0,0,0.8);
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
      font-size: 1.5rem;
      font-weight: 900;
      color: var(--accent-gold);
      text-shadow: 0 0 12px rgba(251, 191, 36, 0.6);
    }
    #net-status-badge {
      font-size: 0.7rem;
      background: rgba(255,255,255,0.1);
      padding: 1px 8px;
      border-radius: 10px;
      border: 1px solid var(--border-color);
      color: #cbd5e1;
    }

    .top-actions {
      display: flex;
      align-items: center;
      gap: 8px;
    }

    .btn-icon {
      background: var(--panel-bg);
      border: 1px solid var(--border-color);
      color: #fff;
      padding: 6px 12px;
      border-radius: 8px;
      cursor: pointer;
      font-size: 0.8rem;
      font-weight: 600;
      display: flex;
      align-items: center;
      gap: 4px;
      transition: all 0.2s;
    }
    .btn-icon:hover {
      background: rgba(255,255,255,0.2);
      border-color: var(--accent-gold);
    }
    .btn-donate {
      background: linear-gradient(135deg, #f59e0b, #d97706);
      color: #000;
      font-weight: 800;
      border: none;
      box-shadow: 0 0 12px rgba(245, 158, 11, 0.4);
    }

    /* Game Viewport */
    #viewport-container {
      flex: 1;
      position: relative;
      overflow: hidden;
      background: #000;
      cursor: crosshair;
    }

    canvas {
      display: block;
      position: absolute;
      top: 0;
      left: 0;
    }

    #vignette-overlay {
      position: absolute;
      inset: 0;
      pointer-events: none;
      box-shadow: inset 0 0 100px rgba(0,0,0,0.8);
      z-index: 10;
    }

    /* Power Bar */
    #power-meter-container {
      position: absolute;
      bottom: 120px;
      left: 50%;
      transform: translateX(-50%);
      width: 240px;
      background: rgba(0, 0, 0, 0.8);
      padding: 5px;
      border-radius: 10px;
      border: 1px solid rgba(255,255,255,0.3);
      display: none;
      z-index: 25;
      box-shadow: 0 0 20px rgba(0,0,0,0.8);
    }
    #power-meter-fill {
      height: 12px;
      width: 0%;
      background: linear-gradient(90deg, #10b981, #f59e0b, #ef4444);
      border-radius: 6px;
      box-shadow: 0 0 10px rgba(239, 68, 68, 0.6);
    }

    /* Weapon Selection Bar */
    #weapon-bar {
      position: absolute;
      bottom: 60px;
      left: 50%;
      transform: translateX(-50%);
      display: flex;
      gap: 6px;
      background: var(--panel-bg);
      padding: 6px 12px;
      border-radius: 14px;
      border: 1px solid var(--border-color);
      z-index: 25;
      backdrop-filter: blur(8px);
      box-shadow: 0 10px 30px rgba(0,0,0,0.7);
      max-width: 95vw;
      overflow-x: auto;
    }
    .wpn-btn {
      background: rgba(30, 41, 59, 0.9);
      border: 1px solid rgba(255,255,255,0.15);
      color: #fff;
      padding: 6px 10px;
      border-radius: 8px;
      font-size: 0.75rem;
      font-weight: 600;
      cursor: pointer;
      display: flex;
      align-items: center;
      gap: 4px;
      white-space: nowrap;
      transition: all 0.15s;
    }
    .wpn-btn:hover { background: rgba(255,255,255,0.15); }
    .wpn-btn.active {
      background: var(--accent-gold);
      border-color: #fff;
      color: #000;
      box-shadow: 0 0 12px rgba(251, 191, 36, 0.6);
    }

    /* Touch Controls */
    #touch-controls {
      position: absolute;
      bottom: 8px;
      left: 0;
      right: 0;
      height: 48px;
      display: flex;
      justify-content: space-between;
      padding: 0 12px;
      z-index: 25;
      pointer-events: none;
    }
    .touch-btn-group { display: flex; gap: 6px; pointer-events: auto; }
    .t-btn {
      width: 44px;
      height: 44px;
      background: rgba(15, 23, 42, 0.9);
      border: 1px solid var(--border-color);
      border-radius: 10px;
      color: #fff;
      font-size: 1.1rem;
      display: flex;
      align-items: center;
      justify-content: center;
      touch-action: manipulation;
      backdrop-filter: blur(4px);
    }
    .t-btn:active { background: var(--accent-gold); color: #000; }
    .t-btn-fire { width: 80px; background: var(--team-red); font-weight: bold; font-size: 0.85rem; }

    /* =========================================================================
       GLOBAL LIVE CHAT BOX (Glassmorphic Floating Widget)
       ========================================================================= */
    #chat-toggle-btn {
      position: absolute;
      bottom: 64px;
      left: 14px;
      z-index: 35;
      background: var(--panel-bg);
      border: 1px solid var(--border-color);
      color: #fff;
      padding: 6px 12px;
      border-radius: 20px;
      font-size: 0.8rem;
      font-weight: 700;
      cursor: pointer;
      display: flex;
      align-items: center;
      gap: 6px;
      backdrop-filter: blur(6px);
      box-shadow: 0 4px 15px rgba(0,0,0,0.5);
      transition: all 0.2s;
    }
    #chat-toggle-btn:hover {
      background: rgba(255, 255, 255, 0.2);
      border-color: var(--accent-gold);
    }
    #chat-unread-badge {
      background: var(--team-red);
      color: #fff;
      font-size: 0.7rem;
      padding: 1px 6px;
      border-radius: 10px;
      display: none;
    }

    #global-chat-container {
      position: absolute;
      bottom: 110px;
      left: 14px;
      width: 320px;
      max-width: calc(100vw - 28px);
      height: 240px;
      background: rgba(15, 23, 42, 0.92);
      border: 1px solid var(--border-color);
      border-radius: 12px;
      z-index: 35;
      display: none;
      flex-direction: column;
      backdrop-filter: blur(10px);
      box-shadow: 0 10px 30px rgba(0,0,0,0.8);
      overflow: hidden;
    }
    #chat-header {
      background: rgba(0, 0, 0, 0.4);
      padding: 8px 12px;
      font-size: 0.8rem;
      font-weight: bold;
      color: var(--accent-gold);
      display: flex;
      justify-content: space-between;
      align-items: center;
      border-bottom: 1px solid rgba(255,255,255,0.08);
    }
    #chat-messages {
      flex: 1;
      padding: 8px 12px;
      overflow-y: auto;
      font-size: 0.8rem;
      display: flex;
      flex-direction: column;
      gap: 6px;
      scrollbar-width: thin;
    }
    .chat-msg {
      line-height: 1.3;
      word-break: break-word;
    }
    .chat-author {
      font-weight: bold;
      color: var(--team-blue);
    }
    .chat-author.red { color: var(--team-red); }
    .chat-author.gold { color: var(--accent-gold); }
    .chat-time {
      font-size: 0.65rem;
      color: #64748b;
      margin-right: 4px;
    }

    #chat-input-bar {
      display: flex;
      padding: 6px;
      gap: 6px;
      background: rgba(0,0,0,0.3);
      border-top: 1px solid rgba(255,255,255,0.08);
    }
    #chat-input {
      flex: 1;
      background: rgba(15, 23, 42, 0.8);
      border: 1px solid #334155;
      color: #fff;
      padding: 6px 10px;
      border-radius: 6px;
      font-size: 0.8rem;
      outline: none;
    }
    #chat-input:focus { border-color: var(--accent-gold); }
    #chat-send-btn {
      background: var(--team-blue);
      border: none;
      color: #fff;
      padding: 6px 10px;
      border-radius: 6px;
      font-size: 0.8rem;
      font-weight: bold;
      cursor: pointer;
    }

    /* Modals */
    .modal-overlay {
      position: absolute;
      inset: 0;
      background: rgba(8, 11, 17, 0.92);
      display: flex;
      align-items: center;
      justify-content: center;
      z-index: 100;
      backdrop-filter: blur(12px);
    }
    .modal-card {
      background: #0f172a;
      border: 1px solid rgba(255,255,255,0.2);
      border-radius: 16px;
      padding: 24px;
      text-align: center;
      max-width: 420px;
      width: 90%;
      display: flex;
      flex-direction: column;
      gap: 12px;
      box-shadow: 0 20px 50px rgba(0,0,0,0.8);
    }
    .modal-card h2 {
      font-size: 1.6rem;
      background: linear-gradient(135deg, #fbbf24, #f59e0b);
      -webkit-background-clip: text;
      -webkit-text-fill-color: transparent;
      font-weight: 900;
    }
    .btn-main {
      background: var(--team-blue);
      color: #fff;
      border: none;
      padding: 12px 16px;
      border-radius: 8px;
      font-weight: bold;
      font-size: 0.95rem;
      cursor: pointer;
      display: flex;
      align-items: center;
      justify-content: center;
      gap: 8px;
      transition: opacity 0.2s;
    }
    .btn-main:hover { opacity: 0.9; }
    .btn-danger { background: var(--team-red); }
    
    .select-input {
      background: #1e293b;
      border: 1px solid #475569;
      color: #fff;
      padding: 10px;
      border-radius: 8px;
      font-size: 0.9rem;
      font-weight: bold;
      outline: none;
    }
    .input-code {
      background: #0b1120;
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
    .input-code:focus { border-color: var(--accent-gold); }

    .donate-grid {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 10px;
      margin: 10px 0;
    }
    .donate-card {
      background: #1e293b;
      border: 1px solid #334155;
      padding: 14px;
      border-radius: 10px;
      text-decoration: none;
      color: #fff;
      font-size: 0.85rem;
      font-weight: bold;
      display: flex;
      flex-direction: column;
      align-items: center;
      gap: 6px;
      transition: all 0.2s;
    }
    .donate-card:hover {
      border-color: var(--accent-gold);
      background: #27354f;
      transform: translateY(-2px);
    }
  </style>
</head>
<body>

  <!-- Top HUD -->
  <header id="hud-top">
    <div class="team-badge red-team">
      <span>🔴 <span id="red-health-summary">100 HP</span></span>
    </div>
    
    <div class="center-hud">
      <div id="turn-timer">30</div>
      <div id="net-status-badge">Local Match</div>
    </div>

    <div class="team-badge blue-team">
      <span>🔵 <span id="blue-health-summary">100 HP</span></span>
    </div>

    <div class="top-actions">
      <button class="btn-icon btn-donate" onclick="openDonateModal()">☕ Support Dev</button>
      <button class="btn-icon" onclick="openSettingsModal()">⚙️</button>
    </div>
  </header>

  <!-- Viewport -->
  <div id="viewport-container">
    <canvas id="sky-canvas"></canvas>
    <canvas id="terrain-canvas"></canvas>
    <canvas id="game-canvas"></canvas>
    <canvas id="light-canvas"></canvas>
    <div id="vignette-overlay"></div>

    <!-- Power Bar -->
    <div id="power-meter-container">
      <div id="power-meter-fill"></div>
    </div>

    <!-- Weapon Radial Dock -->
    <div id="weapon-bar">
      <button class="wpn-btn active" onclick="selectWeapon('bazooka')">🚀 Bazooka</button>
      <button class="wpn-btn" onclick="selectWeapon('grenade')">💣 Grenade</button>
      <button class="wpn-btn" onclick="selectWeapon('cluster')">💥 Cluster Bomb</button>
      <button class="wpn-btn" onclick="selectWeapon('holy')">🕊️ Holy Grenade</button>
      <button class="wpn-btn" onclick="selectWeapon('airstrike')">✈️ Air Strike</button>
      <button class="wpn-btn" onclick="selectWeapon('railgun')">⚡ Railgun</button>
      <button class="wpn-btn" onclick="selectWeapon('dynamite')">🧨 Dynamite</button>
      <button class="wpn-btn" onclick="selectWeapon('jetpack')">🎒 Jetpack</button>
    </div>

    <!-- Global Live Chat Toggle Button -->
    <button id="chat-toggle-btn" onclick="toggleChat()">
      💬 Global Chat <span id="chat-unread-badge">0</span>
    </button>

    <!-- Global Chat Floating Box -->
    <div id="global-chat-container">
      <div id="chat-header">
        <span>🌐 Global Live Lobby Chat</span>
        <button style="background:none; border:none; color:#cbd5e1; cursor:pointer; font-weight:bold;" onclick="toggleChat()">✕</button>
      </div>
      <div id="chat-messages">
        <div class="chat-msg" style="color:#64748b; font-style:italic;">Connecting to global chat server...</div>
      </div>
      <div id="chat-input-bar">
        <input type="text" id="chat-input" placeholder="Type message..." maxlength="120" onkeydown="if(event.key==='Enter') sendChatMessage()">
        <button id="chat-send-btn" onclick="sendChatMessage()">Send</button>
      </div>
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

  <!-- Main Menu / Match Lobby -->
  <div id="lobby-modal" class="modal-overlay">
    <div class="modal-card">
      <h2>ANNELID ASSAULT</h2>
      <p style="font-size:0.85rem; color:#94a3b8;">Unreal Edition • 2D Artillery Tactics</p>
      
      <div style="display:flex; flex-direction:column; gap:6px; text-align:left;">
        <label style="font-size:0.75rem; color:#94a3b8; font-weight:bold;">CHOOSE MAP BIOME:</label>
        <select id="biome-selector" class="select-input">
          <option value="volcano">🌋 Volcanic Hellscape (Lava Sea + Embers)</option>
          <option value="moon">🌙 Lunar Outpost (Low-G + Craters)</option>
          <option value="toxic">☣️ Toxic Wasteland (Acid + Fog)</option>
          <option value="snow">❄️ Arctic Tundra (Ice + Blizzard)</option>
          <option value="classic" selected>🌿 Emerald Highlands (Classic)</option>
        </select>
      </div>

      <button class="btn-main" onclick="startLocalGame()">🎮 Local Pass & Play</button>
      <button class="btn-main" style="background:var(--accent-gold); color:#000;" onclick="showHostUI()">🌐 Host Online Match</button>
      <button class="btn-main" style="background:var(--accent-purple);" onclick="showJoinUI()">🔗 Join Online Match</button>

      <!-- Host Panel -->
      <div id="host-panel" style="display:none; flex-direction:column; gap:10px; margin-top:8px;">
        <div style="font-size:0.8rem; color:#94a3b8;">Share this 4-Digit Room Code:</div>
        <div id="host-room-code" style="font-size:2rem; font-weight:bold; color:var(--accent-gold); font-family:var(--font-code);">----</div>
        <div style="font-size:0.75rem; color:#cbd5e1;" id="host-status-text">Creating peer connection...</div>
        <button class="btn-main btn-danger" onclick="cancelLobby()">Cancel</button>
      </div>

      <!-- Join Panel -->
      <div id="join-panel" style="display:none; flex-direction:column; gap:10px; margin-top:8px;">
        <div style="font-size:0.8rem; color:#94a3b8;">Enter Host 4-Digit Code:</div>
        <input type="text" id="join-code-input" class="input-code" maxlength="4" placeholder="1234">
        <button class="btn-main" onclick="connectToHost()">Connect & Battle</button>
        <button class="btn-main btn-danger" onclick="cancelLobby()">Cancel</button>
      </div>
    </div>
  </div>

  <!-- Donation Modal -->
  <div id="donate-modal" class="modal-overlay" style="display:none;">
    <div class="modal-card">
      <h2 style="color:var(--accent-gold);">☕ SUPPORT THE CREATOR</h2>
      <p style="font-size:0.85rem; color:#cbd5e1;">Enjoying Annelid Assault? Support independent game development with a coffee or tip!</p>
      
      <div class="donate-grid">
        <a href="https://ko-fi.com" target="_blank" class="donate-card">
          <span style="font-size:1.4rem;">☕</span>
          <span>Tip via Ko-fi</span>
        </a>
        <a href="https://paypal.com" target="_blank" class="donate-card">
          <span style="font-size:1.4rem;">💳</span>
          <span>Tip via PayPal</span>
        </a>
      </div>

      <p style="font-size:0.75rem; color:#64748b;">(Replace these links in index.html with your custom creator URL!)</p>
      <button class="btn-main" onclick="closeDonateModal()">Close</button>
    </div>
  </div>

  <!-- Match Over Modal -->
  <div id="gameover-modal" class="modal-overlay" style="display:none;">
    <div class="modal-card">
      <h2 id="winner-title">TEAM RED WINS!</h2>
      <p style="color:#cbd5e1; font-size:0.85rem;" id="winner-desc">All opposing annelids eliminated.</p>
      <button class="btn-main" onclick="restartMatch()">Play Next Round</button>
    </div>
  </div>

  <script>
    /* =========================================================================
       AUDIO SYNTHESIZER ENGINE (Web Audio API)
       ========================================================================= */
    const AudioEngine = {
      ctx: null,
      init() {
        if (!this.ctx) {
          const AudioContext = window.AudioContext || window.webkitAudioContext;
          this.ctx = new AudioContext();
        }
      },
      play(type) {
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
          osc.frequency.exponentialRampToValueAtTime(550, t + 0.16);
          gain.gain.setValueAtTime(0.3, t);
          gain.gain.linearRampToValueAtTime(0.01, t + 0.16);
          osc.start(t); osc.stop(t + 0.16);
        } else if (type === 'explode') {
          const bufferSize = this.ctx.sampleRate * 0.45;
          const buffer = this.ctx.createBuffer(1, bufferSize, this.ctx.sampleRate);
          const data = buffer.getChannelData(0);
          for (let i = 0; i < bufferSize; i++) data[i] = Math.random() * 2 - 1;
          const noise = this.ctx.createBufferSource();
          noise.buffer = buffer;
          const filter = this.ctx.createBiquadFilter();
          filter.type = 'lowpass';
          filter.frequency.setValueAtTime(360, t);
          filter.frequency.linearRampToValueAtTime(30, t + 0.45);
          noise.connect(filter);
          filter.connect(gain);
          gain.gain.setValueAtTime(0.8, t);
          gain.gain.linearRampToValueAtTime(0.01, t + 0.45);
          noise.start(t); noise.stop(t + 0.45);
        } else if (type === 'holy') {
          [523.25, 659.25, 783.99, 1046.50].forEach(freq => {
            const hOsc = this.ctx.createOscillator();
            const hGain = this.ctx.createGain();
            hOsc.type = 'sine';
            hOsc.frequency.setValueAtTime(freq, t);
            hGain.gain.setValueAtTime(0.15, t);
            hGain.gain.linearRampToValueAtTime(0.01, t + 1.2);
            hOsc.connect(hGain);
            hGain.connect(this.ctx.destination);
            hOsc.start(t); hOsc.stop(t + 1.2);
          });
        } else if (type === 'railgun') {
          osc.type = 'sawtooth';
          osc.frequency.setValueAtTime(900, t);
          osc.frequency.exponentialRampToValueAtTime(80, t + 0.22);
          gain.gain.setValueAtTime(0.4, t);
          gain.gain.linearRampToValueAtTime(0.01, t + 0.22);
          osc.start(t); osc.stop(t + 0.22);
        } else if (type === 'jetpack') {
          osc.type = 'triangle';
          osc.frequency.setValueAtTime(90 + Math.random()*40, t);
          gain.gain.setValueAtTime(0.15, t);
          gain.gain.linearRampToValueAtTime(0.01, t + 0.08);
          osc.start(t); osc.stop(t + 0.08);
        } else if (type === 'jet_flyby') {
          osc.type = 'sawtooth';
          osc.frequency.setValueAtTime(200, t);
          osc.frequency.linearRampToValueAtTime(450, t + 0.5);
          osc.frequency.linearRampToValueAtTime(150, t + 1.0);
          gain.gain.setValueAtTime(0.3, t);
          gain.gain.linearRampToValueAtTime(0.01, t + 1.0);
          osc.start(t); osc.stop(t + 1.0);
        } else if (type === 'jump') {
          osc.type = 'sine';
          osc.frequency.setValueAtTime(180, t);
          osc.frequency.exponentialRampToValueAtTime(380, t + 0.12);
          gain.gain.setValueAtTime(0.25, t);
          gain.gain.linearRampToValueAtTime(0.01, t + 0.12);
          osc.start(t); osc.stop(t + 0.12);
        } else if (type === 'splash') {
          osc.type = 'triangle';
          osc.frequency.setValueAtTime(280, t);
          osc.frequency.linearRampToValueAtTime(80, t + 0.25);
          gain.gain.setValueAtTime(0.4, t);
          gain.gain.linearRampToValueAtTime(0.01, t + 0.25);
          osc.start(t); osc.stop(t + 0.25);
        } else if (type === 'chat_ping') {
          osc.type = 'sine';
          osc.frequency.setValueAtTime(440, t);
          osc.frequency.setValueAtTime(880, t + 0.06);
          gain.gain.setValueAtTime(0.12, t);
          gain.gain.linearRampToValueAtTime(0.01, t + 0.12);
          osc.start(t); osc.stop(t + 0.12);
        }
      }
    };

    /* =========================================================================
       GLOBAL LIVE CHAT SYSTEM (MQTT over WebSockets)
       ========================================================================= */
    const CHAT_TOPIC = 'annelid-assault-live-chat-global/lobby';
    let mqttClient = null;
    let chatUserHandle = 'Worm_' + Math.floor(100 + Math.random() * 900);
    let isChatOpen = false;
    let unreadCount = 0;

    function initGlobalChat() {
      // Connect to public MQTT WebSocket broker
      mqttClient = mqtt.connect('wss://broker.emqx.io:8084/mqtt', {
        clientId: 'client_' + Math.random().toString(16).substr(2, 8),
        keepalive: 60
      });

      mqttClient.on('connect', () => {
        mqttClient.subscribe(CHAT_TOPIC);
        const msgArea = document.getElementById('chat-messages');
        msgArea.innerHTML = '<div class="chat-msg" style="color:#22c55e;">● Connected to Global Live Chat!</div>';
      });

      mqttClient.on('message', (topic, message) => {
        try {
          const payload = JSON.parse(message.toString());
          renderIncomingMessage(payload);
        } catch (e) {}
      });
    }

    function toggleChat() {
      isChatOpen = !isChatOpen;
      const box = document.getElementById('global-chat-container');
      box.style.display = isChatOpen ? 'flex' : 'none';
      if (isChatOpen) {
        unreadCount = 0;
        document.getElementById('chat-unread-badge').style.display = 'none';
        document.getElementById('chat-input').focus();
      }
    }

    function sendChatMessage() {
      const input = document.getElementById('chat-input');
      const text = input.value.trim();
      if (!text || !mqttClient) return;

      const payload = {
        user: chatUserHandle,
        team: myTeam,
        text: text,
        time: new Date().toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' })
      };

      mqttClient.publish(CHAT_TOPIC, JSON.stringify(payload));
      input.value = '';
    }

    function renderIncomingMessage(data) {
      const msgArea = document.getElementById('chat-messages');
      const row = document.createElement('div');
      row.className = 'chat-msg';

      const authorColor = data.team === 'red' ? 'red' : data.team === 'blue' ? '' : 'gold';
      const safeUser = escapeHtml(data.user);
      const safeText = escapeHtml(data.text);

      row.innerHTML = `<span class="chat-time">${data.time}</span><span class="chat-author ${authorColor}">[${safeUser}]:</span> ${safeText}`;
      msgArea.appendChild(row);
      msgArea.scrollTop = msgArea.scrollHeight;

      if (!isChatOpen) {
        unreadCount++;
        const badge = document.getElementById('chat-unread-badge');
        badge.innerText = unreadCount;
        badge.style.display = 'inline-block';
        AudioEngine.play('chat_ping');
      }
    }

    function escapeHtml(str) {
      return str.replace(/&/g, '&amp;').replace(/</g, '&lt;').replace(/>/g, '&gt;');
    }

    /* =========================================================================
       MAP BIOMES & ENGINE CONSTANTS
       ========================================================================= */
    const BIOMES = {
      classic: {
        name: 'Emerald Highlands',
        gravity: 0.28,
        friction: 0.85,
        waterColor: 'rgba(14, 165, 233, 0.65)',
        waterType: 'water',
        soil: '#573318',
        grass: '#22c55e',
        sky: ['#0c1938', '#1e3a5f', '#0284c7']
      },
      volcano: {
        name: 'Volcanic Hellscape',
        gravity: 0.30,
        friction: 0.90,
        waterColor: 'rgba(239, 68, 68, 0.85)',
        waterType: 'lava',
        soil: '#261c1a',
        grass: '#ea580c',
        sky: ['#180505', '#450a0a', '#b91c1c']
      },
      moon: {
        name: 'Lunar Outpost',
        gravity: 0.14,
        friction: 0.92,
        waterColor: 'rgba(168, 85, 247, 0.5)',
        waterType: 'gravity_sludge',
        soil: '#334155',
        grass: '#94a3b8',
        sky: ['#030712', '#0f172a', '#3b0764']
      },
      toxic: {
        name: 'Toxic Wasteland',
        gravity: 0.28,
        friction: 0.82,
        waterColor: 'rgba(132, 204, 22, 0.85)',
        waterType: 'acid',
        soil: '#1c1917',
        grass: '#84cc16',
        sky: ['#052e16', '#14532d', '#15803d']
      },
      snow: {
        name: 'Arctic Tundra',
        gravity: 0.28,
        friction: 0.96,
        waterColor: 'rgba(56, 189, 248, 0.7)',
        waterType: 'freezing_water',
        soil: '#475569',
        grass: '#f8fafc',
        sky: ['#082f49', '#0369a1', '#e0f2fe']
      }
    };

    let currentBiome = BIOMES.classic;
    const WORLD_WIDTH = 2200;
    const WORLD_HEIGHT = 1100;
    const WATER_LEVEL = 1000;

    const skyCanvas = document.getElementById('sky-canvas');
    const skyCtx = skyCanvas.getContext('2d');
    const terrainCanvas = document.getElementById('terrain-canvas');
    const terrainCtx = terrainCanvas.getContext('2d');
    const gameCanvas = document.getElementById('game-canvas');
    const gameCtx = gameCanvas.getContext('2d');
    const lightCanvas = document.getElementById('light-canvas');
    const lightCtx = lightCanvas.getContext('2d');

    let viewWidth = window.innerWidth;
    let viewHeight = window.innerHeight;
    let cameraX = 0, cameraY = 0;
    let screenShake = 0;

    let terrainData = new Uint8Array(WORLD_WIDTH * WORLD_HEIGHT);

    /* =========================================================================
       MULTIPLAYER ENGINE (PeerJS)
       ========================================================================= */
    let peer = null, netConn = null, isMultiplayer = false, isHost = false, myTeam = 'red';

    function showHostUI() {
      document.querySelectorAll('#lobby-modal .btn-main').forEach(b => b.style.display = 'none');
      document.getElementById('host-panel').style.display = 'flex';
      const code = Math.floor(1000 + Math.random() * 9000).toString();
      document.getElementById('host-room-code').innerText = code;
      peer = new Peer('annelid-room-' + code);
      isHost = true; myTeam = 'red';

      peer.on('open', () => document.getElementById('host-status-text').innerText = 'Room open! Waiting for guest...');
      peer.on('connection', conn => {
        netConn = conn;
        setupNetEvents();
        document.getElementById('host-status-text').innerText = 'Connected! Starting...';
        setTimeout(() => {
          document.getElementById('lobby-modal').style.display = 'none';
          document.getElementById('net-status-badge').innerText = 'Online: Host (🔴)';
          startMatch(true);
        }, 1000);
      });
    }

    function showJoinUI() {
      document.querySelectorAll('#lobby-modal .btn-main').forEach(b => b.style.display = 'none');
      document.getElementById('join-panel').style.display = 'flex';
      isHost = false; myTeam = 'blue';
    }

    function connectToHost() {
      const code = document.getElementById('join-code-input').value.trim();
      if (code.length !== 4) return alert('Enter valid 4-digit code!');
      peer = new Peer();
      peer.on('open', () => {
        netConn = peer.connect('annelid-room-' + code);
        setupNetEvents();
        netConn.on('open', () => {
          document.getElementById('lobby-modal').style.display = 'none';
          document.getElementById('net-status-badge').innerText = 'Online: Guest (🔵)';
          startMatch(false);
        });
      });
    }

    function setupNetEvents() {
      isMultiplayer = true;
      netConn.on('data', pkg => handleNetworkPacket(pkg));
      netConn.on('close', () => { alert('Opponent disconnected!'); location.reload(); });
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

    function startLocalGame() {
      isMultiplayer = false;
      document.getElementById('lobby-modal').style.display = 'none';
      document.getElementById('net-status-badge').innerText = 'Local Match';
      startMatch(true);
    }

    /* =========================================================================
       PROCEDURAL MAP GENERATOR
       ========================================================================= */
    function generateTerrain(seed = Math.random(), biomeKey = 'classic') {
      currentBiome = BIOMES[biomeKey] || BIOMES.classic;
      terrainCanvas.width = WORLD_WIDTH;
      terrainCanvas.height = WORLD_HEIGHT;
      [skyCanvas, gameCanvas, lightCanvas].forEach(c => { c.width = viewWidth; c.height = viewHeight; });

      terrainCtx.clearRect(0, 0, WORLD_WIDTH, WORLD_HEIGHT);
      terrainData.fill(0);

      const heights = new Float32Array(WORLD_WIDTH);
      for (let x = 0; x < WORLD_WIDTH; x++) {
        const nx = x / WORLD_WIDTH;
        let h = Math.sin(nx * 5.5 + seed * 9) * 150 +
                Math.sin(nx * 13 + seed * 4) * 85 +
                Math.sin(nx * 26) * 35 + 600;
        
        if (x < 160) h += (160 - x) * 3.5;
        if (x > WORLD_WIDTH - 160) h += (x - (WORLD_WIDTH - 160)) * 3.5;
        heights[x] = Math.min(WATER_LEVEL - 50, h);
      }

      // Soil Layer
      terrainCtx.fillStyle = currentBiome.soil;
      terrainCtx.beginPath();
      terrainCtx.moveTo(0, WORLD_HEIGHT);
      terrainCtx.lineTo(0, heights[0]);
      for (let x = 1; x < WORLD_WIDTH; x++) terrainCtx.lineTo(x, heights[x]);
      terrainCtx.lineTo(WORLD_WIDTH, WORLD_HEIGHT);
      terrainCtx.closePath();
      terrainCtx.fill();

      // Top Edge Grass
      terrainCtx.strokeStyle = currentBiome.grass;
      terrainCtx.lineWidth = 9;
      terrainCtx.beginPath();
      terrainCtx.moveTo(0, heights[0]);
      for (let x = 1; x < WORLD_WIDTH; x++) terrainCtx.lineTo(x, heights[x]);
      terrainCtx.stroke();

      // Populate Bitmask Grid
      const imgData = terrainCtx.getImageData(0, 0, WORLD_WIDTH, WORLD_HEIGHT).data;
      for (let y = 0; y < WORLD_HEIGHT; y++) {
        for (let x = 0; x < WORLD_WIDTH; x++) {
          if (imgData[(y * WORLD_WIDTH + x) * 4 + 3] > 64) {
            terrainData[y * WORLD_WIDTH + x] = 1;
          }
        }
      }

      drawSky();
    }

    function drawSky() {
      skyCtx.clearRect(0, 0, viewWidth, viewHeight);
      const grad = skyCtx.createLinearGradient(0, 0, 0, viewHeight);
      grad.addColorStop(0, currentBiome.sky[0]);
      grad.addColorStop(0.6, currentBiome.sky[1]);
      grad.addColorStop(1, currentBiome.sky[2]);
      skyCtx.fillStyle = grad;
      skyCtx.fillRect(0, 0, viewWidth, viewHeight);
    }

    function isTerrainSolid(x, y) {
      const ix = Math.floor(x), iy = Math.floor(y);
      if (ix < 0 || ix >= WORLD_WIDTH || iy >= WORLD_HEIGHT || iy < 0) return false;
      return terrainData[iy * WORLD_WIDTH + ix] === 1;
    }

    function destroyTerrainCircle(cx, cy, radius) {
      terrainCtx.save();
      terrainCtx.globalCompositeOperation = 'destination-out';
      terrainCtx.beginPath();
      terrainCtx.arc(cx, cy, radius, 0, Math.PI * 2);
      terrainCtx.fill();
      terrainCtx.restore();

      const r2 = radius * radius;
      const minX = Math.max(0, Math.floor(cx - radius));
      const maxX = Math.min(WORLD_WIDTH - 1, Math.ceil(cx + radius));
      const minY = Math.max(0, Math.floor(cy - radius));
      const maxY = Math.min(WORLD_HEIGHT - 1, Math.ceil(cy + radius));

      for (let y = minY; y <= maxY; y++) {
        for (let x = minX; x <= maxX; x++) {
          if ((x - cx)**2 + (y - cy)**2 <= r2) {
            terrainData[y * WORLD_WIDTH + x] = 0;
          }
        }
      }
    }

    /* =========================================================================
       WORM ENTITIES & SQUADS
       ========================================================================= */
    class Worm {
      constructor(id, name, team, x, y) {
        this.id = id; this.name = name; this.team = team;
        this.x = x; this.y = y; this.vx = 0; this.vy = 0;
        this.hp = 100; this.maxHp = 100; this.radius = 9;
        this.facingRight = team === 'red';
        this.aimAngle = team === 'red' ? -0.5 : -2.6;
        this.isGrounded = false; this.isDead = false;
        this.fuel = 100;
      }

      update() {
        if (this.isDead) return;

        this.vy += currentBiome.gravity;
        this.vx *= currentBiome.friction;

        let nextX = this.x + this.vx;
        let nextY = this.y + this.vy;

        if (nextY >= WATER_LEVEL) {
          this.hp = 0;
          this.isDead = true;
          AudioEngine.play('splash');
          createWaterSplash(this.x, WATER_LEVEL);
          addFloatingText(this.x, this.y - 15, currentBiome.waterType === 'lava' ? 'INCINERATED!' : 'DROWNED!', '#f43f5e');
          return;
        }

        if (this.vy > 0) {
          if (isTerrainSolid(this.x, nextY + this.radius)) {
            if (this.vy > 8.5) {
              const fallDmg = Math.floor((this.vy - 7) * 9);
              this.hp = Math.max(0, this.hp - fallDmg);
              addScreenShake(fallDmg * 0.4);
              AudioEngine.play('explode');
              addFloatingText(this.x, this.y - 15, `-${fallDmg}`, '#ef4444');
            }
            this.vy = 0;
            this.isGrounded = true;
            this.fuel = 100;
            while (isTerrainSolid(this.x, nextY + this.radius) && nextY > 0) nextY--;
          } else {
            this.isGrounded = false;
          }
        }

        if (Math.abs(this.vx) > 0.05) {
          const sign = Math.sign(this.vx);
          if (isTerrainSolid(nextX + sign * this.radius, nextY)) {
            let climb = 0;
            while (climb < 6 && isTerrainSolid(nextX + sign * this.radius, nextY - climb)) climb++;
            if (climb < 6) nextY -= climb;
            else { this.vx = 0; nextX = this.x; }
          }
        }

        this.x = Math.max(this.radius, Math.min(WORLD_WIDTH - this.radius, nextX));
        this.y = nextY;

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
        this.vy = -6.0;
        this.vx = (this.facingRight ? 1 : -1) * 2.8;
        this.isGrounded = false;
        AudioEngine.play('jump');
      }

      flyJetpack() {
        if (this.fuel <= 0 || (turnPhase !== 'ACTION' && turnPhase !== 'RETREAT')) return;
        this.vy = Math.max(-5.0, this.vy - 0.7);
        this.fuel -= 1.2;
        this.isGrounded = false;
        AudioEngine.play('jetpack');
        particles.push(new Particle(this.x + (this.facingRight ? -8 : 8), this.y + 4, (Math.random()-0.5)*2, Math.random()*3 + 1, '#f59e0b', 16));
      }

      draw(ctx) {
        if (this.isDead) {
          ctx.fillStyle = '#64748b';
          ctx.fillRect(this.x - 6, this.y - 14, 12, 16);
          ctx.beginPath(); ctx.arc(this.x, this.y - 14, 6, Math.PI, 0); ctx.fill();
          return;
        }

        ctx.save();
        ctx.translate(this.x, this.y);

        ctx.font = 'bold 10px sans-serif';
        ctx.textAlign = 'center';
        ctx.fillStyle = '#fff';
        ctx.fillText(this.name, 0, -22);

        ctx.fillStyle = 'rgba(0,0,0,0.7)';
        ctx.fillRect(-15, -18, 30, 4);
        ctx.fillStyle = this.team === 'red' ? '#f43f5e' : '#0ea5e9';
        ctx.fillRect(-15, -18, 30 * (this.hp / this.maxHp), 4);

        if (this === activeWorm && (turnPhase === 'ACTION' || turnPhase === 'RETREAT')) {
          const bob = Math.sin(Date.now() * 0.008) * 4;
          ctx.fillStyle = this.team === 'red' ? '#f43f5e' : '#0ea5e9';
          ctx.beginPath();
          ctx.moveTo(0, -28 + bob);
          ctx.lineTo(-6, -38 + bob);
          ctx.lineTo(6, -38 + bob);
          ctx.fill();
        }

        const grad = ctx.createRadialGradient(0, 0, 2, 0, 0, 9);
        grad.addColorStop(0, '#fda4af');
        grad.addColorStop(1, '#f43f5e');
        ctx.fillStyle = grad;
        ctx.beginPath();
        ctx.arc(0, 0, 9, 0, Math.PI * 2);
        ctx.arc(this.facingRight ? -5 : 5, 3, 7, 0, Math.PI * 2);
        ctx.fill();

        ctx.fillStyle = this.team === 'red' ? '#f43f5e' : '#0ea5e9';
        ctx.fillRect(-7, -8, 14, 3);

        if (this === activeWorm && turnPhase === 'ACTION' && selectedWeapon !== 'airstrike') {
          const aimX = Math.cos(this.aimAngle) * 24;
          const aimY = Math.sin(this.aimAngle) * 24;
          ctx.strokeStyle = '#fbbf24';
          ctx.lineWidth = 2;
          ctx.beginPath(); ctx.moveTo(0, 0); ctx.lineTo(aimX, aimY); ctx.stroke();
          ctx.fillStyle = '#ef4444';
          ctx.beginPath(); ctx.arc(aimX, aimY, 3.5, 0, Math.PI * 2); ctx.fill();
        }

        ctx.restore();
      }
    }

    /* =========================================================================
       PROJECTILES & AIR STRIKE
       ========================================================================= */
    class Projectile {
      constructor(type, x, y, vx, vy) {
        this.type = type;
        this.x = x; this.y = y; this.vx = vx; this.vy = vy;
        this.radius = 5;
        this.timer = type === 'grenade' ? 3.0 : type === 'holy' ? 2.8 : type === 'dynamite' ? 4.0 : type === 'cluster' ? 2.5 : 99;
        this.alive = true;
      }

      update(dt) {
        if (!this.alive) return;
        this.vx += (this.type === 'bazooka' ? wind * 0.018 : 0);
        this.vy += currentBiome.gravity * (this.type === 'bazooka' ? 0.8 : 1);

        if (this.timer < 90) {
          this.timer -= dt;
          if (this.timer <= 0) { this.detonate(); return; }
        }

        const steps = 4;
        const dx = this.vx / steps;
        const dy = this.vy / steps;

        for (let s = 0; s < steps; s++) {
          this.x += dx;
          this.y += dy;

          if (this.y >= WATER_LEVEL) {
            this.alive = false;
            AudioEngine.play('splash');
            createWaterSplash(this.x, WATER_LEVEL);
            break;
          }

          if (isTerrainSolid(this.x, this.y)) {
            if (['bazooka', 'cluster_shard', 'bomb'].includes(this.type)) {
              this.detonate();
              break;
            } else {
              this.vx *= -0.55; this.vy *= -0.45;
              this.x -= dx * 2; this.y -= dy * 2;
              break;
            }
          }

          if (this.type === 'bazooka' || this.type === 'bomb') {
            for (let w of worms) {
              if (!w.isDead && Math.hypot(this.x - w.x, this.y - w.y) < w.radius + this.radius) {
                this.detonate();
                return;
              }
            }
          }
        }
      }

      detonate() {
        this.alive = false;
        if (this.type === 'bazooka') createExplosion(this.x, this.y, 48, 55);
        else if (this.type === 'grenade') createExplosion(this.x, this.y, 55, 60);
        else if (this.type === 'dynamite') createExplosion(this.x, this.y, 80, 90);
        else if (this.type === 'bomb') createExplosion(this.x, this.y, 50, 60);
        else if (this.type === 'holy') {
          AudioEngine.play('holy');
          createExplosion(this.x, this.y, 110, 100);
        } else if (this.type === 'cluster') {
          createExplosion(this.x, this.y, 35, 30);
          for (let i = 0; i < 5; i++) {
            const ang = -Math.PI * 0.8 + i * 0.4;
            projectiles.push(new Projectile('cluster_shard', this.x, this.y - 4, Math.cos(ang)*4.5, Math.sin(ang)*4.5));
          }
        } else if (this.type === 'cluster_shard') createExplosion(this.x, this.y, 25, 25);
      }

      draw(ctx) {
        if (!this.alive) return;
        ctx.save();
        ctx.translate(this.x, this.y);

        if (this.type === 'bazooka' || this.type === 'bomb') {
          ctx.rotate(Math.atan2(this.vy, this.vx));
          ctx.fillStyle = '#64748b'; ctx.fillRect(-6, -3, 12, 6);
          ctx.fillStyle = '#ef4444'; ctx.beginPath(); ctx.moveTo(6, -4); ctx.lineTo(11, 0); ctx.lineTo(6, 4); ctx.fill();
        } else if (this.type === 'holy') {
          ctx.fillStyle = '#fbbf24'; ctx.beginPath(); ctx.arc(0, 0, 7, 0, Math.PI * 2); ctx.fill();
          ctx.fillStyle = '#fff'; ctx.fillRect(-2, -10, 4, 6); ctx.fillRect(-4, -8, 8, 2);
        } else {
          ctx.fillStyle = this.type === 'dynamite' ? '#dc2626' : '#16a34a';
          ctx.beginPath(); ctx.arc(0, 0, 5, 0, Math.PI * 2); ctx.fill();
        }

        ctx.restore();
      }
    }

    class AirStrikeJet {
      constructor(targetX) {
        this.targetX = targetX;
        this.x = targetX - 1000;
        this.y = 80;
        this.speed = 18;
        this.dropped = 0;
        this.alive = true;
        AudioEngine.play('jet_flyby');
      }

      update() {
        this.x += this.speed;
        if (this.dropped < 4 && Math.abs(this.x - (this.targetX - 120 + this.dropped * 80)) < 25) {
          projectiles.push(new Projectile('bomb', this.x, this.y + 10, 2, 4));
          this.dropped++;
        }
        if (this.x > this.targetX + 1000) this.alive = false;
      }

      draw(ctx) {
        ctx.save();
        ctx.translate(this.x, this.y);
        ctx.fillStyle = '#475569';
        ctx.beginPath();
        ctx.moveTo(30, 0); ctx.lineTo(-20, -12); ctx.lineTo(-10, 0); ctx.lineTo(-20, 12);
        ctx.closePath();
        ctx.fill();
        ctx.restore();
      }
    }

    /* =========================================================================
       DYNAMIC PARTICLES & LIGHTING
       ========================================================================= */
    let particles = [], floatingTexts = [], airstrikes = [], lightPoints = [];

    function createExplosion(x, y, radius, maxDamage) {
      AudioEngine.play('explode');
      destroyTerrainCircle(x, y, radius);
      addScreenShake(radius * 0.25);

      lightPoints.push({ x, y, radius: radius * 3, color: 'rgba(251, 191, 36, 0.9)', life: 18, maxLife: 18 });

      worms.forEach(w => {
        if (w.isDead) return;
        const dist = Math.hypot(w.x - x, w.y - y);
        if (dist < radius + 20) {
          const factor = Math.max(0, 1 - (dist / (radius + 20)));
          const dmg = Math.floor(factor * maxDamage);
          if (dmg > 0) {
            w.hp = Math.max(0, w.hp - dmg);
            addFloatingText(w.x, w.y - 15, `-${dmg}`, '#ef4444');
          }
          const angle = Math.atan2(w.y - y, w.x - x);
          w.vx += Math.cos(angle) * factor * 13;
          w.vy += Math.sin(angle) * factor * 13 - 3;
          w.isGrounded = false;
        }
      });

      for (let i = 0; i < 28; i++) {
        const ang = Math.random() * Math.PI * 2;
        const spd = Math.random() * 6 + 1;
        particles.push(new Particle(x, y, Math.cos(ang)*spd, Math.sin(ang)*spd, '#f59e0b', 24));
      }
    }

    function fireRailgun(aimAngle) {
      if (!activeWorm) return;
      AudioEngine.play('railgun');
      addScreenShake(10);
      const startX = activeWorm.x + Math.cos(aimAngle) * 16;
      const startY = activeWorm.y + Math.sin(aimAngle) * 16;
      const endX = startX + Math.cos(aimAngle) * 1500;
      const endY = startY + Math.sin(aimAngle) * 1500;

      worms.forEach(w => {
        if (w !== activeWorm && !w.isDead) {
          const dist = distToSegment({ x: w.x, y: w.y }, { x: startX, y: startY }, { x: endX, y: endY });
          if (dist < 20) {
            w.hp = Math.max(0, w.hp - 45);
            w.vx += Math.cos(aimAngle) * 12;
            w.vy += Math.sin(aimAngle) * 12 - 2;
            w.isGrounded = false;
            addFloatingText(w.x, w.y - 15, '-45', '#0ea5e9');
          }
        }
      });

      for (let d = 0; d < 1200; d += 20) {
        particles.push(new Particle(startX + Math.cos(aimAngle)*d, startY + Math.sin(aimAngle)*d, (Math.random()-0.5)*2, (Math.random()-0.5)*2, '#0ea5e9', 14));
      }

      turnPhase = 'PROJECTILE';
      setTimeout(() => { turnPhase = 'RETREAT'; turnTimer = 4; }, 1000);
    }

    function distToSegment(p, v, w) {
      const l2 = (v.x - w.x)**2 + (v.y - w.y)**2;
      if (l2 === 0) return Math.hypot(p.x - v.x, p.y - v.y);
      let t = ((p.x - v.x) * (w.x - v.x) + (p.y - v.y) * (w.y - v.y)) / l2;
      t = Math.max(0, Math.min(1, t));
      return Math.hypot(p.x - (v.x + t * (w.x - v.x)), p.y - (v.y + t * (w.y - v.y)));
    }

    function createWaterSplash(x, y) {
      for (let i = 0; i < 18; i++) {
        particles.push(new Particle(x, y, (Math.random()-0.5)*5, -Math.random()*5, currentBiome.waterColor, 22));
      }
    }

    class Particle {
      constructor(x, y, vx, vy, color, life) {
        this.x = x; this.y = y; this.vx = vx; this.vy = vy; this.color = color; this.life = life; this.maxLife = life;
      }
      update() { this.x += this.vx; this.y += this.vy; this.vy += 0.1; this.life--; }
      draw(ctx) {
        ctx.fillStyle = this.color;
        ctx.globalAlpha = Math.max(0, this.life / this.maxLife);
        ctx.beginPath(); ctx.arc(this.x, this.y, 3, 0, Math.PI * 2); ctx.fill();
        ctx.globalAlpha = 1.0;
      }
    }

    function addFloatingText(x, y, text, color) { floatingTexts.push({ x, y, text, color, life: 35 }); }
    function addScreenShake(amt) { screenShake = Math.min(25, screenShake + amt); }

    /* =========================================================================
       MATCH ENGINE & GAME STATE
       ========================================================================= */
    let worms = [], projectiles = [];
    let currentTeam = 'red', activeWorm = null, activeWormIndex = { red: 0, blue: 0 };
    let turnTimer = 30, turnPhase = 'ACTION', wind = 0, selectedWeapon = 'bazooka';
    let isChargingPower = false, currentPower = 0, chargeDirection = 1;
    const keys = { left: false, right: false, up: false, down: false, fire: false, jet: false };

    function initSquads() {
      worms = [
        new Worm('r1', 'Bogdan', 'red', 350, 100),
        new Worm('r2', 'Viper', 'red', 650, 100),
        new Worm('b1', 'Glitch', 'blue', WORLD_WIDTH - 650, 100),
        new Worm('b2', 'Shadow', 'blue', WORLD_WIDTH - 350, 100)
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
    }

    function startMatch(asHost) {
      const biomeKey = document.getElementById('biome-selector').value;
      const seed = Math.random();
      generateTerrain(seed, biomeKey);
      initSquads();

      if (isMultiplayer && asHost) {
        sendNet({ type: 'START_SYNC', seed, biomeKey });
      }

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
      wind = Math.floor(Math.random() * 11) - 5;
      turnTimer = 30;
      turnPhase = 'ACTION';
      pickNextActiveWorm();

      if (isMultiplayer && isHost) {
        sendNet({ type: 'TURN_CHANGE', nextTeam: currentTeam, wind });
      }
    }

    function fireWeapon(power, aimAngle, weaponType, targetX) {
      if (!activeWorm) return;

      if (weaponType === 'airstrike') {
        airstrikes.push(new AirStrikeJet(targetX || activeWorm.x));
        turnPhase = 'PROJECTILE';
      } else if (weaponType === 'railgun') {
        fireRailgun(aimAngle);
      } else {
        const p = (power / 100) * 16 + 2;
        const vx = Math.cos(aimAngle) * p;
        const vy = Math.sin(aimAngle) * p;
        const sx = activeWorm.x + Math.cos(aimAngle) * 16;
        const sy = activeWorm.y + Math.sin(aimAngle) * 16;
        projectiles.push(new Projectile(weaponType, sx, sy, vx, vy));
        AudioEngine.play('launch');
        turnPhase = 'PROJECTILE';
      }

      document.getElementById('power-meter-container').style.display = 'none';

      if (canControl()) {
        sendNet({ type: 'FIRE', power, aimAngle, weaponType, targetX });
      }
    }

    function handleNetworkPacket(pkg) {
      if (pkg.type === 'START_SYNC') {
        generateTerrain(pkg.seed, pkg.biomeKey);
        initSquads();
      } else if (pkg.type === 'WORM_SYNC') {
        if (activeWorm && !canControl()) {
          activeWorm.x = pkg.x; activeWorm.y = pkg.y;
          activeWorm.aimAngle = pkg.aimAngle;
          activeWorm.facingRight = pkg.facingRight;
        }
      } else if (pkg.type === 'FIRE') {
        fireWeapon(pkg.power, pkg.aimAngle, pkg.weaponType, pkg.targetX);
      } else if (pkg.type === 'WEAPON_SELECT') {
        selectedWeapon = pkg.weapon;
      } else if (pkg.type === 'TURN_CHANGE') {
        currentTeam = pkg.nextTeam; wind = pkg.wind;
        turnTimer = 30; turnPhase = 'ACTION';
        pickNextActiveWorm();
      }
    }

    function canControl() {
      if (!isMultiplayer) return true;
      return currentTeam === myTeam;
    }

    function selectWeapon(wpn) {
      if (!canControl()) return;
      selectedWeapon = wpn;
      document.querySelectorAll('.wpn-btn').forEach(b => b.classList.remove('active'));
      event.currentTarget.classList.add('active');
      if (isMultiplayer) sendNet({ type: 'WEAPON_SELECT', weapon: wpn });
    }

    function openDonateModal() { document.getElementById('donate-modal').style.display = 'flex'; }
    function closeDonateModal() { document.getElementById('donate-modal').style.display = 'none'; }
    function openSettingsModal() { alert('Map Biome: ' + currentBiome.name); }
    function restartMatch() {
      document.getElementById('gameover-modal').style.display = 'none';
      startMatch(isHost);
    }

    /* =========================================================================
       INPUT SYSTEM & CONTROLS
       ========================================================================= */
    window.addEventListener('keydown', e => {
      if (document.activeElement === document.getElementById('chat-input')) return;
      if (!canControl() || turnPhase !== 'ACTION') return;
      if (e.key === 'a' || e.key === 'ArrowLeft') keys.left = true;
      if (e.key === 'd' || e.key === 'ArrowRight') keys.right = true;
      if (e.key === 'w' || e.key === 'ArrowUp') keys.up = true;
      if (e.key === 's' || e.key === 'ArrowDown') keys.down = true;
      if (e.key === 'Shift') activeWorm?.jump();
      if (e.key === 'e' || e.key === 'f') activeWorm?.flyJetpack();
      if (e.key === ' ' && !keys.fire) { keys.fire = true; startCharging(); }
    });

    window.addEventListener('keyup', e => {
      if (document.activeElement === document.getElementById('chat-input')) return;
      if (!canControl()) return;
      if (e.key === 'a' || e.key === 'ArrowLeft') keys.left = false;
      if (e.key === 'd' || e.key === 'ArrowRight') keys.right = false;
      if (e.key === 'w' || e.key === 'ArrowUp') keys.up = false;
      if (e.key === 's' || e.key === 'ArrowDown') keys.down = false;
      if (e.key === ' ' && keys.fire) { keys.fire = false; releaseCharge(); }
    });

    document.getElementById('viewport-container').addEventListener('click', e => {
      if (!canControl() || turnPhase !== 'ACTION' || selectedWeapon !== 'airstrike') return;
      const targetWorldX = e.clientX + cameraX;
      fireWeapon(100, 0, 'airstrike', targetWorldX);
    });

    function bindTouch(id, onStart, onEnd) {
      const el = document.getElementById(id);
      el.addEventListener('touchstart', e => { e.preventDefault(); onStart(); });
      el.addEventListener('touchend', e => { e.preventDefault(); onEnd(); });
    }

    bindTouch('btn-left', () => { if(canControl()) keys.left = true; }, () => keys.left = false);
    bindTouch('btn-right', () => { if(canControl()) keys.right = true; }, () => keys.right = false);
    bindTouch('btn-up', () => { if(canControl()) keys.up = true; }, () => keys.up = false);
    bindTouch('btn-down', () => { if(canControl()) keys.down = true; }, () => keys.down = false);
    bindTouch('btn-jump', () => { if(canControl()) activeWorm?.jump(); }, () => {});
    bindTouch('btn-fire', () => { if(canControl()) startCharging(); }, () => { if(canControl()) releaseCharge(); });

    function startCharging() {
      if (turnPhase !== 'ACTION' || selectedWeapon === 'airstrike' || selectedWeapon === 'railgun') {
        if (selectedWeapon === 'railgun') fireWeapon(100, activeWorm.aimAngle, 'railgun');
        return;
      }
      isChargingPower = true; currentPower = 0; chargeDirection = 1;
      document.getElementById('power-meter-container').style.display = 'block';
    }

    function releaseCharge() {
      if (!isChargingPower) return;
      isChargingPower = false;
      fireWeapon(currentPower, activeWorm.aimAngle, selectedWeapon);
    }

    /* =========================================================================
       MAIN RENDER & GAME LOOP
       ========================================================================= */
    let lastTime = performance.now();
    let secTimer = 0, syncTimer = 0;

    function gameLoop(now) {
      const dt = Math.min(0.1, (now - lastTime) / 1000);
      lastTime = now;

      if (activeWorm && !activeWorm.isDead && canControl()) {
        if (keys.left) activeWorm.walk(-1);
        if (keys.right) activeWorm.walk(1);
        if (keys.up) activeWorm.aimAngle = Math.max(-Math.PI * 0.95, activeWorm.aimAngle - 0.04);
        if (keys.down) activeWorm.aimAngle = Math.min(0.2, activeWorm.aimAngle + 0.04);

        if (isMultiplayer) {
          syncTimer += dt;
          if (syncTimer > 0.05) {
            syncTimer = 0;
            sendNet({
              type: 'WORM_SYNC', x: activeWorm.x, y: activeWorm.y,
              aimAngle: activeWorm.aimAngle, facingRight: activeWorm.facingRight
            });
          }
        }
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
      airstrikes.forEach(a => a.update());
      airstrikes = airstrikes.filter(a => a.alive);

      if (turnPhase === 'PROJECTILE' && projectiles.length === 0 && airstrikes.length === 0) {
        if (!worms.some(w => Math.hypot(w.vx, w.vy) > 0.4)) {
          turnPhase = 'RETREAT';
          turnTimer = 4;
        }
      }

      particles.forEach(p => p.update());
      particles = particles.filter(p => p.life > 0);

      if (screenShake > 0) {
        screenShake = Math.max(0, screenShake - 0.5);
      }
      const shakeX = (Math.random() - 0.5) * screenShake * 2;
      const shakeY = (Math.random() - 0.5) * screenShake * 2;

      if (projectiles.length > 0) {
        cameraX += (projectiles[0].x - viewWidth / 2 - cameraX) * 0.1;
        cameraY += (projectiles[0].y - viewHeight / 2 - cameraY) * 0.1;
      } else if (activeWorm && !activeWorm.isDead) {
        cameraX += (activeWorm.x - viewWidth / 2 - cameraX) * 0.08;
        cameraY += (activeWorm.y - viewHeight / 2 - cameraY) * 0.08;
      }

      cameraX = Math.max(0, Math.min(WORLD_WIDTH - viewWidth, cameraX)) + shakeX;
      cameraY = Math.max(0, Math.min(WORLD_HEIGHT - viewHeight, cameraY)) + shakeY;

      // -------------------------------------------------------------
      // RENDERING ENGINE
      // -------------------------------------------------------------
      gameCtx.clearRect(0, 0, viewWidth, viewHeight);
      lightCtx.clearRect(0, 0, viewWidth, viewHeight);

      gameCtx.drawImage(terrainCanvas, cameraX, cameraY, viewWidth, viewHeight, 0, 0, viewWidth, viewHeight);

      gameCtx.save();
      gameCtx.translate(-cameraX, -cameraY);

      gameCtx.fillStyle = currentBiome.waterColor;
      gameCtx.fillRect(0, WATER_LEVEL + Math.sin(now * 0.003) * 5, WORLD_WIDTH, WORLD_HEIGHT - WATER_LEVEL);

      worms.forEach(w => w.draw(gameCtx));
      projectiles.forEach(p => p.draw(gameCtx));
      airstrikes.forEach(a => a.draw(gameCtx));
      particles.forEach(p => p.draw(gameCtx));

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

    window.addEventListener('resize', () => {
      viewWidth = window.innerWidth; viewHeight = window.innerHeight;
      [skyCanvas, gameCanvas, lightCanvas].forEach(c => { c.width = viewWidth; c.height = viewHeight; });
      drawSky();
    });

    window.addEventListener('DOMContentLoaded', () => {
      initGlobalChat();
      requestAnimationFrame(gameLoop);
    });
  </script>
</body>
</html>

<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Crimson Ronin — Shadow of the Red Sun</title>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Cinzel:wght@600;800;900&family=Noto+Serif+JP:wght@400;700;900&display=swap" rel="stylesheet">
    <style>
        :root {
            --parchment-light: #ebe7df;
            --parchment-dark: #c5beb0;
            --ink-black: #0c0c0e;
            --ink-charcoal: #1e1e24;
            --crimson-bright: #e01b1b;
            --crimson-deep: #8f0d0d;
            --gold-spark: #f5c156;
        }

        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            user-select: none;
            -webkit-user-select: none;
            -webkit-touch-callout: none;
        }

        html, body {
            width: 100%;
            height: 100%;
            overflow: hidden;
            background-color: var(--ink-black);
            font-family: 'Cinzel', 'Noto Serif JP', serif;
            color: var(--ink-black);
        }

        #game-wrapper {
            position: relative;
            width: 100vw;
            height: 100vh;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            overflow: hidden;
            background: radial-gradient(circle at 50% 38%, #f0ece3 0%, #d4cdc0 60%, #b5ada0 100%);
        }

        canvas {
            display: block;
            width: 100%;
            height: 100%;
            touch-action: none;
        }

        /* Subtle Sumi-e Vignette & Grain Overlay */
        .vignette-overlay {
            position: absolute;
            inset: 0;
            pointer-events: none;
            box-shadow: inset 0 0 90px rgba(10, 10, 12, 0.55);
            z-index: 5;
        }

        /* Top HUD */
        #hud {
            position: absolute;
            top: 0;
            left: 0;
            right: 0;
            padding: 16px 24px;
            display: flex;
            justify-content: space-between;
            align-items: flex-start;
            z-index: 15;
            pointer-events: none;
            transition: opacity 0.4s ease;
        }

        #hud.hidden {
            opacity: 0;
        }

        .hud-left {
            display: flex;
            flex-direction: column;
            gap: 8px;
            min-width: 220px;
            max-width: 42vw;
        }

        .ronin-tag {
            display: flex;
            align-items: center;
            gap: 8px;
            font-weight: 900;
            font-size: 0.95rem;
            letter-spacing: 0.14em;
            color: var(--ink-black);
            text-shadow: 0 1px 0 rgba(255,255,255,0.6);
        }

        .kanji-badge {
            background: var(--crimson-bright);
            color: #fff;
            padding: 2px 6px;
            border-radius: 3px;
            font-family: 'Noto Serif JP', serif;
            font-size: 0.85rem;
            box-shadow: 0 2px 6px rgba(224, 27, 27, 0.4);
        }

        /* Ink Brush Health Bar */
        .health-bar-frame {
            position: relative;
            width: 100%;
            height: 14px;
            background: rgba(15, 15, 18, 0.85);
            border: 1.5px solid #09090b;
            border-radius: 2px 8px 3px 6px;
            overflow: hidden;
            box-shadow: 0 3px 10px rgba(0, 0, 0, 0.25);
        }

        #health-fill {
            width: 100%;
            height: 100%;
            background: linear-gradient(90deg, #8a0c0c 0%, #e01b1b 75%, #ff3b3b 100%);
            transition: width 0.15s ease-out;
        }

        /* Shadow Step Stamina Pips */
        .stamina-row {
            display: flex;
            gap: 6px;
            align-items: center;
        }

        .stamina-pip {
            width: 28px;
            height: 6px;
            background: rgba(15, 15, 18, 0.25);
            border: 1px solid rgba(15, 15, 18, 0.7);
            border-radius: 2px;
            transition: background 0.15s ease, transform 0.15s ease;
        }

        .stamina-pip.ready {
            background: var(--ink-black);
            box-shadow: 0 0 6px rgba(0, 0, 0, 0.4);
        }

        .hud-center {
            text-align: center;
            display: flex;
            flex-direction: column;
            align-items: center;
        }

        .wave-label {
            font-size: 1.15rem;
            font-weight: 900;
            letter-spacing: 0.18em;
            color: var(--ink-black);
            text-transform: uppercase;
        }

        .wave-sub {
            font-size: 0.75rem;
            font-weight: 700;
            color: var(--crimson-deep);
            letter-spacing: 0.12em;
        }

        .hud-right {
            display: flex;
            flex-direction: column;
            align-items: flex-end;
            gap: 6px;
            pointer-events: auto;
        }

        .top-actions {
            display: flex;
            align-items: center;
            gap: 12px;
        }

        .score-box {
            text-align: right;
            pointer-events: none;
        }

        .score-value {
            font-size: 1.35rem;
            font-weight: 900;
            color: var(--ink-black);
            letter-spacing: 0.06em;
            line-height: 1;
        }

        .score-label {
            font-size: 0.65rem;
            font-weight: 800;
            letter-spacing: 0.2em;
            color: #4a4640;
        }

        #combo-container {
            margin-top: 4px;
            text-align: right;
            opacity: 0;
            transform: scale(0.9);
            transition: opacity 0.2s ease, transform 0.15s cubic-bezier(0.18, 0.89, 0.32, 1.28);
            pointer-events: none;
        }

        #combo-container.active {
            opacity: 1;
            transform: scale(1);
        }

        #combo-count {
            font-size: 1.7rem;
            font-weight: 900;
            color: var(--crimson-bright);
            text-shadow: 2px 2px 0px var(--ink-black);
            line-height: 1;
        }

        .combo-text {
            font-size: 0.7rem;
            font-weight: 900;
            letter-spacing: 0.18em;
            color: var(--ink-black);
        }

        .icon-btn {
            background: rgba(235, 231, 223, 0.85);
            border: 1.5px solid var(--ink-black);
            color: var(--ink-black);
            width: 36px;
            height: 36px;
            border-radius: 6px;
            display: flex;
            align-items: center;
            justify-content: center;
            cursor: pointer;
            box-shadow: 0 3px 8px rgba(0,0,0,0.18);
            transition: all 0.15s ease;
        }

        .icon-btn:hover {
            background: var(--ink-black);
            color: var(--parchment-light);
        }

        /* Wave Banner Announcement */
        #wave-banner {
            position: absolute;
            top: 24%;
            left: 50%;
            transform: translate(-50%, -50%) scale(0.92);
            text-align: center;
            pointer-events: none;
            z-index: 14;
            opacity: 0;
            transition: opacity 0.35s ease, transform 0.35s ease;
        }

        #wave-banner.show {
            opacity: 1;
            transform: translate(-50%, -50%) scale(1);
        }

        #wave-banner h2 {
            font-size: clamp(1.8rem, 4.5vw, 3.2rem);
            font-weight: 900;
            letter-spacing: 0.24em;
            color: var(--ink-black);
            text-shadow: 0 2px 14px rgba(235, 231, 223, 0.95);
        }

        #wave-banner p {
            font-size: clamp(0.85rem, 2vw, 1.1rem);
            font-weight: 800;
            letter-spacing: 0.2em;
            color: var(--crimson-bright);
            margin-top: 4px;
            text-shadow: 0 1px 6px rgba(235, 231, 223, 0.95);
        }

        /* Title & Game Over Overlays */
        .screen-overlay {
            position: absolute;
            inset: 0;
            z-index: 25;
            display: flex;
            flex-direction: column;
            justify-content: space-between;
            align-items: center;
            padding: 36px 20px 28px;
            background: radial-gradient(circle at 50% 45%, rgba(235, 231, 223, 0.15) 0%, rgba(12, 12, 14, 0.52) 100%);
            transition: opacity 0.45s ease;
        }

        .screen-overlay.hidden {
            opacity: 0;
            pointer-events: none;
        }

        .title-header {
            text-align: center;
            margin-top: 2vh;
        }

        .jp-subtitle {
            font-family: 'Noto Serif JP', serif;
            font-size: clamp(0.95rem, 2.2vw, 1.25rem);
            font-weight: 900;
            letter-spacing: 0.45em;
            color: var(--crimson-bright);
            text-shadow: 0 2px 8px rgba(0, 0, 0, 0.45);
            margin-bottom: 6px;
        }

        .main-title {
            font-size: clamp(2.4rem, 6.5vw, 4.8rem);
            font-weight: 900;
            letter-spacing: 0.16em;
            color: var(--ink-black);
            line-height: 1.05;
            text-shadow:
                0 0 24px rgba(240, 236, 227, 0.95),
                0 4px 12px rgba(0, 0, 0, 0.35);
        }

        .main-title span {
            color: var(--crimson-bright);
            text-shadow:
                2px 2px 0px var(--ink-black),
                0 0 20px rgba(240, 236, 227, 0.85);
        }

        .title-footer {
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: 16px;
            margin-bottom: 2vh;
        }

        /* Sumi-e Styled Action Buttons */
        .brush-btn {
            position: relative;
            padding: 15px 46px;
            font-family: 'Cinzel', serif;
            font-size: 1.25rem;
            font-weight: 900;
            letter-spacing: 0.24em;
            color: #f4f0e8;
            background: linear-gradient(135deg, #16161a 0%, #09090b 100%);
            border: 2px solid var(--crimson-bright);
            border-radius: 4px 14px 4px 14px;
            cursor: pointer;
            box-shadow:
                0 8px 24px rgba(0, 0, 0, 0.55),
                inset 0 0 12px rgba(224, 27, 27, 0.35);
            transition: transform 0.16s ease, box-shadow 0.16s ease, background 0.16s ease;
        }

        .brush-btn:hover {
            transform: translateY(-2px) scale(1.03);
            background: linear-gradient(135deg, #9e1111 0%, #d91e18 100%);
            border-color: #f4f0e8;
            box-shadow:
                0 12px 28px rgba(217, 30, 24, 0.5),
                inset 0 0 16px rgba(255, 255, 255, 0.25);
        }

        .brush-btn:active {
            transform: translateY(1px) scale(0.98);
        }

        .controls-guide {
            display: flex;
            flex-wrap: wrap;
            justify-content: center;
            gap: 10px 18px;
            background: rgba(14, 14, 17, 0.82);
            color: #e8e4dc;
            padding: 10px 20px;
            border-radius: 8px;
            border: 1px solid rgba(224, 27, 27, 0.45);
            font-size: 0.78rem;
            letter-spacing: 0.08em;
            backdrop-filter: blur(4px);
        }

        .key-cap {
            display: inline-block;
            background: #282830;
            border: 1px solid #555;
            border-bottom-width: 2px;
            padding: 1px 6px;
            border-radius: 4px;
            font-weight: 800;
            color: #fff;
            margin-right: 4px;
        }

        /* Game Over Card */
        .gameover-card {
            background: rgba(14, 14, 17, 0.92);
            border: 2px solid var(--crimson-bright);
            border-radius: 10px;
            padding: 30px 38px;
            text-align: center;
            color: #f0ece3;
            box-shadow: 0 20px 50px rgba(0, 0, 0, 0.75);
            max-width: 92vw;
            width: 430px;
            margin: auto;
        }

        .gameover-card h2 {
            font-size: 2.1rem;
            font-weight: 900;
            letter-spacing: 0.16em;
            color: var(--crimson-bright);
            margin-bottom: 6px;
        }

        .gameover-card .epitaph {
            font-size: 0.85rem;
            color: #b8b2a6;
            letter-spacing: 0.1em;
            margin-bottom: 22px;
        }

        .stats-grid {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 12px;
            margin-bottom: 24px;
            text-align: center;
        }

        .stat-item {
            background: rgba(255, 255, 255, 0.05);
            border: 1px solid rgba(255, 255, 255, 0.1);
            border-radius: 6px;
            padding: 10px;
        }

        .stat-item .val {
            font-size: 1.4rem;
            font-weight: 900;
            color: #fff;
        }

        .stat-item .lbl {
            font-size: 0.68rem;
            letter-spacing: 0.15em;
            color: #a8a296;
            margin-top: 2px;
        }

        /* Touch / On-Screen Controls Bar */
        #touch-controls {
            position: absolute;
            bottom: 14px;
            left: 16px;
            right: 16px;
            display: flex;
            justify-content: space-between;
            align-items: flex-end;
            z-index: 20;
            pointer-events: none;
        }

        #touch-controls.hidden {
            display: none;
        }

        .touch-cluster {
            display: flex;
            gap: 10px;
            pointer-events: auto;
        }

        .touch-btn {
            width: 56px;
            height: 56px;
            border-radius: 50%;
            background: rgba(15, 15, 18, 0.62);
            border: 2px solid rgba(235, 231, 223, 0.65);
            color: #f4f0e8;
            font-family: 'Cinzel', serif;
            font-weight: 900;
            font-size: 0.78rem;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            box-shadow: 0 4px 14px rgba(0, 0, 0, 0.4);
            backdrop-filter: blur(3px);
            cursor: pointer;
            transition: transform 0.08s ease, background 0.1s ease;
        }

        .touch-btn:active, .touch-btn.pressed {
            transform: scale(0.9);
            background: rgba(217, 30, 24, 0.85);
            border-color: #fff;
        }

        .touch-btn.slash-primary {
            width: 66px;
            height: 66px;
            border-color: var(--crimson-bright);
            background: rgba(140, 14, 14, 0.72);
        }

        @media (min-width: 960px) {
            /* Subtle opacity on desktop so screen stays clean, still clickable */
            #touch-controls {
                opacity: 0.55;
                transition: opacity 0.2s ease;
            }
            #touch-controls:hover {
                opacity: 0.95;
            }
        }
    </style>
</head>
<body>
    <div id="game-wrapper">
        <canvas id="gameCanvas"></canvas>
        <div class="vignette-overlay"></div>

        <!-- Top HUD -->
        <div id="hud" class="hidden">
            <div class="hud-left">
                <div class="ronin-tag">
                    <span class="kanji-badge">浪人</span>
                    <span>CRIMSON RONIN</span>
                </div>
                <div class="health-bar-frame">
                    <div id="health-fill"></div>
                </div>
                <div class="stamina-row" id="stamina-pips" title="Shadow Step Charges">
                    <div class="stamina-pip ready"></div>
                    <div class="stamina-pip ready"></div>
                    <div class="stamina-pip ready"></div>
                </div>
            </div>

            <div class="hud-center">
                <div class="wave-label" id="hud-wave">WAVE I</div>
                <div class="wave-sub" id="hud-enemies">FOES REMAINING: 0</div>
            </div>

            <div class="hud-right">
                <div class="top-actions">
                    <div class="score-box">
                        <div class="score-value" id="hud-score">0</div>
                        <div class="score-label">HONOR SCORE</div>
                    </div>
                    <button id="mute-btn" class="icon-btn" title="Toggle Sound" aria-label="Toggle Sound">
                        <svg id="sound-on-icon" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round">
                            <polygon points="11 5 6 9 2 9 2 15 6 15 11 19 11 5"></polygon>
                            <path d="M15.54 8.46a5 5 0 0 1 0 7.07"></path>
                            <path d="M19.07 4.93a10 10 0 0 1 0 14.14"></path>
                        </svg>
                        <svg id="sound-off-icon" style="display:none" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round">
                            <polygon points="11 5 6 9 2 9 2 15 6 15 11 19 11 5"></polygon>
                            <line x1="23" y1="9" x2="17" y2="15"></line>
                            <line x1="17" y1="9" x2="23" y2="15"></line>
                        </svg>
                    </button>
                </div>

                <div id="combo-container">
                    <div id="combo-count">0x</div>
                    <div class="combo-text">CRIMSON CHAIN</div>
                </div>
            </div>
        </div>

        <!-- Wave Announcement Banner -->
        <div id="wave-banner">
            <h2 id="banner-title">WAVE I</h2>
            <p id="banner-sub">THE CROWS GATHER</p>
        </div>

        <!-- Title Screen Overlay -->
        <div id="title-screen" class="screen-overlay">
            <div class="title-header">
                <div class="jp-subtitle">紅蓮の浪人 • 影の刃</div>
                <h1 class="main-title">CRIMSON <span>RONIN</span></h1>
            </div>

            <div class="title-footer">
                <button id="start-btn" class="brush-btn">UNSHEATHE BLADES</button>
                <div class="controls-guide">
                    <span><span class="key-cap">A / D</span> Move</span>
                    <span><span class="key-cap">W / Space</span> Jump</span>
                    <span><span class="key-cap">J / Click</span> Dual Slash</span>
                    <span><span class="key-cap">L / R-Click</span> Parry & Whirl</span>
                    <span><span class="key-cap">Shift / K</span> Shadow Dash</span>
                </div>
            </div>
        </div>

        <!-- Game Over Overlay -->
        <div id="gameover-screen" class="screen-overlay hidden">
            <div class="gameover-card">
                <h2>FALLEN IN INK</h2>
                <div class="epitaph" id="gameover-epitaph">THE RED SUN WITNESSES YOUR END</div>
                <div class="stats-grid">
                    <div class="stat-item">
                        <div class="val" id="final-score">0</div>
                        <div class="lbl">HONOR SCORE</div>
                    </div>
                    <div class="stat-item">
                        <div class="val" id="final-wave">1</div>
                        <div class="lbl">WAVE REACHED</div>
                    </div>
                    <div class="stat-item">
                        <div class="val" id="final-kills">0</div>
                        <div class="lbl">FOES SLAIN</div>
                    </div>
                    <div class="stat-item">
                        <div class="val" id="final-combo">0x</div>
                        <div class="lbl">MAX CHAIN</div>
                    </div>
                </div>
                <button id="restart-btn" class="brush-btn">RISE AGAIN</button>
            </div>
        </div>

        <!-- On-Screen Touch / Mouse Controls -->
        <div id="touch-controls" class="hidden">
            <div class="touch-cluster">
                <button class="touch-btn" id="btn-left" aria-label="Move Left">
                    <svg width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><polyline points="15 18 9 12 15 6"></polyline></svg>
                </button>
                <button class="touch-btn" id="btn-right" aria-label="Move Right">
                    <svg width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><polyline points="9 18 15 12 9 6"></polyline></svg>
                </button>
                <button class="touch-btn" id="btn-jump" aria-label="Jump">
                    <svg width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><polyline points="18 15 12 9 6 15"></polyline></svg>
                </button>
            </div>

            <div class="touch-cluster">
                <button class="touch-btn" id="btn-dash" aria-label="Dash">DASH</button>
                <button class="touch-btn" id="btn-parry" aria-label="Parry">PARRY</button>
                <button class="touch-btn slash-primary" id="btn-attack" aria-label="Slash">SLASH</button>
            </div>
        </div>
    </div>

    <script>
        /**
         * CRIMSON RONIN — Shadow of the Red Sun
         * Complete Sumi-e Ink Wash 2D Action Hack-and-Slash Engine
         */

        // ==========================================
        // 1. PROCEDURAL WEB AUDIO SYNTHESIZER
        // ==========================================
        class SoundEngine {
            constructor() {
                this.ctx = null;
                this.muted = false;
                this.ambientTimer = null;
            }

            init() {
                if (!this.ctx) {
                    const AudioCtx = window.AudioContext || window.webkitAudioContext;
                    if (AudioCtx) {
                        this.ctx = new AudioCtx();
                    }
                }
                if (this.ctx && this.ctx.state === 'suspended') {
                    this.ctx.resume();
                }
            }

            toggleMute() {
                this.muted = !this.muted;
                return this.muted;
            }

            playSlash(pitchMult = 1.0) {
                if (this.muted || !this.ctx) return;
                const now = this.ctx.currentTime;
                // Filtered white noise burst for sharp blade whoosh
                const bufferSize = this.ctx.sampleRate * 0.16;
                const buffer = this.ctx.createBuffer(1, bufferSize, this.ctx.sampleRate);
                const data = buffer.getChannelData(0);
                for (let i = 0; i < bufferSize; i++) {
                    data[i] = (Math.random() * 2 - 1) * Math.exp(-i / (bufferSize * 0.35));
                }

                const noise = this.ctx.createBufferSource();
                noise.buffer = buffer;

                const filter = this.ctx.createBiquadFilter();
                filter.type = 'bandpass';
                filter.frequency.setValueAtTime(1400 * pitchMult, now);
                filter.frequency.exponentialRampToValueAtTime(320 * pitchMult, now + 0.15);
                filter.Q.value = 1.8;

                const gain = this.ctx.createGain();
                gain.gain.setValueAtTime(0.32, now);
                gain.gain.exponentialRampToValueAtTime(0.01, now + 0.16);

                noise.connect(filter);
                filter.connect(gain);
                gain.connect(this.ctx.destination);
                noise.start(now);
            }

            playHit(heavy = false) {
                if (this.muted || !this.ctx) return;
                const now = this.ctx.currentTime;

                // Impact thud oscillator
                const osc = this.ctx.createOscillator();
                const gain = this.ctx.createGain();
                osc.type = 'triangle';
                osc.frequency.setValueAtTime(heavy ? 150 : 210, now);
                osc.frequency.exponentialRampToValueAtTime(38, now + (heavy ? 0.22 : 0.14));

                gain.gain.setValueAtTime(heavy ? 0.45 : 0.3, now);
                gain.gain.exponentialRampToValueAtTime(0.001, now + (heavy ? 0.22 : 0.14));

                osc.connect(gain);
                gain.connect(this.ctx.destination);
                osc.start(now);
                osc.stop(now + (heavy ? 0.23 : 0.15));

                // Crisp ink splatter snap
                this.playSlash(heavy ? 1.5 : 1.8);
            }

            playParry() {
                if (this.muted || !this.ctx) return;
                const now = this.ctx.currentTime;
                // Metallic katana ring (two high harmonic sines)
                [1760, 2637, 3520].forEach((freq, idx) => {
                    const osc = this.ctx.createOscillator();
                    const gain = this.ctx.createGain();
                    osc.type = idx === 0 ? 'sine' : 'triangle';
                    osc.frequency.setValueAtTime(freq, now);
                    osc.frequency.exponentialRampToValueAtTime(freq * 0.96, now + 0.45);

                    gain.gain.setValueAtTime(0.22 / (idx + 1), now);
                    gain.gain.exponentialRampToValueAtTime(0.001, now + 0.45);

                    osc.connect(gain);
                    gain.connect(this.ctx.destination);
                    osc.start(now);
                    osc.stop(now + 0.46);
                });
            }

            playDash() {
                if (this.muted || !this.ctx) return;
                const now = this.ctx.currentTime;
                const bufferSize = this.ctx.sampleRate * 0.2;
                const buffer = this.ctx.createBuffer(1, bufferSize, this.ctx.sampleRate);
                const data = buffer.getChannelData(0);
                for (let i = 0; i < bufferSize; i++) {
                    data[i] = (Math.random() * 2 - 1) * Math.sin((i / bufferSize) * Math.PI);
                }

                const noise = this.ctx.createBufferSource();
                noise.buffer = buffer;

                const filter = this.ctx.createBiquadFilter();
                filter.type = 'lowpass';
                filter.frequency.setValueAtTime(600, now);
                filter.frequency.exponentialRampToValueAtTime(160, now + 0.2);

                const gain = this.ctx.createGain();
                gain.gain.setValueAtTime(0.35, now);
                gain.gain.exponentialRampToValueAtTime(0.01, now + 0.2);

                noise.connect(filter);
                filter.connect(gain);
                gain.connect(this.ctx.destination);
                noise.start(now);
            }

            playHurt() {
                if (this.muted || !this.ctx) return;
                const now = this.ctx.currentTime;
                const osc = this.ctx.createOscillator();
                const gain = this.ctx.createGain();
                osc.type = 'sawtooth';
                osc.frequency.setValueAtTime(115, now);
                osc.frequency.linearRampToValueAtTime(52, now + 0.28);

                gain.gain.setValueAtTime(0.36, now);
                gain.gain.exponentialRampToValueAtTime(0.001, now + 0.28);

                osc.connect(gain);
                gain.connect(this.ctx.destination);
                osc.start(now);
                osc.stop(now + 0.29);
            }

            playTaikoBeat() {
                if (this.muted || !this.ctx) return;
                const now = this.ctx.currentTime;
                const osc = this.ctx.createOscillator();
                const gain = this.ctx.createGain();
                osc.type = 'sine';
                osc.frequency.setValueAtTime(86, now);
                osc.frequency.exponentialRampToValueAtTime(36, now + 0.42);

                gain.gain.setValueAtTime(0.38, now);
                gain.gain.exponentialRampToValueAtTime(0.001, now + 0.45);

                osc.connect(gain);
                gain.connect(this.ctx.destination);
                osc.start(now);
                osc.stop(now + 0.46);
            }
        }

        const sound = new SoundEngine();

        // ==========================================
        // 2. CANVAS & ENVIRONMENT SETUP
        // ==========================================
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');

        let W = 0;
        let H = 0;
        let GROUND_Y = 0;

        // Pre-generated static ink splatters around the Red Sun so it matches the artwork
        const sunSplatters = [];
        function generateSunSplatters() {
            sunSplatters.length = 0;
            for (let i = 0; i < 65; i++) {
                const angle = Math.random() * Math.PI * 2;
                const distRatio = 0.96 + Math.random() * 0.26;
                const radius = (Math.random() < 0.25 ? 4 + Math.random() * 7 : 1.5 + Math.random() * 3.5);
                const isCrimson = Math.random() < 0.65;
                sunSplatters.push({ angle, distRatio, radius, isCrimson });
            }
        }
        generateSunSplatters();

        // Distant ink-wash mountain silhouettes
        const mountainPoints = [];
        function generateMountains() {
            mountainPoints.length = 0;
            const steps = 18;
            for (let i = 0; i <= steps; i++) {
                const nx = i / steps;
                const height = 28 + Math.sin(nx * 7.5) * 18 + Math.cos(nx * 15) * 10 + (i % 3 === 0 ? 14 : 0);
                mountainPoints.push({ nx, height });
            }
        }
        generateMountains();

        // Foreground water puddles for crimson reflection
        const puddles = [
            { nx: 0.16, nw: 0.18 },
            { nx: 0.48, nw: 0.24 },
            { nx: 0.78, nw: 0.16 }
        ];

        function resizeCanvas() {
            const dpr = Math.min(window.devicePixelRatio || 1, 2);
            W = window.innerWidth;
            H = window.innerHeight;
            canvas.width = W * dpr;
            canvas.height = H * dpr;
            ctx.setTransform(dpr, 0, 0, dpr, 0, 0);
            GROUND_Y = Math.floor(H * 0.82);

            if (player) {
                if (player.y > GROUND_Y) player.y = GROUND_Y;
                player.x = Math.max(40, Math.min(W - 40, player.x));
            }
        }

        window.addEventListener('resize', resizeCanvas);

        // ==========================================
        // 3. GAME STATE & ENTITIES
        // ==========================================
        const GAME_STATE = {
            MENU: 'MENU',
            PLAYING: 'PLAYING',
            GAMEOVER: 'GAMEOVER'
        };

        let currentState = GAME_STATE.MENU;
        let score = 0;
        let kills = 0;
        let combo = 0;
        let maxCombo = 0;
        let comboTimer = 0;
        let wave = 1;
        let waveEnemiesToSpawn = [];
        let spawnTimer = 0;
        let waveClearCooldown = 0;

        let hitstopTimer = 0;
        let screenShake = 0;
        let globalTime = 0;

        // Ambient Background Crows
        class AmbientCrow {
            constructor(initial = false) {
                this.reset(initial);
            }

            reset(initial = false) {
                this.dir = Math.random() < 0.5 ? 1 : -1;
                this.x = initial ? Math.random() * W : (this.dir === 1 ? -60 : W + 60);
                this.y = H * (0.12 + Math.random() * 0.42);
                this.speed = (45 + Math.random() * 75) * this.dir;
                this.scale = 0.45 + Math.random() * 0.75;
                this.wingPhase = Math.random() * Math.PI * 2;
                this.flapSpeed = 7 + Math.random() * 5;
                this.alpha = 0.35 + this.scale * 0.5;
            }

            update(dt) {
                this.x += this.speed * dt;
                this.y += Math.sin(globalTime * 2 + this.wingPhase) * 12 * dt;
                this.wingPhase += this.flapSpeed * dt;
                if ((this.dir === 1 && this.x > W + 90) || (this.dir === -1 && this.x < -90)) {
                    this.reset(false);
                }
            }

            draw(ctx) {
                ctx.save();
                ctx.translate(this.x, this.y);
                ctx.scale(this.scale * this.dir, this.scale);
                ctx.fillStyle = `rgba(12, 12, 14, ${this.alpha})`;

                const flap = Math.sin(this.wingPhase);
                const wingTipY = -18 * flap - 4;
                const wingMidY = -10 * flap;

                // Crow silhouette with feathered wing tips
                ctx.beginPath();
                ctx.moveTo(14, 0); // Beak
                ctx.quadraticCurveTo(6, -5, -2, -3); // Head/neck
                // Upper wing
                ctx.quadraticCurveTo(-8, wingMidY - 8, -22, wingTipY);
                ctx.lineTo(-17, wingTipY + 5);
                ctx.lineTo(-20, wingTipY + 8);
                ctx.quadraticCurveTo(-10, -2, -16, 4); // Tail
                ctx.lineTo(-22, 7);
                ctx.lineTo(-15, 1);
                // Lower wing
                ctx.quadraticCurveTo(-4, 4, 2, 2);
                ctx.closePath();
                ctx.fill();
                ctx.restore();
            }
        }

        const ambientCrows = Array.from({ length: 11 }, () => new AmbientCrow(true));

        // Ink & Crimson Particles
        const particles = [];
        function spawnParticles(x, y, count, color, speedMin, speedMax, sizeMin, sizeMax, gravity = 450) {
            for (let i = 0; i < count; i++) {
                const angle = Math.random() * Math.PI * 2;
                const speed = speedMin + Math.random() * (speedMax - speedMin);
                particles.push({
                    x,
                    y,
                    vx: Math.cos(angle) * speed,
                    vy: Math.sin(angle) * speed - 60,
                    size: sizeMin + Math.random() * (sizeMax - sizeMin),
                    color,
                    alpha: 1,
                    life: 0.35 + Math.random() * 0.45,
                    maxLife: 0.35 + Math.random() * 0.45,
                    gravity
                });
            }
        }

        // Slash Arcs (Visual katana trails)
        const slashArcs = [];
        function spawnSlashArc(x, y, facing, radius, startAngle, endAngle, color = '#ffffff', isCrimson = false) {
            slashArcs.push({
                x,
                y,
                facing,
                radius,
                startAngle,
                endAngle,
                color,
                isCrimson,
                life: 0.18,
                maxLife: 0.18
            });
        }

        // Floating Damage / Parry Callouts
        const floatingTexts = [];
        function spawnFloatingText(x, y, text, color = '#d91e18', scale = 1) {
            floatingTexts.push({
                x,
                y,
                text,
                color,
                scale,
                vy: -55,
                life: 0.75,
                maxLife: 0.75
            });
        }

        // ==========================================
        // 4. PLAYER (CRIMSON RONIN)
        // ==========================================
        class Player {
            constructor() {
                this.reset();
            }

            reset() {
                this.x = W * 0.5;
                this.y = GROUND_Y;
                this.vx = 0;
                this.vy = 0;
                this.facing = 1; // 1 = right, -1 = left
                this.width = 34;
                this.height = 84;

                this.maxHp = 100;
                this.hp = 100;

                this.maxStamina = 3;
                this.stamina = 3;
                this.staminaRegenTimer = 0;

                this.grounded = true;
                this.speed = 360;
                this.jumpForce = -660;

                // Combat states
                this.attackCooldown = 0;
                this.attackTimer = 0;
                this.comboStep = 0; // 0, 1, 2
                this.comboWindow = 0;

                this.parryTimer = 0;
                this.parryCooldown = 0;

                this.dashTimer = 0;
                this.invulnTimer = 0;
                this.afterimages = [];

                this.robePhase = 0;
            }

            attack() {
                if (this.attackCooldown > 0 || this.dashTimer > 0) return;

                this.attackCooldown = 0.22;
                this.attackTimer = 0.18;
                this.comboStep = (this.comboStep % 3) + 1;
                this.comboWindow = 0.65;

                // Lunge slightly forward
                this.vx = this.facing * (this.comboStep === 3 ? 280 : 140);

                const isFinisher = this.comboStep === 3;
                sound.playSlash(isFinisher ? 0.85 : 1.05 + this.comboStep * 0.1);

                // Dual Katana Slash Arcs
                const reach = isFinisher ? 115 : 92;
                if (this.comboStep === 1) {
                    spawnSlashArc(this.x, this.y - 44, this.facing, reach, -1.1, 0.8, '#ffffff', false);
                    spawnSlashArc(this.x, this.y - 40, this.facing, reach * 0.85, -0.8, 1.0, '#e01b1b', true);
                } else if (this.comboStep === 2) {
                    spawnSlashArc(this.x, this.y - 46, this.facing, reach, 0.9, -1.1, '#ffffff', false);
                    spawnSlashArc(this.x, this.y - 42, this.facing, reach * 0.88, 1.1, -0.7, '#e01b1b', true);
                } else {
                    // 3rd Hit: Full Dual-Katana X-Cross Crimson Cleave
                    spawnSlashArc(this.x, this.y - 45, this.facing, reach, -1.4, 1.3, '#e01b1b', true);
                    spawnSlashArc(this.x, this.y - 45, this.facing, reach * 0.92, 1.4, -1.3, '#ffffff', false);
                }

                // Check hits against enemies
                const dmg = isFinisher ? 48 : 26;
                let hitCount = 0;

                enemies.forEach(enemy => {
                    if (enemy.dead) return;
                    const dx = enemy.x - this.x;
                    const dy = (enemy.y - enemy.height * 0.5) - (this.y - 44);
                    const inFront = (this.facing === 1 && dx >= -25 && dx <= reach + 24) ||
                                    (this.facing === -1 && dx <= 25 && dx >= -(reach + 24));
                    if (inFront && Math.abs(dy) < 75) {
                        enemy.takeDamage(dmg, this.facing * (isFinisher ? 360 : 190), isFinisher);
                        hitCount++;
                    }
                });

                if (hitCount > 0) {
                    hitstopTimer = isFinisher ? 0.055 : 0.025;
                    screenShake = Math.max(screenShake, isFinisher ? 9 : 4);
                }
            }

            parryAndWhirl() {
                if (this.parryCooldown > 0 || this.dashTimer > 0) return;
                this.parryTimer = 0.28;
                this.parryCooldown = 0.65;
                this.attackTimer = 0.22;

                sound.playSlash(1.35);
                // Circular dual-blade whirl around Ronin
                spawnSlashArc(this.x, this.y - 44, 1, 88, 0, Math.PI * 2, '#e01b1b', true);
                spawnSlashArc(this.x, this.y - 44, 1, 76, 0.5, Math.PI * 2 + 0.5, '#ffffff', false);

                enemies.forEach(enemy => {
                    if (enemy.dead) return;
                    const dist = Math.hypot(enemy.x - this.x, (enemy.y - 35) - (this.y - 44));
                    if (dist < 96) {
                        const pushDir = enemy.x >= this.x ? 1 : -1;
                        enemy.takeDamage(20, pushDir * 290, false);
                        enemy.stunTimer = 0.55;
                    }
                });
            }

            dash() {
                if (this.stamina < 1 || this.dashTimer > 0) return;
                this.stamina -= 1;
                this.dashTimer = 0.18;
                this.invulnTimer = 0.28;
                this.vy = 0;
                this.vx = this.facing * 860;

                sound.playDash();
                spawnParticles(this.x, this.y - 40, 14, '#0c0c0e', 60, 220, 3, 7, 50);
                spawnParticles(this.x, this.y - 40, 6, '#e01b1b', 80, 200, 2, 5, 50);
            }

            jump() {
                if (this.grounded) {
                    this.vy = this.jumpForce;
                    this.grounded = false;
                    spawnParticles(this.x, this.y, 8, '#1e1e24', 40, 130, 2, 5, 200);
                }
            }

            takeDamage(amount, knockbackX = 0) {
                // Check active Parry window!
                if (this.parryTimer > 0) {
                    sound.playParry();
                    hitstopTimer = 0.065;
                    screenShake = 11;
                    this.invulnTimer = 0.45;
                    this.stamina = Math.min(this.maxStamina, this.stamina + 1);
                    spawnParticles(this.x + this.facing * 22, this.y - 46, 22, '#f5c156', 120, 380, 2, 5, 200);
                    spawnParticles(this.x + this.facing * 22, this.y - 46, 12, '#ffffff', 100, 300, 2, 4, 150);
                    spawnFloatingText(this.x, this.y - 95, 'DEFLECT!', '#f5c156', 1.25);

                    // Counter-shockwave stuns nearby enemies
                    enemies.forEach(e => {
                        if (!e.dead && Math.abs(e.x - this.x) < 130) {
                            e.takeDamage(35, (e.x >= this.x ? 1 : -1) * 420, true);
                            e.stunTimer = 1.1;
                        }
                    });
                    return;
                }

                if (this.invulnTimer > 0 || currentState !== GAME_STATE.PLAYING) return;

                this.hp = Math.max(0, this.hp - amount);
                this.invulnTimer = 0.65;
                this.vx = knockbackX;
                this.vy = -180;
                this.grounded = false;

                combo = 0;
                screenShake = 14;
                hitstopTimer = 0.045;
                sound.playHurt();

                spawnParticles(this.x, this.y - 45, 24, '#d91e18', 90, 320, 3, 7, 500);
                spawnFloatingText(this.x, this.y - 90, `-${amount}`, '#e01b1b', 1.1);

                if (this.hp <= 0) {
                    triggerGameOver();
                }
            }

            update(dt) {
                this.robePhase += dt * (Math.abs(this.vx) > 20 ? 14 : 4.5);

                // Cooldowns
                if (this.attackCooldown > 0) this.attackCooldown -= dt;
                if (this.attackTimer > 0) this.attackTimer -= dt;
                if (this.comboWindow > 0) {
                    this.comboWindow -= dt;
                    if (this.comboWindow <= 0) this.comboStep = 0;
                }
                if (this.parryTimer > 0) this.parryTimer -= dt;
                if (this.parryCooldown > 0) this.parryCooldown -= dt;
                if (this.invulnTimer > 0) this.invulnTimer -= dt;

                // Stamina regen
                if (this.stamina < this.maxStamina) {
                    this.staminaRegenTimer += dt;
                    if (this.staminaRegenTimer >= 1.35) {
                        this.stamina++;
                        this.staminaRegenTimer = 0;
                    }
                }

                // Afterimages during dash
                if (this.dashTimer > 0) {
                    this.dashTimer -= dt;
                    this.afterimages.push({ x: this.x, y: this.y, facing: this.facing, alpha: 0.55 });
                }
                for (let i = this.afterimages.length - 1; i >= 0; i--) {
                    this.afterimages[i].alpha -= dt * 3.2;
                    if (this.afterimages[i].alpha <= 0) this.afterimages.splice(i, 1);
                }

                // Movement input (only during PLAYING state)
                if (currentState === GAME_STATE.PLAYING && this.dashTimer <= 0) {
                    let moveDir = 0;
                    if (keys.left) moveDir -= 1;
                    if (keys.right) moveDir += 1;

                    if (moveDir !== 0) {
                        this.facing = moveDir;
                        this.vx += (moveDir * this.speed - this.vx) * Math.min(1, dt * 18);
                    } else {
                        this.vx *= Math.pow(0.001, dt);
                    }
                } else if (currentState === GAME_STATE.MENU) {
                    this.vx = 0;
                    this.y = GROUND_Y;
                }

                // Gravity
                if (!this.grounded && this.dashTimer <= 0) {
                    this.vy += 1750 * dt;
                }

                this.x += this.vx * dt;
                this.y += this.vy * dt;

                if (this.y >= GROUND_Y) {
                    this.y = GROUND_Y;
                    this.vy = 0;
                    this.grounded = true;
                }

                const margin = 36;
                if (this.x < margin) { this.x = margin; this.vx = 0; }
                if (this.x > W - margin) { this.x = W - margin; this.vx = 0; }
            }

            drawSilhouette(ctx, x, y, facing, alpha = 1, isAfterimage = false) {
                ctx.save();
                ctx.translate(x, y);
                ctx.scale(facing, 1);
                ctx.globalAlpha = alpha;

                const sway = Math.sin(this.robePhase) * 4;
                const runLean = Math.min(0.18, Math.abs(this.vx) / 2200);
                ctx.rotate(runLean);

                const inkMain = isAfterimage ? '#1a0808' : '#0b0b0d';
                const inkTrim = isAfterimage ? '#8f0d0d' : '#1b1b20';

                // 1. Tattered Flowing Ronin Cloak / Hakama
                ctx.fillStyle = inkMain;
                ctx.beginPath();
                ctx.moveTo(-12, -62);
                ctx.lineTo(12, -62);
                ctx.lineTo(22 - sway * 0.6, -6);
                // Tattered hem spikes
                ctx.lineTo(14 - sway, 0);
                ctx.lineTo(8 - sway * 0.5, -8);
                ctx.lineTo(1 - sway, 1);
                ctx.lineTo(-6 - sway * 0.7, -7);
                ctx.lineTo(-14 - sway * 1.2, 2);
                ctx.lineTo(-24 - sway * 1.4, -5);
                ctx.closePath();
                ctx.fill();

                // Flowing Scarf / Tattered Shoulder Wraps
                ctx.fillStyle = inkTrim;
                ctx.beginPath();
                ctx.moveTo(-10, -64);
                ctx.quadraticCurveTo(-26 - sway * 1.6, -58 + sway, -36 - sway * 2, -46 + sway * 1.2);
                ctx.lineTo(-28 - sway * 1.5, -42 + sway);
                ctx.quadraticCurveTo(-18 - sway, -50, -8, -56);
                ctx.closePath();
                ctx.fill();

                // 2. Dual Katanas (Matching the provided artwork: held low & angled outward when idle, dynamic during slash)
                ctx.strokeStyle = '#09090b';
                ctx.lineWidth = 3.2;
                ctx.lineCap = 'round';

                if (this.attackTimer > 0) {
                    // Forward extended slash pose
                    ctx.beginPath();
                    ctx.moveTo(8, -46);
                    ctx.lineTo(54, -54);
                    ctx.stroke();

                    ctx.beginPath();
                    ctx.moveTo(4, -38);
                    ctx.lineTo(46, -28);
                    ctx.stroke();
                } else {
                    // Iconic Dual-Katana Stance from the image (blades pointing down-back and down-forward)
                    // Right hand katana (forward-down)
                    ctx.beginPath();
                    ctx.moveTo(12, -36);
                    ctx.lineTo(44, -6);
                    ctx.stroke();

                    // Left hand katana (back-down)
                    ctx.beginPath();
                    ctx.moveTo(-12, -36);
                    ctx.lineTo(-42, -5);
                    ctx.stroke();
                }

                // 3. Conical Straw Hat (Kasa) with top knot
                ctx.fillStyle = '#0a0a0c';
                ctx.beginPath();
                ctx.moveTo(0, -86); // Top finial
                ctx.lineTo(4, -81);
                ctx.lineTo(34, -68); // Right brim
                ctx.quadraticCurveTo(0, -64, -34, -68); // Left brim
                ctx.lineTo(-4, -81);
                ctx.closePath();
                ctx.fill();

                // Subtle straw texture highlight on kasa
                ctx.strokeStyle = 'rgba(235, 231, 223, 0.16)';
                ctx.lineWidth = 1;
                ctx.beginPath();
                ctx.moveTo(-2, -81);
                ctx.lineTo(18, -69);
                ctx.moveTo(-1, -81);
                ctx.lineTo(8, -68);
                ctx.stroke();

                // 4. Shadowed Face & Glowing Crimson Eye(s) under Kasa
                if (!isAfterimage) {
                    ctx.fillStyle = '#ff1e1e';
                    ctx.shadowColor = '#ff1e1e';
                    ctx.shadowBlur = 8;
                    ctx.beginPath();
                    ctx.arc(6, -63, 2.1, 0, Math.PI * 2);
                    ctx.fill();
                    ctx.shadowBlur = 0;
                }

                // Parry Aura Ring if active
                if (this.parryTimer > 0 && !isAfterimage) {
                    ctx.strokeStyle = 'rgba(245, 193, 86, 0.85)';
                    ctx.lineWidth = 2.5;
                    ctx.beginPath();
                    ctx.arc(0, -44, 48, -Math.PI * 0.45, Math.PI * 0.45);
                    ctx.stroke();
                }

                ctx.restore();
            }

            draw(ctx) {
                // Draw dash afterimages
                this.afterimages.forEach(img => {
                    this.drawSilhouette(ctx, img.x, img.y, img.facing, img.alpha, true);
                });

                // Blink slightly when invulnerable from damage
                const blinkAlpha = (this.invulnTimer > 0 && this.dashTimer <= 0 && this.parryTimer <= 0)
                    ? (Math.floor(globalTime * 24) % 2 === 0 ? 0.45 : 1)
                    : 1;

                this.drawSilhouette(ctx, this.x, this.y, this.facing, blinkAlpha, false);
            }
        }

        // ==========================================
        // 5. ENEMY ARCHETYPES
        // ==========================================
        class Enemy {
            constructor(type, side) {
                this.type = type; // 'ashigaru' | 'ninja' | 'brute' | 'crow'
                this.facing = side === 'left' ? 1 : -1;
                this.x = side === 'left' ? -45 : W + 45;
                this.y = type === 'crow' ? GROUND_Y - 150 - Math.random() * 80 : GROUND_Y;
                this.vx = 0;
                this.vy = 0;
                this.dead = false;
                this.deathTimer = 0;
                this.flashTimer = 0;
                this.stunTimer = 0;
                this.attackWindup = 0;
                this.attackCooldown = 0.8 + Math.random() * 0.6;
                this.animPhase = Math.random() * 6;

                if (type === 'ashigaru') {
                    this.maxHp = 50;
                    this.speed = 145 + Math.random() * 35;
                    this.reach = 76;
                    this.damage = 14;
                    this.height = 78;
                    this.points = 100;
                } else if (type === 'ninja') {
                    this.maxHp = 40;
                    this.speed = 250 + Math.random() * 40;
                    this.reach = 60;
                    this.damage = 12;
                    this.height = 72;
                    this.points = 150;
                } else if (type === 'brute') {
                    this.maxHp = 135;
                    this.speed = 88;
                    this.reach = 92;
                    this.damage = 25;
                    this.height = 98;
                    this.points = 300;
                } else if (type === 'crow') {
                    this.maxHp = 30;
                    this.speed = 230;
                    this.reach = 48;
                    this.damage = 12;
                    this.height = 36;
                    this.points = 120;
                    this.swoopPhase = 0;
                }
                this.hp = this.maxHp;
            }

            takeDamage(amount, knockbackX, heavy = false) {
                if (this.dead) return;
                this.hp -= amount;
                this.flashTimer = 0.12;
                this.vx = knockbackX;
                this.attackWindup = 0; // Interrupt enemy windup!

                sound.playHit(heavy);

                // Increment combo
                combo++;
                comboTimer = 3.2;
                if (combo > maxCombo) maxCombo = combo;

                // Crimson ink splatter particles
                spawnParticles(this.x, this.y - this.height * 0.5, heavy ? 22 : 12, '#d91e18', 80, 320, 2.5, 6.5, 480);
                spawnParticles(this.x, this.y - this.height * 0.5, 8, '#0c0c0e', 60, 220, 2, 5, 420);

                if (this.hp <= 0) {
                    this.dead = true;
                    this.deathTimer = 0.35;
                    kills++;
                    const earned = this.points * (1 + Math.floor(combo / 5) * 0.5);
                    score += Math.round(earned);
                    spawnParticles(this.x, this.y - this.height * 0.45, 28, '#0c0c0e', 90, 340, 3, 8, 520);
                    spawnParticles(this.x, this.y - this.height * 0.45, 18, '#e01b1b', 100, 360, 3, 7, 500);
                }
            }

            update(dt) {
                this.animPhase += dt * 10;
                if (this.flashTimer > 0) this.flashTimer -= dt;

                if (this.dead) {
                    this.deathTimer -= dt;
                    this.x += this.vx * dt;
                    this.vx *= 0.9;
                    return;
                }

                if (this.stunTimer > 0) {
                    this.stunTimer -= dt;
                    this.x += this.vx * dt;
                    this.vx *= Math.pow(0.01, dt);
                    return;
                }

                if (this.attackCooldown > 0) this.attackCooldown -= dt;

                const dx = player.x - this.x;
                const distX = Math.abs(dx);
                this.facing = dx >= 0 ? 1 : -1;

                if (this.type === 'crow') {
                    // Aerial swooping shadow crow
                    this.swoopPhase += dt * 3;
                    const targetY = distX < 210 ? GROUND_Y - 46 : GROUND_Y - 155 + Math.sin(this.swoopPhase) * 24;
                    this.vx += (this.facing * this.speed - this.vx) * Math.min(1, dt * 4);
                    this.vy += (targetY - this.y) * 4 * dt;

                    this.x += this.vx * dt;
                    this.y += this.vy * dt;

                    if (Math.hypot(player.x - this.x, (player.y - 44) - this.y) < 46 && this.attackCooldown <= 0) {
                        player.takeDamage(this.damage, this.facing * 200);
                        this.attackCooldown = 1.2;
                        this.vy = -260;
                    }
                    return;
                }

                // Ground Enemies (Ashigaru, Ninja, Brute)
                if (this.attackWindup > 0) {
                    this.attackWindup -= dt;
                    this.vx *= Math.pow(0.001, dt);
                    if (this.attackWindup <= 0) {
                        // Execute strike
                        this.vx = this.facing * (this.type === 'ninja' ? 340 : 180);
                        sound.playSlash(this.type === 'brute' ? 0.7 : 1.15);
                        spawnSlashArc(this.x, this.y - this.height * 0.5, this.facing, this.reach, -0.8, 0.8, '#e01b1b', true);

                        if (Math.abs(player.x - this.x) <= this.reach + 14 && Math.abs(player.y - this.y) < 55) {
                            player.takeDamage(this.damage, this.facing * (this.type === 'brute' ? 380 : 220));
                        }
                        this.attackCooldown = this.type === 'ninja' ? 0.95 : (this.type === 'brute' ? 1.65 : 1.25);
                    }
                } else if (distX > this.reach - 10) {
                    this.vx += (this.facing * this.speed - this.vx) * Math.min(1, dt * 10);
                } else {
                    this.vx *= Math.pow(0.001, dt);
                    if (this.attackCooldown <= 0) {
                        this.attackWindup = this.type === 'brute' ? 0.55 : (this.type === 'ninja' ? 0.32 : 0.42);
                    }
                }

                this.x += this.vx * dt;
                this.y = GROUND_Y;
            }

            draw(ctx) {
                ctx.save();
                ctx.translate(this.x, this.y);
                ctx.scale(this.facing, 1);

                if (this.dead) {
                    ctx.globalAlpha = Math.max(0, this.deathTimer / 0.35);
                }

                const bodyColor = this.flashTimer > 0 ? '#e01b1b' : '#0c0c0e';
                ctx.fillStyle = bodyColor;

                if (this.type === 'crow') {
                    const flap = Math.sin(this.animPhase * 1.6);
                    ctx.beginPath();
                    ctx.moveTo(18, 0);
                    ctx.quadraticCurveTo(4, -8, -14, -2);
                    ctx.lineTo(-26, -18 * flap - 6);
                    ctx.lineTo(-10, 4);
                    ctx.lineTo(-22, 8);
                    ctx.closePath();
                    ctx.fill();

                    // Glowing red eye
                    ctx.fillStyle = '#ff2424';
                    ctx.beginPath();
                    ctx.arc(10, -2, 2.2, 0, Math.PI * 2);
                    ctx.fill();
                    ctx.restore();
                    return;
                }

                const bob = Math.abs(Math.sin(this.animPhase)) * 3;

                if (this.type === 'ashigaru') {
                    // Spearman with Jingasa war hat
                    ctx.beginPath();
                    ctx.moveTo(-12, -58 + bob);
                    ctx.lineTo(12, -58 + bob);
                    ctx.lineTo(18, 0);
                    ctx.lineTo(-18, 0);
                    ctx.closePath();
                    ctx.fill();

                    // Jingasa hat
                    ctx.beginPath();
                    ctx.moveTo(0, -76 + bob);
                    ctx.lineTo(24, -63 + bob);
                    ctx.lineTo(-24, -63 + bob);
                    ctx.closePath();
                    ctx.fill();

                    // Long Yari Spear
                    ctx.strokeStyle = bodyColor;
                    ctx.lineWidth = 3;
                    const thrust = this.attackWindup > 0 ? -14 : 0;
                    ctx.beginPath();
                    ctx.moveTo(-28 + thrust, -34 + bob);
                    ctx.lineTo(66 + thrust, -42 + bob);
                    ctx.stroke();
                } else if (this.type === 'ninja') {
                    // Crouched agile shinobi
                    ctx.beginPath();
                    ctx.moveTo(-10, -52 + bob);
                    ctx.lineTo(14, -50 + bob);
                    ctx.lineTo(20, 0);
                    ctx.lineTo(-18, 0);
                    ctx.closePath();
                    ctx.fill();

                    // Hooded head + flowing headband
                    ctx.beginPath();
                    ctx.arc(4, -60 + bob, 10, 0, Math.PI * 2);
                    ctx.fill();

                    // Dual short tanto blades
                    ctx.strokeStyle = bodyColor;
                    ctx.lineWidth = 2.8;
                    ctx.beginPath();
                    ctx.moveTo(10, -38 + bob);
                    ctx.lineTo(38, -46 + bob);
                    ctx.moveTo(6, -30 + bob);
                    ctx.lineTo(34, -22 + bob);
                    ctx.stroke();
                } else if (this.type === 'brute') {
                    // Towering Armored Samurai / Oni-masked General
                    ctx.beginPath();
                    ctx.moveTo(-22, -78 + bob);
                    ctx.lineTo(22, -78 + bob);
                    ctx.lineTo(28, 0);
                    ctx.lineTo(-28, 0);
                    ctx.closePath();
                    ctx.fill();

                    // Kabuto Helmet with Crescent Horns
                    ctx.beginPath();
                    ctx.arc(0, -84 + bob, 15, Math.PI, 0);
                    ctx.lineTo(18, -78 + bob);
                    ctx.lineTo(-18, -78 + bob);
                    ctx.closePath();
                    ctx.fill();

                    // Crescent horns
                    ctx.strokeStyle = bodyColor;
                    ctx.lineWidth = 3.5;
                    ctx.beginPath();
                    ctx.arc(0, -90 + bob, 16, 0.15 * Math.PI, 0.85 * Math.PI, true);
                    ctx.stroke();

                    // Heavy Odachi Greatsword
                    ctx.lineWidth = 4.5;
                    const lift = this.attackWindup > 0 ? -38 : 0;
                    ctx.beginPath();
                    ctx.moveTo(14, -48 + bob);
                    ctx.lineTo(74, -62 + lift + bob);
                    ctx.stroke();
                }

                // Glowing red eyes
                ctx.fillStyle = '#ff2222';
                ctx.beginPath();
                ctx.arc(7, (this.type === 'brute' ? -80 : -59) + bob, 2, 0, Math.PI * 2);
                ctx.fill();

                // Telegraph glint right before enemy strikes!
                if (this.attackWindup > 0 && this.attackWindup < 0.24) {
                    ctx.strokeStyle = '#ff2a2a';
                    ctx.lineWidth = 2.5;
                    ctx.beginPath();
                    ctx.moveTo(16, -54);
                    ctx.lineTo(32, -38);
                    ctx.moveTo(32, -54);
                    ctx.lineTo(16, -38);
                    ctx.stroke();
                }

                ctx.restore();

                // Mini health bar above damaged Brutes
                if (this.type === 'brute' && this.hp < this.maxHp && !this.dead) {
                    const bw = 48;
                    ctx.fillStyle = 'rgba(0,0,0,0.7)';
                    ctx.fillRect(this.x - bw / 2, this.y - this.height - 16, bw, 5);
                    ctx.fillStyle = '#e01b1b';
                    ctx.fillRect(this.x - bw / 2, this.y - this.height - 16, bw * Math.max(0, this.hp / this.maxHp), 5);
                }
            }
        }

        const player = new Player();
        const enemies = [];

        // ==========================================
        // 6. WAVE DIRECTOR & GAME FLOW
        // ==========================================
        const waveSubtitles = [
            'THE CROWS GATHER',
            'SHADOWS ON THE MARSH',
            'BLADES OF THE SHOGUN',
            'CRIMSON ECLIPSE',
            'WRATH OF THE IRON ONI',
            'ENDLESS BLOOD MOON'
        ];

        function toRoman(num) {
            const map = [
                [10, 'X'], [9, 'IX'], [5, 'V'], [4, 'IV'], [1, 'I']
            ];
            let res = '';
            for (const [val, sym] of map) {
                while (num >= val) { res += sym; num -= val; }
            }
            return res;
        }

        function showWaveBanner(waveNum) {
            const banner = document.getElementById('wave-banner');
            document.getElementById('banner-title').textContent = `WAVE ${toRoman(waveNum)}`;
            document.getElementById('banner-sub').textContent = waveSubtitles[(waveNum - 1) % waveSubtitles.length];
            banner.classList.add('show');
            sound.playTaikoBeat();
            setTimeout(() => {
                banner.classList.remove('show');
            }, 2200);
        }

        function startWave(waveNum) {
            wave = waveNum;
            waveEnemiesToSpawn = [];
            const totalCount = 4 + waveNum * 2;

            for (let i = 0; i < totalCount; i++) {
                const side = Math.random() < 0.5 ? 'left' : 'right';
                let type = 'ashigaru';
                const roll = Math.random();
                if (waveNum >= 3 && i === totalCount - 1) {
                    type = 'brute';
                } else if (waveNum >= 2 && roll < 0.32) {
                    type = 'ninja';
                } else if (waveNum >= 2 && roll < 0.52) {
                    type = 'crow';
                } else if (waveNum >= 4 && roll < 0.68) {
                    type = 'brute';
                }
                waveEnemiesToSpawn.push({ type, side });
            }

            spawnTimer = 0.4;
            showWaveBanner(waveNum);
            updateHUD();
        }

        function startGame() {
            sound.init();
            score = 0;
            kills = 0;
            combo = 0;
            maxCombo = 0;
            enemies.length = 0;
            particles.length = 0;
            slashArcs.length = 0;
            floatingTexts.length = 0;

            player.reset();
            currentState = GAME_STATE.PLAYING;

            document.getElementById('title-screen').classList.add('hidden');
            document.getElementById('gameover-screen').classList.add('hidden');
            document.getElementById('hud').classList.remove('hidden');
            document.getElementById('touch-controls').classList.remove('hidden');

            startWave(1);
        }

        function triggerGameOver() {
            currentState = GAME_STATE.GAMEOVER;
            document.getElementById('hud').classList.add('hidden');
            document.getElementById('touch-controls').classList.add('hidden');

            document.getElementById('final-score').textContent = score.toLocaleString();
            document.getElementById('final-wave').textContent = toRoman(wave);
            document.getElementById('final-kills').textContent = kills;
            document.getElementById('final-combo').textContent = `${maxCombo}x`;

            document.getElementById('gameover-screen').classList.remove('hidden');
        }

        function updateHUD() {
            const hpPct = (player.hp / player.maxHp) * 100;
            document.getElementById('health-fill').style.width = `${hpPct}%`;

            const pips = document.querySelectorAll('#stamina-pips .stamina-pip');
            pips.forEach((pip, idx) => {
                pip.classList.toggle('ready', idx < player.stamina);
            });

            document.getElementById('hud-wave').textContent = `WAVE ${toRoman(wave)}`;
            const remaining = enemies.filter(e => !e.dead).length + waveEnemiesToSpawn.length;
            document.getElementById('hud-enemies').textContent = `FOES REMAINING: ${remaining}`;
            document.getElementById('hud-score').textContent = score.toLocaleString();

            const comboEl = document.getElementById('combo-container');
            if (combo >= 2) {
                comboEl.classList.add('active');
                document.getElementById('combo-count').textContent = `${combo}x`;
            } else {
                comboEl.classList.remove('active');
            }
        }

        // ==========================================
        // 7. INPUT HANDLING (KEYBOARD, MOUSE, TOUCH)
        // ==========================================
        const keys = { left: false, right: false };

        window.addEventListener('keydown', (e) => {
            if (e.repeat) return;
            const code = e.code;

            if (code === 'KeyA' || code === 'ArrowLeft') keys.left = true;
            if (code === 'KeyD' || code === 'ArrowRight') keys.right = true;

            if (currentState !== GAME_STATE.PLAYING) {
                if (code === 'Enter' || code === 'Space') startGame();
                return;
            }

            if (code === 'KeyW' || code === 'ArrowUp' || code === 'Space') {
                e.preventDefault();
                player.jump();
            }
            if (code === 'KeyJ' || code === 'KeyZ') {
                player.attack();
            }
            if (code === 'KeyL' || code === 'KeyX') {
                player.parryAndWhirl();
            }
            if (code === 'ShiftLeft' || code === 'ShiftRight' || code === 'KeyK' || code === 'KeyC') {
                player.dash();
            }
        });

        window.addEventListener('keyup', (e) => {
            const code = e.code;
            if (code === 'KeyA' || code === 'ArrowLeft') keys.left = false;
            if (code === 'KeyD' || code === 'ArrowRight') keys.right = false;
        });

        // Mouse click combat on canvas
        canvas.addEventListener('mousedown', (e) => {
            if (currentState !== GAME_STATE.PLAYING) return;
            sound.init();
            // Turn toward mouse click X
            player.facing = e.clientX >= player.x ? 1 : -1;
            if (e.button === 2) {
                player.parryAndWhirl();
            } else if (e.button === 0) {
                player.attack();
            }
        });

        canvas.addEventListener('contextmenu', (e) => e.preventDefault());

        // Touch / On-screen buttons
        function bindHoldButton(id, onDown, onUp) {
            const btn = document.getElementById(id);
            const start = (e) => {
                e.preventDefault();
                sound.init();
                btn.classList.add('pressed');
                onDown();
            };
            const end = (e) => {
                e.preventDefault();
                btn.classList.remove('pressed');
                if (onUp) onUp();
            };
            btn.addEventListener('touchstart', start, { passive: false });
            btn.addEventListener('touchend', end, { passive: false });
            btn.addEventListener('touchcancel', end, { passive: false });
            btn.addEventListener('mousedown', start);
            btn.addEventListener('mouseup', end);
            btn.addEventListener('mouseleave', end);
        }

        bindHoldButton('btn-left', () => { keys.left = true; }, () => { keys.left = false; });
        bindHoldButton('btn-right', () => { keys.right = true; }, () => { keys.right = false; });
        bindHoldButton('btn-jump', () => { if (currentState === GAME_STATE.PLAYING) player.jump(); });
        bindHoldButton('btn-dash', () => { if (currentState === GAME_STATE.PLAYING) player.dash(); });
        bindHoldButton('btn-parry', () => { if (currentState === GAME_STATE.PLAYING) player.parryAndWhirl(); });
        bindHoldButton('btn-attack', () => { if (currentState === GAME_STATE.PLAYING) player.attack(); });

        document.getElementById('start-btn').addEventListener('click', startGame);
        document.getElementById('restart-btn').addEventListener('click', startGame);

        document.getElementById('mute-btn').addEventListener('click', () => {
            sound.init();
            const isMuted = sound.toggleMute();
            document.getElementById('sound-on-icon').style.display = isMuted ? 'none' : 'block';
            document.getElementById('sound-off-icon').style.display = isMuted ? 'block' : 'none';
        });

        // ==========================================
        // 8. SUMI-E ENVIRONMENT RENDERER
        // ==========================================
        function drawEnvironment(ctx) {
            // 1. Parchment Sky Backdrop
            const skyGrad = ctx.createRadialGradient(W * 0.5, H * 0.36, 20, W * 0.5, H * 0.45, Math.max(W, H) * 0.85);
            skyGrad.addColorStop(0, '#f2eee6');
            skyGrad.addColorStop(0.55, '#dcd5c7');
            skyGrad.addColorStop(1, '#b8b0a2');
            ctx.fillStyle = skyGrad;
            ctx.fillRect(0, 0, W, H);

            // 2. Massive Crimson Sun (Centered behind the Ronin)
            const sunX = W * 0.5;
            const sunY = H * 0.37;
            const sunRadius = Math.min(W, H) * 0.27;

            ctx.save();
            // Subtle ink bleed halo
            const sunHalo = ctx.createRadialGradient(sunX, sunY, sunRadius * 0.85, sunX, sunY, sunRadius * 1.12);
            sunHalo.addColorStop(0, 'rgba(217, 30, 24, 0.35)');
            sunHalo.addColorStop(1, 'rgba(217, 30, 24, 0)');
            ctx.fillStyle = sunHalo;
            ctx.beginPath();
            ctx.arc(sunX, sunY, sunRadius * 1.12, 0, Math.PI * 2);
            ctx.fill();

            // Main Red Sun Disc
            const sunDiscGrad = ctx.createRadialGradient(sunX, sunY - sunRadius * 0.2, 10, sunX, sunY, sunRadius);
            sunDiscGrad.addColorStop(0, '#e8241e');
            sunDiscGrad.addColorStop(0.85, '#cf1914');
            sunDiscGrad.addColorStop(1, '#b3120e');
            ctx.fillStyle = sunDiscGrad;
            ctx.beginPath();
            ctx.arc(sunX, sunY, sunRadius, 0, Math.PI * 2);
            ctx.fill();

            // Sumi-e ink splatters around sun perimeter
            sunSplatters.forEach(sp => {
                const sx = sunX + Math.cos(sp.angle) * sunRadius * sp.distRatio;
                const sy = sunY + Math.sin(sp.angle) * sunRadius * sp.distRatio;
                ctx.fillStyle = sp.isCrimson ? 'rgba(205, 22, 18, 0.78)' : 'rgba(18, 18, 22, 0.65)';
                ctx.beginPath();
                ctx.arc(sx, sy, sp.radius, 0, Math.PI * 2);
                ctx.fill();
            });
            ctx.restore();

            // 3. Ambient Flying Crows (Pass in front of the red sun)
            ambientCrows.forEach(crow => crow.draw(ctx));

            // 4. Misty Distant Mountains (Sumi-e ink wash silhouette)
            ctx.save();
            ctx.fillStyle = 'rgba(40, 40, 46, 0.24)';
            ctx.beginPath();
            ctx.moveTo(0, GROUND_Y);
            mountainPoints.forEach(pt => {
                ctx.lineTo(pt.nx * W, GROUND_Y - pt.height);
            });
            ctx.lineTo(W, GROUND_Y);
            ctx.closePath();
            ctx.fill();

            // Low ground mist band
            const mistGrad = ctx.createLinearGradient(0, GROUND_Y - 45, 0, GROUND_Y + 5);
            mistGrad.addColorStop(0, 'rgba(235, 231, 223, 0)');
            mistGrad.addColorStop(1, 'rgba(235, 231, 223, 0.65)');
            ctx.fillStyle = mistGrad;
            ctx.fillRect(0, GROUND_Y - 45, W, 50);
            ctx.restore();

            // 5. Foreground Marsh Terrain & Crimson Water Reflections
            const groundHeight = H - GROUND_Y;
            const groundGrad = ctx.createLinearGradient(0, GROUND_Y, 0, H);
            groundGrad.addColorStop(0, '#18181c');
            groundGrad.addColorStop(0.25, '#0e0e11');
            groundGrad.addColorStop(1, '#070709');
            ctx.fillStyle = groundGrad;
            ctx.fillRect(0, GROUND_Y, W, groundHeight);

            // Jagged ink brush horizon line
            ctx.strokeStyle = '#09090b';
            ctx.lineWidth = 4;
            ctx.beginPath();
            ctx.moveTo(0, GROUND_Y);
            for (let x = 0; x <= W; x += 40) {
                const jag = Math.sin(x * 0.05) * 2.5;
                ctx.lineTo(x, GROUND_Y + jag);
            }
            ctx.stroke();

            // Reflective water puddles with shimmering crimson sun reflections
            puddles.forEach((p, idx) => {
                const px = p.nx * W;
                const pw = p.nw * W;
                const py = GROUND_Y + 14 + (idx % 2) * 16;

                // Dark mirror water surface
                ctx.fillStyle = 'rgba(32, 34, 40, 0.85)';
                ctx.beginPath();
                ctx.ellipse(px, py, pw * 0.5, 6, 0, 0, Math.PI * 2);
                ctx.fill();

                // Crimson reflection shimmer
                const shimmer = Math.sin(globalTime * 3 + idx) * 6;
                ctx.fillStyle = 'rgba(217, 30, 24, 0.42)';
                ctx.beginPath();
                ctx.ellipse(px + shimmer, py, pw * 0.34, 3.2, 0, 0, Math.PI * 2);
                ctx.fill();
            });
        }

        // ==========================================
        // 9. MAIN ANIMATION & GAME LOOP
        // ==========================================
        let lastTimestamp = 0;

        function update(dt) {
            globalTime += dt;

            // Always update background crows & drifting ash particles
            ambientCrows.forEach(c => c.update(dt));

            // Occasional floating ash/ink embers
            if (Math.random() < 0.35) {
                particles.push({
                    x: Math.random() * W,
                    y: GROUND_Y - Math.random() * H * 0.5,
                    vx: 18 + Math.random() * 25,
                    vy: -12 - Math.random() * 18,
                    size: 1.5 + Math.random() * 2.2,
                    color: Math.random() < 0.3 ? '#d91e18' : '#16161a',
                    alpha: 0.7,
                    life: 2.2,
                    maxLife: 2.2,
                    gravity: 0
                });
            }

            if (hitstopTimer > 0) {
                hitstopTimer -= dt;
                return;
            }

            if (screenShake > 0) {
                screenShake = Math.max(0, screenShake - dt * 42);
            }

            player.update(dt);

            if (currentState === GAME_STATE.PLAYING) {
                // Combo decay timer
                if (combo > 0) {
                    comboTimer -= dt;
                    if (comboTimer <= 0) {
                        combo = 0;
                    }
                }

                // Spawn queued wave enemies
                if (waveEnemiesToSpawn.length > 0) {
                    spawnTimer -= dt;
                    if (spawnTimer <= 0) {
                        const next = waveEnemiesToSpawn.shift();
                        enemies.push(new Enemy(next.type, next.side));
                        spawnTimer = Math.max(0.55, 1.45 - wave * 0.08);
                    }
                } else if (enemies.length === 0) {
                    // Wave cleared! Heal player slightly & start next wave
                    waveClearCooldown += dt;
                    if (waveClearCooldown >= 1.4) {
                        waveClearCooldown = 0;
                        player.hp = Math.min(player.maxHp, player.hp + 15);
                        startWave(wave + 1);
                    }
                }

                // Update enemies
                for (let i = enemies.length - 1; i >= 0; i--) {
                    enemies[i].update(dt);
                    if (enemies[i].dead && enemies[i].deathTimer <= 0) {
                        enemies.splice(i, 1);
                    }
                }

                updateHUD();
            }

            // Update particles
            for (let i = particles.length - 1; i >= 0; i--) {
                const p = particles[i];
                p.life -= dt;
                p.vy += p.gravity * dt;
                p.x += p.vx * dt;
                p.y += p.vy * dt;
                p.alpha = Math.max(0, p.life / p.maxLife);
                if (p.life <= 0) particles.splice(i, 1);
            }

            // Update slash arcs
            for (let i = slashArcs.length - 1; i >= 0; i--) {
                slashArcs[i].life -= dt;
                if (slashArcs[i].life <= 0) slashArcs.splice(i, 1);
            }

            // Update floating combat texts
            for (let i = floatingTexts.length - 1; i >= 0; i--) {
                const ft = floatingTexts[i];
                ft.life -= dt;
                ft.y += ft.vy * dt;
                if (ft.life <= 0) floatingTexts.splice(i, 1);
            }
        }

        function render() {
            ctx.save();

            // Apply screen shake offset
            if (screenShake > 0) {
                const sx = (Math.random() - 0.5) * screenShake * 1.4;
                const sy = (Math.random() - 0.5) * screenShake * 1.4;
                ctx.translate(sx, sy);
            }

            drawEnvironment(ctx);

            // Draw Enemies
            enemies.forEach(enemy => enemy.draw(ctx));

            // Draw Player (Crimson Ronin)
            player.draw(ctx);

            // Draw Katana Slash Arcs
            slashArcs.forEach(arc => {
                ctx.save();
                ctx.translate(arc.x, arc.y);
                ctx.scale(arc.facing, 1);
                const progress = 1 - arc.life / arc.maxLife;
                ctx.globalAlpha = Math.max(0, 1 - progress);

                ctx.strokeStyle = arc.color;
                ctx.lineWidth = arc.isCrimson ? 12 * (1 - progress * 0.5) : 7 * (1 - progress * 0.5);
                ctx.lineCap = 'round';
                ctx.beginPath();
                ctx.arc(0, 0, arc.radius, arc.startAngle, arc.endAngle, arc.startAngle > arc.endAngle);
                ctx.stroke();
                ctx.restore();
            });

            // Draw Particles
            particles.forEach(p => {
                ctx.save();
                ctx.globalAlpha = p.alpha;
                ctx.fillStyle = p.color;
                ctx.beginPath();
                ctx.arc(p.x, p.y, p.size, 0, Math.PI * 2);
                ctx.fill();
                ctx.restore();
            });

            // Draw Floating Callouts
            floatingTexts.forEach(ft => {
                ctx.save();
                ctx.globalAlpha = Math.max(0, ft.life / ft.maxLife);
                ctx.font = `900 ${Math.round(16 * ft.scale)}px 'Cinzel', serif`;
                ctx.textAlign = 'center';
                ctx.fillStyle = '#0c0c0e';
                ctx.fillText(ft.text, ft.x + 1.5, ft.y + 1.5);
                ctx.fillStyle = ft.color;
                ctx.fillText(ft.text, ft.x, ft.y);
                ctx.restore();
            });

            ctx.restore();
        }

        function gameLoop(timestamp) {
            if (!lastTimestamp) lastTimestamp = timestamp;
            const dt = Math.min(0.05, (timestamp - lastTimestamp) / 1000);
            lastTimestamp = timestamp;

            update(dt);
            render();

            requestAnimationFrame(gameLoop);
        }

        window.onload = function () {
            resizeCanvas();
            player.reset();
            requestAnimationFrame(gameLoop);
        };
    </script>
</body>
</html>

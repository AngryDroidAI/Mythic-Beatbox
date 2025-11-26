<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Mythic Beat Generator</title>
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <style>
    :root {
      --bg: #0b0f14;
      --panel: #141a22;
      --accent: #5cf1ff;
      --accent2: #ff5cf1;
      --text: #e9f0f7;
      --muted: #98a6b5;
      --on: #1fd1a1;
      --off: #2a3442;
      --grid-border: #203042;
    }
    body {
      margin: 0;
      background: radial-gradient(1200px 600px at 20% 0%, #10151c, #0b0f14 70%);
      color: var(--text);
      font-family: ui-sans-serif, system-ui, -apple-system, Segoe UI, Roboto, Ubuntu, Cantarell, "Helvetica Neue", Arial;
      display: grid;
      place-items: center;
      min-height: 100vh;
      overflow-x: hidden;
    }
    .app {
      width: min(1200px, 96vw);
      background: linear-gradient(180deg, rgba(255,255,255,0.03), rgba(255,255,255,0.01)),
                  var(--panel);
      border: 1px solid #1b232f;
      border-radius: 16px;
      box-shadow: 0 40px 120px rgba(0,0,0,0.45), inset 0 1px 0 rgba(255,255,255,0.04);
      overflow: hidden;
    }
    header {
      display: flex;
      align-items: center;
      justify-content: space-between;
      padding: 14px 18px;
      background: linear-gradient(0deg, #101620, #0e151e);
      border-bottom: 1px solid #1b232f;
    }
    header .title {
      font-weight: 700;
      letter-spacing: 0.3px;
    }
    header .title span {
      color: var(--accent);
    }
    .controls {
      display: flex;
      gap: 10px;
      flex-wrap: wrap;
      align-items: center;
    }
    .controls button, .controls input, .controls select, .controls label {
      background: #111822;
      color: var(--text);
      border: 1px solid #263243;
      padding: 8px 10px;
      border-radius: 8px;
      font-size: 14px;
    }
    .controls button {
      cursor: pointer;
      transition: transform 0.05s ease, background 0.2s ease;
    }
    .controls button:hover { background: #152132; }
    .controls button:active { transform: translateY(1px); }
    .grid-wrap {
      display: grid;
      grid-template-columns: 200px 1fr;
      gap: 0;
      position: relative;
    }
    .tracks {
      display: grid;
      grid-template-rows: repeat(6, 1fr);
      border-right: 1px solid var(--grid-border);
    }
    .track-label {
      display: flex;
      align-items: center;
      gap: 8px;
      padding: 10px 14px;
      border-bottom: 1px solid var(--grid-border);
      background: #101722;
    }
    .track-label .color {
      width: 10px; height: 10px; border-radius: 50%;
    }
    .sequence {
      display: grid;
      grid-template-rows: repeat(6, 60px);
      grid-template-columns: repeat(16, 1fr);
    }
    .cell {
      border-left: 1px solid #1d2a3a;
      border-bottom: 1px solid #1d2a3a;
      background: #0e1620;
      cursor: pointer;
      position: relative;
    }
    .cell.on {
      background: linear-gradient(180deg, #173040, #122636);
      box-shadow: inset 0 0 0 1px #29465d, inset 0 0 10px rgba(92,241,255,0.18);
    }
    .cell.on::after {
      content: "";
      position: absolute;
      inset: 12px;
      border-radius: 6px;
      background:
        radial-gradient(120% 120% at 0% 0%, rgba(92,241,255,0.30), transparent 60%),
        radial-gradient(100% 100% at 100% 100%, rgba(255,92,241,0.18), transparent 50%);
    }
    .playhead {
      position: absolute;
      top: 0; bottom: 0;
      width: 0;
      border-left: 2px solid var(--accent);
      pointer-events: none;
      z-index: 10;
      transition: left 0.1s linear;
    }
    .meters {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 14px;
      padding: 16px 18px;
      border-top: 1px solid #1b232f;
      background: #0f1621;
    }
    canvas {
      width: 100%;
      height: 120px;
      background: #0c131b;
      border: 1px solid #1d2a3a;
      border-radius: 10px;
    }

    /* Graphic Mixer */
    .mixer-section {
      padding: 20px;
      background: #0f1621;
      border-top: 1px solid #1b232f;
    }
    .mixer-title {
      text-align: center;
      font-weight: 600;
      margin-bottom: 15px;
      color: var(--accent);
    }
    .mixer-controls {
      display: grid;
      grid-template-columns: repeat(3, 1fr);
      gap: 20px;
      max-width: 400px;
      margin: 0 auto;
    }
    .eq-knob {
      display: flex;
      flex-direction: column;
      align-items: center;
      gap: 10px;
    }
    .knob-container {
      position: relative;
      width: 60px;
      height: 60px;
    }
    .knob {
      width: 100%;
      height: 100%;
      border-radius: 50%;
      background: radial-gradient(circle at 30% 30%, #1a2333, #0e1520);
      border: 2px solid #263243;
      position: relative;
      cursor: pointer;
    }
    .knob-indicator {
      position: absolute;
      width: 3px;
      height: 20px;
      background: var(--accent);
      top: 5px;
      left: 50%;
      transform: translateX(-50%) rotate(0deg);
      transform-origin: bottom center;
      border-radius: 2px;
    }
    .knob-value {
      font-size: 12px;
      color: var(--muted);
    }
    .eq-label {
      font-size: 14px;
      font-weight: 600;
      color: var(--accent);
    }

    /* Sample Loader */
    .sample-loader {
      padding: 15px;
      background: #0e151e;
      border-top: 1px solid #1b232f;
    }
    .sample-controls {
      display: flex;
      gap: 10px;
      justify-content: center;
      flex-wrap: wrap;
    }
    .sample-track {
      display: flex;
      align-items: center;
      gap: 8px;
    }

    /* Preset Packs */
    .preset-section {
      padding: 15px;
      background: #101620;
      border-top: 1px solid #1b232f;
    }
    .preset-grid {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
      gap: 10px;
      max-width: 500px;
      margin: 0 auto;
    }
    .preset-btn {
      background: linear-gradient(145deg, #1a2333, #0e1520);
      border: 1px solid #263243;
      padding: 10px;
      border-radius: 8px;
      cursor: pointer;
      transition: all 0.3s ease;
      text-align: center;
    }
    .preset-btn:hover {
      border-color: var(--accent);
      transform: translateY(-2px);
    }
    .preset-btn.active {
      border-color: var(--accent);
      background: linear-gradient(145deg, #1a2a40, #0e1a30);
      box-shadow: 0 0 15px rgba(92, 241, 255, 0.3);
    }

    /* DJ Crossfader */
    .crossfader-section {
      padding: 20px;
      background: #0f1621;
      border-top: 1px solid #1b232f;
    }
    .crossfader-container {
      max-width: 300px;
      margin: 0 auto;
    }
    .crossfader {
      width: 100%;
      height: 20px;
      background: linear-gradient(to right, #5cf1ff, #ff5cf1);
      border-radius: 10px;
      position: relative;
      cursor: pointer;
    }
    .crossfader-handle {
      width: 30px;
      height: 30px;
      background: var(--text);
      border-radius: 50%;
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      box-shadow: 0 0 10px rgba(0,0,0,0.5);
    }
    .pattern-labels {
      display: flex;
      justify-content: space-between;
      margin-top: 10px;
      font-size: 14px;
    }

    /* BPM Visualizer */
    .bpm-visualizer {
      position: fixed;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      pointer-events: none;
      z-index: -1;
    }
    .pulse-circle {
      position: absolute;
      border-radius: 50%;
      opacity: 0;
      transform: scale(0);
    }

    /* Drum Simulator */
    .drum-simulator {
      display: flex;
      justify-content: center;
      gap: 20px;
      padding: 20px;
      background: linear-gradient(0deg, #0e151e, #101620);
      border-top: 1px solid #1b232f;
    }
    .drum-pad {
      width: 80px;
      height: 80px;
      border-radius: 50%;
      background: radial-gradient(circle at 30% 30%, #1a2333, #0e1520);
      border: 2px solid #263243;
      display: flex;
      align-items: center;
      justify-content: center;
      cursor: pointer;
      transition: all 0.1s ease;
      position: relative;
      overflow: hidden;
    }
    .drum-pad.active {
      transform: scale(0.95);
      box-shadow: 0 0 20px rgba(92, 241, 255, 0.4);
      border-color: #5cf1ff;
    }
    .drum-pad.active::after {
      content: "";
      position: absolute;
      inset: 0;
      border-radius: 50%;
      background: radial-gradient(circle at center, rgba(92, 241, 255, 0.2), transparent 70%);
    }
    .drum-pad .label {
      font-weight: 600;
      font-size: 12px;
      text-align: center;
      z-index: 1;
    }

    /* Mini Piano */
    .piano-section {
      padding: 20px;
      background: #0f1621;
      border-top: 1px solid #1b232f;
    }
    .piano-container {
      display: flex;
      justify-content: center;
      position: relative;
      height: 120px;
      background: #0c131b;
      border-radius: 8px;
      border: 1px solid #1d2a3a;
      padding: 10px;
    }
    .piano-key {
      position: relative;
      border: 1px solid #263243;
      cursor: pointer;
      transition: all 0.1s ease;
    }
    .white-key {
      width: 40px;
      height: 100px;
      background: linear-gradient(to bottom, #f0f0f0, #e0e0e0);
      border-radius: 0 0 4px 4px;
      z-index: 1;
    }
    .white-key.active {
      background: linear-gradient(to bottom, #5cf1ff, #4ad1e0);
      box-shadow: inset 0 0 10px rgba(92, 241, 255, 0.5);
    }
    .black-key {
      width: 24px;
      height: 65px;
      background: linear-gradient(to bottom, #333, #000);
      border-radius: 0 0 3px 3px;
      margin-left: -12px;
      margin-right: -12px;
      z-index: 2;
    }
    .black-key.active {
      background: linear-gradient(to bottom, #ff5cf1, #d14ad1);
      box-shadow: inset 0 0 10px rgba(255, 92, 241, 0.5);
    }

    /* Volume Controls */
    .volume-controls {
      display: flex;
      align-items: center;
      gap: 8px;
    }
    .volume-slider {
      width: 80px;
    }
    .mute-btn {
      padding: 6px 8px;
      font-size: 12px;
    }

    .footer {
      display: flex;
      justify-content: space-between;
      padding: 14px 18px;
      font-size: 12px;
      color: var(--muted);
      border-top: 1px solid #1b232f;
      background: #0e151e;
    }
    .hidden { display: none; }
  </style>
</head>
<body>
  <div class="bpm-visualizer" id="bpmVisualizer"></div>
  
  <div class="app">
    <header>
      <div class="title">Mythic <span>Beat Generator</span></div>
      <div class="controls">
        <button id="btnStart">Start</button>
        <button id="btnStop" disabled>Stop</button>
        <label>
          BPM
          <input id="bpm" type="number" min="40" max="220" value="100" style="width:72px" />
        </label>
        <label>
          Swing
          <input id="swing" type="range" min="0" max="0.6" step="0.01" value="0" style="width:120px" />
        </label>
        <label>
          Steps
          <select id="steps">
            <option value="16" selected>16</option>
            <option value="32">32</option>
          </select>
        </label>
        
        <!-- Volume Controls -->
        <div class="volume-controls">
          <label>Vol</label>
          <input id="volume" type="range" min="0" max="100" value="80" class="volume-slider" />
          <button id="btnMute" class="mute-btn">Mute</button>
        </div>
        
        <button id="btnClear">Clear</button>
        <button id="btnSave">Save pattern</button>
        <button id="btnLoad">Load pattern</button>
        <button id="btnRecord">Record</button>
        <a id="downloadLink" class="hidden" download="mythic-beat.webm">Download</a>
      </div>
    </header>

    <div class="grid-wrap">
      <div class="tracks" id="tracks"></div>
      <div class="sequence" id="sequence"></div>
      <div class="playhead" id="playhead"></div>
    </div>

    <!-- DJ Crossfader -->
    <div class="crossfader-section">
      <div class="crossfader-container">
        <div class="crossfader" id="crossfader">
          <div class="crossfader-handle" id="crossfaderHandle"></div>
        </div>
        <div class="pattern-labels">
          <span>Pattern A</span>
          <span>Pattern B</span>
        </div>
      </div>
    </div>

    <!-- Graphic Mixer -->
    <div class="mixer-section">
      <div class="mixer-title">GRAPHIC MIXER</div>
      <div class="mixer-controls">
        <div class="eq-knob">
          <div class="eq-label">BASS</div>
          <div class="knob-container">
            <div class="knob" id="bassKnob">
              <div class="knob-indicator" id="bassIndicator"></div>
            </div>
          </div>
          <div class="knob-value" id="bassValue">0 dB</div>
        </div>
        <div class="eq-knob">
          <div class="eq-label">MID</div>
          <div class="knob-container">
            <div class="knob" id="midKnob">
              <div class="knob-indicator" id="midIndicator"></div>
            </div>
          </div>
          <div class="knob-value" id="midValue">0 dB</div>
        </div>
        <div class="eq-knob">
          <div class="eq-label">TREBLE</div>
          <div class="knob-container">
            <div class="knob" id="trebleKnob">
              <div class="knob-indicator" id="trebleIndicator"></div>
            </div>
          </div>
          <div class="knob-value" id="trebleValue">0 dB</div>
        </div>
      </div>
    </div>

    <!-- Drum Simulator -->
    <div class="drum-simulator" id="drumSimulator"></div>

    <div class="meters">
      <canvas id="wave"></canvas>
      <canvas id="spectrum"></canvas>
    </div>

    <!-- Sample Loader -->
    <div class="sample-loader">
      <div class="sample-controls" id="sampleControls"></div>
    </div>

    <!-- Preset Packs -->
    <div class="preset-section">
      <div class="preset-grid" id="presetGrid"></div>
    </div>

    <!-- Mini Piano -->
    <div class="piano-section">
      <div class="piano-container" id="piano"></div>
    </div>

    <div class="footer">
      <div>Mythic Beat Generator v2.0 • Graphic Mixer • Sample Loader • DJ Crossfader</div>
      <div>BPM-Synced Visuals • Preset Packs • Made for Neon Shrines</div>
    </div>
  </div>

  <script>
    // Instruments (synthetic drum voices)
    const INSTRS = [
      { name: 'Kick',  color: '#1fd1a1' },
      { name: 'Snare', color: '#ff5c8a' },
      { name: 'Hi‑Hat',color: '#5cf1ff' },
      { name: 'Clap',  color: '#f5d15c' },
      { name: 'Tom',   color: '#a1ff5c' },
      { name: 'Ride',  color: '#c65cff' },
    ];

    // Piano notes frequencies (C4 to B4)
    const PIANO_NOTES = [
      { note: 'C', freq: 261.63, type: 'white' },
      { note: 'C#', freq: 277.18, type: 'black' },
      { note: 'D', freq: 293.66, type: 'white' },
      { note: 'D#', freq: 311.13, type: 'black' },
      { note: 'E', freq: 329.63, type: 'white' },
      { note: 'F', freq: 349.23, type: 'white' },
      { note: 'F#', freq: 369.99, type: 'black' },
      { note: 'G', freq: 392.00, type: 'white' },
      { note: 'G#', freq: 415.30, type: 'black' },
      { note: 'A', freq: 440.00, type: 'white' },
      { note: 'A#', freq: 466.16, type: 'black' },
      { note: 'B', freq: 493.88, type: 'white' }
    ];

    // Preset Packs
    const PRESET_PACKS = {
      'Mythic Swamp': {
        kick: { freq: 80, decay: 0.3 },
        snare: { noise: 0.8, freq: 200 },
        hihat: { freq: 8000, decay: 0.1 },
        clap: { delay: 0.02 },
        tom: { freq: 180, decay: 0.25 },
        ride: { freq: 6000, decay: 0.4 }
      },
      'Neon Skies': {
        kick: { freq: 120, decay: 0.2 },
        snare: { noise: 0.9, freq: 400 },
        hihat: { freq: 10000, decay: 0.05 },
        clap: { delay: 0.01 },
        tom: { freq: 260, decay: 0.2 },
        ride: { freq: 8000, decay: 0.3 }
      },
      'Glitch Purge': {
        kick: { freq: 60, decay: 0.4 },
        snare: { noise: 1.0, freq: 150 },
        hihat: { freq: 6000, decay: 0.15 },
        clap: { delay: 0.03 },
        tom: { freq: 140, decay: 0.3 },
        ride: { freq: 4000, decay: 0.5 }
      },
      'Cyber Temple': {
        kick: { freq: 100, decay: 0.25 },
        snare: { noise: 0.7, freq: 300 },
        hihat: { freq: 9000, decay: 0.08 },
        clap: { delay: 0.015 },
        tom: { freq: 220, decay: 0.22 },
        ride: { freq: 7000, decay: 0.35 }
      }
    };

    let ctx, master, analyser, analyser2;
    let playing = false;
    let stepIndex = 0;
    let scheduleAhead = 0.12;
    let nextNoteTime = 0;
    let steps = 16;
    let patternA = INSTRS.map(() => Array(steps).fill(false));
    let patternB = INSTRS.map(() => Array(steps).fill(false));
    let activePattern = patternA;
    let crossfader = 0.5;
    let bpm = 100;
    let swing = 0;
    let timerId = null;
    let mediaRecorder = null;
    let chunks = [];
    let masterVolume = 0.8;
    let isMuted = false;
    let bassGain = 0, midGain = 0, trebleGain = 0;
    let bassFilter, midFilter, trebleFilter;
    let currentPreset = 'Mythic Swamp';
    let loadedSamples = {};

    // DOM elements
    const tracksEl = document.getElementById('tracks');
    const sequenceEl = document.getElementById('sequence');
    const playheadEl = document.getElementById('playhead');
    const drumSimulatorEl = document.getElementById('drumSimulator');
    const pianoEl = document.getElementById('piano');
    const bpmVisualizerEl = document.getElementById('bpmVisualizer');
    const sampleControlsEl = document.getElementById('sampleControls');
    const presetGridEl = document.getElementById('presetGrid');
    const crossfaderEl = document.getElementById('crossfader');
    const crossfaderHandleEl = document.getElementById('crossfaderHandle');
    
    // EQ knobs
    const bassKnob = document.getElementById('bassKnob');
    const midKnob = document.getElementById('midKnob');
    const trebleKnob = document.getElementById('trebleKnob');
    const bassIndicator = document.getElementById('bassIndicator');
    const midIndicator = document.getElementById('midIndicator');
    const trebleIndicator = document.getElementById('trebleIndicator');
    const bassValue = document.getElementById('bassValue');
    const midValue = document.getElementById('midValue');
    const trebleValue = document.getElementById('trebleValue');

    // Control elements
    const bpmEl = document.getElementById('bpm');
    const swingEl = document.getElementById('swing');
    const stepsEl = document.getElementById('steps');
    const volumeEl = document.getElementById('volume');
    const btnMute = document.getElementById('btnMute');
    const btnStart = document.getElementById('btnStart');
    const btnStop = document.getElementById('btnStop');
    const btnClear = document.getElementById('btnClear');
    const btnSave = document.getElementById('btnSave');
    const btnLoad = document.getElementById('btnLoad');
    const btnRecord = document.getElementById('btnRecord');
    const downloadLink = document.getElementById('downloadLink');

    // Build UI
    function buildUI() {
      tracksEl.innerHTML = '';
      sequenceEl.innerHTML = '';
      INSTRS.forEach((instr, r) => {
        const rowLabel = document.createElement('div');
        rowLabel.className = 'track-label';
        const dot = document.createElement('span');
        dot.className = 'color';
        dot.style.background = instr.color;
        const name = document.createElement('span');
        name.textContent = instr.name;
        rowLabel.append(dot, name);
        tracksEl.appendChild(rowLabel);

        for (let c = 0; c < steps; c++) {
          const cell = document.createElement('div');
          cell.className = 'cell';
          if (activePattern[r][c]) cell.classList.add('on');
          cell.dataset.row = r;
          cell.dataset.col = c;
          cell.addEventListener('click', () => {
            const rr = Number(cell.dataset.row);
            const cc = Number(cell.dataset.col);
            activePattern[rr][cc] = !activePattern[rr][cc];
            cell.classList.toggle('on', activePattern[rr][cc]);
          });
          sequenceEl.appendChild(cell);
        }
      });
      playheadEl.style.left = '0px';
    }

    // Build Drum Simulator
    function buildDrumSimulator() {
      drumSimulatorEl.innerHTML = '';
      INSTRS.forEach((instr, index) => {
        const drumPad = document.createElement('div');
        drumPad.className = 'drum-pad';
        drumPad.dataset.instr = index;
        drumPad.style.borderColor = instr.color;
        
        const label = document.createElement('div');
        label.className = 'label';
        label.innerHTML = `${instr.name}<br><small>Click to test</small>`;
        label.style.color = instr.color;
        
        drumPad.appendChild(label);
        drumPad.addEventListener('click', () => {
          drumPad.classList.add('active');
          setTimeout(() => drumPad.classList.remove('active'), 100);
          
          if (ctx) {
            const time = ctx.currentTime;
            switch(instr.name) {
              case 'Kick': voiceKick(time); break;
              case 'Snare': voiceSnare(time); break;
              case 'Hi‑Hat': voiceHiHat(time, false); break;
              case 'Clap': voiceClap(time); break;
              case 'Tom': voiceTom(time); break;
              case 'Ride': voiceRide(time); break;
            }
          }
        });
        
        drumSimulatorEl.appendChild(drumPad);
      });
    }

    // Build Piano
    function buildPiano() {
      pianoEl.innerHTML = '';
      PIANO_NOTES.forEach((note, index) => {
        const key = document.createElement('div');
        key.className = `piano-key ${note.type}-key`;
        key.dataset.note = index;
        
        if (note.type === 'white') {
          const label = document.createElement('div');
          label.style.position = 'absolute';
          label.style.bottom = '5px';
          label.style.width = '100%';
          label.style.textAlign = 'center';
          label.style.color = '#333';
          label.style.fontSize = '10px';
          label.textContent = note.note;
          key.appendChild(label);
        }
        
        key.addEventListener('mousedown', () => playPianoNote(note, key));
        key.addEventListener('mouseup', () => key.classList.remove('active'));
        key.addEventListener('mouseleave', () => key.classList.remove('active'));
        
        pianoEl.appendChild(key);
      });
    }

    // Build Sample Loader
    function buildSampleLoader() {
      sampleControlsEl.innerHTML = '';
      INSTRS.forEach((instr, index) => {
        const trackControl = document.createElement('div');
        trackControl.className = 'sample-track';
        
        const label = document.createElement('span');
        label.textContent = instr.name;
        label.style.color = instr.color;
        label.style.fontSize = '12px';
        label.style.minWidth = '50px';
        
        const fileInput = document.createElement('input');
        fileInput.type = 'file';
        fileInput.accept = 'audio/*';
        fileInput.style.display = 'none';
        fileInput.addEventListener('change', (e) => loadSample(instr.name, e.target.files[0]));
        
        const loadBtn = document.createElement('button');
        loadBtn.textContent = 'Load Sample';
        loadBtn.style.fontSize = '11px';
        loadBtn.addEventListener('click', () => fileInput.click());
        
        trackControl.append(label, fileInput, loadBtn);
        sampleControlsEl.appendChild(trackControl);
      });
    }

    // Build Preset Packs
    function buildPresetPacks() {
      presetGridEl.innerHTML = '';
      Object.keys(PRESET_PACKS).forEach(presetName => {
        const presetBtn = document.createElement('div');
        presetBtn.className = `preset-btn ${presetName === currentPreset ? 'active' : ''}`;
        presetBtn.textContent = presetName;
        presetBtn.addEventListener('click', () => loadPreset(presetName));
        presetGridEl.appendChild(presetBtn);
      });
    }

    // Load preset
    function loadPreset(presetName) {
      currentPreset = presetName;
      buildPresetPacks();
      // In a real implementation, you would apply the preset parameters to the voices
    }

    // Load sample
    function loadSample(instrName, file) {
      if (!file) return;
      
      const reader = new FileReader();
      reader.onload = function(e) {
        if (!ctx) initAudio();
        ctx.decodeAudioData(e.target.result, (buffer) => {
          loadedSamples[instrName] = buffer;
        });
      };
      reader.readAsArrayBuffer(file);
    }

    // Play piano note
    function playPianoNote(note, key) {
      key.classList.add('active');
      if (!ctx) initAudio();
      
      const oscillator = ctx.createOscillator();
      const gainNode = ctx.createGain();
      
      oscillator.type = 'sine';
      oscillator.frequency.value = note.freq;
      
      gainNode.gain.setValueAtTime(0.3, ctx.currentTime);
      gainNode.gain.exponentialRampToValueAtTime(0.001, ctx.currentTime + 1);
      
      oscillator.connect(gainNode);
      gainNode.connect(master);
      
      oscillator.start(ctx.currentTime);
      oscillator.stop(ctx.currentTime + 1);
    }

    function initAudio() {
      if (ctx) return;
      ctx = new (window.AudioContext || window.webkitAudioContext)({ latencyHint: 'interactive' });
      
      // Create EQ filters
      bassFilter = ctx.createBiquadFilter();
      bassFilter.type = 'lowshelf';
      bassFilter.frequency.value = 250;
      
      midFilter = ctx.createBiquadFilter();
      midFilter.type = 'peaking';
      midFilter.frequency.value = 1000;
      midFilter.Q.value = 1;
      
      trebleFilter = ctx.createBiquadFilter();
      trebleFilter.type = 'highshelf';
      trebleFilter.frequency.value = 4000;
      
      master = ctx.createGain();
      master.gain.value = masterVolume;
      
      analyser = ctx.createAnalyser();
      analyser.fftSize = 2048;
      analyser2 = ctx.createAnalyser();
      analyser2.fftSize = 1024;
      
      // Connect: master -> bass -> mid -> treble -> analyser/destination
      master.connect(bassFilter);
      bassFilter.connect(midFilter);
      midFilter.connect(trebleFilter);
      trebleFilter.connect(analyser);
      trebleFilter.connect(analyser2);
      trebleFilter.connect(ctx.destination);
      
      updateEQ();
    }

    // Update EQ
    function updateEQ() {
      if (bassFilter) bassFilter.gain.value = bassGain;
      if (midFilter) midFilter.gain.value = midGain;
      if (trebleFilter) trebleFilter.gain.value = trebleGain;
    }

    // Update volume
    function updateVolume() {
      if (master) {
        master.gain.value = isMuted ? 0 : masterVolume * (volumeEl.value / 100);
      }
    }

    // Schedulers
    function secondsPerStep() {
      return 60 / bpm / 4;
    }

    function schedule() {
      while (nextNoteTime < ctx.currentTime + scheduleAhead) {
        playStep(stepIndex, nextNoteTime);
        const isEven = (stepIndex % 2 === 0);
        const swingOffset = isEven ? 0 : swing * secondsPerStep() * 0.5;
        nextNoteTime += secondsPerStep() + swingOffset;
        stepIndex = (stepIndex + 1) % steps;
      }
      timerId = requestAnimationFrame(schedule);
    }

    function start() {
      initAudio();
      if (playing) return;
      playing = true;
      btnStart.disabled = true;
      btnStop.disabled = false;
      nextNoteTime = ctx.currentTime + 0.05;
      stepIndex = 0;
      schedule();
    }

    function stop() {
      playing = false;
      btnStart.disabled = false;
      btnStop.disabled = true;
      cancelAnimationFrame(timerId);
    }

    // Voices with preset parameters
    function voiceKick(time) {
      highlightDrumPad(0);
      const preset = PRESET_PACKS[currentPreset].kick;
      
      if (loadedSamples['Kick']) {
        const source = ctx.createBufferSource();
        source.buffer = loadedSamples['Kick'];
        source.connect(master);
        source.start(time);
        return;
      }
      
      const o = ctx.createOscillator();
      const g = ctx.createGain();
      o.type = 'sine';
      o.frequency.setValueAtTime(preset.freq, time);
      o.frequency.exponentialRampToValueAtTime(40, time + 0.12);
      g.gain.setValueAtTime(1.1, time);
      g.gain.exponentialRampToValueAtTime(0.0001, time + preset.decay);
      o.connect(g).connect(master);
      o.start(time);
      o.stop(time + 0.25);
    }

    function voiceSnare(time) {
      highlightDrumPad(1);
      const preset = PRESET_PACKS[currentPreset].snare;
      
      if (loadedSamples['Snare']) {
        const source = ctx.createBufferSource();
        source.buffer = loadedSamples['Snare'];
        source.connect(master);
        source.start(time);
        return;
      }
      
      const bufferSize = 2 * ctx.sampleRate;
      const noiseBuffer = ctx.createBuffer(1, bufferSize, ctx.sampleRate);
      const data = noiseBuffer.getChannelData(0);
      for (let i = 0; i < bufferSize; i++) data[i] = Math.random() * 2 - 1;
      const n = ctx.createBufferSource();
      n.buffer = noiseBuffer;
      const hp = ctx.createBiquadFilter();
      hp.type = 'highpass';
      hp.frequency.value = preset.freq;
      const g = ctx.createGain();
      g.gain.setValueAtTime(preset.noise, time);
      g.gain.exponentialRampToValueAtTime(0.0001, time + 0.15);
      n.connect(hp).connect(g).connect(master);
      n.start(time);
      n.stop(time + 0.2);
    }

    function voiceHiHat(time, open=false) {
      highlightDrumPad(2);
      const preset = PRESET_PACKS[currentPreset].hihat;
      
      if (loadedSamples['Hi‑Hat']) {
        const source = ctx.createBufferSource();
        source.buffer = loadedSamples['Hi‑Hat'];
        source.connect(master);
        source.start(time);
        return;
      }
      
      const bufferSize = 2 * ctx.sampleRate;
      const buffer = ctx.createBuffer(1, bufferSize, ctx.sampleRate);
      const data = buffer.getChannelData(0);
      for (let i = 0; i < bufferSize; i++) data[i] = (Math.random() * 2 - 1) * 0.7;
      const src = ctx.createBufferSource();
      src.buffer = buffer;
      const bp = ctx.createBiquadFilter();
      bp.type = 'bandpass';
      bp.frequency.value = preset.freq;
      const g = ctx.createGain();
      g.gain.setValueAtTime(open ? 0.35 : 0.25, time);
      g.gain.exponentialRampToValueAtTime(0.0001, time + preset.decay);
      src.connect(bp).connect(g).connect(master);
      src.start(time);
      src.stop(time + (open ? 0.28 : 0.1));
    }

    function voiceClap(time) {
      highlightDrumPad(3);
      const preset = PRESET_PACKS[currentPreset].clap;
      
      if (loadedSamples['Clap']) {
        const source = ctx.createBufferSource();
        source.buffer = loadedSamples['Clap'];
        source.connect(master);
        source.start(time);
        return;
      }
      
      const hits = [0, preset.delay, preset.delay * 2];
      hits.forEach(h => {
        const bufferSize = 1 * ctx.sampleRate;
        const buffer = ctx.createBuffer(1, bufferSize, ctx.sampleRate);
        const data = buffer.getChannelData(0);
        for (let i = 0; i < bufferSize; i++) data[i] = Math.random() * 2 - 1;
        const src = ctx.createBufferSource();
        src.buffer = buffer;
        const bp = ctx.createBiquadFilter();
        bp.type = 'bandpass';
        bp.frequency.value = 1800;
        const g = ctx.createGain();
        const t = time + h;
        g.gain.setValueAtTime(0.6, t);
        g.gain.exponentialRampToValueAtTime(0.0001, t + 0.12);
        src.connect(bp).connect(g).connect(master);
        src.start(t);
        src.stop(t + 0.14);
      });
    }

    function voiceTom(time) {
      highlightDrumPad(4);
      const preset = PRESET_PACKS[currentPreset].tom;
      
      if (loadedSamples['Tom']) {
        const source = ctx.createBufferSource();
        source.buffer = loadedSamples['Tom'];
        source.connect(master);
        source.start(time);
        return;
      }
      
      const o = ctx.createOscillator();
      const g = ctx.createGain();
      o.type = 'sine';
      o.frequency.setValueAtTime(preset.freq, time);
      o.frequency.exponentialRampToValueAtTime(120, time + 0.18);
      g.gain.setValueAtTime(0.7, time);
      g.gain.exponentialRampToValueAtTime(0.0001, time + preset.decay);
      o.connect(g).connect(master);
      o.start(time);
      o.stop(time + 0.24);
    }

    function voiceRide(time) {
      highlightDrumPad(5);
      const preset = PRESET_PACKS[currentPreset].ride;
      
      if (loadedSamples['Ride']) {
        const source = ctx.createBufferSource();
        source.buffer = loadedSamples['Ride'];
        source.connect(master);
        source.start(time);
        return;
      }
      
      const bufferSize = 2 * ctx.sampleRate;
      const buffer = ctx.createBuffer(1, bufferSize, ctx.sampleRate);
      const data = buffer.getChannelData(0);
      for (let i = 0; i < bufferSize; i++) data[i] = (Math.random() * 2 - 1) * 0.5;
      const src = ctx.createBufferSource();
      src.buffer = buffer;
      const hp = ctx.createBiquadFilter();
      hp.type = 'highpass'; hp.frequency.value = 5000;
      const g = ctx.createGain();
      g.gain.setValueAtTime(0.25, time);
      g.gain.exponentialRampToValueAtTime(0.0001, time + preset.decay);
      src.connect(hp).connect(g).connect(master);
      src.start(time);
      src.stop(time + 0.4);
    }

    // Highlight drum pad when played
    function highlightDrumPad(index) {
      const drumPad = drumSimulatorEl.children[index];
      if (drumPad) {
        drumPad.classList.add('active');
        setTimeout(() => drumPad.classList.remove('active'), 100);
      }
    }

    function playStep(index, time) {
      highlightColumn(index);
      createBPMVisual();

      // Mix patterns based on crossfader position
      const patternAGain = 1 - crossfader;
      const patternBGain = crossfader;

      for (let r = 0; r < INSTRS.length; r++) {
        if (patternA[r][index]) {
          const gainNode = ctx.createGain();
          gainNode.gain.value = patternAGain;
          // Temporarily connect through gain for pattern mixing
          const originalConnect = master.connect;
          master.connect = (dest) => gainNode.connect(dest);
          
          switch (INSTRS[r].name) {
            case 'Kick':  voiceKick(time); break;
            case 'Snare': voiceSnare(time); break;
            case 'Hi‑Hat':voiceHiHat(time, false); break;
            case 'Clap':  voiceClap(time); break;
            case 'Tom':   voiceTom(time); break;
            case 'Ride':  voiceRide(time); break;
          }
          
          master.connect = originalConnect;
        }

        if (patternB[r][index]) {
          const gainNode = ctx.createGain();
          gainNode.gain.value = patternBGain;
          const originalConnect = master.connect;
          master.connect = (dest) => gainNode.connect(dest);
          
          switch (INSTRS[r].name) {
            case 'Kick':  voiceKick(time); break;
            case 'Snare': voiceSnare(time); break;
            case 'Hi‑Hat':voiceHiHat(time, false); break;
            case 'Clap':  voiceClap(time); break;
            case 'Tom':   voiceTom(time); break;
            case 'Ride':  voiceRide(time); break;
          }
          
          master.connect = originalConnect;
        }
      }
    }

    function highlightColumn(index) {
      const colWidth = sequenceEl.clientWidth / steps;
      playheadEl.style.left = `${index * colWidth}px`;
    }

    // BPM Visualizer
    function createBPMVisual() {
      if (!playing) return;
      
      const circle = document.createElement('div');
      circle.className = 'pulse-circle';
      circle.style.width = '50px';
      circle.style.height = '50px';
      circle.style.background = `radial-gradient(circle, ${getRandomNeonColor()}, transparent)`;
      circle.style.left = `${Math.random() * 100}%`;
      circle.style.top = `${Math.random() * 100}%`;
      
      bpmVisualizerEl.appendChild(circle);
      
      // Animate with BPM sync
      const pulseDuration = (60 / bpm) * 1000;
      
      circle.animate([
        { transform: 'scale(0)', opacity: 1 },
        { transform: 'scale(2)', opacity: 0 }
      ], {
        duration: pulseDuration,
        easing: 'ease-out'
      });
      
      setTimeout(() => {
        if (circle.parentNode) {
          circle.parentNode.removeChild(circle);
        }
      }, pulseDuration);
    }

    function getRandomNeonColor() {
      const colors = ['#5cf1ff', '#ff5cf1', '#a1ff5c', '#f5d15c', '#c65cff'];
      return colors[Math.floor(Math.random() * colors.length)];
    }

    // EQ Knob Controls
    function setupKnobControls() {
      setupKnob(bassKnob, bassIndicator, bassValue, 'bass', -20, 20);
      setupKnob(midKnob, midIndicator, midValue, 'mid', -20, 20);
      setupKnob(trebleKnob, trebleIndicator, trebleValue, 'treble', -20, 20);
    }

    function setupKnob(knob, indicator, valueDisplay, type, min, max) {
      let isDragging = false;
      let startY = 0;
      let currentValue = 0;

      knob.addEventListener('mousedown', (e) => {
        isDragging = true;
        startY = e.clientY;
        knob.style.cursor = 'grabbing';
      });

      document.addEventListener('mousemove', (e) => {
        if (!isDragging) return;
        
        const deltaY = startY - e.clientY;
        currentValue = Math.max(min, Math.min(max, currentValue + deltaY * 0.5));
        startY = e.clientY;
        
        updateKnob(indicator, valueDisplay, currentValue, type);
      });

      document.addEventListener('mouseup', () => {
        isDragging = false;
        knob.style.cursor = 'grab';
      });

      updateKnob(indicator, valueDisplay, currentValue, type);
    }

    function updateKnob(indicator, valueDisplay, value, type) {
      const angle = (value + 20) * 4.5 - 135; // -135 to +135 degrees
      indicator.style.transform = `translateX(-50%) rotate(${angle}deg)`;
      valueDisplay.textContent = `${value > 0 ? '+' : ''}${value.toFixed(1)} dB`;
      
      // Update EQ gains
      if (type === 'bass') bassGain = value;
      if (type === 'mid') midGain = value;
      if (type === 'treble') trebleGain = value;
      updateEQ();
      
      // Visual feedback
      indicator.style.background = value !== 0 ? 
        (value > 0 ? '#1fd1a1' : '#ff5c8a') : '#5cf1ff';
    }

    // Crossfader Control
    function setupCrossfader() {
      let isDragging = false;

      crossfaderEl.addEventListener('mousedown', (e) => {
        isDragging = true;
        updateCrossfader(e);
      });

      document.addEventListener('mousemove', (e) => {
        if (!isDragging) return;
        updateCrossfader(e);
      });

      document.addEventListener('mouseup', () => {
        isDragging = false;
      });

      function updateCrossfader(e) {
        const rect = crossfaderEl.getBoundingClientRect();
        let x = e.clientX - rect.left;
        x = Math.max(0, Math.min(rect.width, x));
        crossfader = x / rect.width;
        crossfaderHandleEl.style.left = `${x}px`;
        
        // Update pattern mixing
        if (crossfader < 0.5) {
          activePattern = patternA;
        } else {
          activePattern = patternB;
        }
        buildUI();
      }
    }

    // Visualizers
    const waveCanvas = document.getElementById('wave');
    const specCanvas = document.getElementById('spectrum');
    const wCtx = waveCanvas.getContext('2d');
    const sCtx = specCanvas.getContext('2d');

    function drawVisuals() {
      if (!analyser || !analyser2) return requestAnimationFrame(drawVisuals);
      
      // Waveform
      const bufferLength = analyser.fftSize;
      const dataArray = new Uint8Array(bufferLength);
      analyser.getByteTimeDomainData(dataArray);
      wCtx.clearRect(0, 0, waveCanvas.width, waveCanvas.height);
      wCtx.strokeStyle = '#5cf1ff';
      wCtx.lineWidth = 2;
      wCtx.beginPath();
      const slice = waveCanvas.width / bufferLength;
      for (let i = 0; i < bufferLength; i++) {
        const v = dataArray[i] / 128.0;
        const y = (v * waveCanvas.height) / 2;
        const x = i * slice;
        i ? wCtx.lineTo(x, y) : wCtx.moveTo(x, y);
      }
      wCtx.stroke();

      // Spectrum
      const len = analyser2.frequencyBinCount;
      const freq = new Uint8Array(len);
      analyser2.getByteFrequencyData(freq);
      sCtx.clearRect(0, 0, specCanvas.width, specCanvas.height);
      const barWidth = specCanvas.width / len;
      for (let i = 0; i < len; i++) {
        const val = freq[i];
        const h = (val / 255) * specCanvas.height;
        sCtx.fillStyle = `hsl(${180 + i / 2}, 80%, ${35 + val/8}%)`;
        sCtx.fillRect(i * barWidth, specCanvas.height - h, barWidth, h);
      }
      requestAnimationFrame(drawVisuals);
    }
    requestAnimationFrame(drawVisuals);

    // Event Listeners
    bpmEl.addEventListener('input', () => bpm = Number(bpmEl.value));
    swingEl.addEventListener('input', () => swing = Number(swingEl.value));
    volumeEl.addEventListener('input', updateVolume);
    btnMute.addEventListener('click', () => {
      isMuted = !isMuted;
      btnMute.textContent = isMuted ? 'Unmute' : 'Mute';
      updateVolume();
    });
    stepsEl.addEventListener('change', () => {
      const newSteps = Number(stepsEl.value);
      const newPatternA = INSTRS.map(() => Array(newSteps).fill(false));
      const newPatternB = INSTRS.map(() => Array(newSteps).fill(false));
      
      for (let r = 0; r < INSTRS.length; r++) {
        for (let c = 0; c < Math.min(steps, newSteps); c++) {
          newPatternA[r][c] = patternA[r][c];
          newPatternB[r][c] = patternB[r][c];
        }
      }
      steps = newSteps;
      patternA = newPatternA;
      patternB = newPatternB;
      activePattern = crossfader < 0.5 ? patternA : patternB;
      buildUI();
    });
    btnStart.addEventListener('click', () => start());
    btnStop.addEventListener('click', () => stop());
    btnClear.addEventListener('click', () => {
      activePattern = activePattern === patternA ? patternA : patternB;
      activePattern.forEach((track, i) => {
        track.fill(false);
      });
      buildUI();
    });
    btnSave.addEventListener('click', () => {
      const data = { 
        bpm, swing, steps, 
        patternA, patternB, crossfader,
        bassGain, midGain, trebleGain,
        currentPreset 
      };
      const blob = new Blob([JSON.stringify(data, null, 2)], { type: 'application/json' });
      const a = document.createElement('a');
      a.href = URL.createObjectURL(blob);
      a.download = 'mythic-beat-project.json';
      a.click();
      URL.revokeObjectURL(a.href);
    });
    btnLoad.addEventListener('click', async () => {
      const input = document.createElement('input');
      input.type = 'file';
      input.accept = 'application/json';
      input.onchange = async () => {
        const file = input.files[0];
        if (!file) return;
        const text = await file.text();
        const data = JSON.parse(text);
        bpm = Number(data.bpm ?? bpm);
        swing = Number(data.swing ?? swing);
        steps = Number(data.steps ?? steps);
        patternA = data.patternA ?? patternA;
        patternB = data.patternB ?? patternB;
        crossfader = data.crossfader ?? crossfader;
        bassGain = data.bassGain ?? bassGain;
        midGain = data.midGain ?? midGain;
        trebleGain = data.trebleGain ?? trebleGain;
        currentPreset = data.currentPreset ?? currentPreset;
        
        bpmEl.value = bpm;
        swingEl.value = swing;
        stepsEl.value = steps;
        updateKnob(bassIndicator, bassValue, bassGain, 'bass');
        updateKnob(midIndicator, midValue, midGain, 'mid');
        updateKnob(trebleIndicator, trebleValue, trebleGain, 'treble');
        buildPresetPacks();
        buildUI();
      };
      input.click();
    });

    btnRecord.addEventListener('click', async () => {
      initAudio();
      if (!mediaRecorder || mediaRecorder.state === 'inactive') {
        const dest = ctx.createMediaStreamDestination();
        trebleFilter.disconnect();
        trebleFilter.connect(dest);
        trebleFilter.connect(analyser);
        trebleFilter.connect(analyser2);
        trebleFilter.connect(ctx.destination);
        
        mediaRecorder = new MediaRecorder(dest.stream);
        chunks = [];
        mediaRecorder.ondataavailable = e => { if (e.data.size > 0) chunks.push(e.data); };
        mediaRecorder.onstop = () => {
          const blob = new Blob(chunks, { type: 'audio/webm' });
          downloadLink.href = URL.createObjectURL(blob);
          downloadLink.classList.remove('hidden');
          downloadLink.click();
        };
        mediaRecorder.start();
        btnRecord.textContent = 'Stop recording';
        if (!playing) start();
      } else {
        mediaRecorder.stop();
        btnRecord.textContent = 'Record';
      }
    });

    // Initialize
    patternA = INSTRS.map((_, r) => {
      const arr = Array(steps).fill(false);
      if (INSTRS[r].name === 'Kick')  { arr[0] = true; arr[4] = true; arr[8] = true; arr[12] = true; }
      if (INSTRS[r].name === 'Snare') { arr[4] = true; arr[12] = true; }
      if (INSTRS[r].name === 'Hi‑Hat'){ for (let i=0;i<steps;i++) if (i%2===0) arr[i] = true; }
      return arr;
    });
    
    patternB = INSTRS.map(() => Array(steps).fill(false));
    activePattern = patternA;

    buildUI();
    buildDrumSimulator();
    buildPiano();
    buildSampleLoader();
    buildPresetPacks();
    setupKnobControls();
    setupCrossfader();

    // Resize canvases
    function fitCanvas(c) {
      const rect = c.getBoundingClientRect();
      c.width = Math.floor(rect.width * devicePixelRatio);
      c.height = Math.floor(rect.height * devicePixelRatio);
    }
    window.addEventListener('resize', () => { fitCanvas(waveCanvas); fitCanvas(specCanvas); });
    fitCanvas(waveCanvas); fitCanvas(specCanvas);
  </script>
</body>
</html>

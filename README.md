<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>本物AI顔認識 宿題集中タイマー</title>
  <!-- 🎉 クラッカー紙吹雪 -->
  <script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1.9.3/dist/confetti.browser.min.js"></script>
  <!-- 📈 リアルタイムグラフ (Chart.js) -->
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <!-- 🤖 MediaPipe FaceMesh & Camera Utils (リアルタイムAI顔認識) -->
  <script src="https://cdn.jsdelivr.net/npm/@mediapipe/camera_utils/camera_utils.js" crossorigin="anonymous"></script>
  <script src="https://cdn.jsdelivr.net/npm/@mediapipe/drawing_utils/drawing_utils.js" crossorigin="anonymous"></script>
  <script src="https://cdn.jsdelivr.net/npm/@mediapipe/face_mesh/face_mesh.js" crossorigin="anonymous"></script>

  <style>
    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
    }
    body {
      font-family: 'Helvetica Neue', Arial, sans-serif;
      background: #0f172a;
      color: #f8fafc;
      display: flex;
      justify-content: center;
      align-items: center;
      min-height: 100vh;
      padding: 20px;
    }
    .timer-card {
      background: #1e293b;
      border-radius: 24px;
      padding: 28px 24px;
      width: 100%;
      max-width: 480px;
      text-align: center;
      box-shadow: 0 20px 40px rgba(0, 0, 0, 0.5);
      border: 2px solid #334155;
    }
    h1 {
      font-size: 1.3rem;
      margin-bottom: 12px;
      color: #93c5fd;
      letter-spacing: 0.05em;
    }
    /* 顔ランドマーク描画領域（実映像は非表示） */
    .face-container {
      position: relative;
      width: 100%;
      height: 220px;
      background: #090d16;
      border-radius: 16px;
      overflow: hidden;
      border: 1px solid #334155;
      margin-bottom: 12px;
      display: flex;
      justify-content: center;
      align-items: center;
    }
    #faceCanvas {
      width: 100%;
      height: 100%;
      object-fit: contain;
    }
    #webcamVideo {
      display: none; /* ビデオ映像自体は絶対に非表示 */
    }
    .status-badge {
      position: absolute;
      bottom: 10px;
      left: 50%;
      transform: translateX(-50%);
      padding: 4px 14px;
      border-radius: 12px;
      font-size: 0.85rem;
      font-weight: bold;
      background: rgba(15, 23, 42, 0.85);
      border: 1px solid #475569;
      color: #38bdf8;
      backdrop-filter: blur(4px);
      white-space: nowrap;
    }

    .display {
      font-size: 4.5rem;
      font-weight: 800;
      font-family: 'Courier New', Courier, monospace;
      margin: 4px 0;
      color: #4ade80;
      transition: color 0.3s ease;
    }
    .interval-badge {
      display: inline-block;
      padding: 4px 14px;
      border-radius: 16px;
      background: #334155;
      font-size: 0.85rem;
      font-weight: 600;
      margin-bottom: 16px;
      color: #cbd5e1;
    }

    /* グラフ描画エリア */
    .chart-container {
      width: 100%;
      height: 110px;
      margin-bottom: 16px;
      background: #0f172a;
      padding: 8px;
      border-radius: 12px;
      border: 1px solid #1e293b;
    }

    .presets {
      display: flex;
      justify-content: center;
      gap: 6px;
      margin-bottom: 12px;
      flex-wrap: wrap;
    }
    .preset-btn {
      background: #334155;
      color: #cbd5e1;
      border: none;
      padding: 6px 12px;
      border-radius: 8px;
      cursor: pointer;
      font-weight: bold;
      font-size: 0.9rem;
      transition: all 0.2s;
    }
    .preset-btn:hover, .preset-btn.active {
      background: #3b82f6;
      color: #ffffff;
    }
    .time-setter {
      display: flex;
      align-items: center;
      justify-content: center;
      gap: 6px;
      margin-bottom: 18px;
      background: #0f172a;
      padding: 8px;
      border-radius: 12px;
      border: 1px solid #334155;
    }
    .time-setter label {
      font-size: 0.85rem;
      color: #94a3b8;
      font-weight: bold;
    }
    .custom-input {
      width: 55px;
      padding: 6px;
      border-radius: 6px;
      border: 1px solid #475569;
      background: #1e293b;
      color: #ffffff;
      text-align: center;
      font-size: 1rem;
      font-weight: bold;
    }
    .controls {
      display: flex;
      gap: 10px;
      justify-content: center;
    }
    .btn {
      flex: 1;
      padding: 12px;
      border: none;
      border-radius: 12px;
      font-size: 1.05rem;
      font-weight: bold;
      cursor: pointer;
      transition: transform 0.1s, opacity 0.2s;
    }
    .btn:active {
      transform: scale(0.96);
    }
    .btn-start { background: #22c55e; color: #052e16; }
    .btn-pause { background: #eab308; color: #422006; }
    .btn-reset { background: #ef4444; color: #450a0a; }

    /* 緊急度カラー演出 */
    .urgency-medium .display { color: #facc15; }
    .urgency-high .display { color: #f97316; }
    .urgency-critical .display {
      color: #ef4444;
      animation: pulse 0.5s infinite alternate;
    }
    @keyframes pulse {
      from { opacity: 1; }
      to { opacity: 0.3; }
    }
  </style>
</head>
<body>

<div class="timer-card" id="card">
  <h1>🧠 本物AI集中力測定タイマー</h1>

  <!-- 顔ランドマーク描画領域（映像は非表示） -->
  <div class="face-container">
    <canvas id="faceCanvas" width="640" height="480"></canvas>
    <video id="webcamVideo" playsinline></video>
    <div class="status-badge" id="statusBadge">カメラ未有効化（スタートで起動）</div>
  </div>

  <div class="display" id="display">10:00</div>
  <div class="interval-badge" id="intervalBadge">通知間隔: 1分おき</div>

  <!-- 集中力リアルタイムグラフ -->
  <div class="chart-container">
    <canvas id="chartCanvas"></canvas>
  </div>

  <!-- プリセットボタン -->
  <div class="presets">
    <button class="preset-btn" onclick="setPreset(1, 0)">1分</button>
    <button class="preset-btn" onclick="setPreset(3, 0)">3分</button>
    <button class="preset-btn" onclick="setPreset(5, 0)">5分</button>
    <button class="preset-btn active" onclick="setPreset(10, 0)">10分</button>
  </div>

  <!-- 秒単位の自由時間設定 -->
  <div class="time-setter">
    <input type="number" id="customMin" class="custom-input" value="10" min="0" max="180" oninput="setCustomTime()">
    <label for="customMin">分</label>
    <input type="number" id="customSec" class="custom-input" value="0" min="0" max="59" oninput="setCustomTime()">
    <label for="customSec">秒</label>
  </div>

  <div class="controls">
    <button class="btn btn-start" id="startBtn" onclick="toggleTimer()">スタート</button>
    <button class="btn btn-reset" onclick="resetTimer()">リセット</button>
  </div>
</div>

<script>
  let totalSeconds = 600;
  let remainingSeconds = 600;
  let timerId = null;
  let audioCtx = null;

  // AI & カメラ関連
  let faceMesh = null;
  let camera = null;
  let latestConcentrationScore = 100;
  let smoothedScore = 100;
  let scoreHistory = [];
  let chart = null;

  const videoElement = document.getElementById('webcamVideo');
  const faceCanvas = document.getElementById('faceCanvas');
  const faceCtx = faceCanvas.getContext('2d');
  const statusBadge = document.getElementById('statusBadge');

  const display = document.getElementById('display');
  const intervalBadge = document.getElementById('intervalBadge');
  const startBtn = document.getElementById('startBtn');
  const card = document.getElementById('card');
  const customMinInput = document.getElementById('customMin');
  const customSecInput = document.getElementById('customSec');

  // 音響設定: 学校のスクールタイマー風「ピピッ」音
  function playSchoolPipipi() {
    if (!audioCtx) {
      audioCtx = new (window.AudioContext || window.webkitAudioContext)();
    }
    if (audioCtx.state === 'suspended') {
      audioCtx.resume();
    }

    const now = audioCtx.currentTime;
    const freq = remainingSeconds <= 5 ? 3000 : 2500;
    const beepTimes = [0, 0.08];

    beepTimes.forEach(delay => {
      const osc = audioCtx.createOscillator();
      const gain = audioCtx.createGain();

      osc.type = 'sine';
      osc.frequency.setValueAtTime(freq, now + delay);

      gain.gain.setValueAtTime(0.3, now + delay);
      gain.gain.exponentialRampToValueAtTime(0.001, now + delay + 0.05);

      osc.connect(gain);
      gain.connect(audioCtx.destination);

      osc.start(now + delay);
      osc.stop(now + delay + 0.05);
    });
  }

  // 終了音（ピロリーンメロディ）
  function playFinishSound() {
    if (!audioCtx) return;
    const now = audioCtx.currentTime;
    const notes = [523.25, 659.25, 783.99, 1046.50];
    notes.forEach((freq, idx) => {
      const osc = audioCtx.createOscillator();
      const gain = audioCtx.createGain();
      osc.type = 'triangle';
      osc.frequency.setValueAtTime(freq, now + idx * 0.12);
      gain.gain.setValueAtTime(0.3, now + idx * 0.12);
      gain.gain.exponentialRampToValueAtTime(0.001, now + idx * 0.12 + 0.3);
      osc.connect(gain);
      gain.connect(audioCtx.destination);
      osc.start(now + idx * 0.12);
      osc.stop(now + idx * 0.12 + 0.3);
    });
  }

  // 🎉 紙吹雪
  function fireConfetti() {
    if (typeof confetti === 'function') {
      const count = 200;
      const defaults = { origin: { y: 0.7 } };
      function fire(particleRatio, opts) {
        confetti({ ...defaults, ...opts, particleCount: Math.floor(count * particleRatio) });
      }
      fire(0.25, { spread: 26, startVelocity: 55 });
      fire(0.2, { spread: 60 });
      fire(0.35, { spread: 100, decay: 0.91, scalar: 0.8 });
      fire(0.1, { spread: 120, startVelocity: 25, decay: 0.92, scalar: 1.2 });
      fire(0.1, { spread: 120, startVelocity: 45 });
    }
  }

  // 📈 Chart.js グラフの初期化 (ヌルヌルアニメーション設定)
  function initChart() {
    const ctx = document.getElementById('chartCanvas').getContext('2d');
    chart = new Chart(ctx, {
      type: 'line',
      data: {
        labels: [],
        datasets: [{
          label: '集中度',
          data: scoreHistory,
          borderColor: '#4ade80',
          backgroundColor: 'rgba(74, 222, 128, 0.1)',
          borderWidth: 2,
          pointRadius: 0,
          fill: true,
          tension: 0.4 // ヌルヌルとした滑らかな曲線を表現
        }]
      },
      options: {
        responsive: true,
        maintainAspectRatio: false,
        animation: {
          duration: 400,
          easing: 'easeOutQuad'
        },
        scales: {
          x: { display: false },
          y: {
            min: 0,
            max: 100,
            ticks: { color: '#64748b', font: { size: 10 } },
            grid: { color: '#334155' }
          }
        },
        plugins: { legend: { display: false } }
      }
    });
  }

  function addChartData(score) {
    scoreHistory.push(Math.round(score));
    if (scoreHistory.length > 40) scoreHistory.shift();

    chart.data.labels = Array.from({ length: scoreHistory.length }, (_, i) => i);
    chart.data.datasets[0].data = scoreHistory;

    // スコアに応じて線の色を動的変化 (緑/黄/赤)
    if (score >= 70) chart.data.datasets[0].borderColor = '#4ade80';
    else if (score >= 40) chart.data.datasets[0].borderColor = '#facc15';
    else chart.data.datasets[0].borderColor = '#ef4444';

    chart.update();
  }

  // 🤖 MediaPipe AI顔認識 & 本物の集中力計算
  function initFaceMesh() {
    if (faceMesh) return;

    faceMesh = new FaceMesh({
      locateFile: (file) => `https://cdn.jsdelivr.net/npm/@mediapipe/face_mesh/${file}`
    });

    faceMesh.setOptions({
      maxNumFaces: 1,
      refineLandmarks: true,
      minDetectionConfidence: 0.5,
      minTrackingConfidence: 0.5
    });

    faceMesh.onResults(onFaceResults);
  }

  function calculateRealConcentration(lm) {
    // 1. 顔の正面度 (左右の頬[234, 454]に対する鼻[1]の位置)
    const leftCheek = lm[234].x;
    const rightCheek = lm[454].x;
    const nose = lm[1].x;
    const faceWidth = Math.abs(rightCheek - leftCheek);

    let yawScore = 1.0;
    if (faceWidth > 0.01) {
      const noseRel = (nose - leftCheek) / faceWidth;
      const offset = Math.abs(noseRel - 0.5); // 0なら完全正面
      yawScore = Math.max(0, 1 - offset * 3.5);
    }

    // 2. 上下の向き (額[10], 顎[152]に対する鼻[1]の位置)
    const forehead = lm[10].y;
    const chin = lm[152].y;
    const noseY = lm[1].y;
    const faceHeight = Math.abs(chin - forehead);

    let pitchScore = 1.0;
    if (faceHeight > 0.01) {
      const noseRelY = (noseY - forehead) / faceHeight;
      const offsetY = Math.abs(noseRelY - 0.55);
      pitchScore = Math.max(0, 1 - offsetY * 3.5);
    }

    // 3. 目の開き具合 (EAR: Eye Aspect Ratio)
    const leftVert = Math.hypot(lm[159].x - lm[145].x, lm[159].y - lm[145].y);
    const leftHoriz = Math.hypot(lm[33].x - lm[133].x, lm[33].y - lm[133].y);
    const earLeft = leftVert / (leftHoriz + 0.0001);

    const rightVert = Math.hypot(lm[386].x - lm[374].x, lm[386].y - lm[374].y);
    const rightHoriz = Math.hypot(lm[362].x - lm[263].x, lm[362].y - lm[263].y);
    const earRight = rightVert / (rightHoriz + 0.0001);

    const avgEar = (earLeft + earRight) / 2;
    let eyeScore = 1.0;
    if (avgEar < 0.12) {
      eyeScore = 0.2; // まばたき/目を閉じている
    } else if (avgEar < 0.18) {
      eyeScore = 0.6;
    }

    // 総合判定 (0~100%)
    const rawScore = (yawScore * 0.45 + pitchScore * 0.35 + eyeScore * 0.20) * 100;

    // 滑らかな値の補正 (指数移動平均)
    smoothedScore = smoothedScore * 0.75 + rawScore * 0.25;
    return Math.min(100, Math.max(0, smoothedScore));
  }

  function onFaceResults(results) {
    faceCtx.save();
    faceCtx.clearRect(0, 0, faceCanvas.width, faceCanvas.height);

    if (results.multiFaceLandmarks && results.multiFaceLandmarks.length > 0) {
      const landmarks = results.multiFaceLandmarks[0];

      // リアルタイム集中力スコア算出
      const score = calculateRealConcentration(landmarks);
      latestConcentrationScore = score;

      // 映像は描画せず、顔の点・メッシュのみを描画
      drawFaceMeshOverlay(landmarks, score);

      const rounded = Math.round(score);
      let statusText = "超集中！";
      if (rounded < 40) statusText = "脇見・集中力低";
      else if (rounded < 70) statusText = "やや不調";

      statusBadge.textContent = `🎯 集中度: ${rounded}% (${statusText})`;
      statusBadge.style.color = rounded >= 70 ? '#4ade80' : rounded >= 40 ? '#facc15' : '#f87171';
    } else {
      latestConcentrationScore = 0;
      statusBadge.textContent = `⚠️ 顔が認識されていません`;
      statusBadge.style.color = '#ef4444';
    }
    faceCtx.restore();
  }

  // 顔ランドマーク（点・メッシュ）の描画（背景映像なし）
  function drawFaceMeshOverlay(landmarks, score) {
    const w = faceCanvas.width;
    const h = faceCanvas.height;

    // 集中度に応じたネオンカラー
    const hue = Math.max(0, Math.min(120, (score / 100) * 120));
    const mainColor = `hsl(${hue}, 90%, 55%)`;

    // コネクター描画 (標準描画関数がある場合)
    if (typeof FACEMESH_TESSELATION !== 'undefined' && typeof drawConnectors === 'function') {
      drawConnectors(faceCtx, landmarks, FACEMESH_TESSELATION, { color: `hsla(${hue}, 80%, 50%, 0.18)`, lineWidth: 0.8 });
      if (typeof FACEMESH_RIGHT_EYE !== 'undefined') drawConnectors(faceCtx, landmarks, FACEMESH_RIGHT_EYE, { color: mainColor, lineWidth: 1.5 });
      if (typeof FACEMESH_LEFT_EYE !== 'undefined') drawConnectors(faceCtx, landmarks, FACEMESH_LEFT_EYE, { color: mainColor, lineWidth: 1.5 });
      if (typeof FACEMESH_LIPS !== 'undefined') drawConnectors(faceCtx, landmarks, FACEMESH_LIPS, { color: '#f43f5e', lineWidth: 1.2 });
      if (typeof FACEMESH_FACE_OVAL !== 'undefined') drawConnectors(faceCtx, landmarks, FACEMESH_FACE_OVAL, { color: mainColor, lineWidth: 1.5 });
    }

    // ドット描画 (鏡像反転して自然な見た目に)
    faceCtx.fillStyle = mainColor;
    for (let i = 0; i < landmarks.length; i += 3) {
      const pt = landmarks[i];
      const x = (1 - pt.x) * w;
      const y = pt.y * h;
      faceCtx.beginPath();
      faceCtx.arc(x, y, 1.2, 0, 2 * Math.PI);
      faceCtx.fill();
    }
  }

  // カメラの初期化とブラウザの権限要求
  async function startCamera() {
    initFaceMesh();
    if (!camera) {
      statusBadge.textContent = "📷 カメラアクセスの許可を求めています...";
      camera = new Camera(videoElement, {
        onFrame: async () => {
          if (videoElement) {
            await faceMesh.send({ image: videoElement });
          }
        },
        width: 640,
        height: 480
      });
    }
    try {
      await camera.start();
    } catch (err) {
      console.error("Camera access error:", err);
      statusBadge.textContent = "❌ カメラのアクセスが拒否されました";
      statusBadge.style.color = "#ef4444";
    }
  }

  function stopCamera() {
    if (camera) {
      camera.stop();
      camera = null;
    }
    faceCtx.clearRect(0, 0, faceCanvas.width, faceCanvas.height);
    statusBadge.textContent = "カメラ停止中";
    statusBadge.style.color = "#94a3b8";
  }

  // 通知間隔ルール
  function getBeepInterval(rem) {
    if (rem > 120) return 60;
    if (rem > 60)  return 30;
    if (rem > 30)  return 15;
    if (rem > 15)  return 10;
    if (rem > 5)   return 5;
    return 1;
  }

  function updateDisplay() {
    const m = Math.floor(remainingSeconds / 60);
    const s = remainingSeconds % 60;
    display.textContent = `${String(m).padStart(2, '0')}:${String(s).padStart(2, '0')}`;

    const interval = getBeepInterval(remainingSeconds);
    if (remainingSeconds > 0) {
      intervalBadge.textContent = `通知間隔: ${interval}秒おき`;
    } else {
      intervalBadge.textContent = `🎉 タイムアップ！お疲れ様！ 🎉`;
    }

    card.classList.remove('urgency-medium', 'urgency-high', 'urgency-critical');
    if (remainingSeconds <= 5) {
      card.classList.add('urgency-critical');
    } else if (remainingSeconds <= 30) {
      card.classList.add('urgency-high');
    } else if (remainingSeconds <= 120) {
      card.classList.add('urgency-medium');
    }
  }

  function tick() {
    if (remainingSeconds <= 0) {
      clearInterval(timerId);
      timerId = null;
      startBtn.textContent = 'スタート';
      startBtn.className = 'btn btn-start';
      playFinishSound();
      fireConfetti();
      updateDisplay();
      stopCamera();
      return;
    }

    remainingSeconds--;
    updateDisplay();

    // 集中力データのグラフ更新
    addChartData(latestConcentrationScore);

    // 学校風「ピピッ」音
    const interval = getBeepInterval(remainingSeconds);
    if (remainingSeconds > 0 && remainingSeconds % interval === 0) {
      playSchoolPipipi();
    }
  }

  function toggleTimer() {
    if (!audioCtx) {
      audioCtx = new (window.AudioContext || window.webkitAudioContext)();
    }

    if (timerId) {
      clearInterval(timerId);
      timerId = null;
      startBtn.textContent = '再開';
      startBtn.className = 'btn btn-start';
      stopCamera();
    } else {
      if (remainingSeconds <= 0) resetTimer();
      timerId = setInterval(tick, 1000);
      startBtn.textContent = '一時停止';
      startBtn.className = 'btn btn-pause';
      playSchoolPipipi();
      startCamera(); // ブラウザ標準のカメラ許可を要求＆起動
    }
  }

  function resetTimer() {
    if (timerId) {
      clearInterval(timerId);
      timerId = null;
    }
    remainingSeconds = totalSeconds;
    startBtn.textContent = 'スタート';
    startBtn.className = 'btn btn-start';
    scoreHistory = [];
    chart.data.labels = [];
    chart.data.datasets[0].data = [];
    chart.update();
    updateDisplay();
    stopCamera();
  }

  function setPreset(min, sec) {
    document.querySelectorAll('.preset-btn').forEach(btn => btn.classList.remove('active'));
    if (event && event.target) event.target.classList.add('active');
    customMinInput.value = min;
    customSecInput.value = sec;
    totalSeconds = min * 60 + sec;
    resetTimer();
  }

  function setCustomTime() {
    document.querySelectorAll('.preset-btn').forEach(btn => btn.classList.remove('active'));
    const min = parseInt(customMinInput.value) || 0;
    const sec = parseInt(customSecInput.value) || 0;
    totalSeconds = min * 60 + sec;
    resetTimer();
  }

  window.addEventListener('DOMContentLoaded', () => {
    initChart();
    updateDisplay();
  });
</script>

</body>
</html>

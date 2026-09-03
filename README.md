<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>集中力分析タイマー</title>
  <!-- 🎉 紙吹雪ライブラリ -->
  <script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1.9.3/dist/confetti.browser.min.js"></script>
  <!-- 📈 グラフ描画ライブラリ (Chart.js) -->
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <!-- 👤 リアルタイム顔認識ライブラリ (MediaPipe FaceMesh) -->
  <script src="https://cdn.jsdelivr.net/npm/@mediapipe/camera_utils/camera_utils.js" crossorigin="anonymous"></script>
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
      padding: 32px 24px;
      width: 100%;
      max-width: 500px;
      text-align: center;
      box-shadow: 0 20px 40px rgba(0, 0, 0, 0.5);
      border: 2px solid #334155;
      transition: all 0.3s ease;
    }
    h1 {
      font-size: 1.3rem;
      margin-bottom: 16px;
      color: #93c5fd;
      letter-spacing: 0.05em;
    }
    
    /* カメラ ＆ ランドマーク表示エリア */
    .face-landmarkers-container {
      position: relative;
      width: 100%;
      height: 220px;
      margin-bottom: 16px;
      background: #020617;
      border-radius: 16px;
      overflow: hidden;
      border: 2px solid #334155;
      display: flex;
      justify-content: center;
      align-items: center;
    }
    #video {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      object-fit: cover;
      transform: scaleX(-1); /* 鏡合わせ表示 */
    }
    #faceCanvas {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      transform: scaleX(-1); /* 鏡合わせ表示 */
      pointer-events: none;
    }
    .camera-status {
      position: absolute;
      top: 10px;
      left: 10px;
      background: rgba(15, 23, 42, 0.85);
      padding: 4px 12px;
      border-radius: 12px;
      font-size: 0.8rem;
      font-weight: bold;
      color: #38bdf8;
      border: 1px solid #0284c7;
      z-index: 10;
    }

    .display {
      font-size: 4.5rem;
      font-weight: 800;
      font-family: 'Courier New', Courier, monospace;
      margin: 8px 0;
      color: #4ade80;
      transition: color 0.3s ease;
    }
    .interval-badge {
      display: inline-block;
      padding: 6px 16px;
      border-radius: 20px;
      background: #334155;
      font-size: 0.9rem;
      font-weight: 600;
      margin-bottom: 16px;
      color: #f1f5f9;
    }

    .chart-container {
      position: relative;
      width: 100%;
      height: 100px;
      margin-bottom: 16px;
    }

    .presets {
      display: flex;
      justify-content: center;
      gap: 8px;
      margin-bottom: 16px;
      flex-wrap: wrap;
    }
    .preset-btn {
      background: #334155;
      color: #cbd5e1;
      border: none;
      padding: 8px 14px;
      border-radius: 10px;
      cursor: pointer;
      font-weight: bold;
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
      gap: 8px;
      margin-bottom: 20px;
      background: #0f172a;
      padding: 10px;
      border-radius: 14px;
      border: 1px solid #334155;
    }
    .time-setter label {
      font-size: 0.9rem;
      color: #94a3b8;
      font-weight: bold;
    }
    .custom-input {
      width: 60px;
      padding: 6px;
      border-radius: 8px;
      border: 1px solid #475569;
      background: #1e293b;
      color: #ffffff;
      text-align: center;
      font-size: 1rem;
      font-weight: bold;
    }
    .controls {
      display: flex;
      gap: 12px;
      justify-content: center;
    }
    .btn {
      flex: 1;
      padding: 12px;
      border: none;
      border-radius: 14px;
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

    /* 残り時間に応じた緊張感演出 */
    .urgency-medium .display { color: #facc15; }
    .urgency-high .display { color: #f97316; }
    .urgency-critical .display {
      color: #ef4444;
      animation: pulse 0.5s infinite alternate;
    }
    @keyframes pulse {
      from { opacity: 1; }
      to { opacity: 0.4; }
    }
  </style>
</head>
<body>

<div class="timer-card" id="card">
  <h1>🏫 集中力分析タイマー</h1>
  
  <div class="face-landmarkers-container">
    <div class="camera-status" id="cameraStatus">📷 カメラ準備中...</div>
    <video id="video" playsinline></video>
    <canvas id="faceCanvas"></canvas>
  </div>

  <div class="display" id="display">10:00</div>
  <div class="interval-badge" id="intervalBadge">通知間隔: 1分おき</div>

  <div class="chart-container">
    <canvas id="chartCanvas"></canvas>
  </div>

  <!-- プリセットボタン -->
  <div class="presets">
    <button class="preset-btn" data-min="1" data-sec="0" onclick="handlePresetClick(this)">1分</button>
    <button class="preset-btn" data-min="3" data-sec="0" onclick="handlePresetClick(this)">3分</button>
    <button class="preset-btn" data-min="5" data-sec="0" onclick="handlePresetClick(this)">5分</button>
    <button class="preset-btn active" data-min="10" data-sec="0" onclick="handlePresetClick(this)">10分</button>
  </div>

  <!-- 時間設定 -->
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
  let concentrationData = [];
  let concentrationChart = null;
  let videoStream = null;
  let currentScore = 50; // リアルタイム集中度スコア
  let faceMesh = null;

  const faceCanvas = document.getElementById('faceCanvas');
  const faceCtx = faceCanvas.getContext('2d');
  const video = document.getElementById('video');
  const cameraStatus = document.getElementById('cameraStatus');

  const display = document.getElementById('display');
  const intervalBadge = document.getElementById('intervalBadge');
  const startBtn = document.getElementById('startBtn');
  const card = document.getElementById('card');
  const customMinInput = document.getElementById('customMin');
  const customSecInput = document.getElementById('customSec');

  // 音声フィードバック
  function playSchoolPipipi() {
    if (!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)();
    if (audioCtx.state === 'suspended') audioCtx.resume();

    const now = audioCtx.currentTime;
    const freq = remainingSeconds <= 5 ? 3000 : 2500;
    [0, 0.08].forEach(delay => {
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

  function fireConfetti() {
    if (typeof confetti === 'function') {
      confetti({ particleCount: 150, spread: 80, origin: { y: 0.6 } });
    }
  }

  // 👤 顔認識モデル (MediaPipe FaceMesh) 初期化
  function initFaceMesh() {
    if (typeof FaceMesh === 'undefined') return;
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

  function onFaceResults(results) {
    if (video.videoWidth > 0 && video.videoHeight > 0) {
      if (faceCanvas.width !== video.videoWidth || faceCanvas.height !== video.videoHeight) {
        faceCanvas.width = video.videoWidth;
        faceCanvas.height = video.videoHeight;
      }
    }

    faceCtx.save();
    faceCtx.clearRect(0, 0, faceCanvas.width, faceCanvas.height);

    if (results.multiFaceLandmarks && results.multiFaceLandmarks.length > 0) {
      cameraStatus.textContent = "👤 顔認識中（測定中）";
      cameraStatus.style.borderColor = "#22c55e";
      cameraStatus.style.color = "#4ade80";

      const landmarks = results.multiFaceLandmarks[0];

      // 顔ランドマークの描画
      faceCtx.fillStyle = '#4ade80';
      for (let i = 0; i < landmarks.length; i += 6) {
        const x = landmarks[i].x * faceCanvas.width;
        const y = landmarks[i].y * faceCanvas.height;
        faceCtx.beginPath();
        faceCtx.arc(x, y, 1.8, 0, 2 * Math.PI);
        faceCtx.fill();
      }

      // 顔の位置から集中度を計算（正面に近いほど高スコア）
      const nose = landmarks[1];
      const dist = Math.hypot(nose.x - 0.5, nose.y - 0.5);
      let targetScore = Math.max(20, Math.min(100, Math.floor(100 - dist * 160)));
      currentScore = Math.floor(currentScore * 0.7 + targetScore * 0.3);
    } else {
      cameraStatus.textContent = "⚠️ 顔が検出されません";
      cameraStatus.style.borderColor = "#ef4444";
      cameraStatus.style.color = "#f87171";
      currentScore = Math.max(10, currentScore - 5);
    }
    faceCtx.restore();
  }

  async function startCamera() {
    if (!navigator.mediaDevices || !navigator.mediaDevices.getUserMedia) {
      cameraStatus.textContent = "⚠️ カメラ非対応";
      return;
    }
    try {
      videoStream = await navigator.mediaDevices.getUserMedia({ video: true });
      video.srcObject = videoStream;
      await video.play();

      async function processVideo() {
        if (video.readyState >= 2 && faceMesh) {
          await faceMesh.send({ image: video });
        }
        if (videoStream) requestAnimationFrame(processVideo);
      }
      processVideo();
    } catch (err) {
      console.warn("カメラアクセスエラー:", err);
      cameraStatus.textContent = "⚠️ カメラの使用が拒否されました";
    }
  }

  function stopCamera() {
    if (videoStream) {
      videoStream.getTracks().forEach(track => track.stop());
      video.srcObject = null;
      videoStream = null;
    }
    cameraStatus.textContent = "📷 カメラ停止中";
    cameraStatus.style.borderColor = "#0284c7";
    cameraStatus.style.color = "#38bdf8";
    faceCtx.clearRect(0, 0, faceCanvas.width, faceCanvas.height);
  }

  // 📈 グラフ設定
  function initChart() {
    const ctx = document.getElementById('chartCanvas').getContext('2d');
    concentrationChart = new Chart(ctx, {
      type: 'line',
      data: {
        labels: [],
        datasets: [{
          label: '集中力 (%)',
          data: concentrationData,
          borderColor: '#4ade80',
          borderWidth: 2,
          pointRadius: 0,
          fill: false,
          tension: 0.3
        }]
      },
      options: {
        responsive: true,
        maintainAspectRatio: false,
        animation: { duration: 200 },
        scales: {
          x: { display: false },
          y: { suggestedMin: 0, suggestedMax: 100, ticks: { color: '#94a3b8' }, grid: { color: '#334155' } }
        },
        plugins: { legend: { display: false } }
      }
    });
  }

  function updateChart() {
    concentrationData.push(currentScore);
    if (concentrationData.length > 30) concentrationData.shift();
    concentrationChart.data.labels = Array.from({length: concentrationData.length}, (_, i) => i);
    concentrationChart.update('none');
  }

  function getBeepInterval(rem) {
    if (rem > 120) return 60;
    if (rem > 60) return 30;
    if (rem > 30) return 15;
    if (rem > 15) return 10;
    if (rem > 5) return 5;
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
    if (remainingSeconds <= 5) card.classList.add('urgency-critical');
    else if (remainingSeconds <= 30) card.classList.add('urgency-high');
    else if (remainingSeconds <= 120) card.classList.add('urgency-medium');
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
    updateChart();

    const interval = getBeepInterval(remainingSeconds);
    if (remainingSeconds > 0 && remainingSeconds % interval === 0) {
      playSchoolPipipi();
    }
  }

  function toggleTimer() {
    if (!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)();

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
      startCamera();
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
    concentrationData = [];
    concentrationChart.data.labels = [];
    concentrationChart.update('none');
    updateDisplay();
    stopCamera();
  }

  function handlePresetClick(btn) {
    document.querySelectorAll('.preset-btn').forEach(b => b.classList.remove('active'));
    btn.classList.add('active');
    const min = parseInt(btn.dataset.min);
    const sec = parseInt(btn.dataset.sec);
    customMinInput.value = min;
    customSecInput.value = sec;
    totalSeconds = min * 60 + sec;
    resetTimer();
  }

  function setCustomTime() {
    document.querySelectorAll('.preset-btn').forEach(b => b.classList.remove('active'));
    const min = parseInt(customMinInput.value) || 0;
    const sec = parseInt(customSecInput.value) || 0;
    totalSeconds = min * 60 + sec;
    resetTimer();
  }

  window.addEventListener('load', () => {
    initChart();
    initFaceMesh();
    updateDisplay();
  });
</script>

</body>
</html>

<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>多角解析 宿題集中タイマー</title>
  <!-- 🎉 クラッカー紙吹雪 -->
  <script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1.9.3/dist/confetti.browser.min.js"></script>
  <!-- 📈 リアルタイムグラフ (Chart.js) -->
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <!-- 👤 リアルタイム顔認識 (MediaPipe FaceMesh) -->
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
      padding: 28px 24px;
      width: 100%;
      max-width: 500px;
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
    .face-landmarkers-container {
      position: relative;
      width: 100%;
      height: 220px;
      margin-bottom: 12px;
      background: #020617;
      border-radius: 16px;
      overflow: hidden;
      border: 2px solid #334155;
      display: flex;
      justify-content: center;
      align-items: center;
    }
    #video {
      display: none;
    }
    #faceCanvas {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      transform: scaleX(-1);
    }
    .status-badge {
      position: absolute;
      bottom: 10px;
      left: 50%;
      transform: translateX(-50%);
      padding: 6px 16px;
      border-radius: 16px;
      font-size: 0.85rem;
      font-weight: bold;
      background: rgba(15, 23, 42, 0.85);
      border: 1px solid #0284c7;
      color: #38bdf8;
      backdrop-filter: blur(4px);
      white-space: nowrap;
      z-index: 10;
    }
    .audio-indicator {
      position: absolute;
      top: 10px;
      right: 10px;
      padding: 4px 10px;
      border-radius: 10px;
      font-size: 0.75rem;
      font-weight: bold;
      background: rgba(15, 23, 42, 0.8);
      border: 1px solid #334155;
      color: #94a3b8;
      z-index: 10;
    }
    .audio-indicator.active {
      color: #4ade80;
      border-color: #22c55e;
      background: rgba(22, 101, 52, 0.4);
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
      margin-bottom: 12px;
      color: #cbd5e1;
    }

    .chart-container {
      position: relative;
      width: 100%;
      height: 100px;
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
  <h1>🧠 多角集中力解析タイマー</h1>

  <div class="face-landmarkers-container">
    <div class="audio-indicator" id="audioIndicator">✏️ 筆記音: 未検知</div>
    <div class="status-badge" id="statusBadge">カメラ・マイク未起動（スタートで要求）</div>
    <video id="video" playsinline></video>
    <canvas id="faceCanvas"></canvas>
  </div>

  <div class="display" id="display">10:00</div>
  <div class="interval-badge" id="intervalBadge">通知間隔: 1分おき</div>

  <div class="chart-container">
    <canvas id="chartCanvas"></canvas>
  </div>

  <div class="presets">
    <button class="preset-btn" data-min="1" data-sec="0" onclick="handlePresetClick(this)">1分</button>
    <button class="preset-btn" data-min="3" data-sec="0" onclick="handlePresetClick(this)">3分</button>
    <button class="preset-btn" data-min="5" data-sec="0" onclick="handlePresetClick(this)">5分</button>
    <button class="preset-btn active" data-min="10" data-sec="0" onclick="handlePresetClick(this)">10分</button>
  </div>

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

  // Web Audio API & マイク入力
  let audioCtx = null;
  let micStream = null;
  let analyser = null;
  let isPenSoundDetected = false;

  // 顔認識 & 分析
  let faceMesh = null;
  let videoStream = null;
  let latestConcentrationScore = 100;
  let smoothedScore = 100;
  let scoreHistory = [];
  let chart = null;

  const video = document.getElementById('video');
  const faceCanvas = document.getElementById('faceCanvas');
  const faceCtx = faceCanvas.getContext('2d');
  const statusBadge = document.getElementById('statusBadge');
  const audioIndicator = document.getElementById('audioIndicator');

  const display = document.getElementById('display');
  const intervalBadge = document.getElementById('intervalBadge');
  const startBtn = document.getElementById('startBtn');
  const card = document.getElementById('card');
  const customMinInput = document.getElementById('customMin');
  const customSecInput = document.getElementById('customSec');

  // スクールタイマー風「ピピッ」音
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

  // 📈 Chart.js グラフ初期化
  function initChart() {
    const ctx = document.getElementById('chartCanvas').getContext('2d');
    chart = new Chart(ctx, {
      type: 'line',
      data: {
        labels: [],
        datasets: [{
          data: scoreHistory,
          borderColor: '#4ade80',
          backgroundColor: 'rgba(74, 222, 128, 0.1)',
          borderWidth: 2,
          pointRadius: 0,
          fill: true,
          tension: 0.4
        }]
      },
      options: {
        responsive: true,
        maintainAspectRatio: false,
        animation: { duration: 300 },
        scales: {
          x: { display: false },
          y: { min: 0, max: 100, ticks: { color: '#64748b' }, grid: { color: '#334155' } }
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

    if (score >= 70) chart.data.datasets[0].borderColor = '#4ade80';
    else if (score >= 40) chart.data.datasets[0].borderColor = '#facc15';
    else chart.data.datasets[0].borderColor = '#ef4444';

    chart.update('none');
  }

  // 🎙️ マイク音響解析（シャーペン音・ノック音・高音域クリックノイズ検出）
  async function initMicrophone() {
    if (!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)();
    if (audioCtx.state === 'suspended') await audioCtx.resume();

    try {
      micStream = await navigator.mediaDevices.getUserMedia({ audio: true });
      const source = audioCtx.createMediaStreamSource(micStream);
      analyser = audioCtx.createAnalyser();
      analyser.fftSize = 512;
      source.connect(analyser);

      analyzeAudioLoop();
    } catch (err) {
      console.warn("マイクのアクセスが許可されませんでした", err);
      audioIndicator.textContent = "🎙️ マイク無効";
    }
  }

  function analyzeAudioLoop() {
    if (!analyser || !micStream) return;

    const dataArray = new Uint8Array(analyser.frequencyBinCount);
    analyser.getByteFrequencyData(dataArray);

    // 高音域帯（シャーペンのカチャカチャ音・筆記擦れ音 3kHz〜8kHz 付近）の強度計算
    let highFreqSum = 0;
    const startIndex = Math.floor(dataArray.length * 0.35);
    const endIndex = Math.floor(dataArray.length * 0.85);

    for (let i = startIndex; i < endIndex; i++) {
      highFreqSum += dataArray[i];
    }
    const avgHighFreq = highFreqSum / (endIndex - startIndex);

    // カチャカチャ音が一定以上の高周波スパイクを持った場合
    if (avgHighFreq > 35) {
      isPenSoundDetected = true;
      audioIndicator.textContent = "✏️ 筆記音・作業音検知中";
      audioIndicator.classList.add('active');
    } else {
      isPenSoundDetected = false;
      audioIndicator.textContent = "✏️ 筆記音: 未検知";
      audioIndicator.classList.remove('active');
    }

    if (timerId) requestAnimationFrame(analyzeAudioLoop);
  }

  // 👤 MediaPipe FaceMesh & 姿勢・表情・居眠り判定
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

  function calculateAdvancedConcentration(lm) {
    // 1. 顔のピッチ（上下角度）: 額(10)と顎(152)に対する鼻(1)の相対位置
    const forehead = lm[10].y;
    const chin = lm[152].y;
    const noseY = lm[1].y;
    const faceHeight = Math.abs(chin - forehead) || 0.001;
    const noseRelY = (noseY - forehead) / faceHeight;

    let pitchScore = 0;
    let pitchStatus = "適正";

    // 0.55付近が正面。0.60〜0.72 付近が「適度な下向き（ノート集中）」
    if (noseRelY >= 0.58 && noseRelY <= 0.75) {
      pitchScore = 100; // 適度に下を向いている（最も良い集中状態）
      pitchStatus = "ノート集中（適度な下向き）";
    } else if (noseRelY < 0.52) {
      pitchScore = 30;  // 上を向きすぎ（よそ見）
      pitchStatus = "上向き（よそ見）";
    } else if (noseRelY > 0.82) {
      pitchScore = 20;  // 下を向きすぎ（うつ伏せ・居眠り）
      pitchStatus = "うつ伏せ傾向";
    } else {
      pitchScore = 80;  // 正面付近
      pitchStatus = "正面向き";
    }

    // 2. 左右向き（Yaw）: 左右頬(234, 454)に対する鼻(1)の位置
    const leftCheek = lm[234].x;
    const rightCheek = lm[454].x;
    const noseX = lm[1].x;
    const faceWidth = Math.abs(rightCheek - leftCheek) || 0.001;
    const noseRelX = (noseX - leftCheek) / faceWidth;
    const yawOffset = Math.abs(noseRelX - 0.5);

    let yawScore = Math.max(0, 100 - yawOffset * 300); // 横を向くほどスコア低下

    // 3. 目の開き（EAR: 居眠り検知）
    const leftVert = Math.hypot(lm[159].x - lm[145].x, lm[159].y - lm[145].y);
    const leftHoriz = Math.hypot(lm[33].x - lm[133].x, lm[33].y - lm[133].y);
    const earLeft = leftVert / (leftHoriz + 0.0001);

    const rightVert = Math.hypot(lm[386].x - lm[374].x, lm[386].y - lm[374].y);
    const rightHoriz = Math.hypot(lm[362].x - lm[263].x, lm[362].y - lm[263].y);
    const earRight = rightVert / (rightHoriz + 0.0001);

    const avgEar = (earLeft + earRight) / 2;
    let eyeScore = 100;
    if (avgEar < 0.12) {
      eyeScore = 10; // 目を閉じている（眠気/居眠り）
      pitchStatus = "居眠り・閉眼検知";
    }

    // 4. シャーペン音・筆記音ボーナス補正
    let soundBonus = isPenSoundDetected ? 15 : 0;

    // 総合判定算出
    let baseScore = (pitchScore * 0.45 + yawScore * 0.35 + eyeScore * 0.20) + soundBonus;
    let targetScore = Math.min(100, Math.max(0, baseScore));

    smoothedScore = smoothedScore * 0.7 + targetScore * 0.3;
    return { score: smoothedScore, status: pitchStatus };
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
      const landmarks = results.multiFaceLandmarks[0];

      // 顔認識ポイント（ドットメッシュ）の描画
      faceCtx.fillStyle = '#38bdf8';
      for (let i = 0; i < landmarks.length; i += 5) {
        const x = landmarks[i].x * faceCanvas.width;
        const y = landmarks[i].y * faceCanvas.height;
        faceCtx.beginPath();
        faceCtx.arc(x, y, 1.5, 0, 2 * Math.PI);
        faceCtx.fill();
      }

      // 集中度スコア計算
      const { score, status } = calculateAdvancedConcentration(landmarks);
      latestConcentrationScore = score;

      const rounded = Math.round(score);
      statusBadge.textContent = `🎯 集中度: ${rounded}% (${status})`;
      statusBadge.style.borderColor = rounded >= 70 ? '#22c55e' : rounded >= 40 ? '#facc15' : '#ef4444';
      statusBadge.style.color = rounded >= 70 ? '#4ade80' : rounded >= 40 ? '#facc15' : '#f87171';

    } else {
      // 顔非検出 ＝ 立席 / 画面外
      latestConcentrationScore = 0;
      smoothedScore = 0;
      statusBadge.textContent = `🚶 離席・顔が検出されません (0%)`;
      statusBadge.style.borderColor = '#ef4444';
      statusBadge.style.color = '#f87171';
    }
    faceCtx.restore();
  }

  async function startCamera() {
    initFaceMesh();
    initMicrophone();

    if (!navigator.mediaDevices || !navigator.mediaDevices.getUserMedia) {
      statusBadge.textContent = "⚠️ メディア機能非対応";
      return;
    }
    try {
      videoStream = await navigator.mediaDevices.getUserMedia({ video: true });
      video.srcObject = videoStream;
      await video.play();

      async function processVideo() {
        if (video.readyState >= 2 && faceMesh && videoStream) {
          await faceMesh.send({ image: video });
        }
        if (videoStream) requestAnimationFrame(processVideo);
      }
      processVideo();
    } catch (err) {
      statusBadge.textContent = "❌ カメラへのアクセスが拒否されました";
    }
  }

  function stopCamera() {
    if (videoStream) {
      videoStream.getTracks().forEach(track => track.stop());
      video.srcObject = null;
      videoStream = null;
    }
    if (micStream) {
      micStream.getTracks().forEach(track => track.stop());
      micStream = null;
    }
    statusBadge.textContent = "カメラ・マイク停止中";
    statusBadge.style.borderColor = "#0284c7";
    statusBadge.style.color = "#38bdf8";
    faceCtx.clearRect(0, 0, faceCanvas.width, faceCanvas.height);
  }

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

    // グラフ更新
    addChartData(latestConcentrationScore);

    // スクールタイマー音
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
    scoreHistory = [];
    chart.data.labels = [];
    chart.data.datasets[0].data = [];
    chart.update('none');
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

  window.addEventListener('DOMContentLoaded', () => {
    initChart();
    updateDisplay();
  });
</script>

</body>
</html>

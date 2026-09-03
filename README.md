<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>集中力分析シミュレーションタイマー</title>
  <!-- 🎉 紙吹雪アニメーション用ライブラリ -->
  <script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1.9.3/dist/confetti.browser.min.js"></script>
  <!-- 📈 グラフ描画用ライブラリ (Chart.js) -->
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
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
      padding: 36px 28px;
      width: 100%;
      max-width: 500px;
      text-align: center;
      box-shadow: 0 20px 40px rgba(0, 0, 0, 0.5);
      border: 2px solid #334155;
      transition: all 0.3s ease;
    }
    h1 {
      font-size: 1.4rem;
      margin-bottom: 16px;
      color: #93c5fd;
      letter-spacing: 0.05em;
    }
    .display {
      font-size: 4.8rem;
      font-weight: 800;
      font-family: 'Courier New', Courier, monospace;
      margin: 10px 0;
      color: #4ade80;
      transition: color 0.3s ease;
    }
    .interval-badge {
      display: inline-block;
      padding: 6px 16px;
      border-radius: 20px;
      background: #334155;
      font-size: 0.95rem;
      font-weight: 600;
      margin-bottom: 24px;
      color: #f1f5f9;
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
      margin-bottom: 24px;
      background: #0f172a;
      padding: 12px;
      border-radius: 14px;
      border: 1px solid #334155;
    }
    .time-setter label {
      font-size: 0.95rem;
      color: #94a3b8;
      font-weight: bold;
    }
    .custom-input {
      width: 65px;
      padding: 8px;
      border-radius: 8px;
      border: 1px solid #475569;
      background: #1e293b;
      color: #ffffff;
      text-align: center;
      font-size: 1.1rem;
      font-weight: bold;
    }
    .controls {
      display: flex;
      gap: 12px;
      justify-content: center;
    }
    .btn {
      flex: 1;
      padding: 14px;
      border: none;
      border-radius: 14px;
      font-size: 1.1rem;
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

    /* カメラ・集中力グラフエリア */
    .face-landmarkers-container {
      position: relative;
      width: 100%;
      height: 200px;
      margin-bottom: 24px;
      background: #0f172a;
      border-radius: 14px;
      overflow: hidden;
      border: 1px solid #334155;
    }
    #faceCanvas {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
    }
    #chartCanvas {
      width: 100%;
      height: 100px;
      margin-bottom: 24px;
    }

    /* 非表示のビデオ要素 */
    #video {
      display: none;
    }

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
  <h1>🏫 集中力分析シミュレーションタイマー</h1>
  
  <div class="face-landmarkers-container">
    <canvas id="faceCanvas"></canvas>
    <video id="video" autoplay playsinline></video>
  </div>

  <div class="display" id="display">10:00</div>
  <div class="interval-badge" id="intervalBadge">通知間隔: 1分おき</div>

  <canvas id="chartCanvas"></canvas>

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
  let concentrationData = [];
  let concentrationChart = null;
  let videoStream = null;
  let faceLandmarks = [];
  const faceCanvas = document.getElementById('faceCanvas');
  const faceCtx = faceCanvas.getContext('2d');
  const video = document.getElementById('video');

  const display = document.getElementById('display');
  const intervalBadge = document.getElementById('intervalBadge');
  const startBtn = document.getElementById('startBtn');
  const card = document.getElementById('card');
  const customMinInput = document.getElementById('customMin');
  const customSecInput = document.getElementById('customSec');

  // 学校のスクールタイマー風「ピピッ」音
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
    const notes = [523.25, 659.25, 783.99, 1046.50]; // C, E, G, High C
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

  // 🎉 クラッカーの紙吹雪アニメーション
  function fireConfetti() {
    if (typeof confetti === 'function') {
      const count = 200;
      const defaults = { origin: { y: 0.7 } };

      function fire(particleRatio, opts) {
        confetti({
          ...defaults,
          ...opts,
          particleCount: Math.floor(count * particleRatio)
        });
      }

      fire(0.25, { spread: 26, startVelocity: 55 });
      fire(0.2, { spread: 60 });
      fire(0.35, { spread: 100, decay: 0.91, scalar: 0.8 });
      fire(0.1, { spread: 120, startVelocity: 25, decay: 0.92, scalar: 1.2 });
      fire(0.1, { spread: 120, startVelocity: 45 });
    }
  }

  // 集中力スコアのシミュレーション（ダミーデータ）
  function generateConcentrationScore() {
    // 0~100の間でランダムに変化させる
    return Math.floor(Math.random() * 61) + 20; // 20~80
  }

  // 表情ランドマークのシミュレーション描画
  function drawFaceLandmarks(score) {
    if (!faceCtx) return;
    faceCtx.clearRect(0, 0, faceCanvas.width, faceCanvas.height);
    if (!faceLandmarks.length) return;

    // 集中力スコアに応じてランドマークの色を変化させる
    const r = Math.floor((100 - score) * 2.55);
    const g = Math.floor(score * 2.55);
    const b = 150;
    const color = `rgb(${r}, ${g}, ${b})`;

    faceCtx.fillStyle = color;
    faceCtx.strokeStyle = color;
    faceCtx.lineWidth = 2;

    const centerX = faceCanvas.width / 2;
    const centerY = faceCanvas.height / 2;
    const scale = 50; // 顔の大きさのスケール

    // 集中力スコアに応じてランドマークの位置を微調整する
    const shift = (score - 50) / 10; 

    // ダミーのランドマーク描画（顔の形、目、眉、口）
    // 集中力スコアに合わせて目の位置や口の形を動かす

    // 顔の輪郭
    faceCtx.beginPath();
    faceCtx.arc(centerX, centerY, scale, 0, 2 * Math.PI);
    faceCtx.stroke();

    // 眉
    faceCtx.beginPath();
    faceCtx.moveTo(centerX - 30 + shift, centerY - 30);
    faceCtx.lineTo(centerX - 10 + shift, centerY - 35 + shift);
    faceCtx.stroke();
    faceCtx.beginPath();
    faceCtx.moveTo(centerX + 30 - shift, centerY - 30);
    faceCtx.lineTo(centerX + 10 - shift, centerY - 35 + shift);
    faceCtx.stroke();

    // 目
    const eyeSize = 5;
    faceCtx.beginPath();
    faceCtx.arc(centerX - 20 + shift, centerY - 15, eyeSize, 0, 2 * Math.PI);
    faceCtx.fill();
    faceCtx.beginPath();
    faceCtx.arc(centerX + 20 - shift, centerY - 15, eyeSize, 0, 2 * Math.PI);
    faceCtx.fill();

    // 口
    const mouthWidth = 20 + shift;
    const mouthHeight = 10 - shift;
    faceCtx.beginPath();
    faceCtx.moveTo(centerX - mouthWidth, centerY + 20);
    faceCtx.quadraticCurveTo(centerX, centerY + 20 + mouthHeight, centerX + mouthWidth, centerY + 20);
    faceCtx.stroke();

  }

  // 集中力グラフの描画（ヌルヌルアニメーション）
  function initConcentrationChart() {
    const ctx = document.getElementById('chartCanvas').getContext('2d');
    concentrationChart = new Chart(ctx, {
      type: 'line',
      data: {
        labels: [],
        datasets: [{
          label: '集中力 (シミュレーション)',
          data: concentrationData,
          borderColor: '#4ade80',
          borderWidth: 2,
          pointRadius: 0,
          fill: false,
          tension: 0.4 // ヌルヌルとした曲線
        }]
      },
      options: {
        responsive: true,
        maintainAspectRatio: false,
        animation: {
          duration: 300, // スムーズなアニメーション
          easing: 'linear'
        },
        scales: {
          x: {
            display: false,
            ticks: {
              maxTicksLimit: 10
            }
          },
          y: {
            suggestedMin: 0,
            suggestedMax: 100,
            ticks: {
              color: '#94a3b8'
            },
            grid: {
              color: '#334155'
            }
          }
        },
        plugins: {
          legend: {
            display: false
          }
        }
      }
    });
  }

  function updateConcentrationData() {
    const score = generateConcentrationScore();
    concentrationData.push(score);
    if (concentrationData.length > 30) {
      concentrationData.shift();
    }
    
    concentrationChart.data.labels = Array.from({length: concentrationData.length}, (_, i) => i);
    concentrationChart.update('none'); // アニメーションなしでデータ更新

    drawFaceLandmarks(score);
  }

  // 残り時間に応じた通知間隔（秒）
  function getBeepInterval(rem) {
    if (rem > 120) return 60; // 2分超: 1分おき
    if (rem > 60)  return 30; // 1分〜2分: 30秒おき
    if (rem > 30)  return 15; // 30秒〜1分: 15秒おき
    if (rem > 15)  return 10; // 15秒〜30秒: 10秒おき
    if (rem > 5)   return 5;  // 5秒〜15秒: 5秒おき
    return 1;                 // 5秒以下: 毎秒
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

    // 警戒レベルの色・演出切り替え
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
      fireConfetti(); // クラッカー発射！
      updateDisplay();
      stopCamera();
      return;
    }

    remainingSeconds--;
    updateDisplay();
    updateConcentrationData();

    // 指定間隔で学校風の「ピピッ」音を鳴らす
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
      playSchoolPipipi(); // スタート合図
      initCamera();
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
    faceCtx.clearRect(0, 0, faceCanvas.width, faceCanvas.height);
    updateDisplay();
    stopCamera();
  }

  function setPreset(min, sec) {
    document.querySelectorAll('.preset-btn').forEach(btn => btn.classList.remove('active'));
    if (event && event.target && event.target.classList) {
      event.target.classList.add('active');
    }
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

  async function initCamera() {
    if (navigator.mediaDevices && navigator.mediaDevices.getUserMedia) {
      try {
        videoStream = await navigator.mediaDevices.getUserMedia({ video: true });
        video.srcObject = videoStream;
        await video.play();

        // ランドマーク（シミュレーション）の設定
        faceCanvas.width = video.videoWidth;
        faceCanvas.height = video.videoHeight;
        faceLandmarks = Array.from({length: 468}, (_, i) => ({x: 0, y: 0}));

        drawFaceLandmarks(50);
      } catch (err) {
        console.error("Camera access denied or not available.", err);
        // カメラが使えない場合の処理（シミュレーションを停止するなど）
      }
    }
  }

  function stopCamera() {
    if (videoStream) {
      const tracks = videoStream.getTracks();
      tracks.forEach(track => track.stop());
      video.srcObject = null;
      videoStream = null;
    }
  }

  window.addEventListener('load', () => {
    inito

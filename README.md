<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>多角解析 勉強集中タイマー & 独り言AIレポート</title>
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
      max-width: 520px;
      text-align: center;
      box-shadow: 0 20px 40px rgba(0, 0, 0, 0.5);
      border: 2px solid #334155;
      position: relative;
    }
    h1 {
      font-size: 1.3rem;
      margin-bottom: 12px;
      color: #93c5fd;
      letter-spacing: 0.05em;
    }

    /* 勉強テーマ入力欄 */
    .topic-container {
      margin-bottom: 12px;
      text-align: left;
    }
    .topic-container label {
      font-size: 0.8rem;
      color: #94a3b8;
      font-weight: bold;
      display: block;
      margin-bottom: 4px;
    }
    .topic-input {
      width: 100%;
      padding: 8px 12px;
      border-radius: 8px;
      border: 1px solid #475569;
      background: #0f172a;
      color: #38bdf8;
      font-size: 0.95rem;
      font-weight: bold;
    }

    .face-landmarkers-container {
      position: relative;
      width: 100%;
      height: 200px;
      margin-bottom: 12px;
      background: #020617;
      border-radius: 16px;
      overflow: hidden;
      border: 2px solid #334155;
      display: flex;
      justify-content: center;
      align-items: center;
    }
    #video { display: none; }
    #faceCanvas {
      position: absolute;
      top: 0; left: 0;
      width: 100%; height: 100%;
      transform: scaleX(-1);
    }
    .status-badge {
      position: absolute;
      bottom: 10px;
      left: 50%;
      transform: translateX(-50%);
      padding: 6px 14px;
      border-radius: 16px;
      font-size: 0.8rem;
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

    /* リアルタイム独り言テロップ */
    .speech-box {
      background: #0f172a;
      border: 1px solid #334155;
      border-radius: 10px;
      padding: 8px;
      font-size: 0.85rem;
      color: #cbd5e1;
      margin-bottom: 12px;
      min-height: 38px;
      display: flex;
      align-items: center;
      justify-content: center;
    }
    .speech-box span {
      color: #38bdf8;
      font-weight: bold;
    }

    .display {
      font-size: 4.2rem;
      font-weight: 800;
      font-family: 'Courier New', Courier, monospace;
      margin: 2px 0;
      color: #4ade80;
      transition: color 0.3s ease;
    }
    .interval-badge {
      display: inline-block;
      padding: 4px 14px;
      border-radius: 16px;
      background: #334155;
      font-size: 0.8rem;
      font-weight: 600;
      margin-bottom: 12px;
      color: #cbd5e1;
    }

    .chart-container {
      position: relative;
      width: 100%;
      height: 90px;
      margin-bottom: 14px;
      background: #0f172a;
      padding: 6px;
      border-radius: 12px;
      border: 1px solid #1e293b;
    }

    .presets {
      display: flex;
      justify-content: center;
      gap: 6px;
      margin-bottom: 12px;
    }
    .preset-btn {
      background: #334155;
      color: #cbd5e1;
      border: none;
      padding: 6px 12px;
      border-radius: 8px;
      cursor: pointer;
      font-weight: bold;
      font-size: 0.85rem;
    }
    .preset-btn.active {
      background: #3b82f6;
      color: #ffffff;
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
      font-size: 1rem;
      font-weight: bold;
      cursor: pointer;
    }
    .btn-start { background: #22c55e; color: #052e16; }
    .btn-pause { background: #eab308; color: #422006; }
    .btn-reset { background: #ef4444; color: #450a0a; }

    /* 📊 終了時要約モーダル */
    .modal-overlay {
      display: none;
      position: fixed;
      top:0; left:0; width:100%; height:100%;
      background: rgba(0,0,0,0.8);
      justify-content: center;
      align-items: center;
      z-index: 100;
      padding: 20px;
    }
    .modal-content {
      background: #1e293b;
      border-radius: 20px;
      padding: 24px;
      max-width: 480px;
      width: 100%;
      border: 2px solid #38bdf8;
      text-align: left;
      max-height: 90vh;
      overflow-y: auto;
    }
    .modal-title {
      font-size: 1.4rem;
      color: #38bdf8;
      text-align: center;
      margin-bottom: 16px;
      font-weight: bold;
    }
    .score-box {
      text-align: center;
      background: #0f172a;
      padding: 16px;
      border-radius: 16px;
      margin-bottom: 16px;
      border: 1px solid #334155;
    }
    .score-number {
      font-size: 3.5rem;
      font-weight: 800;
      color: #4ade80;
    }
    .summary-section {
      margin-bottom: 14px;
    }
    .summary-section h3 {
      font-size: 0.95rem;
      color: #93c5fd;
      margin-bottom: 6px;
      border-bottom: 1px solid #334155;
      padding-bottom: 4px;
    }
    .log-list {
      background: #0f172a;
      border-radius: 8px;
      padding: 10px;
      max-height: 120px;
      overflow-y: auto;
      font-size: 0.85rem;
    }
    .log-item {
      color: #f87171;
      margin-bottom: 6px;
      padding-bottom: 4px;
      border-bottom: 1px dashed #334155;
    }
    .log-item:last-child { border-bottom: none; }
    .close-btn {
      width: 100%;
      padding: 10px;
      background: #3b82f6;
      color: white;
      border: none;
      border-radius: 10px;
      font-weight: bold;
      cursor: pointer;
      margin-top: 10px;
    }
  </style>
</head>
<body>

<div class="timer-card">
  <h1>🧠 独り言AI解析 集中タイマー</h1>

  <div class="topic-container">
    <label for="studyTopic">📝 今日の勉強テーマ（例: 数学の微分積分 / 英単語）</label>
    <input type="text" id="studyTopic" class="topic-input" value="数学の微分積分">
  </div>

  <div class="face-landmarkers-container">
    <div class="audio-indicator" id="audioIndicator">✏️ 筆記音: 未検知</div>
    <div class="status-badge" id="statusBadge">カメラ・マイク未起動</div>
    <video id="video" playsinline></video>
    <canvas id="faceCanvas"></canvas>
  </div>

  <div class="speech-box" id="speechBox">
    🗣️ 独り言検知: <span id="speechText">（待機中…）</span>
  </div>

  <div class="display" id="display">10:00</div>
  <div class="interval-badge" id="intervalBadge">通知間隔: 1分おき</div>

  <div class="chart-container">
    <canvas id="chartCanvas"></canvas>
  </div>

  <div class="presets">
    <button class="preset-btn" onclick="setPreset(1)">1分</button>
    <button class="preset-btn" onclick="setPreset(3)">3分</button>
    <button class="preset-btn" onclick="setPreset(5)">5分</button>
    <button class="preset-btn active" onclick="setPreset(10)">10分</button>
  </div>

  <div class="controls">
    <button class="btn btn-start" id="startBtn" onclick="toggleTimer()">スタート</button>
    <button class="btn btn-reset" onclick="resetTimer()">リセット</button>
  </div>
</div>

<!-- 📊 終了時要約レポート モーダル -->
<div class="modal-overlay" id="summaryModal">
  <div class="modal-content">
    <div class="modal-title">🎓 学習集中度 要約レポート</div>

    <div class="score-box">
      <div style="font-size:0.85rem; color:#94a3b8;">総合集中スコア</div>
      <div class="score-number" id="finalScoreDisplay">88</div>
      <div id="scoreEvalText" style="font-size:0.9rem; color:#38bdf8;">素晴らしい集中力でした！</div>
    </div>

    <div class="summary-section">
      <h3>📈 集中度の内訳</h3>
      <p id="postureSummary" style="font-size:0.85rem; color:#cbd5e1; margin-bottom:4px;"></p>
      <p id="writingSummary" style="font-size:0.85rem; color:#cbd5e1;"></p>
    </div>

    <div class="summary-section">
      <h3>💬 無関係な独り言・脱線ログ</h3>
      <div class="log-list" id="irrelevantLogList">
        <!-- JSで動的挿入 -->
      </div>
    </div>

    <button class="close-btn" onclick="closeModal()">閉じる</button>
  </div>
</div>

<script>
  let totalSeconds = 600;
  let remainingSeconds = 600;
  let timerId = null;

  // Web Audio & マイク
  let audioCtx = null;
  let micStream = null;
  let analyser = null;
  let isPenSoundDetected = false;
  let writingDetectCount = 0;
  let totalTicks = 0;

  // 顔認識 & スコア履歴
  let faceMesh = null;
  let videoStream = null;
  let latestConcentrationScore = 100;
  let smoothedScore = 100;
  let scoreHistory = [];
  let chart = null;

  // 🗣️ 音声認識 & 独り言解析
  let recognition = null;
  let irrelevantSpeechLogs = []; // 無関係な発話ログ [{ time: "08:12", text: "..." }]
  let totalSpeechCount = 0;
  let irrelevantSpeechCount = 0;

  const video = document.getElementById('video');
  const faceCanvas = document.getElementById('faceCanvas');
  const faceCtx = faceCanvas.getContext('2d');
  const statusBadge = document.getElementById('statusBadge');
  const audioIndicator = document.getElementById('audioIndicator');
  const speechText = document.getElementById('speechText');

  const display = document.getElementById('display');
  const intervalBadge = document.getElementById('intervalBadge');
  const startBtn = document.getElementById('startBtn');
  const studyTopicInput = document.getElementById('studyTopic');

  // 無関係な発話を検出するためのNG判定フレーズ（日常会話・脱線ワード）
  const NG_WORDS = ["お腹減った", "お腹すいた", "眠い", "だるい", "ゲーム", "漫画", "マンガ", "遊ぶ", "スマホ", "Youtube", "ねむい", "どこか行く", "帰りたい", "めんどくさい", "疲れ"];

  // 音声認識（SpeechRecognition）の初期化
  function initSpeechRecognition() {
    const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
    if (!SpeechRecognition) {
      speechText.textContent = "※お使いのブラウザは音声認識非対応です";
      return;
    }

    recognition = new SpeechRecognition();
    recognition.lang = 'ja-JP';
    recognition.continuous = true;
    recognition.interimResults = false;

    recognition.onresult = (event) => {
      const lastIndex = event.results.length - 1;
      const transcript = event.results[lastIndex][0].transcript.trim();
      speechText.textContent = `「${transcript}」`;

      analyzeSpeechRelevance(transcript);
    };

    recognition.onerror = (err) => {
      console.warn("音声認識エラー:", err.error);
    };

    recognition.onend = () => {
      // タイマー作動中なら再起動
      if (timerId && recognition) {
        try { recognition.start(); } catch(e){}
      }
    };
  }

  // 独り言の関連性判定（AI / キーワードエンジン）
  function analyzeSpeechRelevance(text) {
    totalSpeechCount++;
    const topic = studyTopicInput.value.trim();

    // 1. NGワード（雑談・愚痴）が含まれているか
    const containsNG = NG_WORDS.some(word => text.includes(word));

    // 2. 勉強テーマのキーワードが含まれているか（簡易マッチ）
    const topicKeywords = topic.split(/[\s　]+/);
    const containsTopic = topicKeywords.some(kw => kw.length > 1 && text.includes(kw));

    // 無関係と判定される条件: NGワードを含む、あるいはテーマと一切無関係な短い単語
    let isIrrelevant = false;
    if (containsNG) {
      isIrrelevant = true;
    } else if (!containsTopic && text.length > 4) {
      // テーマ単語を含まず、かつ勉強用のつぶやき（「なるほど」「OK」「ここか」等）でもない場合
      const studyPhrases = ["なるほど", "わかった", "つまり", "こうして", "次は", "えーと", "答え", "計算", "確認"];
      const isStudyPhrase = studyPhrases.some(p => text.includes(p));
      if (!isStudyPhrase) isIrrelevant = true;
    }

    if (isIrrelevant) {
      irrelevantSpeechCount++;
      const timeStr = display.textContent;
      irrelevantSpeechLogs.push({ time: timeStr, text: text });
      speechText.style.color = "#f87171"; // 赤文字警告
    } else {
      speechText.style.color = "#38bdf8"; // 青文字
    }
  }

  // スクールタイマー音
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

  // 🎙️ マイク音響解析
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
      audioIndicator.textContent = "🎙️ マイク無効";
    }
  }

  function analyzeAudioLoop() {
    if (!analyser || !micStream) return;

    const dataArray = new Uint8Array(analyser.frequencyBinCount);
    analyser.getByteFrequencyData(dataArray);

    let highFreqSum = 0;
    const startIndex = Math.floor(dataArray.length * 0.35);
    const endIndex = Math.floor(dataArray.length * 0.85);

    for (let i = startIndex; i < endIndex; i++) highFreqSum += dataArray[i];
    const avgHighFreq = highFreqSum / (endIndex - startIndex);

    if (avgHighFreq > 35) {
      isPenSoundDetected = true;
      writingDetectCount++;
      audioIndicator.textContent = "✏️ 筆記音検知中";
      audioIndicator.classList.add('active');
    } else {
      isPenSoundDetected = false;
      audioIndicator.textContent = "✏️ 筆記音: 未検知";
      audioIndicator.classList.remove('active');
    }

    if (timerId) requestAnimationFrame(analyzeAudioLoop);
  }

  // 👤 MediaPipe FaceMesh
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

  function calculateConcentration(lm) {
    const forehead = lm[10].y;
    const chin = lm[152].y;
    const noseY = lm[1].y;
    const faceHeight = Math.abs(chin - forehead) || 0.001;
    const noseRelY = (noseY - forehead) / faceHeight;

    let pitchScore = 80;
    let pitchStatus = "正面向き";

    if (noseRelY >= 0.58 && noseRelY <= 0.75) {
      pitchScore = 100;
      pitchStatus = "ノート集中（適度な下向き）";
    } else if (noseRelY < 0.52) {
      pitchScore = 30;
      pitchStatus = "上向き（よそ見）";
    } else if (noseRelY > 0.82) {
      pitchScore = 20;
      pitchStatus = "うつ伏せ傾向";
    }

    const noseX = lm[1].x;
    const faceWidth = Math.abs(lm[454].x - lm[234].x) || 0.001;
    const yawOffset = Math.abs((noseX - lm[234].x) / faceWidth - 0.5);
    let yawScore = Math.max(0, 100 - yawOffset * 300);

    let soundBonus = isPenSoundDetected ? 15 : 0;
    let targetScore = Math.min(100, Math.max(0, (pitchScore * 0.6 + yawScore * 0.4) + soundBonus));

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

      faceCtx.fillStyle = '#38bdf8';
      for (let i = 0; i < landmarks.length; i += 5) {
        const x = landmarks[i].x * faceCanvas.width;
        const y = landmarks[i].y * faceCanvas.height;
        faceCtx.beginPath();
        faceCtx.arc(x, y, 1.5, 0, 2 * Math.PI);
        faceCtx.fill();
      }

      const { score, status } = calculateConcentration(landmarks);
      latestConcentrationScore = score;

      const rounded = Math.round(score);
      statusBadge.textContent = `🎯 集中度: ${rounded}% (${status})`;
      statusBadge.style.borderColor = rounded >= 70 ? '#22c55e' : rounded >= 40 ? '#facc15' : '#ef4444';
      statusBadge.style.color = rounded >= 70 ? '#4ade80' : rounded >= 40 ? '#facc15' : '#f87171';
    } else {
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
      statusBadge.textContent = "❌ カメラアクセス拒否";
    }
  }

  function stopCamera() {
    if (videoStream) {
      videoStream.getTracks().forEach(track => track.stop());
      videoStream = null;
    }
    if (micStream) {
      micStream.getTracks().forEach(track => track.stop());
      micStream = null;
    }
    if (recognition) {
      try { recognition.stop(); } catch(e){}
    }
  }

  function updateDisplay() {
    const m = Math.floor(remainingSeconds / 60);
    const s = remainingSeconds % 60;
    display.textContent = `${String(m).padStart(2, '0')}:${String(s).padStart(2, '0')}`;
  }

  function tick() {
    if (remainingSeconds <= 0) {
      clearInterval(timerId);
      timerId = null;
      startBtn.textContent = 'スタート';
      startBtn.className = 'btn btn-start';
      playFinishSound();
      if (typeof confetti === 'function') confetti({ particleCount: 150, spread: 80 });
      showSummaryReport();
      stopCamera();
      return;
    }

    remainingSeconds--;
    totalTicks++;
    updateDisplay();
    addChartData(latestConcentrationScore);

    const rem = remainingSeconds;
    const interval = rem > 120 ? 60 : rem > 60 ? 30 : rem > 30 ? 15 : rem > 5 ? 5 : 1;
    if (rem > 0 && rem % interval === 0) playSchoolPipipi();
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

      // 音声認識スタート
      if (!recognition) initSpeechRecognition();
      if (recognition) {
        try { recognition.start(); } catch(e){}
      }
    }
  }

  function resetTimer() {
    if (timerId) {
      clearInterval(timerId);
      timerId = null;
    }
    remainingSeconds = totalSeconds;
    totalTicks = 0;
    writingDetectCount = 0;
    irrelevantSpeechLogs = [];
    totalSpeechCount = 0;
    irrelevantSpeechCount = 0;

    startBtn.textContent = 'スタート';
    startBtn.className = 'btn btn-start';
    scoreHistory = [];
    chart.data.labels = [];
    chart.data.datasets[0].data = [];
    chart.update('none');
    speechText.textContent = "（待機中…）";
    updateDisplay();
    stopCamera();
  }

  function setPreset(min) {
    document.querySelectorAll('.preset-btn').forEach(b => b.classList.remove('active'));
    event.target.classList.add('active');
    totalSeconds = min * 60;
    resetTimer();
  }

  // 📊 最終要約レポート＆総合スコアの表示
  function showSummaryReport() {
    const avgScore = scoreHistory.length > 0
      ? Math.round(scoreHistory.reduce((a, b) => a + b, 0) / scoreHistory.length)
      : 80;

    // 独り言による減点 (無関係な発言1回につき-5点)
    const speechPenalty = irrelevantSpeechCount * 5;
    const finalScore = Math.max(0, Math.min(100, avgScore - speechPenalty));

    document.getElementById('finalScoreDisplay').textContent = finalScore;

    const evalElem = document.getElementById('scoreEvalText');
    if (finalScore >= 85) evalElem.textContent = "🎉 素晴らしい集中力でした！大変よくできました。";
    else if (finalScore >= 60) evalElem.textContent = "👍 概ね集中できていました。脱線に気をつけましょう。";
    else evalElem.textContent = "⚠️ 姿勢の乱れや無関係な独り言が多く見られました。次頑張ろう！";

    document.getElementById('postureSummary').textContent = `・姿勢・視線安定度: 平均 ${avgScore} 点`;
    const writingRatio = totalTicks > 0 ? Math.round((writingDetectCount / totalTicks) * 100) : 0;
    document.getElementById('writingSummary').textContent = `・筆記音（作業量）検知割合: 時間の ${writingRatio}%`;

    const logList = document.getElementById('irrelevantLogList');
    logList.innerHTML = '';

    if (irrelevantSpeechLogs.length === 0) {
      logList.innerHTML = '<div style="color:#4ade80; text-align:center;">無関係な独り言はありませんでした！👏</div>';
    } else {
      irrelevantSpeechLogs.forEach(log => {
        const item = document.createElement('div');
        item.className = 'log-item';
        item.textContent = `⏱️ [残り ${log.time}] 「${log.text}」`;
        logList.appendChild(item);
      });
    }

    document.getElementById('summaryModal').style.display = 'flex';
  }

  function closeModal() {
    document.getElementById('summaryModal').style.display = 'none';
  }

  window.addEventListener('DOMContentLoaded', () => {
    initChart();
    updateDisplay();
  });
</script>

</body>
</html>

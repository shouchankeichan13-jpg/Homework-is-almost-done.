<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>スパイダー・コンセントレーション ＆ 独り言AI思考解析タイマー</title>
  <!-- 🎉 クラッカー紙吹雪 -->
  <script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1.9.3/dist/confetti.browser.min.js"></script>
  <!-- 📈 リアルタイムグラフ (Chart.js) -->
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <!-- 👤 MediaPipe FaceMesh -->
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
      background: #090d16;
      color: #f8fafc;
      display: flex;
      justify-content: center;
      align-items: center;
      min-height: 100vh;
      padding: 20px;
    }
    .timer-card {
      background: #111827;
      border-radius: 24px;
      padding: 24px;
      width: 100%;
      max-width: 540px;
      text-align: center;
      box-shadow: 0 20px 50px rgba(220, 38, 38, 0.15), 0 10px 30px rgba(0, 0, 0, 0.8);
      border: 2px solid #374151;
      position: relative;
    }
    h1 {
      font-size: 1.25rem;
      margin-bottom: 12px;
      color: #ef4444;
      letter-spacing: 0.05em;
      display: flex;
      align-items: center;
      justify-content: center;
      gap: 8px;
    }

    /* 勉強テーマ入力欄 */
    .topic-container {
      margin-bottom: 12px;
      text-align: left;
    }
    .topic-container label {
      font-size: 0.78rem;
      color: #9ca3af;
      font-weight: bold;
      display: block;
      margin-bottom: 4px;
    }
    .topic-input {
      width: 100%;
      padding: 8px 12px;
      border-radius: 8px;
      border: 1px solid #4b5563;
      background: #030712;
      color: #38bdf8;
      font-size: 0.95rem;
      font-weight: bold;
      outline: none;
    }
    .topic-input:focus {
      border-color: #ef4444;
    }

    /* カメラ＆マスクビュー */
    .face-landmarkers-container {
      position: relative;
      width: 100%;
      height: 220px;
      margin-bottom: 12px;
      background: #000;
      border-radius: 16px;
      overflow: hidden;
      border: 2px solid #b91c1c;
      display: flex;
      justify-content: center;
      align-items: center;
    }
    #video { display: none; }
    #faceCanvas {
      position: absolute;
      top: 0; left: 0;
      width: 100%; height: 100%;
      transform: scaleX(-1); /* 鏡映像 */
    }
    .status-badge {
      position: absolute;
      bottom: 10px;
      left: 50%;
      transform: translateX(-50%);
      padding: 6px 14px;
      border-radius: 16px;
      font-size: 0.78rem;
      font-weight: bold;
      background: rgba(3, 7, 18, 0.85);
      border: 1px solid #ef4444;
      color: #fca5a5;
      backdrop-filter: blur(6px);
      white-space: nowrap;
      z-index: 10;
    }
    .audio-indicator {
      position: absolute;
      top: 10px;
      right: 10px;
      padding: 4px 10px;
      border-radius: 10px;
      font-size: 0.72rem;
      font-weight: bold;
      background: rgba(17, 24, 39, 0.85);
      border: 1px solid #374151;
      color: #9ca3af;
      z-index: 10;
    }
    .audio-indicator.active {
      color: #4ade80;
      border-color: #22c55e;
      background: rgba(22, 101, 52, 0.4);
    }

    /* 🗣️ 独り言リアルタイム解析テロップ */
    .speech-card {
      background: #030712;
      border: 1px solid #1f2937;
      border-radius: 12px;
      padding: 10px 12px;
      margin-bottom: 12px;
      text-align: left;
    }
    .speech-header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 4px;
      font-size: 0.75rem;
      color: #9ca3af;
    }
    .speech-category-tag {
      padding: 2px 8px;
      border-radius: 6px;
      font-size: 0.7rem;
      font-weight: bold;
    }
    .tag-thinking { background: #0284c7; color: #e0f2fe; }
    .tag-understood { background: #15803d; color: #dcfce7; }
    .tag-question { background: #b45309; color: #fef3c7; }
    .tag-offtopic { background: #b91c1c; color: #fee2e2; }
    .tag-idle { background: #374151; color: #9ca3af; }

    .speech-body {
      font-size: 0.88rem;
      color: #f3f4f6;
      min-height: 22px;
      font-weight: 500;
    }
    .speech-ai-advice {
      margin-top: 6px;
      font-size: 0.78rem;
      color: #38bdf8;
      background: rgba(14, 165, 233, 0.1);
      padding: 4px 8px;
      border-radius: 6px;
      border-left: 3px solid #0284c7;
      display: none;
    }

    .display {
      font-size: 4.2rem;
      font-weight: 800;
      font-family: 'Courier New', Courier, monospace;
      margin: 2px 0;
      color: #ef4444;
      text-shadow: 0 0 20px rgba(239, 68, 68, 0.4);
      transition: color 0.3s ease;
    }
    .interval-badge {
      display: inline-block;
      padding: 4px 14px;
      border-radius: 16px;
      background: #1f2937;
      font-size: 0.78rem;
      font-weight: 600;
      margin-bottom: 10px;
      color: #9ca3af;
    }

    .chart-container {
      position: relative;
      width: 100%;
      height: 90px;
      margin-bottom: 12px;
      background: #030712;
      padding: 6px;
      border-radius: 12px;
      border: 1px solid #1f2937;
    }

    .presets {
      display: flex;
      justify-content: center;
      gap: 6px;
      margin-bottom: 12px;
    }
    .preset-btn {
      background: #1f2937;
      color: #cbd5e1;
      border: 1px solid #374151;
      padding: 6px 12px;
      border-radius: 8px;
      cursor: pointer;
      font-weight: bold;
      font-size: 0.85rem;
    }
    .preset-btn.active {
      background: #dc2626;
      color: #ffffff;
      border-color: #ef4444;
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
      transition: all 0.2s ease;
    }
    .btn-start { background: #dc2626; color: #ffffff; box-shadow: 0 4px 14px rgba(220, 38, 38, 0.4); }
    .btn-start:hover { background: #ef4444; }
    .btn-reset { background: #374151; color: #d1d5db; }

    /* 📊 終了時要約モーダル */
    .modal-overlay {
      display: none;
      position: fixed;
      top:0; left:0; width:100%; height:100%;
      background: rgba(0,0,0,0.85);
      justify-content: center;
      align-items: center;
      z-index: 100;
      padding: 20px;
      backdrop-filter: blur(4px);
    }
    .modal-content {
      background: #111827;
      border-radius: 20px;
      padding: 24px;
      max-width: 500px;
      width: 100%;
      border: 2px solid #ef4444;
      text-align: left;
      max-height: 90vh;
      overflow-y: auto;
      box-shadow: 0 25px 50px rgba(220, 38, 38, 0.2);
    }
    .modal-title {
      font-size: 1.3rem;
      color: #f87171;
      text-align: center;
      margin-bottom: 16px;
      font-weight: bold;
    }
    .score-box {
      text-align: center;
      background: #030712;
      padding: 16px;
      border-radius: 16px;
      margin-bottom: 16px;
      border: 1px solid #1f2937;
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
      font-size: 0.9rem;
      color: #9ca3af;
      margin-bottom: 6px;
      border-bottom: 1px solid #1f2937;
      padding-bottom: 4px;
    }
    .log-list {
      background: #030712;
      border-radius: 8px;
      padding: 10px;
      max-height: 140px;
      overflow-y: auto;
      font-size: 0.82rem;
    }
    .log-item {
      margin-bottom: 6px;
      padding-bottom: 4px;
      border-bottom: 1px dashed #1f2937;
      display: flex;
      flex-direction: column;
      gap: 2px;
    }
    .log-item:last-child { border-bottom: none; }
    .close-btn {
      width: 100%;
      padding: 12px;
      background: #dc2626;
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
  <h1>🕷️ スパイダーAI 集中タイマー</h1>

  <div class="topic-container">
    <label for="studyTopic">📝 今日の学習テーマ（例: 数学の微分積分 / 英単語）</label>
    <input type="text" id="studyTopic" class="topic-input" value="数学の微分積分">
  </div>

  <div class="face-landmarkers-container">
    <div class="audio-indicator" id="audioIndicator">✏️ 筆記音: 未検知</div>
    <div class="status-badge" id="statusBadge">カメラ・マイク準備中…</div>
    <video id="video" playsinline></video>
    <canvas id="faceCanvas"></canvas>
  </div>

  <!-- 🗣️ 高度化された独り言AI解析エリア -->
  <div class="speech-card">
    <div class="speech-header">
      <span>🗣️ 独り言・思考リアルタイムAI解析</span>
      <span class="speech-category-tag tag-idle" id="speechCategoryTag">待機中</span>
    </div>
    <div class="speech-body" id="speechText">（マイクに向かって思考をつぶやいてみましょう）</div>
    <div class="speech-ai-advice" id="speechAiAdvice"></div>
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
    <div class="modal-title">🕷️ 学習・思考集中度 分析レポート</div>

    <div class="score-box">
      <div style="font-size:0.82rem; color:#9ca3af;">総合集中スコア</div>
      <div class="score-number" id="finalScoreDisplay">88</div>
      <div id="scoreEvalText" style="font-size:0.88rem; color:#38bdf8;">素晴らしい集中力でした！</div>
    </div>

    <div class="summary-section">
      <h3>📈 姿勢・筆記・思考の総合分析</h3>
      <p id="postureSummary" style="font-size:0.82rem; color:#d1d5db; margin-bottom:4px;"></p>
      <p id="writingSummary" style="font-size:0.82rem; color:#d1d5db; margin-bottom:4px;"></p>
      <p id="speechSummary" style="font-size:0.82rem; color:#38bdf8;"></p>
    </div>

    <div class="summary-section">
      <h3>💬 発話ログ & 思考セッション履歴</h3>
      <div class="log-list" id="speechLogList">
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

  // 🗣️ 独り言AI解析エンジン
  let recognition = null;
  let speechLogs = []; // 全発話ログ [{ time, text, category, advice }]
  let categoryCounts = { thinking: 0, understood: 0, question: 0, offtopic: 0 };

  const video = document.getElementById('video');
  const faceCanvas = document.getElementById('faceCanvas');
  const faceCtx = faceCanvas.getContext('2d');
  const statusBadge = document.getElementById('statusBadge');
  const audioIndicator = document.getElementById('audioIndicator');
  
  const speechText = document.getElementById('speechText');
  const speechCategoryTag = document.getElementById('speechCategoryTag');
  const speechAiAdvice = document.getElementById('speechAiAdvice');

  const display = document.getElementById('display');
  const intervalBadge = document.getElementById('intervalBadge');
  const startBtn = document.getElementById('startBtn');
  const studyTopicInput = document.getElementById('studyTopic');

  // 辞書・パターン定義（独り言AI解析）
  const DICT = {
    thinking: ["なぜ", "だから", "つまり", "公式", "代入", "仮定", "まず", "整理", "計算", "解法", "まとめると", "求める", "定義", "手順", "順番"],
    understood: ["なるほど", "わかった", "解けた", "合ってる", "オッケー", "理解", "これか", "完璧", "見えた", "よし"],
    question: ["わからない", "どういうこと", "変だな", "間違えた", "なんで", "合わない", "難しい", "どこで", "不可解"],
    offtopic: ["眠い", "スマホ", "Youtube", "お腹すいた", "だるい", "ゲーム", "疲れた", "帰りたい", "遊ぶ", "漫画", "めんどい", "後でいい"]
  };

  // 🎙️ 高度化された音声認識 & AI解析初期化
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
      if (!transcript) return;

      analyzeSpeechWithAI(transcript);
    };

    recognition.onerror = (err) => {
      console.warn("音声認識エラー:", err.error);
    };

    recognition.onend = () => {
      if (timerId && recognition) {
        try { recognition.start(); } catch(e){}
      }
    };
  }

  // 🤖 独り言のリアルタイムAI文脈解析
  function analyzeSpeechWithAI(text) {
    const topic = studyTopicInput.value.trim();
    const topicKeywords = topic.split(/[\s　]+/);
    const timeStr = display.textContent;

    let category = 'thinking'; // デフォルトは思考中
    let advice = "";

    // 1. 各カテゴリのスコア算出
    let scores = {
      thinking: 0,
      understood: 0,
      question: 0,
      offtopic: 0
    };

    // 勉強テーマキーワードが含まれている場合は思考スコア加算
    const containsTopic = topicKeywords.some(kw => kw.length > 1 && text.includes(kw));
    if (containsTopic) scores.thinking += 2;

    for (let cat in DICT) {
      DICT[cat].forEach(word => {
        if (text.includes(word)) scores[cat] += 1;
      });
    }

    // 最高スコアのカテゴリを特定
    let maxCat = 'thinking';
    let maxVal = -1;
    for (let cat in scores) {
      if (scores[cat] > maxVal) {
        maxVal = scores[cat];
        maxCat = cat;
      }
    }

    // 判定調整（辞書に引っかからずテーマ単語もない短文は保留/脱線チェック）
    if (maxVal === 0) {
      if (text.length > 6) maxCat = 'offtopic';
      else maxCat = 'thinking';
    }

    category = maxCat;
    categoryCounts[category]++;

    // カテゴリごとのタグ演出 & AIアドバイス生成
    speechCategoryTag.className = 'speech-category-tag';
    speechText.textContent = `「${text}」`;

    if (category === 'thinking') {
      speechCategoryTag.textContent = "🟢 思考・学習中";
      speechCategoryTag.classList.add('tag-thinking');
      speechAiAdvice.style.display = 'none';
    } else if (category === 'understood') {
      speechCategoryTag.textContent = "✨ 理解・アハ体験";
      speechCategoryTag.classList.add('tag-understood');
      speechAiAdvice.textContent = "💡 素晴らしい！解法が定着しています。この調子で進めましょう。";
      speechAiAdvice.style.display = 'block';
      smoothedScore = Math.min(100, smoothedScore + 10); // 集中度ボーナス
    } else if (category === 'question') {
      speechCategoryTag.textContent = "💡 疑問・悩み";
      speechCategoryTag.classList.add('tag-question');
      speechAiAdvice.textContent = `💡 疑問を言語化できていますね。テーマ「${topicKeywords[0] || topic}」の基本定義や条件を見直してみましょう。`;
      speechAiAdvice.style.display = 'block';
    } else if (category === 'offtopic') {
      speechCategoryTag.textContent = "⚠️ 脱線・集中低下";
      speechCategoryTag.classList.add('tag-offtopic');
      speechAiAdvice.textContent = `⚠️ 集中が散漫になっています。『${topic}』に意識を戻しましょう！`;
      speechAiAdvice.style.display = 'block';
      smoothedScore = Math.max(0, smoothedScore - 15); // 集中度ペナルティ
    }

    // ログに保存
    speechLogs.push({
      time: timeStr,
      text: text,
      category: category,
      advice: advice || (speechAiAdvice.style.display === 'block' ? speechAiAdvice.textContent : "")
    });
  }

  // 🕷️【スパイダーマンマスクを描画する高度描画エンジン】
  function drawSpiderManMask(landmarks) {
    if (!landmarks || landmarks.length === 0) return;

    const w = faceCanvas.width;
    const h = faceCanvas.height;

    // キーポインターの取得（ランドマークインデックス）
    const getPt = (idx) => ({ x: landmarks[idx].x * w, y: landmarks[idx].y * w, z: landmarks[idx].z }); // アスペクト正方化

    // 1. 顔の外郭（Silhouette）のランドマークインデックス列
    const faceOutlineIndices = [
      10, 338, 297, 332, 284, 251, 389, 356, 454, 323, 361, 288, 397, 365, 379, 378, 400, 377,
      152, 148, 176, 149, 150, 136, 172, 58, 132, 93, 234, 127, 162, 21, 54, 103, 67, 109
    ];

    faceCtx.save();

    // 顔の外郭パスの作成
    faceCtx.beginPath();
    faceOutlineIndices.forEach((idx, i) => {
      const pt = getPt(idx);
      if (i === 0) faceCtx.moveTo(pt.x, pt.y);
      else faceCtx.lineTo(pt.x, pt.y);
    });
    faceCtx.closePath();

    // --- A. マスクのベースカラー & 立体グラデーション ---
    const nosePt = getPt(1); // 鼻先を中心
    const chinPt = getPt(152);
    const grad = faceCtx.createRadialGradient(nosePt.x, nosePt.y, 10, nosePt.x, nosePt.y, Math.abs(chinPt.y - nosePt.y) * 1.5);
    grad.addColorStop(0, '#ef4444');   // 中央：明るい赤
    grad.addColorStop(0.6, '#dc2626'); // 中間：スパイダーレッド
    grad.addColorStop(1, '#7f1d1d');   // 外縁：ダークレッド

    faceCtx.fillStyle = grad;
    faceCtx.fill();

    // クリップして顔の範囲内にすべての描画を制限
    faceCtx.clip();

    // --- B. スーツのヘキサゴン・微細テクスチャ演出 ---
    faceCtx.strokeStyle = 'rgba(0, 0, 0, 0.15)';
    faceCtx.lineWidth = 1;
    for (let x = 0; x < w; x += 12) {
      faceCtx.beginPath();
      faceCtx.moveTo(x, 0);
      faceCtx.lineTo(x, h);
      faceCtx.stroke();
    }

    // --- C. 蜘蛛の巣（ウェブ）ネットワークの描画 ---
    // 放射状のメインウェブ（鼻中心 1/4 から伸びる線）
    const webCenter = getPt(4); // 鼻筋中央
    const radialTargets = [10, 338, 297, 332, 284, 251, 454, 323, 361, 152, 136, 172, 58, 234, 127, 21, 54, 103, 67];

    faceCtx.strokeStyle = 'rgba(15, 23, 42, 0.85)';
    faceCtx.lineWidth = 1.8;

    radialTargets.forEach(idx => {
      const target = getPt(idx);
      faceCtx.beginPath();
      faceCtx.moveTo(webCenter.x, webCenter.y);
      faceCtx.lineTo(target.x, target.y);
      faceCtx.stroke();
    });

    // 同心円状のウェブリング（3段階の高さ）
    const ringFactors = [0.25, 0.5, 0.75, 0.95];
    ringFactors.forEach(factor => {
      faceCtx.beginPath();
      radialTargets.forEach((idx, i) => {
        const target = getPt(idx);
        const rx = webCenter.x + (target.x - webCenter.x) * factor;
        const ry = webCenter.y + (target.y - webCenter.y) * factor;

        if (i === 0) faceCtx.moveTo(rx, ry);
        else {
          const prevIdx = radialTargets[i - 1];
          const prevTarget = getPt(prevIdx);
          const prx = webCenter.x + (prevTarget.x - webCenter.x) * factor;
          const pry = webCenter.y + (prevTarget.y - webCenter.y) * factor;
          // ベジェ曲線でたるみを持たせる
          const cx = (prx + rx) / 2 + (webCenter.x - (prx + rx) / 2) * 0.15;
          const cy = (pry + ry) / 2 + (webCenter.y - (pry + ry) / 2) * 0.15;
          faceCtx.quadraticCurveTo(cx, cy, rx, ry);
        }
      });
      faceCtx.closePath();
      faceCtx.stroke();
    });

    // --- D. リアル・スパイダーアイ（太い黒枠＋メタリック白レンズ） ---
    // 左目・右目の周囲骨格ランドマーク
    const leftEyeIndices = [70, 63, 105, 66, 107, 55, 193, 245, 128, 121, 120, 119, 215, 138, 135, 169, 170];
    const rightEyeIndices = [300, 293, 334, 296, 336, 285, 417, 465, 357, 350, 349, 348, 435, 367, 364, 394, 395];

    function drawSpideyEye(indices, isLeft) {
      faceCtx.save();
      faceCtx.beginPath();
      indices.forEach((idx, i) => {
        const pt = getPt(idx);
        if (i === 0) faceCtx.moveTo(pt.x, pt.y);
        else faceCtx.lineTo(pt.x, pt.y);
      });
      faceCtx.closePath();

      // 目内部グラデーション（メタリックシルバー～ホワイト）
      const firstPt = getPt(indices[0]);
      const lastPt = getPt(indices[8] || indices[4]);
      const eyeGrad = faceCtx.createLinearGradient(firstPt.x, firstPt.y, lastPt.x, lastPt.y);
      eyeGrad.addColorStop(0, '#ffffff');
      eyeGrad.addColorStop(0.7, '#e2e8f0');
      eyeGrad.addColorStop(1, '#94a3b8');

      faceCtx.fillStyle = eyeGrad;
      faceCtx.fill();

      // レンズ縁の漆黒太フレーム
      faceCtx.strokeStyle = '#090d16';
      faceCtx.lineWidth = 6;
      faceCtx.stroke();

      // 内側の光沢ハイライト線
      faceCtx.lineWidth = 2;
      faceCtx.strokeStyle = 'rgba(255, 255, 255, 0.8)';
      faceCtx.stroke();

      faceCtx.restore();
    }

    drawSpideyEye(leftEyeIndices, true);
    drawSpideyEye(rightEyeIndices, false);

    // --- E. 鼻筋・頬の立体シャドウ（シェーディング） ---
    const noseShadowGrad = faceCtx.createLinearGradient(webCenter.x - 20, webCenter.y, webCenter.x + 20, webCenter.y);
    noseShadowGrad.addColorStop(0, 'rgba(0,0,0,0.3)');
    noseShadowGrad.addColorStop(0.5, 'rgba(0,0,0,0)');
    noseShadowGrad.addColorStop(1, 'rgba(0,0,0,0.3)');
    faceCtx.fillStyle = noseShadowGrad;
    faceCtx.fillRect(0, 0, w, h);

    faceCtx.restore();
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
          borderColor: '#ef4444',
          backgroundColor: 'rgba(239, 68, 68, 0.15)',
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
          y: { min: 0, max: 100, ticks: { color: '#6b7280' }, grid: { color: '#1f2937' } }
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

  // 🎙️ マイク音響解析（筆記音検出）
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

  // 👤 MediaPipe FaceMesh & スパイダーマン描画ループ
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
      pitchStatus = "手元・ノート集中";
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

      // 🕷️ スパイダーマンマスクの動的フィット描画
      drawSpiderManMask(landmarks);

      const { score, status } = calculateConcentration(landmarks);
      latestConcentrationScore = score;

      const rounded = Math.round(score);
      statusBadge.textContent = `🎯 集中度: ${rounded}% (${status})`;
      statusBadge.style.borderColor = rounded >= 70 ? '#22c55e' : rounded >= 40 ? '#facc15' : '#ef4444';
      statusBadge.style.color = rounded >= 70 ? '#4ade80' : rounded >= 40 ? '#facc15' : '#f87171';
    } else {
      latestConcentrationScore = 0;
      smoothedScore = 0;
      statusBadge.textContent = `🚶 離席・顔未検出 (0%)`;
      statusBadge.style.borderColor = '#ef4444';
      statusBadge.style.color = '#f87171';
    }
    faceCtx.restore();
  }

  async function startCamera() {
    initFaceMesh();
    initMicrophone();
    initSpeechRecognition();

    try {
      videoStream = await navigator.mediaDevices.getUserMedia({ video: { width: 640, height: 480 } });
      video.srcObject = videoStream;
      await video.play();

      const processVideo = async () => {
        if (faceMesh && video.readyState >= 2) {
          await faceMesh.send({ image: video });
        }
        requestAnimationFrame(processVideo);
      };
      processVideo();
    } catch (err) {
      statusBadge.textContent = "❌ カメラの起動に失敗しました";
    }
  }

  // タイマー制御
  function updateDisplay() {
    const mins = Math.floor(remainingSeconds / 60);
    const secs = remainingSeconds % 60;
    display.textContent = `${String(mins).padStart(2, '0')}:${String(secs).padStart(2, '0')}`;
  }

  function toggleTimer() {
    if (timerId) {
      pauseTimer();
    } else {
      startTimer();
    }
  }

  function startTimer() {
    if (!videoStream) startCamera();
    if (recognition) {
      try { recognition.start(); } catch(e){}
    }

    startBtn.textContent = '一時停止';
    startBtn.style.background = '#eab308';
    startBtn.style.color = '#422006';

    timerId = setInterval(() => {
      remainingSeconds--;
      totalTicks++;
      updateDisplay();

      // 毎秒グラフ更新
      addChartData(latestConcentrationScore);

      // 1分ごとのチャイム
      if (remainingSeconds > 0 && remainingSeconds % 60 === 0) {
        playSchoolPipipi();
      }

      if (remainingSeconds <= 0) {
        finishTimer();
      }
    }, 1000);
  }

  function pauseTimer() {
    clearInterval(timerId);
    timerId = null;
    startBtn.textContent = '再開';
    startBtn.style.background = '#22c55e';
    startBtn.style.color = '#052e16';
  }

  function resetTimer() {
    pauseTimer();
    remainingSeconds = totalSeconds;
    updateDisplay();
    scoreHistory = [];
    speechLogs = [];
    categoryCounts = { thinking: 0, understood: 0, question: 0, offtopic: 0 };
    if (chart) {
      chart.data.labels = [];
      chart.data.datasets[0].data = [];
      chart.update();
    }
    startBtn.textContent = 'スタート';
    startBtn.style.background = '#dc2626';
    startBtn.style.color = '#ffffff';
  }

  function setPreset(mins) {
    totalSeconds = mins * 60;
    document.querySelectorAll('.preset-btn').forEach(btn => btn.classList.remove('active'));
    event.target.classList.add('active');
    resetTimer();
  }

  function finishTimer() {
    pauseTimer();
    playFinishSound();

    if (typeof confetti === 'function') {
      confetti({ particleCount: 120, spread: 80, origin: { y: 0.6 } });
    }

    showSummaryReport();
  }

  // 📊 要約レポートの計算 & 表示
  function showSummaryReport() {
    const modal = document.getElementById('summaryModal');
    const finalScoreDisplay = document.getElementById('finalScoreDisplay');
    const scoreEvalText = document.getElementById('scoreEvalText');
    const postureSummary = document.getElementById('postureSummary');
    const writingSummary = document.getElementById('writingSummary');
    const speechSummary = document.getElementById('speechSummary');
    const speechLogList = document.getElementById('speechLogList');

    // 総合スコア計算
    const avgScore = scoreHistory.length > 0 ? Math.round(scoreHistory.reduce((a, b) => a + b, 0) / scoreHistory.length) : 85;
    finalScoreDisplay.textContent = avgScore;

    if (avgScore >= 80) scoreEvalText.textContent = "🦸‍♂️ ヒーロー級の驚異的な集中力です！";
    else if (avgScore >= 60) scoreEvalText.textContent = "👍 良好な集中度を維持できました。";
    else scoreEvalText.textContent = "⚠️ 少し疲れが見られます。小休憩を挟みましょう。";

    // 姿勢・筆記分析
    postureSummary.textContent = `・正面・手元視線の維持率: ${Math.min(100, Math.round(avgScore * 1.05))}%`;
    const writingRatio = totalTicks > 0 ? Math.round((writingDetectCount / totalTicks) * 100) : 0;
    writingSummary.textContent = `・アクティブ筆記・作業時間割合: ${writingRatio}%`;

    // 思考・独り言分析
    const totalSpeech = speechLogs.length;
    speechSummary.textContent = `・全発話数: ${totalSpeech}回（思考: ${categoryCounts.thinking} / 理解: ${categoryCounts.understood} / 疑問: ${categoryCounts.question} / 脱線: ${categoryCounts.offtopic}）`;

    // ログリストの生成
    speechLogList.innerHTML = '';
    if (speechLogs.length === 0) {
      speechLogList.innerHTML = '<div style="color:#6b7280; text-align:center;">発話ログはありませんでした</div>';
    } else {
      speechLogs.forEach(log => {
        const item = document.createElement('div');
        item.className = 'log-item';
        
        let tagHtml = '';
        if (log.category === 'thinking') tagHtml = '<span style="color:#38bdf8;">[思考]</span>';
        else if (log.category === 'understood') tagHtml = '<span style="color:#4ade80;">[理解]</span>';
        else if (log.category === 'question') tagHtml = '<span style="color:#facc15;">[疑問]</span>';
        else if (log.category === 'offtopic') tagHtml = '<span style="color:#f87171;">[脱線]</span>';

        item.innerHTML = `
          <div><strong style="color:#9ca3af;">${log.time}</strong> ${tagHtml} ${log.text}</div>
          ${log.advice ? `<div style="font-size:0.75rem; color:#0284c7; margin-left:12px;">└ ${log.advice}</div>` : ''}
        `;
        speechLogList.appendChild(item);
      });
    }

    modal.style.display = 'flex';
  }

  function closeModal() {
    document.getElementById('summaryModal').style.display = 'none';
  }

  // 初期化実行
  window.addEventListener('DOMContentLoaded', () => {
    initChart();
    updateDisplay();
  });
</script>
</body>
</html>

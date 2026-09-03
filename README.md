<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>独り言ログ & 顔変身ARアプリ</title>
    <style>
        :root {
            --bg-color: #0f172a;
            --card-bg: #1e293b;
            --accent-color: #38bdf8;
            --accent-hover: #0284c7;
            --text-color: #f8fafc;
            --text-muted: #94a3b8;
            --spider-red: #ef4444;
        }

        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }

        body {
            background-color: var(--bg-color);
            color: var(--text-color);
            display: flex;
            flex-direction: column;
            align-items: center;
            min-height: 100vh;
            padding: 20px;
        }

        h1 {
            margin-bottom: 20px;
            font-size: 1.8rem;
            color: var(--accent-color);
            text-align: center;
        }

        .container {
            display: grid;
            grid-template-columns: 1fr;
            gap: 20px;
            max-width: 1000px;
            width: 100%;
        }

        @media (min-width: 768px) {
            .container {
                grid-template-columns: 1fr 1fr;
            }
        }

        .card {
            background: var(--card-bg);
            border-radius: 12px;
            padding: 20px;
            box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.3);
            display: flex;
            flex-direction: column;
            gap: 15px;
        }

        .card h2 {
            font-size: 1.3rem;
            border-bottom: 2px solid var(--accent-color);
            padding-bottom: 8px;
            display: flex;
            align-items: center;
            gap: 8px;
        }

        /* 独り言ログ用スタイル */
        .setting-group {
            display: flex;
            flex-direction: column;
            gap: 5px;
        }

        label {
            font-size: 0.9rem;
            color: var(--text-muted);
        }

        .slider-container {
            display: flex;
            align-items: center;
            gap: 10px;
        }

        input[type="range"] {
            flex: 1;
            accent-color: var(--accent-color);
        }

        .sec-display {
            font-weight: bold;
            color: var(--accent-color);
            min-width: 45px;
        }

        button {
            background: var(--accent-color);
            color: #000;
            border: none;
            padding: 12px 20px;
            border-radius: 8px;
            font-weight: bold;
            cursor: pointer;
            transition: background 0.2s, transform 0.1s;
        }

        button:hover {
            background: var(--accent-hover);
        }

        button:active {
            transform: scale(0.98);
        }

        button:disabled {
            background: #475569;
            color: #94a3b8;
            cursor: not-allowed;
        }

        .status {
            font-size: 0.85rem;
            color: #f59e0b;
            min-height: 20px;
        }

        .log-box, .ai-box {
            background: #090d16;
            border-radius: 8px;
            padding: 12px;
            min-height: 80px;
            max-height: 150px;
            overflow-y: auto;
            font-size: 0.95rem;
            border: 1px solid #334155;
            white-space: pre-wrap;
        }

        .ai-box {
            border-color: #3b82f6;
            color: #93c5fd;
        }

        /* 顔変身カメラ用スタイル */
        .camera-container {
            position: relative;
            width: 100%;
            aspect-ratio: 4/3;
            background: #000;
            border-radius: 8px;
            overflow: hidden;
        }

        video, canvas {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            object-fit: cover;
        }

        video {
            transform: scaleX(-1); /* 鏡像表示 */
        }

        canvas {
            transform: scaleX(-1);
            pointer-events: none;
        }

        select {
            background: #0f172a;
            color: #fff;
            border: 1px solid #334155;
            padding: 10px;
            border-radius: 6px;
            font-size: 1rem;
        }
    </style>
</head>
<body>

    <h1>🎤 独り言ログ & 🎭 顔変身ARアプリ</h1>

    <div class="container">
        <!-- 1. 独り言ログセクション -->
        <div class="card">
            <h2>🎙️ 独り言ログ（STT → AI）</h2>
            
            <div class="setting-group">
                <label for="duration">録音時間（秒数調整）:</label>
                <div class="slider-container">
                    <input type="range" id="duration" min="2" max="20" value="5">
                    <span class="sec-display" id="secVal">5 秒</span>
                </div>
            </div>

            <button id="startRecBtn">録音 & 認識スタート</button>
            <div class="status" id="statusText">待機中</div>

            <label>1. 認識された音声テキスト (STT):</label>
            <div class="log-box" id="sttResult">ここに認識結果が表示されます...</div>

            <label>2. AIの応答・分析結果:</label>
            <div class="ai-box" id="aiResponse">AIが読み込んだ内容の応答がここに表示されます...</div>
        </div>

        <!-- 2. 顔認識・顔変身セクション -->
        <div class="card">
            <h2>🎭 顔変身カメラ（ARフィルタ）</h2>

            <div class="setting-group">
                <label for="maskSelect">変換するマスクを選択:</label>
                <select id="maskSelect">
                    <option value="spiderman">🕷️ スパイダーマン</option>
                    <option value="cyber">🤖 サイバーマスク</option>
                    <option value="anon">🎭 映画風仮面</option>
                </select>
            </div>

            <button id="toggleCamBtn">カメラ起動</button>

            <div class="camera-container">
                <video id="webcam" autoplay playsinline muted></video>
                <canvas id="arCanvas"></canvas>
            </div>
        </div>
    </div>

    <script>
        // ==========================================
        // 1. 独り言ログ機能（音声認識 + AI読み込み）
        // ==========================================
        const durationInput = document.getElementById('duration');
        const secVal = document.getElementById('secVal');
        const startRecBtn = document.getElementById('startRecBtn');
        const statusText = document.getElementById('statusText');
        const sttResult = document.getElementById('sttResult');
        const aiResponse = document.getElementById('aiResponse');

        let recTimer = null;
        let isRecording = false;

        // 秒数スライダーの反映
        durationInput.addEventListener('input', (e) => {
            secVal.textContent = `${e.target.value} 秒`;
        });

        // Web Speech API（音声認識）の準備
        const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
        let recognition = null;

        if (SpeechRecognition) {
            recognition = new SpeechRecognition();
            recognition.lang = 'ja-JP';
            recognition.continuous = true;
            recognition.interimResults = true;

            recognition.onresult = (event) => {
                let transcript = '';
                for (let i = event.resultIndex; i < event.results.length; i++) {
                    transcript += event.results[i][0].transcript;
                }
                sttResult.textContent = transcript;
            };

            recognition.onerror = (event) => {
                statusText.textContent = `エラーが発生しました: ${event.error}`;
                stopRecording();
            };

            recognition.onend = () => {
                if (isRecording) {
                    stopRecording();
                }
            };
        } else {
            statusText.textContent = 'お使いのブラウザは音声認識に対応していません。(Chrome推奨)';
            startRecBtn.disabled = true;
        }

        // 録音 & 認識スタート
        startRecBtn.addEventListener('click', () => {
            if (isRecording) {
                stopRecording();
            } else {
                startRecording();
            }
        });

        function startRecording() {
            const seconds = parseInt(durationInput.value, 10);
            sttResult.textContent = '音声を聞き取っています...';
            aiResponse.textContent = 'AIの処理待ち...';
            statusText.textContent = `録音中... (あと ${seconds} 秒)`;
            startRecBtn.textContent = '停止';
            isRecording = true;

            try {
                recognition.start();
            } catch (e) {
                console.log(e);
            }

            let timeLeft = seconds;
            recTimer = setInterval(() => {
                timeLeft--;
                if (timeLeft > 0) {
                    statusText.textContent = `録音中... (あと ${timeLeft} 秒)`;
                } else {
                    stopRecording();
                }
            }, 1000);
        }

        function stopRecording() {
            clearInterval(recTimer);
            isRecording = false;
            startRecBtn.textContent = '録音 & 認識スタート';
            statusText.textContent = '認識完了。AIに送信中...';

            if (recognition) {
                try {
                    recognition.stop();
                } catch (e) {}
            }

            // 音声認識テキストを AI に読ませる（シミュレーション処理）
            setTimeout(() => {
                processTextWithAI(sttResult.textContent);
            }, 800);
        }

        // 認識されたテキストをAIが処理する関数
        function processTextWithAI(text) {
            if (!text || text === '音声を聞き取っています...' || text.trim() === '') {
                statusText.textContent = '音声が検出されませんでした。';
                aiResponse.textContent = '（音声がありませんでした）';
                return;
            }

            statusText.textContent = 'AIの分析完了';

            // 擬似的なAIレスポンス（API接続も可能）
            const responses = [
                `AI: 「${text}」という独り言を記録しました。とても興味深いアイデアですね！`,
                `AI: 了解しました。「${text}」についてのメモをデータベースに保存しました。`,
                `AI分析: 「${text}」→ 思考が整理されています。関連タスクを追加しますか？`
            ];
            const randomResponse = responses[Math.floor(Math.random() * responses.length)];
            aiResponse.textContent = randomResponse;
        }


        // ==========================================
        // 2. 顔認識・顔変身（Webcam + ARマスク描画）
        // ==========================================
        const webcam = document.getElementById('webcam');
        const canvas = document.getElementById('arCanvas');
        const ctx = canvas.getContext('2d');
        const toggleCamBtn = document.getElementById('toggleCamBtn');
        const maskSelect = document.getElementById('maskSelect');

        let isCameraOn = false;
        let animationFrameId = null;

        toggleCamBtn.addEventListener('click', async () => {
            if (isCameraOn) {
                stopCamera();
            } else {
                await startCamera();
            }
        });

        async function startCamera() {
            try {
                const stream = await navigator.mediaDevices.getUserMedia({
                    video: { width: 640, height: 480 }
                });
                webcam.srcObject = stream;
                isCameraOn = true;
                toggleCamBtn.textContent = 'カメラ停止';

                webcam.onloadedmetadata = () => {
                    canvas.width = webcam.videoWidth;
                    canvas.height = webcam.videoHeight;
                    drawAROverlay();
                };
            } catch (err) {
                alert('カメラへのアクセスが拒否されたか、利用できません。');
            }
        }

        function stopCamera() {
            if (webcam.srcObject) {
                webcam.srcObject.getTracks().forEach(track => track.stop());
            }
            isCameraOn = false;
            toggleCamBtn.textContent = 'カメラ起動';
            cancelAnimationFrame(animationFrameId);
            ctx.clearRect(0, 0, canvas.width, canvas.height);
        }

        // Face Detection (ブラウザ標準FaceDetectorまたはCanvasオーバーレイ)
        async function drawAROverlay() {
            if (!isCameraOn) return;

            ctx.clearRect(0, 0, canvas.width, canvas.height);

            // 顔検出（Shape Detection APIに対応しているブラウザの場合）
            let faceFound = false;
            let faceX = canvas.width / 2 - 100;
            let faceY = canvas.height / 2 - 120;
            let faceWidth = 200;
            let faceHeight = 240;

            if ('FaceDetector' in window) {
                try {
                    const faceDetector = new FaceDetector({ fastMode: true });
                    const faces = await faceDetector.detect(webcam);
                    if (faces.length > 0) {
                        const box = faces[0].boundingBox;
                        faceX = box.x;
                        faceY = box.y;
                        faceWidth = box.width;
                        faceHeight = box.height;
                        faceFound = true;
                    }
                } catch (e) {
                    // API未対応時は中央固定
                }
            }

            // 選択されたマスクを描画
            const selectedMask = maskSelect.value;
            if (selectedMask === 'spiderman') {
                drawSpidermanMask(faceX, faceY, faceWidth, faceHeight);
            } else if (selectedMask === 'cyber') {
                drawCyberMask(faceX, faceY, faceWidth, faceHeight);
            } else if (selectedMask === 'anon') {
                drawAnonMask(faceX, faceY, faceWidth, faceHeight);
            }

            animationFrameId = requestAnimationFrame(drawAROverlay);
        }

        // 🕷️ スパイダーマンマスクの描画ロジック
        function drawSpidermanMask(x, y, w, h) {
            ctx.save();
            
            // 赤いマスクベース
            ctx.fillStyle = '#dc2626';
            ctx.beginPath();
            ctx.ellipse(x + w/2, y + h/2, w/2, h/1.8, 0, 0, Math.PI * 2);
            ctx.fill();

            // 蜘蛛の巣模様（ライン）
            ctx.strokeStyle = '#000000';
            ctx.lineWidth = 3;
            ctx.beginPath();
            // 放射状の線
            for (let i = 0; i < 8; i++) {
                const angle = (i * Math.PI) / 4;
                ctx.moveTo(x + w/2, y + h/2);
                ctx.lineTo(x + w/2 + (w/2) * Math.cos(angle), y + h/2 + (h/1.8) * Math.sin(angle));
            }
            ctx.stroke();

            // 巨大な目の特徴（左目 & 右目）
            const drawEye = (centerX, centerY) => {
                ctx.fillStyle = '#ffffff';
                ctx.strokeStyle = '#000000';
                ctx.lineWidth = 8;
                ctx.beginPath();
                ctx.moveTo(centerX - w*0.15, centerY - h*0.05);
                ctx.quadraticCurveTo(centerX, centerY - h*0.18, centerX + w*0.15, centerY - h*0.05);
                ctx.quadraticCurveTo(centerX + w*0.05, centerY + h*0.12, centerX - w*0.15, centerY - h*0.05);
                ctx.fill();
                ctx.stroke();
            };

            drawEye(x + w*0.3, y + h*0.4);
            drawEye(x + w*0.7, y + h*0.4);

            ctx.restore();
        }

        // 🤖 サイバーマスク
        function drawCyberMask(x, y, w, h) {
            ctx.save();
            ctx.fillStyle = 'rgba(14, 165, 233, 0.8)';
            ctx.fillRect(x + w*0.1, y + h*0.3, w*0.8, h*0.25);
            ctx.strokeStyle = '#38bdf8';
            ctx.lineWidth = 4;
            ctx.strokeRect(x + w*0.05, y + h*0.25, w*0.9, h*0.35);
            ctx.restore();
        }

        // 🎭 映画風仮面
        function drawAnonMask(x, y, w, h) {
            ctx.save();
            ctx.fillStyle = '#f8fafc';
            ctx.beginPath();
            ctx.ellipse(x + w/2, y + h/2, w/2.2, h/1.9, 0, 0, Math.PI * 2);
            ctx.fill();
            // ひげ
            ctx.fillStyle = '#000';
            ctx.fillRect(x + w*0.35, y + h*0.7, w*0.3, h*0.05);
            ctx.restore();
        }
    </script>
</body>
</html>

<!DOCTYPE html>
<html lang="sw">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Piga Picha na Pakua</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            font-family: 'Arial', sans-serif;
        }

        body {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
            padding: 20px;
        }

        .container {
            background: white;
            border-radius: 20px;
            box-shadow: 0 20px 60px rgba(0,0,0,0.3);
            padding: 30px;
            width: 100%;
            max-width: 600px;
            text-align: center;
        }

        h1 {
            color: #333;
            margin-bottom: 20px;
            font-size: 28px;
        }

        .camera-container {
            position: relative;
            width: 100%;
            max-width: 500px;
            margin: 0 auto 20px;
        }

        video {
            width: 100%;
            height: auto;
            border-radius: 15px;
            border: 3px solid #ddd;
            background: #000;
            transform: scaleX(-1); /* Kutengeneza kioo */
        }

        canvas {
            display: none;
        }

        .preview-container {
            margin: 20px 0;
        }

        #preview {
            max-width: 100%;
            border-radius: 10px;
            border: 3px solid #4CAF50;
            display: none;
        }

        .controls {
            display: flex;
            flex-wrap: wrap;
            gap: 15px;
            justify-content: center;
            margin-top: 20px;
        }

        button {
            padding: 15px 30px;
            border: none;
            border-radius: 50px;
            font-size: 16px;
            font-weight: bold;
            cursor: pointer;
            transition: all 0.3s ease;
            display: flex;
            align-items: center;
            justify-content: center;
            gap: 8px;
            min-width: 180px;
        }

        .capture-btn {
            background: linear-gradient(45deg, #FF416C, #FF4B2B);
            color: white;
        }

        .download-btn {
            background: linear-gradient(45deg, #2196F3, #21CBF3);
            color: white;
        }

        .retake-btn {
            background: linear-gradient(45deg, #FF9800, #FF5722);
            color: white;
        }

        button:hover {
            transform: translateY(-3px);
            box-shadow: 0 10px 20px rgba(0,0,0,0.2);
        }

        button:active {
            transform: translateY(-1px);
        }

        button:disabled {
            opacity: 0.5;
            cursor: not-allowed;
            transform: none;
        }

        .permission-msg {
            background: #ffebee;
            color: #c62828;
            padding: 15px;
            border-radius: 10px;
            margin: 20px 0;
            display: none;
        }

        .instruction {
            color: #666;
            margin: 15px 0;
            font-size: 14px;
            line-height: 1.5;
        }

        .icon {
            font-size: 20px;
        }

        @media (max-width: 600px) {
            .container {
                padding: 20px;
            }
            
            button {
                width: 100%;
            }
            
            h1 {
                font-size: 24px;
            }
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>📸 Piga Picha na Pakua</h1>
        
        <p class="instruction">
            Bonyeza "Washa Kamera" kuanza. Bonyeza "Piga Picha" kuchukua picha. 
            Bonyeza "Pakua Picha" kuhifadhi picha yako.
        </p>
        
        <div class="permission-msg" id="permissionMsg">
            Tafadhali ruhusu matumizi ya kamera. Hakikisha umebonyeza "Allow" unapoombwa ruhusa.
        </div>
        
        <div class="camera-container">
            <video id="cameraView" autoplay playsinline></video>
            <canvas id="canvas"></canvas>
        </div>
        
        <div class="preview-container">
            <img id="preview" alt="Picha iliyopigwa">
        </div>
        
        <div class="controls">
            <button id="startBtn" class="capture-btn">
                <span class="icon">🎥</span> Washa Kamera
            </button>
            
            <button id="captureBtn" class="capture-btn" disabled>
                <span class="icon">📸</span> Piga Picha
            </button>
            
            <button id="retakeBtn" class="retake-btn" disabled>
                <span class="icon">🔄</span> Piga Tena
            </button>
            
            <button id="downloadBtn" class="download-btn" disabled>
                <span class="icon">⬇️</span> Pakua Picha
            </button>
        </div>
    </div>

    <script>
        // Vitu muhimu
        const video = document.getElementById('cameraView');
        const canvas = document.getElementById('canvas');
        const preview = document.getElementById('preview');
        const startBtn = document.getElementById('startBtn');
        const captureBtn = document.getElementById('captureBtn');
        const retakeBtn = document.getElementById('retakeBtn');
        const downloadBtn = document.getElementById('downloadBtn');
        const permissionMsg = document.getElementById('permissionMsg');
        
        let stream = null;
        let capturedImage = null;

        // 1. Washa kamera
        startBtn.addEventListener('click', async () => {
            try {
                // Omba ruhusa ya kamera
                stream = await navigator.mediaDevices.getUserMedia({ 
                    video: { 
                        facingMode: 'user', // Tumia kamera ya mbele
                        width: { ideal: 1280 },
                        height: { ideal: 720 }
                    }, 
                    audio: false 
                });
                
                video.srcObject = stream;
                permissionMsg.style.display = 'none';
                
                // Wezesha vitufe
                captureBtn.disabled = false;
                startBtn.disabled = true;
                startBtn.innerHTML = '<span class="icon">✅</span> Kamera Imewashwa';
                
            } catch (error) {
                console.error('Hitilafu ya kamera:', error);
                permissionMsg.style.display = 'block';
                permissionMsg.innerHTML = `
                    Hitilafu: ${error.name}<br>
                    Tafadhali hakikisha:<br>
                    1. Umeruhusu matumizi ya kamera<br>
                    2. Hakuna programu nyingine inayotumia kamera<br>
                    3. Kifaa chako kinaunga mkono kamera ya wavuti
                `;
            }
        });

        // 2. Piga picha
        captureBtn.addEventListener('click', () => {
            // Weka ukubwa wa canvas sawa na video
            canvas.width = video.videoWidth;
            canvas.height = video.videoHeight;
            
            // Chora picha kwenye canvas
            const context = canvas.getContext('2d');
            context.drawImage(video, 0, 0, canvas.width, canvas.height);
            
            // Pata picha kwenye muundo wa Data URL
            capturedImage = canvas.toDataURL('image/png');
            
            // Onyesha picha iliyopigwa
            preview.src = capturedImage;
            preview.style.display = 'block';
            
            // Ficha video
            video.style.display = 'none';
            
            // Wezesha vitufe vya kupakua na kuchukua tena
            downloadBtn.disabled = false;
            retakeBtn.disabled = false;
            captureBtn.disabled = true;
            
            // Zima kamera ili kuhifadhi nishati
            if (stream) {
                stream.getTracks().forEach(track => track.stop());
            }
        });

        // 3. Piga picha tena
        retakeBtn.addEventListener('click', () => {
            // Onyesha video tena
            preview.style.display = 'none';
            video.style.display = 'block';
            
            // Zima vitufe
            downloadBtn.disabled = true;
            retakeBtn.disabled = true;
            captureBtn.disabled = false;
            
            // Washa kamera tena
            startBtn.click();
        });

        // 4. Pakua picha
        downloadBtn.addEventListener('click', () => {
            if (!capturedImage) return;
            
            // Unda jina la faili la kipekee
            const timestamp = new Date().toISOString().replace(/[:.]/g, '-');
            const filename = `picha-${timestamp}.png`;
            
            // Unda kiungo cha kupakulia
            const link = document.createElement('a');
            link.href = capturedImage;
            link.download = filename;
            link.click();
            
            // Onyesha ujumbe wa mafanikio
            alert(`Picha imepakuliwa kama: ${filename}`);
        });

        // 5. Safisha (zima kamera unapoondoka)
        window.addEventListener('beforeunload', () => {
            if (stream) {
                stream.getTracks().forEach(track => track.stop());
            }
        });

        // 6. Kukagua ikiwa kifaa kinaunga mkono kamera
        window.addEventListener('load', () => {
            if (!navigator.mediaDevices || !navigator.mediaDevices.getUserMedia) {
                permissionMsg.style.display = 'block';
                permissionMsg.innerHTML = 'Kifaa chako hakiungii mkono kamera ya wavuti. Tafadhali tumia kifaa kipya.';
                startBtn.disabled = true;
            }
        });
    </script>
</body>
</html># Nyaya
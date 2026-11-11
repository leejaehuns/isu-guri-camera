<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>워터마크 카메라</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- FontAwesome Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <!-- Google Font (Noto Sans KR) -->
    <link href="https://fonts.googleapis.com/css2?family=Noto+Sans+KR:wght@400;500;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Noto Sans KR', sans-serif;
            background-color: #000000;
            color: #ffffff;
            touch-action: manipulation;
        }
        /* Hide scrollbars for app-like feel */
        ::-webkit-scrollbar {
            display: none;
        }
        /* Mobile viewport height fix */
        .h-screen-ios {
            height: 100vh;
            height: -webkit-fill-available;
        }
    </style>
</head>
<body class="bg-black text-white h-screen-ios flex flex-col justify-between overflow-hidden select-none">

    <!-- Header / Top Bar -->
    <header class="relative z-20 px-4 py-3 bg-neutral-900/80 backdrop-blur-md flex items-center justify-between border-b border-neutral-800">
        <div class="flex items-center gap-2">
            <i class="fa-solid font-bold fa-camera-retro text-amber-400 text-xl"></i>
            <h1 class="font-bold text-lg tracking-wide text-gray-100">워터마크 카메라</h1>
        </div>
        <button id="switchCamBtn" class="p-2 rounded-full bg-neutral-800 hover:bg-neutral-700 active:scale-95 transition text-gray-300 hover:text-white" title="카메라 전환">
            <i class="fa-solid fa-rotate-left text-lg"></i>
        </button>
    </header>

    <!-- Main Viewfinder Section -->
    <main class="relative flex-1 bg-black flex items-center justify-center overflow-hidden">
        
        <!-- Video Element (Hidden or layered beneath Canvas overlay) -->
        <video id="webcam" autoplay playsinline class="hidden"></video>

        <!-- Live Viewfinder Canvas -->
        <canvas id="viewfinderCanvas" class="w-full h-full object-contain"></canvas>

        <!-- Loading / Camera Off State Overlay -->
        <div id="cameraPlaceholder" class="absolute inset-0 flex flex-col items-center justify-center bg-neutral-950 gap-3 z-10">
            <div class="animate-spin rounded-full h-12 w-12 border-4 border-amber-400 border-t-transparent"></div>
            <p id="statusText" class="text-sm text-neutral-400 font-medium">카메라 권한을 확인하는 중...</p>
        </div>

        <!-- Preview Modal / Screen after taking photo -->
        <div id="previewContainer" class="absolute inset-0 bg-black z-30 hidden flex flex-col">
            <div class="relative flex-1 flex items-center justify-center overflow-hidden p-2">
                <img id="capturedImage" class="max-w-full max-h-full object-contain rounded-lg shadow-2xl" alt="촬영된 이미지">
            </div>
            <!-- Preview Action Bar -->
            <div class="p-4 bg-neutral-900/90 backdrop-blur-md flex items-center justify-around gap-4 border-t border-neutral-800">
                <button id="retakeBtn" class="flex-1 py-3 px-4 rounded-xl bg-neutral-800 hover:bg-neutral-700 text-white font-semibold flex items-center justify-center gap-2 active:scale-95 transition">
                    <i class="fa-solid fa-arrows-rotate"></i>
                    <span>다시 촬영</span>
                </button>
                <button id="downloadBtn" class="flex-1 py-3 px-4 rounded-xl bg-amber-500 hover:bg-amber-400 text-neutral-950 font-bold flex items-center justify-center gap-2 active:scale-95 transition shadow-lg shadow-amber-500/20">
                    <i class="fa-solid fa-download"></i>
                    <span>사진 저장</span>
                </button>
            </div>
        </div>

    </main>

    <!-- Watermark Inputs & Capture Controls Section -->
    <footer class="relative z-20 bg-neutral-900 border-t border-neutral-800 flex flex-col gap-3 p-4">
        
        <!-- Input Fields Panel (Date, Dong, Floor) -->
        <div class="grid grid-cols-3 gap-2">
            <!-- Date Input -->
            <div class="flex flex-col gap-1">
                <label class="text-[11px] text-neutral-400 font-medium px-1 flex items-center gap-1">
                    <i class="fa-regular fa-calendar text-amber-400"></i> 날짜
                </label>
                <input type="date" id="inputDate" 
                    class="w-full bg-neutral-800 border border-neutral-700 focus:border-amber-400 focus:outline-none rounded-lg px-2.5 py-1.5 text-xs text-white transition text-center font-mono">
            </div>
            
            <!-- Dong Input -->
            <div class="flex flex-col gap-1">
                <label class="text-[11px] text-neutral-400 font-medium px-1 flex items-center gap-1">
                    <i class="fa-solid fa-building text-amber-400"></i> 동
                </label>
                <input type="text" id="inputDong" placeholder="예: 101동" 
                    class="w-full bg-neutral-800 border border-neutral-700 focus:border-amber-400 focus:outline-none rounded-lg px-2.5 py-1.5 text-xs text-white placeholder-neutral-500 transition text-center">
            </div>

            <!-- Floor Input -->
            <div class="flex flex-col gap-1">
                <label class="text-[11px] text-neutral-400 font-medium px-1 flex items-center gap-1">
                    <i class="fa-solid fa-layer-group text-amber-400"></i> 층
                </label>
                <input type="text" id="inputFloor" placeholder="예: 5층" 
                    class="w-full bg-neutral-800 border border-neutral-700 focus:border-amber-400 focus:outline-none rounded-lg px-2.5 py-1.5 text-xs text-white placeholder-neutral-500 transition text-center">
            </div>
        </div>

        <!-- Shutter / Capture Button -->
        <div class="flex items-center justify-center pt-1 pb-2">
            <button id="captureBtn" class="group relative flex items-center justify-center w-16 h-16 rounded-full bg-neutral-800 border-4 border-amber-400 active:scale-90 transition duration-150 shadow-lg shadow-amber-400/10">
                <span class="w-12 h-12 rounded-full bg-amber-400 group-hover:bg-amber-300 transition"></span>
            </button>
        </div>
    </footer>

    <!-- Notification Toast -->
    <div id="toast" class="fixed top-16 left-1/2 -translate-x-1/2 bg-neutral-800/95 border border-neutral-700 text-white text-xs px-4 py-2 rounded-full shadow-lg opacity-0 transition-opacity duration-300 pointer-events-none z-50 flex items-center gap-2">
        <i class="fa-solid fa-circle-info text-amber-400"></i>
        <span id="toastMsg">메시지</span>
    </div>

    <!-- Application Script -->
    <script>
        // DOM Elements
        const webcam = document.getElementById('webcam');
        const canvas = document.getElementById('viewfinderCanvas');
        const ctx = canvas.getContext('2d');
        const cameraPlaceholder = document.getElementById('cameraPlaceholder');
        const statusText = document.getElementById('statusText');
        
        const inputDate = document.getElementById('inputDate');
        const inputDong = document.getElementById('inputDong');
        const inputFloor = document.getElementById('inputFloor');

        const captureBtn = document.getElementById('captureBtn');
        const switchCamBtn = document.getElementById('switchCamBtn');
        
        const previewContainer = document.getElementById('previewContainer');
        const capturedImage = document.getElementById('capturedImage');
        const retakeBtn = document.getElementById('retakeBtn');
        const downloadBtn = document.getElementById('downloadBtn');
        const toast = document.getElementById('toast');
        const toastMsg = document.getElementById('toastMsg');

        // State Variables
        let currentStream = null;
        let facingMode = 'environment'; // Default: Rear camera on mobiles
        let isCameraReady = false;
        let animationFrameId = null;

        // Set default date to today (YYYY-MM-DD)
        function initDefaultDate() {
            const today = new Date();
            const year = today.getFullYear();
            const month = String(today.getMonth() + 1).padStart(2, '0');
            const day = String(today.getDate()).padStart(2, '0');
            inputDate.value = `${year}-${month}-${day}`;
        }

        // Show Toast Message
        function showToast(message) {
            toastMsg.textContent = message;
            toast.classList.remove('opacity-0');
            toast.classList.add('opacity-100');
            setTimeout(() => {
                toast.classList.remove('opacity-100');
                toast.classList.add('opacity-0');
            }, 2500);
        }

        // Start Web Camera Stream
        async function startCamera() {
            isCameraReady = false;
            cameraPlaceholder.classList.remove('hidden');
            statusText.textContent = "카메라를 연결하는 중...";

            if (currentStream) {
                currentStream.getTracks().forEach(track => track.stop());
            }

            const constraints = {
                video: {
                    facingMode: facingMode,
                    width: { ideal: 1920 },
                    height: { ideal: 1080 }
                },
                audio: false
            };

            try {
                currentStream = await navigator.mediaDevices.getUserMedia(constraints);
                webcam.srcObject = currentStream;

                webcam.onloadedmetadata = () => {
                    webcam.play();
                    isCameraReady = true;
                    cameraPlaceholder.classList.add('hidden');
                    renderViewfinder();
                };
            } catch (err) {
                console.error("Camera access error:", err);
                // Fallback attempt without facing mode constraint
                try {
                    currentStream = await navigator.mediaDevices.getUserMedia({ video: true, audio: false });
                    webcam.srcObject = currentStream;
                    webcam.onloadedmetadata = () => {
                        webcam.play();
                        isCameraReady = true;
                        cameraPlaceholder.classList.add('hidden');
                        renderViewfinder();
                    };
                } catch (fallbackErr) {
                    statusText.textContent = "카메라에 접근할 수 없습니다. 권한을 허용해주세요.";
                    showToast("카메라 권한을 확인해주세요.");
                }
            }
        }

        // Build Watermark Text Array from Inputs
        function getWatermarkText() {
            const dateVal = inputDate.value.trim();
            const dongVal = inputDong.value.trim();
            const floorVal = inputFloor.value.trim();

            const parts = [];
            if (dateVal) parts.push(dateVal);
            if (dongVal) parts.push(dongVal);
            if (floorVal) parts.push(floorVal);

            return parts.join(' | ');
        }

        // Draw Watermark on Target Canvas Context
        function drawWatermark(targetCtx, width, height) {
            const text = getWatermarkText();
            if (!text) return; // Skip if all fields are empty

            // Scale font size dynamically relative to image dimensions
            const baseSize = Math.max(width, height);
            const fontSize = Math.max(16, Math.round(baseSize * 0.028));
            
            targetCtx.font = `500 ${fontSize}px 'Noto Sans KR', sans-serif`;

            // Calculate Text & Badge Padding
            const metrics = targetCtx.measureText(text);
            const textWidth = metrics.width;
            const textHeight = fontSize;

            const paddingX = fontSize * 0.75;
            const paddingY = fontSize * 0.5;
            const margin = fontSize * 0.8; // Distance from bottom-left corner

            // Watermark position: Left-Bottom
            const rectWidth = textWidth + (paddingX * 2);
            const rectHeight = textHeight + (paddingY * 2);
            const rectX = margin;
            const rectY = height - margin - rectHeight;

            // Draw Semi-transparent Dark Background Box
            targetCtx.save();
            targetCtx.fillStyle = 'rgba(0, 0, 0, 0.65)';
            
            // Rounded Rectangle Box
            const radius = Math.min(8, fontSize * 0.3);
            targetCtx.beginPath();
            targetCtx.moveTo(rectX + radius, rectY);
            targetCtx.lineTo(rectX + rectWidth - radius, rectY);
            targetCtx.quadraticCurveTo(rectX + rectWidth, rectY, rectX + rectWidth, rectY + radius);
            targetCtx.lineTo(rectX + rectWidth, rectY + rectHeight - radius);
            targetCtx.quadraticCurveTo(rectX + rectWidth, rectY + rectHeight, rectX + rectWidth - radius, rectY + rectHeight);
            targetCtx.lineTo(rectX + radius, rectY + rectHeight);
            targetCtx.quadraticCurveTo(rectX, rectY + rectHeight, rectX, rectY + rectHeight - radius);
            targetCtx.lineTo(rectX, rectY + radius);
            targetCtx.quadraticCurveTo(rectX, rectY, rectX + radius, rectY);
            targetCtx.closePath();
            targetCtx.fill();

            // Draw White Text Inside Box
            targetCtx.fillStyle = '#FFFFFF';
            targetCtx.textBaseline = 'middle';
            targetCtx.fillText(text, rectX + paddingX, rectY + (rectHeight / 2) + (fontSize * 0.05));

            targetCtx.restore();
        }

        // Live Viewfinder Loop
        function renderViewfinder() {
            if (!isCameraReady) return;

            // Adjust Canvas resolution to match actual video dimensions
            if (canvas.width !== webcam.videoWidth || canvas.height !== webcam.videoHeight) {
                canvas.width = webcam.videoWidth;
                canvas.height = webcam.videoHeight;
            }

            // Draw current frame from Video
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            ctx.drawImage(webcam, 0, 0, canvas.width, canvas.height);

            // Overlay Watermark at Left-Bottom
            drawWatermark(ctx, canvas.width, canvas.height);

            // Request next frame
            animationFrameId = requestAnimationFrame(renderViewfinder);
        }

        // Capture High-Res Image with Watermark
        function capturePhoto() {
            if (!isCameraReady || !webcam.videoWidth) {
                showToast("카메라 준비 중입니다. 잠시 후 시도해주세요.");
                return;
            }

            // Create offscreen high-res canvas
            const offscreenCanvas = document.createElement('canvas');
            offscreenCanvas.width = webcam.videoWidth;
            offscreenCanvas.height = webcam.videoHeight;
            const offCtx = offscreenCanvas.getContext('2d');

            // Draw Full Resolution Frame
            offCtx.drawImage(webcam, 0, 0, offscreenCanvas.width, offscreenCanvas.height);

            // Overlay Watermark
            drawWatermark(offCtx, offscreenCanvas.width, offscreenCanvas.height);

            // Convert Canvas to Data URL and Display in Preview
            const dataUrl = offscreenCanvas.toDataURL('image/jpeg', 0.95);
            capturedImage.src = dataUrl;
            
            // Show Preview UI
            previewContainer.classList.remove('hidden');
        }

        // Download Captured Image
        function downloadPhoto() {
            if (!capturedImage.src) return;

            const now = new Date();
            const timestamp = `${now.getFullYear()}${String(now.getMonth()+1).padStart(2,'0')}${String(now.getDate()).padStart(2,'0')}_${String(now.getHours()).padStart(2,'0')}${String(now.getMinutes()).padStart(2,'0')}${String(now.getSeconds()).padStart(2,'0')}`;
            const fileName = `watermark_${timestamp}.jpg`;

            const link = document.createElement('a');
            link.href = capturedImage.src;
            link.download = fileName;
            document.body.appendChild(link);
            link.click();
            document.body.removeChild(link);

            showToast("사진이 저장되었습니다.");
        }

        // Switch Camera (Front <-> Rear)
        function toggleCamera() {
            facingMode = (facingMode === 'user') ? 'environment' : 'user';
            if (animationFrameId) {
                cancelAnimationFrame(animationFrameId);
            }
            startCamera();
        }

        // Event Listeners
        captureBtn.addEventListener('click', capturePhoto);

        switchCamBtn.addEventListener('click', toggleCamera);

        retakeBtn.addEventListener('click', () => {
            previewContainer.classList.add('hidden');
            capturedImage.src = '';
        });

        downloadBtn.addEventListener('click', downloadPhoto);

        // Initialize App
        window.addEventListener('load', () => {
            initDefaultDate();
            startCamera();
        });

        // Handle window resize dynamically
        window.addEventListener('resize', () => {
            if (isCameraReady && webcam.videoWidth) {
                canvas.width = webcam.videoWidth;
                canvas.height = webcam.videoHeight;
            }
        });
    </script>
</body>
</html>

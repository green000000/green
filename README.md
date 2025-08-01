<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>화이트보드 (분필 사운드)</title>
    <style>
        body {
            margin: 0;
            display: flex;
            flex-direction: column;
            align-items: center;
            background: #f0f0f0;
            font-family: sans-serif;
            user-select: none;
        }
        .toolbar {
            display: flex;
            flex-wrap: wrap;
            gap: 10px;
            margin: 10px;
        }
        canvas {
            background: white;
            border: 2px solid #ccc;
            touch-action: none;
            image-rendering: crisp-edges;
        }
    </style>
</head>
<body>
    <div class="toolbar">
        <input type="color" id="colorPicker" value="#000000">
        <input type="range" id="thickness" min="1" max="20" value="3">
        <button id="eraser">지우개</button>
        <button id="clear">전체 지우기</button>
        <button id="save">저장</button>
    </div>

    <canvas id="board" width="800" height="500"></canvas>
    <audio id="chalkSound" src="zapsplat_office_chalk_stick_thick_set_down_on_small_chalk_board_002_34617.mp3"></audio>

    <script>
        const canvas = document.getElementById('board');
        const ctx = canvas.getContext('2d');
        const sound = document.getElementById('chalkSound');
        const colorPicker = document.getElementById('colorPicker');
        const thickness = document.getElementById('thickness');
        const eraserBtn = document.getElementById('eraser');
        const clearBtn = document.getElementById('clear');
        const saveBtn = document.getElementById('save');

        let drawing = false;
        let erasing = false;

        ctx.lineCap = 'round';
        ctx.lineJoin = 'round';

        function getX(e) {
            return e.touches ? e.touches[0].clientX - canvas.offsetLeft : e.clientX - canvas.offsetLeft;
        }

        function getY(e) {
            return e.touches ? e.touches[0].clientY - canvas.offsetTop : e.clientY - canvas.offsetTop;
        }

        function startDrawing(e) {
            drawing = true;
            ctx.beginPath();
            ctx.moveTo(getX(e), getY(e));
            playSound();
        }

        function draw(e) {
            if (!drawing) return;
            ctx.strokeStyle = erasing ? 'white' : colorPicker.value;
            ctx.lineWidth = thickness.value;
            ctx.lineTo(getX(e), getY(e));
            ctx.stroke();
        }

        function stopDrawing() {
            drawing = false;
            sound.pause();
            sound.currentTime = 0;
        }

        function playSound() {
            if (sound.paused) {
                sound.loop = true;
                sound.play().catch(() => {});
            }
        }

        function setupEventListeners() {
            canvas.addEventListener('mousedown', startDrawing);
            canvas.addEventListener('mousemove', draw);
            canvas.addEventListener('mouseup', stopDrawing);
            canvas.addEventListener('mouseleave', stopDrawing);

            canvas.addEventListener('touchstart', startDrawing);
            canvas.addEventListener('touchmove', (e) => {
                e.preventDefault();
                draw(e);
            }, { passive: false });
            canvas.addEventListener('touchend', stopDrawing);
        }

        setupEventListeners();

        eraserBtn.addEventListener('click', () => {
            erasing = !erasing;
            eraserBtn.textContent = erasing ? '펜' : '지우개';
        });

        clearBtn.addEventListener('click', () => {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
        });

        saveBtn.addEventListener('click', () => {
            const link = document.createElement('a');
            link.download = 'whiteboard.png';
            link.href = canvas.toDataURL('image/png');
            link.click();
        });

        // 사운드 재생 권한 확보 (모바일 브라우저용)
        document.body.addEventListener('touchstart', () => {
            sound.play().then(() => sound.pause());
            sound.currentTime = 0;
        }, { once: true });
    </script>
</body>
</html>

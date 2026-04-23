# project2
<!DOCTYPE html>
<html lang="uz">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SpeedType Pro | Test</title>
    <style>
        :root {
            --primary: #6366f1;
            --success: #10b981;
            --danger: #ef4444;
            --bg: #0f172a;
        }

        * { margin: 0; padding: 0; box-sizing: border-box; font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; }

        body {
            background: var(--bg);
            color: #f8fafc;
            height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
            padding: 20px;
        }

        .container {
            background: rgba(30, 41, 59, 0.7);
            backdrop-filter: blur(10px);
            padding: 40px;
            border-radius: 24px;
            width: 100%;
            max-width: 800px;
            border: 1px solid rgba(255,255,255,0.1);
            box-shadow: 0 25px 50px -12px rgba(0,0,0,0.5);
            text-align: center;
        }

        h2 { margin-bottom: 20px; color: var(--primary); font-size: 28px; }

        .text-display {
            background: rgba(15, 23, 42, 0.5);
            padding: 25px;
            border-radius: 16px;
            margin-bottom: 25px;
            font-size: 22px;
            line-height: 1.6;
            text-align: left;
            border: 1px solid rgba(255,255,255,0.05);
            letter-spacing: 0.5px;
            min-height: 100px;
        }

        textarea {
            width: 100%;
            height: 120px;
            background: rgba(0,0,0,0.2);
            color: white;
            border: 2px solid #334155;
            border-radius: 16px;
            padding: 20px;
            font-size: 18px;
            outline: none;
            resize: none;
            transition: 0.3s;
        }

        textarea:focus { border-color: var(--primary); box-shadow: 0 0 15px rgba(99, 102, 241, 0.3); }

        .dashboard {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 20px;
            margin-top: 30px;
        }

        .stat-card {
            background: rgba(255,255,255,0.03);
            padding: 15px;
            border-radius: 12px;
            border: 1px solid rgba(255,255,255,0.05);
        }

        .stat-card span { display: block; font-size: 24px; font-weight: bold; color: var(--primary); }
        .stat-card p { font-size: 12px; opacity: 0.6; text-transform: uppercase; margin-top: 5px; }

        #resultPanel {
            display: none;
            margin-top: 30px;
            padding: 20px;
            background: var(--success);
            color: white;
            border-radius: 16px;
            animation: slideUp 0.5s ease;
        }

        @keyframes slideUp { from { opacity: 0; transform: translateY(20px); } to { opacity: 1; transform: translateY(0); } }

        .btn {
            margin-top: 25px;
            padding: 12px 30px;
            background: var(--primary);
            border: none;
            color: white;
            border-radius: 10px;
            cursor: pointer;
            font-weight: bold;
            transition: 0.3s;
        }

        .btn:hover { filter: brightness(1.2); transform: translateY(-2px); }
        
        .char-correct { color: var(--success); }
        .char-wrong { background: var(--danger); color: white; border-radius: 4px; }
    </style>
</head>
<body>

    <div class="container">
        <h2>SpeedType Pro</h2>
        
        <div class="text-display" id="quoteDisplay">Yuklanmoqda...</div>
        
        <textarea id="quoteInput" placeholder="Matnni terishni boshlang..."></textarea>

        <div class="dashboard">
            <div class="stat-card">

> Ⓔ︎Ⓛ︎Ⓑ︎Ⓔ︎Ⓚ︎:
<span id="timer">0</span>
                <p>Vaqt (sek)</p>
            </div>
            <div class="stat-card">
                <span id="wpm">0</span>
                <p>Tezlik (WPM)</p>
            </div>
            <div class="stat-card">
                <span id="errors">0</span>
                <p>Xatolar</p>
            </div>
        </div>

        <div id="resultPanel">
            <h3>Ajoyib natija! 🎉</h3>
            <p id="finalStats"></p>
        </div>

        <button class="btn" onclick="initGame()">Yangi matn / Qayta boshlash</button>
    </div>

    <script>
        const quotes = [
            "Muvaffaqiyat kaliti tinimsiz harakat va o'ziga bo'lgan ishonchdadir.",
            "Dasturlash - bu kelajak tili, uni o'rganish sizga cheksiz imkoniyatlar ochadi.",
            "Vaqt eng qimmatli boylik, uni foydali ishlarga sarflash kerak.",
            "Har bir katta muvaffaqiyat kichik qadamlardan boshlanadi.",
            "Bilim - bu dunyoni o'zgartirish uchun eng kuchli qurol.",
            "O'rganishdan to'xtagan inson qarigan hisoblanadi, xoh u yigirmada bo'lsin, xoh saksonda.",
            "Eng yaxshi dam - bu mehnatdan keyingi oromdir."
        ];

        const quoteDisplay = document.getElementById('quoteDisplay');
        const quoteInput = document.getElementById('quoteInput');
        const timerDisplay = document.getElementById('timer');
        const wpmDisplay = document.getElementById('wpm');
        const errorDisplay = document.getElementById('errors');
        const resultPanel = document.getElementById('resultPanel');
        const finalStats = document.getElementById('finalStats');

        let timeLeft = 0;
        let timerInterval;
        let isStarted = false;

        function initGame() {
            clearInterval(timerInterval);
            isStarted = false;
            timeLeft = 0;
            timerDisplay.innerText = 0;
            wpmDisplay.innerText = 0;
            errorDisplay.innerText = 0;
            quoteInput.value = "";
            quoteInput.disabled = false;
            resultPanel.style.display = "none";
            
            // Tasodifiy matn tanlash (avvalgisidan farqli bo'lishi uchun)
            const currentText = quoteDisplay.innerText;
            let newQuote = quotes[Math.floor(Math.random() * quotes.length)];
            while(newQuote === currentText && quotes.length > 1) {
                newQuote = quotes[Math.floor(Math.random() * quotes.length)];
            }
            
            quoteDisplay.innerHTML = '';
            newQuote.split('').forEach(char => {
                const charSpan = document.createElement('span');
                charSpan.innerText = char;
                quoteDisplay.appendChild(charSpan);
            });
            quoteInput.focus();
        }

        quoteInput.addEventListener('input', () => {
            if(!isStarted && quoteInput.value.length > 0) {
                isStarted = true;
                timerInterval = setInterval(() => {
                    timeLeft++;
                    timerDisplay.innerText = timeLeft;
                }, 1000);
            }

            const arrayQuote = quoteDisplay.querySelectorAll('span');
            const arrayValue = quoteInput.value.split('');
            let errors = 0;
            let charactersTyped = arrayValue.length;

            arrayQuote.forEach((characterSpan, index) => {
                const character = arrayValue[index];
                if (character == null) {
                    characterSpan.classList.remove('char-correct');
                    characterSpan.classList.remove('char-wrong');
                } else if (character === characterSpan.innerText) {
                    characterSpan.classList.add('char-correct');
                    characterSpan.classList.remove('char-wrong');
                } else {
                    characterSpan.classList.remove('char-correct');
                    characterSpan.classList.add('char-wrong');
                    errors++;
                }
            });

> Ⓔ︎Ⓛ︎Ⓑ︎Ⓔ︎Ⓚ︎:
errorDisplay.innerText = errors;

            // Agar hamma harf terilgan bo'lsa
            if (charactersTyped >= arrayQuote.length) {
                finishGame();
            }
        });

        function finishGame() {
            clearInterval(timerInterval);
            quoteInput.disabled = true;
            
            const text = quoteDisplay.innerText;
            const wordCount = text.split(/\s+/).length;
            const wpm = Math.round((wordCount / (timeLeft / 60))) || 0;
            
            wpmDisplay.innerText = wpm;
            resultPanel.style.display = "block";
            finalStats.innerText = Siz ${timeLeft} soniyada ${wpm} WPM tezlik bilan yozdingiz. Jami xatolar: ${errorDisplay.innerText} ta.;
        }

        // Birinchi yuklashda ishga tushirish
        initGame();
    </script>
</body>
</html>

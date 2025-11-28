<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>شطرنج برو | Chess Pro</title>
    <style>
        /* --- CSS: التصميم والمظهر --- */
        :root {
            --primary-color: #2c3e50;
            --secondary-color: #e67e22;
            --accent-color: #27ae60;
            --bg-color: #1a1a1a;
            --board-light: #f0d9b5;
            --board-dark: #b58863;
            --text-color: #ecf0f1;
            --font-main: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }

        body {
            margin: 0;
            font-family: var(--font-main);
            background-color: var(--bg-color);
            color: var(--text-color);
            display: flex;
            flex-direction: column;
            align-items: center;
            min-height: 100vh;
            overflow-x: hidden;
        }

        /* شاشات التحميل والتنقل */
        .screen {
            display: none;
            width: 100%;
            max-width: 600px;
            padding: 20px;
            box-sizing: border-box;
            text-align: center;
            animation: fadeIn 0.5s;
        }

        .screen.active {
            display: block;
        }

        @keyframes fadeIn {
            from { opacity: 0; }
            to { opacity: 1; }
        }

        h1, h2 { color: var(--secondary-color); text-shadow: 2px 2px 4px #000; }
        
        button {
            background: linear-gradient(45deg, var(--secondary-color), #d35400);
            border: none;
            padding: 12px 24px;
            color: white;
            font-size: 16px;
            cursor: pointer;
            border-radius: 8px;
            margin: 10px;
            transition: transform 0.2s, box-shadow 0.2s;
            font-weight: bold;
        }

        button:hover {
            transform: scale(1.05);
            box-shadow: 0 0 15px rgba(230, 126, 34, 0.6);
        }

        input, select {
            padding: 10px;
            margin: 10px;
            border-radius: 5px;
            border: 1px solid #555;
            background: #333;
            color: white;
            width: 80%;
        }

        /* رقعة الشطرنج */
        #game-container {
            display: flex;
            flex-direction: column;
            align-items: center;
        }

        #chessboard {
            display: grid;
            grid-template-columns: repeat(8, 1fr);
            width: 100%;
            max-width: 400px;
            aspect-ratio: 1;
            border: 5px solid #4a3021;
            user-select: none;
        }

        .square {
            width: 100%;
            height: 100%;
            display: flex;
            justify-content: center;
            align-items: center;
            font-size: 30px;
            cursor: pointer;
            position: relative;
        }

        .light { background-color: var(--board-light); color: black; }
        .dark { background-color: var(--board-dark); color: black; }
        
        .selected { background-color: rgba(255, 255, 0, 0.5) !important; }
        .possible-move::after {
            content: '';
            width: 15px;
            height: 15px;
            background: rgba(0, 0, 0, 0.3);
            border-radius: 50%;
            position: absolute;
        }

        /* معلومات اللعب */
        .player-info {
            display: flex;
            justify-content: space-between;
            width: 100%;
            max-width: 400px;
            margin: 10px 0;
            background: #333;
            padding: 10px;
            border-radius: 8px;
        }

        /* المتجر والعملات */
        .currency-display {
            position: fixed;
            top: 10px;
            left: 10px;
            background: rgba(0,0,0,0.7);
            padding: 5px 10px;
            border-radius: 15px;
            color: gold;
            font-weight: bold;
            z-index: 1000;
        }

        .shop-item {
            background: #333;
            border: 1px solid #555;
            padding: 10px;
            margin: 10px 0;
            border-radius: 8px;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }

        /* الرموز التعبيرية */
        .emote-panel {
            display: flex;
            gap: 10px;
            margin-top: 10px;
        }
        .emote-btn {
            background: none;
            font-size: 20px;
            padding: 5px;
            margin: 0;
        }

        /* التنبيهات */
        .modal {
            position: fixed;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background: #222;
            padding: 20px;
            border: 2px solid var(--secondary-color);
            z-index: 2000;
            display: none;
            box-shadow: 0 0 20px rgba(0,0,0,0.8);
            width: 80%;
            max-width: 400px;
        }

        /* سياسة الخصوصية */
        .legal-text {
            font-size: 12px;
            color: #aaa;
            text-align: justify;
            max-height: 100px;
            overflow-y: scroll;
            background: #222;
            padding: 5px;
            border: 1px solid #444;
            margin-bottom: 10px;
        }

        @media (max-width: 600px) {
            #chessboard { max-width: 95vw; }
            .square { font-size: 24px; }
        }
    </style>
</head>
<body>

    <div class="currency-display">
        💰 <span id="coin-balance">0</span> | 💎 <span id="gem-balance">0</span>
    </div>

    <div id="screen-login" class="screen active">
        <h1>شطرنج برو</h1>
        <h2>Chess Pro</h2>
        <p>الاحترافية، المتعة، والتحدي</p>
        
        <input type="text" id="username" placeholder="اسم المستخدم (Username)">
        <input type="email" id="email" placeholder="البريد الإلكتروني (Email)">
        <input type="password" id="password" placeholder="كلمة المرور (Password)">
        
        <div class="legal-text">
            يجب الموافقة على شروط الاستخدام وسياسة الخصوصية. نحن نستخدم إجراءات أمان عالية لحماية بياناتك. يمنع استخدام الألفاظ النابية أو محاولة الاختراق تحت طائلة المسؤولية القانونية.
        </div>
        <label><input type="checkbox" id="terms-check"> أوافق على الشروط والسياسات</label>
        
        <br>
        <button onclick="app.login()">تسجيل الدخول / إنشاء حساب</button>
    </div>

    <div id="screen-menu" class="screen">
        <h2 id="welcome-msg">مرحباً</h2>
        <p>المستوى: <span id="player-level">1</span> (مبتدئ)</p>
        
        <button onclick="app.startGame('ai')">⚔️ لعب ضد الذكاء الاصطناعي</button>
        <button onclick="app.showScreen('screen-training')">🎓 التدريب والاستراتيجيات</button>
        <button onclick="app.showScreen('screen-shop')">🛒 المتجر (Shop)</button>
        <button onclick="app.showScreen('screen-settings')">⚙️ الإعدادات</button>
        <button onclick="app.showScreen('screen-profile')">👤 الملف الشخصي</button>
        
        <div style="margin-top: 20px; border-top: 1px solid #555; padding-top: 10px;">
            <p>🏆 الدوري الشهري | 📅 البطولة السنوية</p>
            <small style="color: #888;">الاشتراك البريميوم: 19$ / شهر</small>
        </div>
    </div>

    <div id="screen-game" class="screen">
        <div class="player-info">
            <span>🤖 الخصم (AI/Player)</span>
            <span id="timer-opponent">10:00</span>
        </div>
        
        <div id="game-container">
            <div id="chessboard"></div>
        </div>
        
        <div class="player-info">
            <span id="player-name-display">أنت</span>
            <span id="timer-player">10:00</span>
        </div>

        <div class="emote-panel">
            <button class="emote-btn" onclick="game.sendEmote('👏')">👏</button>
            <button class="emote-btn" onclick="game.sendEmote('🤔')">🤔</button>
            <button class="emote-btn" onclick="game.sendEmote('🔥')">🔥</button>
            <button class="emote-btn" onclick="game.sendEmote('🏳️')">🏳️</button>
        </div>
        
        <button onclick="app.showScreen('screen-menu')" style="background: #c0392b;">إنهاء المباراة</button>
    </div>

    <div id="screen-shop" class="screen">
        <h2>المتجر</h2>
        <div class="shop-item">
            <span>🎨 قطع شطرنج ذهبية</span>
            <button onclick="shop.buy(500, 'coins')">500 💰</button>
        </div>
        <div class="shop-item">
            <span>🌌 خلفية الفضاء</span>
            <button onclick="shop.buy(100, 'gems')">100 💎</button>
        </div>
        <div class="shop-item">
            <span>👑 اشتراك بريميوم (شهر)</span>
            <button onclick="shop.buyRealMoney(19)">$19.00</button>
        </div>
        <button onclick="app.showScreen('screen-menu')">عودة</button>
    </div>

    <div id="screen-settings" class="screen">
        <h2>الإعدادات</h2>
        <label>اللغة:
            <select id="lang-select" onchange="app.changeLang()">
                <option value="ar">العربية</option>
                <option value="en">English</option>
                <option value="fr">Français</option>
                <option value="es">Español</option>
                <option value="de">Deutsch</option>
                </select>
        </label>
        <br>
        <label>وقت المباراة:
            <select>
                <option>قصيرة (10د)</option>
                <option>متوسطة (20د)</option>
                <option>طويلة (30د)</option>
            </select>
        </label>
        <br>
        <button onclick="app.deleteAccount()" style="background:red">حذف الحساب</button>
        <button onclick="app.showScreen('screen-menu')">عودة</button>
    </div>

    <div id="screen-training" class="screen">
        <h2>مركز التدريب</h2>
        <p>تعلم الاستراتيجيات الأساسية وافتتاحيات اللعب.</p>
        <div style="text-align: right; padding: 10px;">
            <h3>1. السيطرة على المنتصف</h3>
            <p>حاول وضع البيادق في المربعات e4 و d4.</p>
            <h3>2. حماية الملك</h3>
            <p>قم بالتبييت مبكراً.</p>
        </div>
        <button onclick="app.showScreen('screen-menu')">تم</button>
    </div>

    <div id="modal-popup" class="modal">
        <h3 id="modal-title">تنبيه</h3>
        <p id="modal-msg">رسالة</p>
        <button onclick="document.getElementById('modal-popup').style.display='none'">حسناً</button>
    </div>

    <script>
        /* --- JavaScript: المنطق والتحكم --- */
        
        // كائن التطبيق الرئيسي لإدارة الحالة
        const app = {
            user: {
                name: "Guest",
                coins: 100,
                gems: 10,
                level: 1,
                xp: 0
            },
            
            init: function() {
                this.loadData();
                this.updateCurrencyUI();
            },

            loadData: function() {
                const saved = localStorage.getItem('chessProUser');
                if (saved) {
                    this.user = JSON.parse(saved);
                }
            },

            saveData: function() {
                localStorage.setItem('chessProUser', JSON.stringify(this.user));
                this.updateCurrencyUI();
            },

            updateCurrencyUI: function() {
                document.getElementById('coin-balance').innerText = this.user.coins;
                document.getElementById('gem-balance').innerText = this.user.gems;
                document.getElementById('player-level').innerText = this.user.level;
            },

            showScreen: function(screenId) {
                document.querySelectorAll('.screen').forEach(s => s.classList.remove('active'));
                document.getElementById(screenId).classList.add('active');
            },

            login: function() {
                const name = document.getElementById('username').value;
                const terms = document.getElementById('terms-check').checked;
                
                // فلتر الأسماء المسيئة (محاكاة بسيطة)
                const badWords = ["bad", "ugly", "شتيمة"];
                if (badWords.some(w => name.includes(w))) {
                    this.showModal("خطأ", "هذا الاسم غير مسموح به.");
                    return;
                }

                if (!name || !terms) {
                    this.showModal("تنبيه", "الرجاء إدخال الاسم والموافقة على الشروط.");
                    return;
                }

                this.user.name = name;
                this.saveData();
                document.getElementById('welcome-msg').innerText = "مرحباً " + name;
                document.getElementById('player-name-display').innerText = name;
                
                // سؤال التدريب
                if (confirm("هل تود عرض البرنامج التدريبي لتعلم الأساسيات؟")) {
                    this.showScreen('screen-training');
                } else {
                    this.showScreen('screen-menu');
                }
            },

            startGame: function(mode) {
                this.showScreen('screen-game');
                game.initBoard();
                // محاكاة إعلان مدفوع
                this.showModal("إعلان", "جاري عرض إعلان من الشريك... (محاكاة 5 ثواني)");
                setTimeout(() => document.getElementById('modal-popup').style.display='none', 3000);
            },

            showModal: function(title, msg) {
                document.getElementById('modal-title').innerText = title;
                document.getElementById('modal-msg').innerText = msg;
                document.getElementById('modal-popup').style.display = 'block';
            },

            deleteAccount: function() {
                if(confirm("هل أنت متأكد؟ لا يمكن التراجع عن هذا الإجراء.")) {
                    localStorage.removeItem('chessProUser');
                    location.reload();
                }
            },
            
            changeLang: function() {
                const lang = document.getElementById('lang-select').value;
                document.dir = (lang === 'ar') ? 'rtl' : 'ltr';
                alert("تم تغيير تفضيلات اللغة إلى: " + lang);
            }
        };

        // منطق المتجر
        const shop = {
            buy: function(cost, type) {
                if (app.user[type] >= cost) {
                    app.user[type] -= cost;
                    app.saveData();
                    app.showModal("نجاح", "تمت عملية الشراء بنجاح!");
                } else {
                    app.showModal("فشل", "رصيدك غير كافٍ.");
                }
            },
            buyRealMoney: function(amount) {
                // محاكاة بوابة دفع آمنة
                const card = prompt("محاكاة: أدخل رقم البطاقة الائتمانية (وهمي):");
                if (card) {
                    app.showModal("نظام الدفع", "تم الدفع بنجاح! شكرا لاشتراكك.");
                    app.user.gems += 500; // مكافأة
                    app.saveData();
                }
            }
        };

        // منطق اللعبة (مبسط جداً كنموذج)
        const game = {
            board: [],
            selectedSquare: null,
            turn: 'white', // white moves first
            
            // تمثيل القطع باليونيكود
            pieces: {
                w: { k: '♔', q: '♕', r: '♖', b: '♗', n: '♘', p: '♙' },
                b: { k: '♚', q: '♛', r: '♜', b: '♝', n: '♞', p: '♟' }
            },

            initBoard: function() {
                const boardEl = document.getElementById('chessboard');
                boardEl.innerHTML = '';
                // وضعية ابتدائية مبسطة للتجربة
                const initialLayout = [
                    ['r','n','b','q','k','b','n','r'],
                    ['p','p','p','p','p','p','p','p'],
                    ['','','','','','','',''],
                    ['','','','','','','',''],
                    ['','','','','','','',''],
                    ['','','','','','','',''],
                    ['P','P','P','P','P','P','P','P'],
                    ['R','N','B','Q','K','B','N','R']
                ];

                for (let r = 0; r < 8; r++) {
                    for (let c = 0; c < 8; c++) {
                        const sq = document.createElement('div');
                        sq.className = `square ${(r + c) % 2 === 0 ? 'light' : 'dark'}`;
                        sq.dataset.row = r;
                        sq.dataset.col = c;
                        
                        const pieceChar = initialLayout[r][c];
                        if (pieceChar) {
                            const color = pieceChar === pieceChar.toUpperCase() ? 'w' : 'b';
                            const type = pieceChar.toLowerCase();
                            sq.innerText = this.pieces[color][type];
                            sq.dataset.piece = pieceChar;
                            sq.dataset.color = color;
                        }
                        
                        sq.onclick = () => this.handleClick(sq);
                        boardEl.appendChild(sq);
                    }
                }
            },

            handleClick: function(sq) {
                // منطق تحريك بسيط جداً (غير كامل القوانين) لأغراض العرض
                if (this.selectedSquare) {
                    // محاولة النقل
                    if (sq !== this.selectedSquare) {
                        // نقل القطعة
                        sq.innerText = this.selectedSquare.innerText;
                        sq.dataset.piece = this.selectedSquare.dataset.piece;
                        sq.dataset.color = this.selectedSquare.dataset.color;
                        
                        // إفراغ المربع القديم
                        this.selectedSquare.innerText = '';
                        delete this.selectedSquare.dataset.piece;
                        delete this.selectedSquare.dataset.color;
                        this.selectedSquare.classList.remove('selected');
                        this.selectedSquare = null;
                        
                        // تشغيل الصوت
                        this.playSound('move');
                        
                        // تبديل الدور وتشغيل الذكاء الاصطناعي
                        this.turn = 'black';
                        setTimeout(() => this.aiMove(), 1000);
                    } else {
                        // إلغاء التحديد
                        this.selectedSquare.classList.remove('selected');
                        this.selectedSquare = null;
                    }
                } else {
                    // تحديد قطعة
                    if (sq.innerText && sq.dataset.color === 'w') { // اللاعب يلعب بالأبيض فقط
                        this.selectedSquare = sq;
                        sq.classList.add('selected');
                    }
                }
            },

            aiMove: function() {
                // ذكاء اصطناعي عشوائي بسيط جداً
                const squares = Array.from(document.querySelectorAll('.square'));
                const blackPieces = squares.filter(s => s.dataset.color === 'b');
                
                if (blackPieces.length > 0) {
                    const randomPiece = blackPieces[Math.floor(Math.random() * blackPieces.length)];
                    // حركة عشوائية لأسفل (فقط للمحاكاة)
                    const currentRow = parseInt(randomPiece.dataset.row);
                    const currentCol = parseInt(randomPiece.dataset.col);
                    
                    // محاولة إيجاد مربع فارغ عشوائي
                    const targetSq = squares.find(s => 
                        parseInt(s.dataset.row) === currentRow + 1 && 
                        Math.abs(parseInt(s.dataset.col) - currentCol) <= 1
                    );

                    if (targetSq) {
                        targetSq.innerText = randomPiece.innerText;
                        targetSq.dataset.piece = randomPiece.dataset.piece;
     

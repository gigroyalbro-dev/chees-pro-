<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>شطرنج برو - Chess Pro</title>
    <link href="https://fonts.googleapis.com/css2?family=Cairo:wght@300;400;600;700&display=swap" rel="stylesheet">
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            font-family: 'Cairo', sans-serif;
        }

        body {
            background: linear-gradient(135deg, #1e3c72 0%, #2a5298 100%);
            color: white;
            direction: rtl;
            min-height: 100vh;
        }

        .navbar {
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 1rem 2rem;
            background: rgba(255, 255, 255, 0.1);
            backdrop-filter: blur(10px);
            border-bottom: 1px solid rgba(255, 255, 255, 0.2);
        }

        .nav-brand {
            font-size: 1.5rem;
            font-weight: bold;
        }

        .nav-links {
            display: flex;
            gap: 1rem;
        }

        .nav-btn {
            padding: 0.5rem 1rem;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            transition: all 0.3s ease;
        }

        .nav-btn.primary {
            background: #4CAF50;
            color: white;
        }

        .nav-btn:hover {
            transform: translateY(-2px);
            box-shadow: 0 5px 15px rgba(0,0,0,0.2);
        }

        .main-container {
            max-width: 1200px;
            margin: 0 auto;
            padding: 2rem;
        }

        .hero-section {
            text-align: center;
            padding: 4rem 0;
        }

        .hero-section h1 {
            font-size: 3.5rem;
            margin-bottom: 1rem;
            text-shadow: 2px 2px 4px rgba(0,0,0,0.5);
        }

        .hero-section p {
            font-size: 1.2rem;
            margin-bottom: 2rem;
            opacity: 0.9;
        }

        .game-options {
            display: flex;
            gap: 1rem;
            justify-content: center;
            flex-wrap: wrap;
            margin-top: 2rem;
        }

        .game-btn {
            padding: 1rem 2rem;
            font-size: 1.1rem;
            border: none;
            border-radius: 10px;
            cursor: pointer;
            transition: all 0.3s ease;
            min-width: 200px;
        }

        .game-btn.primary {
            background: linear-gradient(45deg, #4CAF50, #45a049);
            color: white;
        }

        .game-btn.secondary {
            background: linear-gradient(45deg, #2196F3, #1976D2);
            color: white;
        }

        .game-btn.accent {
            background: linear-gradient(45deg, #FF9800, #F57C00);
            color: white;
        }

        .game-btn:hover {
            transform: translateY(-3px);
            box-shadow: 0 10px 25px rgba(0,0,0,0.3);
        }

        .game-section {
            margin-top: 2rem;
        }

        .game-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 2rem;
            padding: 1rem;
            background: rgba(255, 255, 255, 0.1);
            border-radius: 10px;
        }

        .player-info {
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: 0.5rem;
            padding: 1rem;
            border-radius: 8px;
            min-width: 150px;
        }

        .player-info.white {
            background: rgba(255, 255, 255, 0.9);
            color: #333;
        }

        .player-info.black {
            background: rgba(0, 0, 0, 0.8);
            color: white;
        }

        .timer {
            font-size: 1.5rem;
            font-weight: bold;
            font-family: monospace;
        }

        .game-controls {
            display: flex;
            gap: 0.5rem;
        }

        .game-controls button {
            padding: 0.5rem 1rem;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            background: #555;
            color: white;
            transition: background 0.3s ease;
        }

        .game-controls button:hover {
            background: #666;
        }

        .chess-container {
            display: flex;
            gap: 2rem;
            justify-content: center;
            align-items: flex-start;
        }

        .chess-board {
            display: grid;
            grid-template-columns: repeat(8, 70px);
            grid-template-rows: repeat(8, 70px);
            border: 15px solid #8B4513;
            border-radius: 5px;
            box-shadow: 0 20px 40px rgba(0,0,0,0.3);
            background: #8B4513;
        }

        .square {
            width: 70px;
            height: 70px;
            display: flex;
            align-items: center;
            justify-content: center;
            cursor: pointer;
            position: relative;
            transition: all 0.2s ease;
        }

        .square.light {
            background-color: #f0d9b5;
        }

        .square.dark {
            background-color: #b58863;
        }

        .square.selected {
            background-color: #aec6cf !important;
        }

        .square.valid-move::before {
            content: '';
            width: 20px;
            height: 20px;
            border-radius: 50%;
            background: rgba(0, 0, 0, 0.3);
            position: absolute;
        }

        .square.check {
            background-color: #ff6b6b !important;
        }

        .piece {
            width: 60px;
            height: 60px;
            background-size: cover;
            background-repeat: no-repeat;
            transition: all 0.3s ease;
            z-index: 10;
            cursor: pointer;
        }

        .piece.dragging {
            transform: scale(1.1);
            z-index: 100;
        }

        .piece.wp { background-image: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 45 45"><path d="M22.5 9c-2.21 0-4 1.79-4 4 0 .89.29 1.71.78 2.38C17.33 16.5 16 18.59 16 21c0 2.03.94 3.84 2.41 5.03-3 1.06-7.41 5.55-7.41 13.47h23c0-7.92-4.41-12.41-7.41-13.47 1.47-1.19 2.41-3 2.41-5.03 0-2.41-1.33-4.5-3.28-5.62.49-.67.78-1.49.78-2.38 0-2.21-1.79-4-4-4z" fill="white" stroke="black"/></svg>'); }
        .piece.wn { background-image: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 45 45"><path d="M22 10c10.5 1 16.5 8 16 29H15c0-9 10-6.5 8-21" fill="white" stroke="black"/><path d="M24 18c.38 2.91-5.55 7.37-8 9-3 2-2.82 4.34-5 4-1.04-.94 1.41-3.04 0-3-1 0 .19 1.23-1 2-1 0-4.003 1-4-4 0-2 6-12 6-12s1.89-1.9 2-3.5c-.73-.994-.5-2-.5-3 1-1 3 2.5 3 2.5h2s.78-1.992 2.5-3c1 0 1 3 1 3" fill="white" stroke="black"/><path d="M9.5 25.5a.5.5 0 1 1-1 0 .5.5 0 1 1 1 0z" fill="black"/><path d="M15 15.5a.5 1.5 0 1 1-1 0 .5 1.5 0 1 1 1 0z" transform="rotate(30 13.5 15.5)" fill="black"/></svg>'); }
        .piece.wb { background-image: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 45 45"><g fill="none" fill-rule="evenodd" stroke="black" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><g fill="white" stroke-linecap="butt"><path d="M9 36c3.39-.97 10.11.43 13.5-2 3.39 2.43 10.11 1.03 13.5 2 0 0 1.65.54 3 2-.68.97-1.65.99-3 .5-3.39-.97-10.11.46-13.5-1-3.39 1.46-10.11.03-13.5 1-1.35.49-2.32.47-3-.5 1.35-1.46 3-2 3-2z"/><path d="M15 32c2.5 2.5 12.5 2.5 15 0 .5-1.5 0-2 0-2 0-2.5-2.5-4-2.5-4 5.5-1.5 6-11.5-5-15.5-11 4-10.5 14-5 15.5 0 0-2.5 1.5-2.5 4 0 0-.5.5 0 2z"/><path d="M25 8a2.5 2.5 0 1 1-5 0 2.5 2.5 0 1 1 5 0z"/></g><path d="M17.5 26h10M15 30h15m-7.5-14.5v5M20 18h5" stroke="black"/></g></svg>'); }
        .piece.wr { background-image: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 45 45"><path d="M9 39h27v-3H9v3zm3-3v-4h21v4H12zm-1-22V9h4v2h5V9h5v2h5V9h4v5" fill="white" stroke="black"/><path d="M34 14l-3 3H14l-3-3" fill="white" stroke="black"/><path d="M31 17v12.5H14V17" fill="white" stroke="black"/><path d="M31 29.5l1.5 2.5h-20l1.5-2.5" fill="white" stroke="black"/><path d="M11 14h23" fill="white" stroke="black"/></svg>'); }
        .piece.wq { background-image: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 45 45"><path d="M9 26c8.5-1.5 21-1.5 27 0l2.5-12.5L31 25l-.3-14.1-5.2 13.6-3-14.5-3 14.5-5.2-13.6L14 25 6.5 13.5 9 26z" fill="white" stroke="black"/><path d="M9 26c0 2 1.5 2 2.5 4 1 1.5 1 1 .5 3.5-1.5 1-1 2.5-1 2.5-1.5 1.5 0 2.5 0 2.5 6.5 1 16.5 1 23 0 0 0 1.5-1 0-2.5 0 0 .5-1.5-1-2.5-.5-2.5-.5-2 .5-3.5 1-2 2.5-2 2.5-4-8.5-1.5-18.5-1.5-27 0z" fill="white" stroke="black"/><path d="M11.5 30c3.5-1 18.5-1 22 0M12 33.5c6-1 15-1 21 0" fill="white"/></svg>'); }
        .piece.wk { background-image: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 45 45"><path d="M22.5 11.63V6M20 8h5" stroke="black" stroke-width="1.5" stroke-linejoin="round"/><path d="M22.5 25s4.5-7.5 3-10.5c0 0-1-2.5-3-2.5s-3 2.5-3 2.5c-1.5 3 3 10.5 3 10.5" fill="white" stroke-linecap="butt" stroke-linejoin="round"/><path d="M12.5 37c5.5 3.5 14.5 3.5 20 0v-7s9-4.5 6-10.5c-4-6.5-13.5-3.5-16 4V27v-3.5c-2.5-7.5-12-10.5-16-4-3 6 6 10.5 6 10.5v7" fill="white"/><path d="M12.5 30c5.5-3 14.5-3 20 0M12.5 33.5c5.5-3 14.5-3 20 0M12.5 37c5.5-3 14.5-3 20 0" stroke="black"/></svg>'); }
        .piece.bp { background-image: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 45 45"><path d="M22.5 9c-2.21 0-4 1.79-4 4 0 .89.29 1.71.78 2.38C17.33 16.5 16 18.59 16 21c0 2.03.94 3.84 2.41 5.03-3 1.06-7.41 5.55-7.41 13.47h23c0-7.92-4.41-12.41-7.41-13.47 1.47-1.19 2.41-3 2.41-5.03 0-2.41-1.33-4.5-3.28-5.62.49-.67.78-1.49.78-2.38 0-2.21-1.79-4-4-4z" fill="black" stroke="black"/></svg>'); }
        .piece.bn { background-image: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 45 45"><path d="M22 10c10.5 1 16.5 8 16 29H15c0-9 10-6.5 8-21" fill="black" stroke="black"/><path d="M24 18c.38 2.91-5.55 7.37-8 9-3 2-2.82 4.34-5 4-1.04-.94 1.41-3.04 0-3-1 0 .19 1.23-1 2-1 0-4.003 1-4-4 0-2 6-12 6-12s1.89-1.9 2-3.5c-.73-.994-.5-2-.5-3 1-1 3 2.5 3 2.5h2s.78-1.992 2.5-3c1 0 1 3 1 3" fill="black" stroke="black"/><path d="M9.5 25.5a.5.5 0 1 1-1 0 .5.5 0 1 1 1 0z" fill="white"/><path d="M15 15.5a.5 1.5 0 1 1-1 0 .5 1.5 0 1 1 1 0z" transform="rotate(30 13.5 15.5)" fill="white"/></svg>'); }
        .piece.bb { background-image: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 45 45"><g fill="none" fill-rule="evenodd" stroke="black" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><g fill="black" stroke-linecap="butt"><path d="M9 36c3.39-.97 10.11.43 13.5-2 3.39 2.43 10.11 1.03 13.5 2 0 0 1.65.54 3 2-.68.97-1.65.99-3 .5-3.39-.97-10.11.46-13.5-1-3.39 1.46-10.11.03-13.5 1-1.35.49-2.32.47-3-.5 1.35-1.46 3-2 3-2z"/><path d="M15 32c2.5 2.5 12.5 2.5 15 0 .5-1.5 0-2 0-2 0-2.5-2.5-4-2.5-4 5.5-1.5 6-11.5-5-15.5-11 4-10.5 14-5 15.5 0 0-2.5 1.5-2.5 4 0 0-.5.5 0 2z"/><path d="M25 8a2.5 2.5 0 1 1-5 0 2.5 2.5 0 1 1 5 0z"/></g><path d="M17.5 26h10M15 30h15m-7.5-14.5v5M20 18h5" stroke="white"/></g></svg>'); }
        .piece.br { background-image: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 45 45"><path d="M9 39h27v-3H9v3zm3-3v-4h21v4H12zm-1-22V9h4v2h5V9h5v2h5V9h4v5" fill="black" stroke="black"/><path d="M34 14l-3 3H14l-3-3" fill="black" stroke="black"/><path d="M31 17v12.5H14V17" fill="black" stroke="black"/><path d="M31 29.5l1.5 2.5h-20l1.5-2.5" fill="black" stroke="black"/><path d="M11 14h23" fill="black" stroke="black"/></svg>'); }
        .piece.bq { background-image: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 45 45"><path d="M9 26c8.5-1.5 21-1.5 27 0l2.5-12.5L31 25l-.3-14.1-5.2 13.6-3-14.5-3 14.5-5.2-13.6L14 25 6.5 13.5 9 26z" fill="black" stroke="black"/><path d="M9 26c0 2 1.5 2 2.5 4 1 1.5 1 1 .5 3.5-1.5 1-1 2.5-1 2.5-1.5 1.5 0 2.5 0 2.5 6.5 1 16.5 1 23 0 0 0 1.5-1 0-2.5 0 0 .5-1.5-1-2.5-.5-2.5-.5-2 .5-3.5 1-2 2.5-2 2.5-4-8.5-1.5-18.5-1.5-27 0z" fill="black" stroke="black"/><path d="M11.5 30c3.5-1 18.5-1 22 0M12 33.5c6-1 15-1 21 0" fill="black"/></svg>'); }
        .piece.bk { background-image: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 45 45"><path d="M22.5 11.63V6M20 8h5" stroke="black" stroke-width="1.5" stroke-linejoin="round"/><path d="M22.5 25s4.5-7.5 3-10.5c0 0-1-2.5-3-2.5s-3 2.5-3 2.5c-1.5 3 3 10.5 3 10.5" fill="black" stroke-linecap="butt" stroke-linejoin="round"/><path d="M12.5 37c5.5 3.5 14.5 3.5 20 0v-7s9-4.5 6-10.5c-4-6.5-13.5-3.5-16 4V27v-3.5c-2.5-7.5-12-10.5-16-4-3 6 6 10.5 6 10.5v7" fill="black"/><path d="M12.5 30c5.5-3 14.5-3 20 0M12.5 33.5c5.5-3 14.5-3 20 0M12.5 37c5.5-3 14.5-3 20 0" stroke="white"/></svg>'); }

        .game-sidebar {
            background: rgba(255, 255, 255, 0.1);
            padding: 1rem;
            border-radius: 10px;
            min-width: 250px;
            backdrop-filter: blur(10px);
        }

        .move-history {
            max-height: 300px;
            overflow-y: auto;
        }

        .move-list {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 0.5rem;
            margin-top: 1rem;
        }

        .move-item {
            background: rgba(255, 255, 255, 0.2);
            padding: 0.5rem;
            border-radius: 5px;
            text-align: center;
            font-family: monospace;
        }

        .emote-section {
            margin-top: 1rem;
        }

        .emotes {
            display: flex;
            gap: 0.5rem;
            margin-top: 0.5rem;
            flex-wrap: wrap;
        }

        .emote {
            background: rgba(255, 255, 255, 0.2);
            border: none;
            border-radius: 50%;
            width: 40px;
            height: 40px;
            cursor: pointer;
            font-size: 1.2rem;
            transition: all 0.2s ease;
        }

        .emote:hover {
            background: rgba(255, 255, 255, 0.3);
            transform: scale(1.1);
        }

        .modal {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0, 0, 0, 0.8);
            display: flex;
            align-items: center;
            justify-content: center;
            z-index: 1000;
        }

        .modal-content {
            background: white;
            padding: 2rem;
            border-radius: 10px;
            color: #333;
            min-width: 400px;
            text-align: center;
        }

        .modal-content h2 {
            margin-bottom: 1.5rem;
            color: #2a5298;
        }

        .modal-content input {
            width: 100%;
            padding: 0.75rem;
            margin: 0.5rem 0;
            border: 1px solid #ddd;
            border-radius: 5px;
        }

        .modal-content button {
            width: 100%;
            padding: 0.75rem;
            margin: 0.5rem 0;
            border: none;
            border-radius: 5px;
            background: #4CAF50;
            color: white;
            cursor: pointer;
            transition: background 0.3s ease;
        }

        .modal-content button:hover {
            background: #45a049;
        }

        .loading {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0, 0, 0, 0.8);
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            z-index: 2000;
        }

        .spinner {
            font-size: 4rem;
            animation: spin 1s linear infinite;
        }

        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }

        .hidden {
            display: none !important;
        }

        @media (max-width: 768px) {
            .chess-board {
                grid-template-columns: repeat(8, 40px);
                grid-template-rows: repeat(8, 40px);
            }
            
            .square {
                width: 40px;
                height: 40px;
            }
            
            .piece {
                width: 35px;
                height: 35px;
            }
            
            .chess-container {
                flex-direction: column;
                align-items: center;
            }
            
            .game-header {
                flex-direction: column;
                gap: 1rem;
            }
            
            .hero-section h1 {
                font-size: 2.5rem;
            }
            
            .game-options {
                flex-direction: column;
                align-items: center;
            }
            
            .game-btn {
                min-width: 250px;
            }
        }
    </style>
</head>
<body>
    <div id="app">
        <nav class="navbar">
            <div class="nav-brand">
                <span>♛ شطرنج برو</span>
            </div>
            <div class="nav-links">
                <button id="loginBtn" class="nav-btn">تسجيل الدخول</button>
                <button id="registerBtn" class="nav-btn primary">إنشاء حساب</button>
            </div>
        </nav>

        <main class="main-container">
            <div class="hero-section">
                <h1>♚ شطرنج برو</h1>
                <p>اللعبة الأذكى في العالم العربي</p>
                
                <div class="game-options">
                    <button id="playAiBtn" class="game-btn primary">
                        ♟ اللعب ضد الكمبيوتر
                    </button>
                    <button id="playOnlineBtn" class="game-btn secondary">
                        🌐 اللعب عبر الإنترنت
                    </button>
                    <button id="trainingBtn" class="game-btn accent">
                        📚 التدريب
                    </button>
                </div>
            </div>

            <div id="gameSection" class="game-section hidden">
                <div class="game-header">
                    <div class="player-info white">
                        <span>الأبيض: <span id="whitePlayer">أنت</span></span>
                        <div class="timer" id="whiteTimer">10:00</div>
                    </div>
                    
                    <div class="game-controls">
                        <button id="newGameBtn">لعبة جديدة</button>
                        <button id="resignBtn">استسلام</button>
                        <button id="hintBtn">تلميح</button>
                    </div>

                    <div class="player-info black">
                        <span>الأسود: <span id="blackPlayer">الكمبيوتر</span></span>
                        <div class="timer" id="blackTimer">10:00</div>
                    </div>
                </div>

                <div class="chess-container">
                    <div id="chessBoard" class="chess-board"></div>
                    <div class="game-sidebar">
                        <div class="move-history">
                            <h3>سجل النقلات</h3>
                            <div id="moveList" class="move-list"></div>
                        </div>
                        <div class="emote-section">
                            <h3>التعبيرات</h3>
                            <div class="emotes">
                                <button class="emote" data-emote="👍">👍</button>
                                <button class="emote" data-emote="🤔">🤔</button>
                                <button class="emote" data-emote="🎉">🎉</button>
                                <

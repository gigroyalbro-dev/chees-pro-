import React, { useState, useRef, useEffect } from 'react';
import { Chess } from 'chess.js';

// الملف الكامل بلغة واحدة - Chess Master
const ChessMaster = () => {
  // === حالات التطبيق ===
  const [currentView, setCurrentView] = useState('main'); // main, game, profile, training
  const [user, setUser] = useState(null);
  const [game, setGame] = useState(new Chess());
  const [selectedSquare, setSelectedSquare] = useState(null);
  const [possibleMoves, setPossibleMoves] = useState([]);
  const [playerColor, setPlayerColor] = useState('white');
  const [gameMode, setGameMode] = useState('ai');
  const [aiDifficulty, setAiDifficulty] = useState('medium');
  const [gameHistory, setGameHistory] = useState([]);

  // === تأثيرات ===
  useEffect(() => {
    // تحميل بيانات المستخدم من localStorage
    const savedUser = localStorage.getItem('chessUser');
    if (savedUser) {
      setUser(JSON.parse(savedUser));
    }
  }, []);

  // === دوال اللعبة ===
  const handleSquareClick = (square) => {
    if (game.isGameOver()) return;

    const piece = game.get(square);

    if (selectedSquare) {
      try {
        const move = game.move({
          from: selectedSquare,
          to: square,
          promotion: 'q'
        });

        if (move) {
          const newGame = new Chess(game.fen());
          setGame(newGame);
          setSelectedSquare(null);
          setPossibleMoves([]);
          
          // إضافة الحركة للسجل
          setGameHistory(prev => [...prev, move]);
          
          // إذا كان ضد الذكاء الاصطناعي
          if (gameMode === 'ai' && newGame.turn() !== playerColor[0]) {
            setTimeout(() => makeAIMove(newGame), 500);
          }
        }
      } catch (e) {
        // إذا النقر على قطعة أخرى
        if (piece && piece.color === game.turn()) {
          setSelectedSquare(square);
          setPossibleMoves(game.moves({ square, verbose: true }));
          return;
        }
        setSelectedSquare(null);
        setPossibleMoves([]);
      }
    } else {
      if (piece && piece.color === game.turn()) {
        setSelectedSquare(square);
        setPossibleMoves(game.moves({ square, verbose: true }));
      }
    }
  };

  const makeAIMove = (currentGame) => {
    const moves = currentGame.moves({ verbose: true });
    if (moves.length === 0) return;

    // خوارزمية بسيطة للذكاء الاصطناعي
    let bestMove = null;
    let bestScore = -Infinity;

    moves.forEach(move => {
      currentGame.move(move);
      const score = evaluateBoard(currentGame);
      currentGame.undo();

      if (score > bestScore) {
        bestScore = score;
        bestMove = move;
      }
    });

    if (bestMove) {
      currentGame.move(bestMove);
      setGame(new Chess(currentGame.fen()));
      setGameHistory(prev => [...prev, bestMove]);
    }
  };

  const evaluateBoard = (game) => {
    let score = 0;
    const board = game.board();
    const pieceValues = { 'p': 10, 'n': 30, 'b': 30, 'r': 50, 'q': 90, 'k': 900 };

    for (let i = 0; i < 8; i++) {
      for (let j = 0; j < 8; j++) {
        const piece = board[i][j];
        if (piece) {
          const value = pieceValues[piece.type];
          score += piece.color === 'w' ? value : -value;
        }
      }
    }
    return game.turn() === 'w' ? score : -score;
  };

  const resetGame = () => {
    setGame(new Chess());
    setSelectedSquare(null);
    setPossibleMoves([]);
    setGameHistory([]);
  };

  const startNewGame = (mode, color = 'white') => {
    setGameMode(mode);
    setPlayerColor(color);
    resetGame();
    setCurrentView('game');
  };

  // === دوال المستخدم ===
  const handleLogin = (userData) => {
    const newUser = {
      id: Date.now(),
      username: userData.username,
      email: userData.email,
      profile: {
        level: 'مبتدئ',
        eloRating: 1200,
        coins: 100,
        country: userData.country,
        language: userData.language
      },
      statistics: {
        gamesPlayed: 0,
        gamesWon: 0,
        gamesLost: 0,
        gamesDrawn: 0
      },
      preferences: {
        theme: 'classic',
        soundEnabled: true
      }
    };
    setUser(newUser);
    localStorage.setItem('chessUser', JSON.stringify(newUser));
  };

  const handleLogout = () => {
    setUser(null);
    localStorage.removeItem('chessUser');
  };

  // === مكونات الواجهة ===
  const ChessBoard = () => {
    const renderSquare = (i, j) => {
      const file = 'abcdefgh'[j];
      const rank = 8 - i;
      const square = `${file}${rank}`;
      const isLight = (i + j) % 2 === 0;
      const piece = game.get(square);
      const isSelected = selectedSquare === square;
      const isPossibleMove = possibleMoves.some(move => move.to === square);

      return (
        <div
          key={square}
          className={`square ${isLight ? 'light' : 'dark'} ${
            isSelected ? 'selected' : ''
          } ${isPossibleMove ? 'possible-move' : ''}`}
          onClick={() => handleSquareClick(square)}
          style={{
            width: '60px',
            height: '60px',
            display: 'flex',
            alignItems: 'center',
            justifyContent: 'center',
            cursor: 'pointer',
            position: 'relative',
            backgroundColor: isLight ? '#f0d9b5' : '#b58863'
          }}
        >
          {piece && (
            <div style={{
              fontSize: '45px',
              width: '100%',
              height: '100%',
              display: 'flex',
              alignItems: 'center',
              justifyContent: 'center',
              userSelect: 'none'
            }}>
              {getPieceSymbol(piece)}
            </div>
          )}
          {isPossibleMove && !piece && (
            <div style={{
              width: '20px',
              height: '20px',
              backgroundColor: 'rgba(0,0,0,0.3)',
              borderRadius: '50%'
            }} />
          )}
        </div>
      );
    };

    const getPieceSymbol = (piece) => {
      const symbols = {
        w: { k: '♔', q: '♕', r: '♖', b: '♗', n: '♘', p: '♙' },
        b: { k: '♚', q: '♛', r: '♜', b: '♝', n: '♞', p: '♟' }
      };
      return symbols[piece.color][piece.type];
    };

    const squares = [];
    for (let i = 0; i < 8; i++) {
      for (let j = 0; j < 8; j++) {
        squares.push(renderSquare(i, j));
      }
    }

    return (
      <div style={{
        display: 'flex',
        flexDirection: 'column',
        alignItems: 'center',
        gap: '20px',
        padding: '20px',
        background: '#2c3e50',
        borderRadius: '15px',
        boxShadow: '0 10px 30px rgba(0,0,0,0.3)'
      }}>
        <div style={{
          display: 'flex',
          gap: '20px',
          alignItems: 'center',
          color: 'white',
          fontSize: '18px',
          fontWeight: 'bold'
        }}>
          <div style={{
            padding: '10px 20px',
            background: '#34495e',
            borderRadius: '25px'
          }}>
            دور: {game.turn() === 'w' ? 'الأبيض' : 'الأسود'}
          </div>
          {game.isCheck() && (
            <div style={{
              padding: '10px 20px',
              background: '#e74c3c',
              borderRadius: '25px',
              animation: 'pulse 1.5s infinite'
            }}>
              كش ملك!
            </div>
          )}
        </div>

        <div style={{
          display: 'grid',
          gridTemplateColumns: 'repeat(8, 60px)',
          gridTemplateRows: 'repeat(8, 60px)',
          border: '3px solid #34495e',
          borderRadius: '8px',
          overflow: 'hidden'
        }}>
          {squares}
        </div>

        <div style={{ display: 'flex', gap: '15px' }}>
          <button
            onClick={resetGame}
            style={{
              padding: '12px 24px',
              background: '#3498db',
              color: 'white',
              border: 'none',
              borderRadius: '25px',
              cursor: 'pointer',
              fontSize: '16px',
              fontWeight: 'bold'
            }}
          >
            لعبة جديدة
          </button>
          <button
            onClick={() => setCurrentView('main')}
            style={{
              padding: '12px 24px',
              background: '#95a5a6',
              color: 'white',
              border: 'none',
              borderRadius: '25px',
              cursor: 'pointer',
              fontSize: '16px',
              fontWeight: 'bold'
            }}
          >
            القائمة الرئيسية
          </button>
        </div>
      </div>
    );
  };

  const LoginForm = () => {
    const [formData, setFormData] = useState({
      username: '',
      email: '',
      password: '',
      country: 'SA',
      language: 'ar'
    });

    const handleSubmit = (e) => {
      e.preventDefault();
      if (formData.username && formData.email && formData.password) {
        handleLogin(formData);
      }
    };

    return (
      <div style={{
        minHeight: '100vh',
        display: 'flex',
        alignItems: 'center',
        justifyContent: 'center',
        background: 'linear-gradient(135deg, #667eea 0%, #764ba2 100%)',
        padding: '20px'
      }}>
        <div style={{
          background: 'white',
          padding: '40px',
          borderRadius: '20px',
          boxShadow: '0 20px 40px rgba(0,0,0,0.1)',
          width: '100%',
          maxWidth: '400px'
        }}>
          <h2 style={{ textAlign: 'center', marginBottom: '30px', color: '#2c3e50' }}>
            إنشاء حساب جديد
          </h2>
          
          <form onSubmit={handleSubmit} style={{ display: 'flex', flexDirection: 'column', gap: '20px' }}>
            <div>
              <label style={{ display: 'block', marginBottom: '5px', fontWeight: 'bold' }}>
                اسم المستخدم
              </label>
              <input
                type="text"
                value={formData.username}
                onChange={(e) => setFormData({...formData, username: e.target.value})}
                style={{
                  width: '100%',
                  padding: '12px',
                  border: '2px solid #bdc3c7',
                  borderRadius: '10px',
                  fontSize: '16px'
                }}
                placeholder="اختر اسمًا فريدًا"
                required
              />
            </div>

            <div>
              <label style={{ display: 'block', marginBottom: '5px', fontWeight: 'bold' }}>
                البريد الإلكتروني
              </label>
              <input
                type="email"
                value={formData.email}
                onChange={(e) => setFormData({...formData, email: e.target.value})}
                style={{
                  width: '100%',
                  padding: '12px',
                  border: '2px solid #bdc3c7',
                  borderRadius: '10px',
                  fontSize: '16px'
                }}
                placeholder="example@email.com"
                required
              />
            </div>

            <div>
              <label style={{ display: 'block', marginBottom: '5px', fontWeight: 'bold' }}>
                كلمة المرور
              </label>
              <input
                type="password"
                value={formData.password}
                onChange={(e) => setFormData({...formData, password: e.target.value})}
                style={{
                  width: '100%',
                  padding: '12px',
                  border: '2px solid #bdc3c7',
                  borderRadius: '10px',
                  fontSize: '16px'
                }}
                placeholder="6 أحرف على الأقل"
                required
              />
            </div>

            <button
              type="submit"
              style={{
                padding: '15px',
                background: 'linear-gradient(135deg, #3498db, #2980b9)',
                color: 'white',
                border: 'none',
                borderRadius: '10px',
                fontSize: '18px',
                fontWeight: 'bold',
                cursor: 'pointer',
                marginTop: '10px'
              }}
            >
              إنشاء الحساب
            </button>
          </form>
        </div>
      </div>
    );
  };

  const MainMenu = () => (
    <div style={{
      minHeight: '100vh',
      background: 'linear-gradient(135deg, #667eea 0%, #764ba2 100%)',
      padding: '20px',
      display: 'flex',
      flexDirection: 'column',
      alignItems: 'center',
      justifyContent: 'center'
    }}>
      <div style={{
        background: 'rgba(255,255,255,0.95)',
        padding: '40px',
        borderRadius: '20px',
        boxShadow: '0 20px 40px rgba(0,0,0,0.1)',
        textAlign: 'center',
        maxWidth: '600px',
        width: '100%'
      }}>
        <h1 style={{ color: '#2c3e50', marginBottom: '10px', fontSize: '2.5em' }}>
          ♞ Chess Master
        </h1>
        <p style={{ color: '#7f8c8d', marginBottom: '40px', fontSize: '1.2em' }}>
          استمتع بلعبة الشطرنج بتجربة فريدة ومميزة
        </p>

        {user && (
          <div style={{
            background: '#ecf0f1',
            padding: '20px',
            borderRadius: '15px',
            marginBottom: '30px'
          }}>
            <h3 style={{ color: '#2c3e50', marginBottom: '10px' }}>
              مرحباً، {user.username}!
            </h3>
            <div style={{ display: 'flex', gap: '20px', justifyContent: 'center', flexWrap: 'wrap' }}>
              <div style={{ textAlign: 'center' }}>
                <div style={{ fontSize: '1.5em', fontWeight: 'bold', color: '#3498db' }}>
                  {user.profile.eloRating}
                </div>
                <div style={{ fontSize: '0.9em', color: '#7f8c8d' }}>التصنيف</div>
              </div>
              <div style={{ textAlign: 'center' }}>
                <div style={{ fontSize: '1.5em', fontWeight: 'bold', color: '#e74c3c' }}>
                  {user.statistics.gamesWon}
                </div>
                <div style={{ fontSize: '0.9em', color: '#7f8c8d' }}>الفوز</div>
              </div>
              <div style={{ textAlign: 'center' }}>
                <div style={{ fontSize: '1.5em', fontWeight: 'bold', color: '#f39c12' }}>
                  {user.profile.coins}
                </div>
                <div style={{ fontSize: '0.9em', color: '#7f8c8d' }}>العملات</div>
              </div>
            </div>
          </div>
        )}

        <div style={{ display: 'grid', gridTemplateColumns: 'repeat(auto-fit, minmax(250px, 1fr))', gap: '20px' }}>
          <div
            onClick={() => startNewGame('ai', 'white')}
            style={{
              background: 'linear-gradient(135deg, #3498db, #2980b9)',
              color: 'white',
              padding: '30px 20px',
              borderRadius: '15px',
              cursor: 'pointer',
              transition: 'transform 0.3s ease',
              textAlign: 'center'
            }}
            onMouseEnter={(e) => e.target.style.transform = 'translateY(-5px)'}
            onMouseLeave={(e) => e.target.style.transform = 'translateY(0)'}
          >
            <div style={{ fontSize: '3em', marginBottom: '10px' }}>🤖</div>
            <h3>العب ضد الكمبيوتر</h3>
            <p>تدرب ضد ذكاء اصطناعي</p>
          </div>

          <div
            onClick={() => startNewGame('local', 'white')}
            style={{
              background: 'linear-gradient(135deg, #2ecc71, #27ae60)',
              color: 'white',
              padding: '30px 20px',
              borderRadius: '15px',
              cursor: 'pointer',
              transition: 'transform 0.3s ease',
              textAlign: 'center'
            }}
            onMouseEnter={(e) => e.target.style.transform = 'translateY(-5px)'}
            onMouseLeave={(e) => e.target.style.transform = 'translateY(0)'}
          >
            <div style={{ fontSize: '3em', marginBottom: '10px' }}>👥</div>
            <h3>لاعب ضد لاعب</h3>
            <p>على نفس الجهاز</p>
          </div>

          <div
            onClick={() => setCurrentView('training')}
            style={{
              background: 'linear-gradient(135deg, #e74c3c, #c0392b)',
              color: 'white',
              padding: '30px 20px',
              borderRadius: '15px',
              cursor: 'pointer',
              transition: 'transform 0.3s ease',
              textAlign: 'center'
            }}
            onMouseEnter={(e) => e.target.style.transform = 'translateY(-5px)'}
            onMouseLeave={(e) => e.target.style.transform = 'translateY(0)'}
          >
            <div style={{ fontSize: '3em', marginBottom: '10px' }}>🎯</div>
            <h3>جلسات تدريبية</h3>
            <p>حسن مهاراتك</p>
          </div>

          <div
            onClick={() => setCurrentView('profile')}
            style={{
              background: 'linear-gradient(135deg, #9b59b6, #8e44ad)',
              color: 'white',
              padding: '30px 20px',
              borderRadius: '15px',
              cursor: 'pointer',
              transition: 'transform 0.3s ease',
              textAlign: 'center'
            }}
            onMouseEnter={(e) => e.target.style.transform = 'translateY(-5px)'}
            onMouseLeave={(e) => e.target.style.transform = 'translateY(0)'}
          >
            <div style={{ fontSize: '3em', marginBottom: '10px' }}>👤</div>
            <h3>الملف الشخصي</h3>
            <p>إدارة حسابك</p>
          </div>
        </div>

        {user && (
          <button
            onClick={handleLogout}
            style={{
              marginTop: '30px',
              padding: '12px 30px',
              background: 'transparent',
              color: '#e74c3c',
              border: '2px solid #e74c3c',
              borderRadius: '25px',
              cursor: 'pointer',
              fontSize: '16px',
              fontWeight: 'bold',
              transition: 'all 0.3s ease'
            }}
            onMouseEnter={(e) => {
              e.target.style.background = '#e74c3c';
              e.target.style.color = 'white';
            }}
            onMouseLeave={(e) => {
              e.target.style.background = 'transparent';
              e.target.style.color = '#e74c3c';
            }}
          >
            تسجيل الخروج
          </button>
        )}
      </div>
    </div>
  );

  const TrainingView = () => (
    <div style={{
      minHeight: '100vh',
      background: 'linear-gradient(135deg, #667eea 0%, #764ba2 100%)',
      padding: '20px'
    }}>
      <div style={{
        background: 'rgba(255,255,255,0.95)',
        padding: '40px',
        borderRadius: '20px',
        maxWidth: '800px',
        margin: '0 auto'
      }}>
        <div style={{ display: 'flex', justifyContent: 'space-between', alignItems: 'center', marginBottom: '30px' }}>
          <h1 style={{ color: '#2c3e50' }}>🎯 برنامج التدريب</h1>
          <button
            onClick={() => setCurrentView('main')}
            style={{
              padding: '10px 20px',
              background: '#95a5a6',
              color: 'white',
              border: 'none',
              borderRadius: '10px',
              cursor: 'pointer'
            }}
          >
            العودة
          </button>
        </div>

        <div style={{ display: 'grid', gap: '20px' }}>
          <div style={{
            background: 'linear-gradient(135deg, #3498db, #2980b9)',
            color: 'white',
            padding: '25px',
            borderRadius: '15px',
            cursor: 'pointer'
          }}>
            <h3>🌱 المستوى المبتدئ</h3>
            <p>تعلم حركات القطع والأساسيات</p>
            <div style={{ background: 'rgba(255,255,255,0.3)', height: '10px', borderRadius: '5px', marginTop: '10px' }}>
              <div style={{ background: 'white', height: '100%', width: '75%', borderRadius: '5px' }}></div>
            </div>
          </div>

          <div style={{
            background: 'linear-gradient(135deg, #2ecc71, #27ae60)',
            color: 'white',
            padding: '25px',
            borderRadius: '15px',
            cursor: 'pointer'
          }}>
            <h3>⚔️ التكتيكات الأساسية</h3>
            <p>تعلم الهجمات والدفاعات</p>
            <div style={{ background: 'rgba(255,255,255,0.3)', height: '10px', borderRadius: '5px', marginTop: '10px' }}>
              <div style={{ background: 'white', height: '100%', width: '40%', borderRadius: '5px' }}></div>
            </div>
          </div>

          <div style={{
            background: 'linear-gradient(135deg, #e74c3c, #c0392b)',
            color: 'white',
            padding: '25px',
            borderRadius: '15px',
            cursor: 'pointer'
          }}>
            <h3>🎯 الاستراتيجيات المتقدمة</h3>
            <p>تخطيط طويل المدى</p>
            <div style={{ background: 'rgba(255,255,255,0.3)', height: '10px', borderRadius: '5px', marginTop: '10px' }}>
              <div style={{ background: 'white', height: '100%', width: '20%', borderRadius: '5px' }}></div>
            </div>
          </div>
        </div>
      </div>
    </div>
  );

  const ProfileView = () => (
    <div style={{
      minHeight: '100vh',
      background: 'linear-gradient(135deg, #667eea 0%, #764ba2 100%)',
      padding: '20px'
    }}>
      <div style={{
        background: 'rgba(255,255,255,0.95)',
        padding: '40px',
        borderRadius: '20px',
        maxWidth: '600px',
        margin: '0 auto'
      }}>
        <div style={{ display: 'flex', justifyContent: 'space-between', alignItems: 'center', marginBottom: '30px' }}>
          <h1 style={{ color: '#2c3e50' }}>👤 الملف الشخصي</h1>
          <button
            onClick={() => setCurrentView('main')}
            style={{
              padding: '10px 20px',
              background: '#95a5a6',
              color: 'white',
              border: 'none',
              borderRadius: '10px',
              cursor: 'pointer'
            }}
          >
            العودة
          </button>
        </div>

        {user && (
          <div style={{ display: 'grid', gap: '20px' }}>
            <div style={{
              background: '#ecf0f1',
              padding: '20px',
              borderRadius: '15px',
              textAlign: 'center'
            }}>
              <div style={{ fontSize: '3em', marginBottom: '10px' }}>👑</div>
              <h2 style={{ color: '#2c3e50', marginBottom: '5px' }}>{user.username}</h2>
              <p style={{ color: '#7f8c8d', marginBottom: '15px' }}>{user.email}</p>
              <div style={{ background: '#3498db', color: 'white', padding: '5px 15px', borderRadius: '15px', display: 'inline-block' }}>
                {user.profile.level}
              </div>
            </div>

            <div style={{
              display: 'grid',
              gridTemplateColumns: 'repeat(2, 1fr)',
              gap: '15px'
            }}>
              <div style={{ background: '#3498db', color: 'white', padding: '20px', borderRadius: '10px', textAlign: 'center' }}>
                <div style={{ fontSize: '1.5em', fontWeight: 'bold' }}>{user.profile.eloRating}</div>
                <div>التصنيف</div>
              </div>
              <div style={{ background: '#2ecc71', color: 'white', padding: '20px', borderRadius: '10px', textAlign: 'center' }}>
                <div style={{ fontSize: '1.5em', fontWeight: 'bold' }}>{user.statistics.gamesPlayed}</div>
                <div>المباريات</div>
              </div>
              <div style={{ background: '#e74c3c', color: 'white', padding: '20px', borderRadius: '10px', textAlign: 'center' }}>
                <div style={{ fontSize: '1.5em', fontWeight: 'bold' }}>{user.statistics.gamesWon}</div>
                <div>الفوز</div>
              </div>
              <div style={{ background: '#f39c12', color: 'white', padding: '20px', borderRadius: '10px', textAlign: 'center' }}>
                <div style={{ fontSize: '1.5em', fontWeight: 'bold' }}>{user.profile.coins}</div>
                <div>العملات</div>
              </div>
            </div>

            <div style={{
              background: '#34495e',
              color: 'white',
              padding: '20px',
              borderRadius: '15px'
            }}>
              <h3 style={{ marginBottom: '15px' }}>الإحصائيات</h3>
              <div style={{ display: 'grid', gap: '10px' }}>
                <div style={{ display: 'flex', justifyContent: 'space-between' }}>
                  <span>معدل الفوز:</span>
                  <span>
                    {user.statistics.gamesPlayed > 0 
                      ? Math.round((user.statistics.gamesWon / user.statistics.gamesPlayed) * 100)
                      : 0}%
                  </span>
                </div>
                <div style={{ display: 'flex', justifyContent: 'space-between' }}>
                  <span>البلد:</span>
                  <span>{user.profile.country === 'SA' ? 'السعودية' : user.profile.country}</span>
                </div>
                <div style={{ display: 'flex', justifyContent: 'space-between' }}>
                  <span>اللغة:</span>
                  <span>{user.profile.language === 'ar' ? 'العربية' : user.profile.language}</span>
                </div>
              </div>
            </div>
          </div>
        )}
      </div>
    </div>
  );

  // === التصيير الرئيسي ===
  if (!user) {
    return <LoginForm />;
  }

  switch (currentView) {
    case 'game':
      return <ChessBoard />;
    case 'training':
      return <TrainingView />;
    case 'profile':
      return <ProfileView />;
    default:
      return <MainMenu />;
  }
};

// إضافة أنماط CSS مدمجة
const styles = `
  @keyframes pulse {
    0% { opacity: 1; }
    50% { opacity: 0.7; }
    100% { opacity: 1; }
  }
  
  * {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  }
  
  body {
    background: #ecf0f1;
  }
`;

// إضافة الأنماط إلى head
const styleSheet = document.createElement("style");
styleSheet.innerText = styles;
document.head.appendChild(styleSheet);

export default ChessMaster;

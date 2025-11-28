
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>画一个完美的圆 - Draw a Perfect Circle</title>
    <script crossorigin src="https://cdnjs.cloudflare.com/ajax/libs/react/18.2.0/umd/react.production.min.js"></script>
    <script crossorigin src="https://cdnjs.cloudflare.com/ajax/libs/react-dom/18.2.0/umd/react-dom.production.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/babel-standalone/7.23.5/babel.min.js"></script>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        body {
            margin: 0;
            padding: 0;
            overflow: hidden;
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
        }
        @keyframes float-up {
            0% {
                transform: translateY(0) scale(1);
                opacity: 1;
            }
            100% {
                transform: translateY(-100px) scale(0.5);
                opacity: 0;
            }
        }
        .particle {
            position: absolute;
            pointer-events: none;
            animation: float-up 1s ease-out forwards;
        }
    </style>
</head>
<body>
    <div id="root"></div>

    <script type="text/babel">
        const { useState, useRef, useEffect } = React;

        function PerfectCircleGame() {
            const canvasRef = useRef(null);
            const fireworksCanvasRef = useRef(null);
            const [isDrawing, setIsDrawing] = useState(false);
            const [points, setPoints] = useState([]);
            const [score, setScore] = useState(null);
            const [message, setMessage] = useState("画一个圆，一笔完成");
            const [particles, setParticles] = useState([]);

            useEffect(() => {
                const canvas = canvasRef.current;
                const fireworksCanvas = fireworksCanvasRef.current;
                if (canvas && fireworksCanvas) {
                    canvas.width = window.innerWidth;
                    canvas.height = window.innerHeight;
                    fireworksCanvas.width = window.innerWidth;
                    fireworksCanvas.height = window.innerHeight;
                }

                const handleResize = () => {
                    if (canvas && fireworksCanvas) {
                        canvas.width = window.innerWidth;
                        canvas.height = window.innerHeight;
                        fireworksCanvas.width = window.innerWidth;
                        fireworksCanvas.height = window.innerHeight;
                    }
                };

                window.addEventListener('resize', handleResize);
                return () => window.removeEventListener('resize', handleResize);
            }, []);

            const createFireworks = (centerX, centerY, scoreValue) => {
                const canvas = fireworksCanvasRef.current;
                const ctx = canvas.getContext('2d');
                
                // 根据分数决定烟花数量和颜色
                const fireworkCount = scoreValue >= 90 ? 5 : scoreValue >= 75 ? 3 : 2;
                const colors = scoreValue >= 90 
                    ? ['#FFD700', '#FF69B4', '#00CED1', '#FF6347', '#9370DB']
                    : scoreValue >= 75
                    ? ['#FFD700', '#FF69B4', '#00CED1']
                    : ['#FFD700', '#FF69B4'];

                let allParticles = [];

                for (let f = 0; f < fireworkCount; f++) {
                    const offsetX = (Math.random() - 0.5) * 200;
                    const offsetY = (Math.random() - 0.5) * 200;
                    const particleCount = 30;
                    const color = colors[f % colors.length];

                    for (let i = 0; i < particleCount; i++) {
                        const angle = (Math.PI * 2 * i) / particleCount;
                        const velocity = 2 + Math.random() * 3;
                        allParticles.push({
                            x: centerX + offsetX,
                            y: centerY + offsetY,
                            vx: Math.cos(angle) * velocity,
                            vy: Math.sin(angle) * velocity,
                            life: 1,
                            color: color,
                            size: 3 + Math.random() * 3
                        });
                    }
                }

                let animationId;
                const animate = () => {
                    ctx.clearRect(0, 0, canvas.width, canvas.height);
                    
                    allParticles = allParticles.filter(p => p.life > 0);
                    
                    if (allParticles.length === 0) {
                        cancelAnimationFrame(animationId);
                        return;
                    }

                    allParticles.forEach(p => {
                        p.x += p.vx;
                        p.y += p.vy;
                        p.vy += 0.1; // 重力
                        p.life -= 0.015;

                        ctx.globalAlpha = p.life;
                        ctx.fillStyle = p.color;
                        ctx.beginPath();
                        ctx.arc(p.x, p.y, p.size, 0, Math.PI * 2);
                        ctx.fill();
                    });

                    ctx.globalAlpha = 1;
                    animationId = requestAnimationFrame(animate);
                };

                animate();

                // 添加表情粒子
                createEmojiParticles(centerX, centerY, scoreValue);
            };

            const createEmojiParticles = (x, y, scoreValue) => {
                const emojis = scoreValue >= 90 
                    ? ['🎉', '⭐', '✨', '💫', '🌟']
                    : scoreValue >= 75
                    ? ['👍', '😊', '✨']
                    : ['🙂', '👌'];

                const newParticles = [];
                for (let i = 0; i < 8; i++) {
                    const angle = (Math.PI * 2 * i) / 8;
                    newParticles.push({
                        id: Date.now() + i,
                        emoji: emojis[Math.floor(Math.random() * emojis.length)],
                        x: x,
                        y: y,
                        vx: Math.cos(angle) * 3,
                        vy: Math.sin(angle) * 3
                    });
                }
                setParticles(newParticles);

                setTimeout(() => setParticles([]), 1000);
            };

            const getMousePos = (e) => {
                const canvas = canvasRef.current;
                const rect = canvas.getBoundingClientRect();
                return {
                    x: e.clientX - rect.left,
                    y: e.clientY - rect.top
                };
            };

            const getTouchPos = (e) => {
                const canvas = canvasRef.current;
                const rect = canvas.getBoundingClientRect();
                return {
                    x: e.touches[0].clientX - rect.left,
                    y: e.touches[0].clientY - rect.top
                };
            };

            const startDrawing = (pos) => {
                setIsDrawing(true);
                setPoints([pos]);
                setScore(null);
                setMessage("继续画...");
                
                const canvas = canvasRef.current;
                const ctx = canvas.getContext('2d');
                ctx.clearRect(0, 0, canvas.width, canvas.height);
            };

            const draw = (pos) => {
                if (!isDrawing) return;

                const newPoints = [...points, pos];
                setPoints(newPoints);

                const canvas = canvasRef.current;
                const ctx = canvas.getContext('2d');
                
                ctx.strokeStyle = '#2c3e50';
                ctx.lineWidth = 3;
                ctx.lineCap = 'round';
                ctx.lineJoin = 'round';

                if (newPoints.length > 1) {
                    const lastPoint = newPoints[newPoints.length - 2];
                    ctx.beginPath();
                    ctx.moveTo(lastPoint.x, lastPoint.y);
                    ctx.lineTo(pos.x, pos.y);
                    ctx.stroke();
                }
            };

            const calculateCircleScore = (pts) => {
                if (pts.length < 50) return 0;

                const centerX = pts.reduce((sum, p) => sum + p.x, 0) / pts.length;
                const centerY = pts.reduce((sum, p) => sum + p.y, 0) / pts.length;

                const radii = pts.map(p => Math.sqrt(Math.pow(p.x - centerX, 2) + Math.pow(p.y - centerY, 2)));
                const avgRadius = radii.reduce((sum, r) => sum + r, 0) / radii.length;

                const deviations = radii.map(r => Math.abs(r - avgRadius));
                const avgDeviation = deviations.reduce((sum, d) => sum + d, 0) / deviations.length;

                const startPoint = pts[0];
                const endPoint = pts[pts.length - 1];
                const distance = Math.sqrt(Math.pow(endPoint.x - startPoint.x, 2) + Math.pow(endPoint.y - startPoint.y, 2));
                const closedBonus = distance < avgRadius * 0.15 ? 1 : 0.7;

                const circularityScore = Math.max(0, 100 - (avgDeviation / avgRadius) * 200);
                const finalScore = Math.round(circularityScore * closedBonus);

                return {
                    score: Math.min(100, Math.max(0, finalScore)),
                    center: { x: centerX, y: centerY }
                };
            };

            const getScoreMessage = (score) => {
                if (score >= 95) return "🎯 完美！你是画圆大师！";
                if (score >= 85) return "⭐ 太棒了！接近完美！";
                if (score >= 75) return "👍 很好！非常圆！";
                if (score >= 60) return "😊 不错！继续练习！";
                if (score >= 40) return "🙂 还可以！再试一次！";
                return "😅 继续努力！熟能生巧！";
            };

            const endDrawing = () => {
                if (!isDrawing || points.length < 10) {
                    setIsDrawing(false);
                    return;
                }

                setIsDrawing(false);
                const result = calculateCircleScore(points);
                setScore(result.score);
                setMessage(getScoreMessage(result.score));
                
                // 触发烟花效果
                setTimeout(() => {
                    createFireworks(result.center.x, result.center.y, result.score);
                }, 300);
            };

            const reset = () => {
                const canvas = canvasRef.current;
                const fireworksCanvas = fireworksCanvasRef.current;
                const ctx = canvas.getContext('2d');
                const fwCtx = fireworksCanvas.getContext('2d');
                ctx.clearRect(0, 0, canvas.width, canvas.height);
                fwCtx.clearRect(0, 0, fireworksCanvas.width, fireworksCanvas.height);
                setPoints([]);
                setScore(null);
                setIsDrawing(false);
                setMessage("画一个圆，一笔完成");
                setParticles([]);
            };

            return (
                <div className="fixed inset-0 bg-gray-50 overflow-hidden">
                    <div className="absolute inset-0 opacity-5 pointer-events-none" 
                         style={{backgroundImage: 'url("data:image/svg+xml,%3Csvg width=\'100\' height=\'100\' xmlns=\'http://www.w3.org/2000/svg\'%3E%3Cfilter id=\'noise\'%3E%3CfeTurbulence baseFrequency=\'0.9\' numOctaves=\'3\' /%3E%3C/filter%3E%3Crect width=\'100%25\' height=\'100%25\' filter=\'url(%23noise)\' /%3E%3C/svg%3E")'}} />
                    
                    <div className="absolute top-0 left-0 right-0 p-6 flex justify-between items-center z-10">
                        <div>
                            <h1 className="text-3xl font-bold text-gray-800 mb-1">画一个完美的圆</h1>
                            <p className="text-gray-600">{message}</p>
                        </div>
                        <button
                            onClick={reset}
                            className="bg-white hover:bg-gray-100 text-gray-700 font-semibold py-3 px-6 rounded-lg shadow-md transition-all flex items-center gap-2"
                        >
                            <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round">
                                <path d="M3 12a9 9 0 1 0 9-9 9.75 9.75 0 0 0-6.74 2.74L3 8"/>
                                <path d="M3 3v5h5"/>
                            </svg>
                            重置
                        </button>
                    </div>

                    {score !== null && (
                        <div className="absolute top-1/2 left-1/2 transform -translate-x-1/2 -translate-y-1/2 z-20 pointer-events-none">
                            <div className="bg-white rounded-2xl shadow-2xl p-8 text-center animate-bounce">
                                <div className="text-6xl font-bold text-gray-800 mb-2">{score}%</div>
                                <div className="text-sm text-gray-500 uppercase tracking-wide">完美度</div>
                            </div>
                        </div>
                    )}

                    {/* 表情粒子 */}
                    {particles.map(p => (
                        <div
                            key={p.id}
                            className="particle text-3xl"
                            style={{
                                left: p.x + 'px',
                                top: p.y + 'px',
                                '--vx': p.vx,
                                '--vy': p.vy
                            }}
                        >
                            {p.emoji}
                        </div>
                    ))}

                    <canvas
                        ref={canvasRef}
                        className="cursor-crosshair absolute inset-0"
                        style={{ zIndex: 1 }}
                        onMouseDown={(e) => startDrawing(getMousePos(e))}
                        onMouseMove={(e) => draw(getMousePos(e))}
                        onMouseUp={endDrawing}
                        onMouseLeave={endDrawing}
                        onTouchStart={(e) => {
                            e.preventDefault();
                            startDrawing(getTouchPos(e));
                        }}
                        onTouchMove={(e) => {
                            e.preventDefault();
                            draw(getTouchPos(e));
                        }}
                        onTouchEnd={(e) => {
                            e.preventDefault();
                            endDrawing();
                        }}
                    />

                    {/* 烟花画布 */}
                    <canvas
                        ref={fireworksCanvasRef}
                        className="absolute inset-0 pointer-events-none"
                        style={{ zIndex: 2 }}
                    />

                    <div className="absolute bottom-6 left-1/2 transform -translate-x-1/2 bg-white rounded-lg shadow-md px-6 py-3 text-gray-600 text-sm z-10">
                        点击并拖动来画圆 • 尽量闭合圆形 • 越接近完美圆，分数越高！
                    </div>
                </div>
            );
        }

        ReactDOM.render(<PerfectCircleGame />, document.getElementById('root'));
    </script>
</body>
</html>

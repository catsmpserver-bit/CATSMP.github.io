<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CatSMP — Minecraft Server</title>
    <link href="https://fonts.googleapis.com/css2?family=Cinzel+Decorative:wght@700;900&family=Exo+2:wght@300;400;600;700&display=swap" rel="stylesheet">
    <style>
        :root {
            --bg:#f5f7fa; --bg2:#eef1f6; --bg3:#e2e8f2;
            --card:#ffffff; --card2:#f8fafd;
            --border:#d4dce8; --border2:#c0cfe0;
            --accent:#3b7dd8; --accent2:#5b9cf6; --accent-soft:#dbeafe;
            --text:#1e293b; --text2:#475569; --text3:#94a3b8;
            --gold:#f59e0b;
            --shadow:0 2px 16px rgba(59,125,216,0.10);
            --shadow2:0 8px 32px rgba(59,125,216,0.13);
        }

        [data-theme="dark"] {
            --bg:#0f1117; --bg2:#161b27; --bg3:#1e2536;
            --card:#1a2030; --card2:#151c2c;
            --border:#2a3550; --border2:#374060;
            --accent:#5b9cf6; --accent2:#7cb4ff; --accent-soft:#1e3a6e;
            --text:#e8f0fe; --text2:#9ab0cc; --text3:#5a7a99;
            --shadow:0 2px 16px rgba(0,0,0,0.35);
            --shadow2:0 8px 32px rgba(0,0,0,0.4);
        }

        * { margin:0; padding:0; box-sizing:border-box; cursor:none !important; }

        body {
            background: var(--bg);
            color: var(--text);
            font-family: 'Exo 2', sans-serif;
            min-height: 100vh;
            overflow-x: hidden;
            transition: background .3s, color .3s;
        }

        /* Фоновый паттерн */
        body::before {
            content: ''; position: fixed; inset: 0; z-index: 0; pointer-events: none;
            background-image: radial-gradient(circle, rgba(91,156,246,0.07) 1px, transparent 1px);
            background-size: 28px 28px; opacity: .6;
        }
        [data-theme='dark'] body::before { opacity: .22; }

        /* Кастомные курсоры */
        #cursor {
            width: 26px; height: 26px; border-radius: 50%;
            border: 2px solid var(--accent); position: fixed; pointer-events: none; z-index: 99999;
            transform: translate(-50%,-50%);
            background: radial-gradient(circle, rgba(91,156,246,0.18) 0%, transparent 70%);
            box-shadow: 0 0 10px rgba(91,156,246,0.3);
            transition: width .18s, height .18s, background .18s, opacity .2s;
        }
        #cursor::after { content: ''; position: absolute; width: 5px; height: 5px; background: var(--accent); border-radius: 50%; top: 50%; left: 50%; transform: translate(-50%,-50%); }
        #cursor-trail { width: 6px; height: 6px; border-radius: 50%; background: var(--accent2); position: fixed; pointer-events: none; z-index: 99998; transform: translate(-50%,-50%); opacity: 0.4; transition: left .16s ease, top .16s ease; }

        /* Навигация */
        nav {
            position: fixed; top: 0; left: 0; right: 0; z-index: 1000; display: flex; align-items: center; justify-content: space-between;
            padding: 14px 40px; background: rgba(245,247,250,0.92); backdrop-filter: blur(12px);
            border-bottom: 1px solid var(--border); transition: background .3s, border-color .3s;
        }
        [data-theme='dark'] nav { background: rgba(15,17,23,0.92); }
        .nav-logo { font-family: 'Cinzel Decorative', serif; font-size: 1.5rem; font-weight: 900; color: var(--accent); letter-spacing: 3px; text-decoration: none; }
        .nav-link { font-weight: 600; font-size: 0.78rem; text-transform: uppercase; color: var(--text2); text-decoration: none; padding: 7px 13px; border-radius: 8px; transition: all .25s; }
        .nav-link:hover, .nav-link.active { color: var(--accent); background: var(--accent-soft); }

        /* Секции */
        .section { display: none; position: relative; z-index: 1; min-height: 100vh; padding: 110px 0 80px; }
        .section.active { display: flex; flex-direction: column; align-items: center; justify-content: center; }

        /* Герой-блок */
        .hero-title { font-family: 'Cinzel Decorative', serif; font-size: clamp(3rem, 8.5vw, 6.5rem); text-align: center; margin-bottom: 14px; }
        .hero-title span { color: var(--accent); }
        .hero-ip { display: inline-flex; align-items: center; gap: 14px; background: var(--card); border: 1.5px solid var(--border); border-radius: 12px; padding: 13px 26px; box-shadow: var(--shadow); }
        .copy-btn { background: var(--accent-soft); border: 1px solid rgba(91,156,246,0.3); color: var(--accent); border-radius: 7px; padding: 5px 12px; transition: all .2s; }

        /* Анимации */
        @keyframes fadeUp { from { opacity: 0; transform: translateY(24px); } to { opacity: 1; transform: translateY(0); } }

        /* Темная тема кнопка */
        .theme-btn { position: fixed; bottom: 28px; right: 28px; z-index: 2000; width: 48px; height: 48px; border-radius: 50%; background: var(--card); border: 1.5px solid var(--border2); display: flex; align-items: center; justify-content: center; }

        @media(max-width: 768px) {
            nav { padding: 12px 14px; }
            .nav-link { font-size: 0.68rem; padding: 6px 8px; }
        }
    </style>
</head>
<body>

    <div id="cursor"></div>
    <div id="cursor-trail"></div>

    <button class="theme-btn" onclick="toggleTheme()" id="themeBtn">🌙</button>

    <nav id="mainNav">
        <a href="#" class="nav-logo" onclick="showSection('home')">Cat<span>SMP</span></a>
        <div class="nav-links">
            <a href="#" class="nav-link active" onclick="showSection('home')">Главная</a>
            <a href="#" class="nav-link" onclick="showSection('about')">О сервере</a>
            <a href="https://discord.gg/QjUfawNQW" target="_blank" class="nav-link" style="color:var(--gold);">Discord</a>
        </div>
    </nav>

    <section id="home" class="section active">
        <h1 class="hero-title">Cat<span>SMP</span></h1>
        <p style="margin-bottom: 30px; color: var(--text2);">Добро пожаловать на лучший Minecraft SMP сервер с модами Create!</p>
        <div class="hero-ip">
            <code>catsmp.net</code>
            <button class="copy-btn" onclick="copyIP()">Копировать IP</button>
        </div>
    </section>

    <section id="about" class="section">
        <h2 class="hero-title">О сервере</h2>
        <p style="max-width: 600px; text-align: center; color: var(--text2);">CatSMP — это уникальное выживание с упором на механизмы, авиацию и политическое взаимодействие игроков.</p>
    </section>

    <script>
        // Логика курсора
        const cursor = document.getElementById('cursor');
        const trail = document.getElementById('cursor-trail');
        document.addEventListener('mousemove', (e) => {
            cursor.style.left = e.clientX + 'px';
            cursor.style.top = e.clientY + 'px';
            setTimeout(() => {
                trail.style.left = e.clientX + 'px';
                trail.style.top = e.clientY + 'px';
            }, 50);
        });

        // Переключение секций
        function showSection(id) {
            document.querySelectorAll('.section').forEach(s => s.classList.remove('active'));
            document.getElementById(id).classList.add('active');
            document.querySelectorAll('.nav-link').forEach(l => l.classList.remove('active'));
            event.target.classList.add('active');
        }

        // Копирование IP
        function copyIP() {
            navigator.clipboard.writeText('catsmp.net');
            alert('IP скопирован!');
        }

        // Темная тема
        function toggleTheme() {
            const body = document.documentElement;
            const isDark = body.getAttribute('data-theme') === 'dark';
            body.setAttribute('data-theme', isDark ? 'light' : 'dark');
            document.getElementById('themeBtn').innerText = isDark ? '🌙' : '☀️';
        }
    </script>
</body>
</html>

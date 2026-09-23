<!DOCTYPE html>
<html lang="nl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>VogelSpot Kiosk Dashboard</title>
    <link href="https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@400;500;600;700;800&family=Playfair+Display:wght@700&display=swap" rel="stylesheet">
    <style>
        :root {
            --bg-dark: #0b130e;
            --card-bg: rgba(18, 32, 23, 0.85);
            --card-border: rgba(64, 115, 81, 0.25);
            --accent-green: #2ecc71;
            --accent-emerald: #10b981;
            --accent-gold: #f59e0b;
            --text-light: #ecfdf5;
            --text-muted: #809a8a;
            --font-sans: 'Plus Jakarta Sans', sans-serif;
            --font-serif: 'Playfair Display', serif;
        }

        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
        }

        body {
            background-color: var(--bg-dark);
            color: var(--text-light);
            font-family: var(--font-sans);
            height: 100vh;
            overflow: hidden;
            display: flex;
            flex-direction: column;
        }

        /* Top Bar / Header */
        .header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 1rem 2rem;
            background: rgba(8, 16, 11, 0.95);
            border-bottom: 1px solid var(--card-border);
            z-index: 10;
        }

        .brand {
            display: flex;
            align-items: center;
            gap: 0.75rem;
        }

        .brand-icon {
            width: 40px;
            height: 40px;
            background: linear-gradient(135deg, var(--accent-green), var(--accent-emerald));
            border-radius: 12px;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 1.3rem;
        }

        .brand-title {
            font-family: var(--font-serif);
            font-size: 1.4rem;
            font-weight: 700;
            letter-spacing: -0.5px;
        }

        .brand-title span {
            color: var(--accent-green);
        }

        .kiosk-status {
            display: flex;
            align-items: center;
            gap: 1.5rem;
        }

        .slide-indicator {
            display: flex;
            align-items: center;
            gap: 0.5rem;
            background: rgba(255, 255, 255, 0.05);
            padding: 0.4rem 1rem;
            border-radius: 20px;
            font-weight: 600;
            font-size: 0.85rem;
            border: 1px solid var(--card-border);
        }

        .dot {
            width: 8px;
            height: 8px;
            border-radius: 50%;
            background: var(--accent-green);
            box-shadow: 0 0 10px var(--accent-green);
            animation: pulse 2s infinite;
        }

        @keyframes pulse {
            0% { opacity: 0.4; }
            50% { opacity: 1; }
            100% { opacity: 0.4; }
        }

        .clock-container {
            text-align: right;
        }

        .clock-time {
            font-size: 1.4rem;
            font-weight: 800;
            font-family: monospace;
            color: #fff;
            letter-spacing: 1px;
        }

        .clock-date {
            font-size: 0.75rem;
            color: var(--text-muted);
            text-transform: capitalize;
        }

        /* Progress Bar */
        .progress-bar-container {
            width: 100%;
            height: 4px;
            background: rgba(255, 255, 255, 0.05);
            position: relative;
        }

        .progress-bar {
            height: 100%;
            width: 0%;
            background: linear-gradient(90deg, var(--accent-green), var(--accent-emerald));
            transition: width 0.1s linear;
        }

        /* Slide Viewer Area */
        .slides-wrapper {
            flex: 1;
            position: relative;
            width: 100%;
            height: calc(100vh - 70px);
            overflow: hidden;
        }

        .slide {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            opacity: 0;
            visibility: hidden;
            transition: opacity 0.8s ease-in-out, transform 0.8s ease-in-out;
            transform: scale(0.98);
            padding: 2rem;
            display: flex;
            flex-direction: column;
            gap: 1.5rem;
        }

        .slide.active {
            opacity: 1;
            visibility: visible;
            transform: scale(1);
        }

        /* Slide 1 Layout - Live Stream */
        .stream-grid {
            display: grid;
            grid-template-columns: 2fr 1fr;
            gap: 1.5rem;
            height: 100%;
        }

        .video-box {
            background: #000;
            border-radius: 16px;
            overflow: hidden;
            position: relative;
            border: 1px solid var(--card-border);
            box-shadow: 0 20px 40px rgba(0, 0, 0, 0.5);
            display: flex;
            flex-direction: column;
        }

        .video-container {
            width: 100%;
            height: 100%;
            position: relative;
        }

        .video-container iframe {
            width: 100%;
            height: 100%;
            border: none;
        }

        .cam-overlay {
            position: absolute;
            top: 1rem;
            left: 1rem;
            right: 1rem;
            display: flex;
            justify-content: space-between;
            align-items: center;
            pointer-events: none;
            z-index: 5;
        }

        .badge-live {
            background: rgba(220, 38, 38, 0.9);
            color: white;
            padding: 0.3rem 0.8rem;
            border-radius: 6px;
            font-size: 0.75rem;
            font-weight: 800;
            letter-spacing: 1px;
            display: flex;
            align-items: center;
            gap: 0.5rem;
            backdrop-filter: blur(4px);
        }

        .cam-info {
            background: rgba(0, 0, 0, 0.6);
            backdrop-filter: blur(8px);
            padding: 0.3rem 0.8rem;
            border-radius: 6px;
            font-size: 0.75rem;
            font-weight: 600;
            color: #fff;
            border: 1px solid rgba(255, 255, 255, 0.1);
        }

        /* Feed Side Column */
        .spotters-panel {
            background: var(--card-bg);
            border: 1px solid var(--card-border);
            border-radius: 16px;
            padding: 1.5rem;
            display: flex;
            flex-direction: column;
            gap: 1rem;
            backdrop-filter: blur(12px);
        }

        .panel-title {
            font-size: 1.1rem;
            font-weight: 700;
            color: var(--accent-green);
            display: flex;
            align-items: center;
            justify-content: space-between;
            border-bottom: 1px solid var(--card-border);
            padding-bottom: 0.75rem;
        }

        .spot-list {
            list-style: none;
            display: flex;
            flex-direction: column;
            gap: 0.75rem;
            overflow-y: auto;
        }

        .spot-item {
            background: rgba(255, 255, 255, 0.03);
            border: 1px solid rgba(255, 255, 255, 0.05);
            padding: 0.8rem 1rem;
            border-radius: 10px;
            display: flex;
            align-items: center;
            gap: 1rem;
        }

        .spot-icon {
            font-size: 1.5rem;
            background: rgba(46, 204, 113, 0.1);
            width: 40px;
            height: 40px;
            border-radius: 8px;
            display: flex;
            align-items: center;
            justify-content: center;
        }

        .spot-details h4 {
            font-size: 0.95rem;
            font-weight: 600;
        }

        .spot-details p {
            font-size: 0.75rem;
            color: var(--text-muted);
        }

        .spot-time {
            margin-left: auto;
            font-size: 0.75rem;
            color: var(--accent-green);
            font-weight: 600;
        }

        /* Slide 2: Waarnemingen */
        .waarneming-container {
            background: var(--card-bg);
            border: 1px solid var(--card-border);
            border-radius: 16px;
            padding: 2rem;
            height: 100%;
            display: flex;
            flex-direction: column;
            gap: 1.5rem;
            backdrop-filter: blur(12px);
        }

        .obs-table {
            width: 100%;
            border-collapse: collapse;
            text-align: left;
        }

        .obs-table th {
            padding: 1rem;
            color: var(--text-muted);
            font-size: 0.85rem;
            border-bottom: 1px solid var(--card-border);
            text-transform: uppercase;
            letter-spacing: 0.5px;
        }

        .obs-table td {
            padding: 1.2rem 1rem;
            border-bottom: 1px solid rgba(255, 255, 255, 0.05);
            font-size: 1rem;
        }

        .badge-rarity {
            padding: 0.25rem 0.6rem;
            border-radius: 6px;
            font-size: 0.75rem;
            font-weight: 700;
        }

        .rarity-high {
            background: rgba(245, 158, 11, 0.2);
            color: var(--accent-gold);
            border: 1px solid rgba(245, 158, 11, 0.4);
        }

        .rarity-med {
            background: rgba(16, 185, 129, 0.2);
            color: var(--accent-emerald);
            border: 1px solid rgba(16, 185, 129, 0.4);
        }

        /* Slide 3: Vogel van de dag */
        .spotlight-grid {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 2rem;
            height: 100%;
            align-items: center;
        }

        .bird-card {
            background: var(--card-bg);
            border: 1px solid var(--card-border);
            border-radius: 20px;
            padding: 2.5rem;
            display: flex;
            flex-direction: column;
            gap: 1.5rem;
            height: 100%;
            justify-content: center;
        }

        .bird-image-box {
            position: relative;
            height: 100%;
            border-radius: 20px;
            overflow: hidden;
            border: 1px solid var(--card-border);
            box-shadow: 0 15px 30px rgba(0, 0, 0, 0.4);
        }

        .bird-image-box img {
            width: 100%;
            height: 100%;
            object-fit: cover;
        }

        .bird-title-tag {
            font-size: 0.85rem;
            color: var(--accent-green);
            text-transform: uppercase;
            letter-spacing: 1.5px;
            font-weight: 700;
        }

        .bird-name {
            font-family: var(--font-serif);
            font-size: 2.5rem;
            font-weight: 700;
        }

        .bird-latin {
            font-style: italic;
            color: var(--text-muted);
            margin-top: -0.5rem;
            font-size: 1.1rem;
        }

        .bird-traits {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 1rem;
            margin-top: 1rem;
        }

        .trait-box {
            background: rgba(255, 255, 255, 0.03);
            border: 1px solid rgba(255, 255, 255, 0.05);
            padding: 1rem;
            border-radius: 12px;
        }

        .trait-label {
            font-size: 0.75rem;
            color: var(--text-muted);
            margin-bottom: 0.3rem;
        }

        .trait-value {
            font-weight: 600;
            font-size: 0.95rem;
        }
    </style>
</head>
<body>

    <!-- Header -->
    <header class="header">
        <div class="brand">
            <div class="brand-icon">🐤</div>
            <div class="brand-title">VogelSpot <span>KIOSK</span></div>
        </div>

        <div class="kiosk-status">
            <div class="slide-indicator">
                <div class="dot"></div>
                <span id="slide-name">SLIDE 1: LIVE VOERSTATION</span>
            </div>
        </div>

        <div class="clock-container">
            <div class="clock-time" id="clock-time">00:00:00</div>
            <div class="clock-date" id="clock-date">Laden...</div>
        </div>
    </header>

    <!-- Progress Bar -->
    <div class="progress-bar-container">
        <div class="progress-bar" id="progress-bar"></div>
    </div>

    <!-- Main Content Area -->
    <main class="slides-wrapper">

        <!-- Slide 1: Live Voerstation Stream -->
        <section class="slide active" id="slide-1">
            <div class="stream-grid">
                <div class="video-box">
                    <div class="cam-overlay">
                        <div class="badge-live">
                            <span style="width:8px;height:8px;background:#fff;border-radius:50%;display:inline-block;"></span>
                            YOUTUBE LIVE 4K
                        </div>
                        <div class="cam-info">NATURETEC BIRD CAM</div>
                    </div>
                    <div class="video-container">
                        <iframe 
                            src="https://www.youtube-nocookie.com/embed/4kRzwJXaeIM?autoplay=1&mute=1&controls=0&loop=1&playlist=4kRzwJXaeIM&enablejsapi=1" 
                            title="NatureTec 4K Bird Feeder Stream"
                            allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
                            referrerpolicy="strict-origin-when-cross-origin"
                            allowfullscreen>
                        </iframe>
                    </div>
                </div>

                <div class="spotters-panel">
                    <div class="panel-title">
                        <span>Zojuist Gespot op Cam</span>
                        <span style="font-size: 0.8rem; color: var(--accent-green);">● Live Update</span>
                    </div>

                    <ul class="spot-list">
                        <li class="spot-item">
                            <div class="spot-icon">🐦</div>
                            <div class="spot-details">
                                <h4>Koolmees</h4>
                                <p>Op de zonnebloemsilo</p>
                            </div>
                            <div class="spot-time">Zojuist</div>
                        </li>
                        <li class="spot-item">
                            <div class="spot-icon">🐤</div>
                            <div class="spot-details">
                                <h4>Roodborst</h4>
                                <p>Pikt meelwormen op plank</p>
                            </div>
                            <div class="spot-time">2 min geleden</div>
                        </li>
                        <li class="spot-item">
                            <div class="spot-icon">🫐</div>
                            <div class="spot-details">
                                <h4>Pimpelmees</h4>
                                <p>Acrobatisch aan de vetbol</p>
                            </div>
                            <div class="spot-time">5 min geleden</div>
                        </li>
                        <li class="spot-item">
                            <div class="spot-icon">🪵</div>
                            <div class="spot-details">
                                <h4>Grote Bonte Specht</h4>
                                <p>Kort op de pindakaashouder</p>
                            </div>
                            <div class="spot-time">12 min geleden</div>
                        </li>
                    </ul>
                </div>
            </div>
        </section>

        <!-- Slide 2: Waarneming.nl Feeds -->
        <section class="slide" id="slide-2">
            <div class="waarneming-container">
                <div class="panel-title" style="font-size: 1.4rem;">
                    <span>Laatste Waarnemingen (Regio Groningen / Delfzijl)</span>
                    <span style="font-size: 0.85rem; color: var(--text-muted);">Bron: Waarneming.nl Feed</span>
                </div>

                <table class="obs-table">
                    <thead>
                        <tr>
                            <th>Soort</th>
                            <th>Aantal</th>
                            <th>Locatie</th>
                            <th>Tijdstip</th>
                            <th>Status</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr>
                            <td style="font-weight: 700;">IJsvogel (Alcedo atthis)</td>
                            <td>1x</td>
                            <td>Eemskanaal / Delfzijl</td>
                            <td>14:12 uur</td>
                            <td><span class="badge-rarity rarity-high">Zeldzaam</span></td>
                        </tr>
                        <tr>
                            <td style="font-weight: 700;">Slechtvalk (Falco peregrinus)</td>
                            <td>2x</td>
                            <td>Industriepark Delfzijl</td>
                            <td>13:45 uur</td>
                            <td><span class="badge-rarity rarity-high">Vrij Zeldzaam</span></td>
                        </tr>
                        <tr>
                            <td style="font-weight: 700;">Grote Zilverreiger</td>
                            <td>3x</td>
                            <td>Polder Schildmeer</td>
                            <td>12:30 uur</td>
                            <td><span class="badge-rarity rarity-med">Algemeen</span></td>
                        </tr>
                        <tr>
                            <td style="font-weight: 700;">Putter / Distelvink</td>
                            <td>6x</td>
                            <td>Tuinbuurt Eemsdelta</td>
                            <td>11:15 uur</td>
                            <td><span class="badge-rarity rarity-med">Algemeen</span></td>
                        </tr>
                    </tbody>
                </table>
            </div>
        </section>

        <!-- Slide 3: Vogel van de Dag -->
        <section class="slide" id="slide-3">
            <div class="spotlight-grid">
                <div class="bird-card">
                    <div class="bird-title-tag">Vogel van de Dag Spotlight</div>
                    <div>
                        <h2 class="bird-name">Roodborst</h2>
                        <div class="bird-latin">Erithacus rubecula</div>
                    </div>
                    <p style="color: var(--text-muted); line-height: 1.6;">
                        De roodborst is een nieuwsgierige en moedige tuinvogel. Ze zijn erg territoriaal ingesteld en komen graag op de voederplank af als er meelwormen of ongezouten pinda's liggen.
                    </p>
                    <div class="bird-traits">
                        <div class="trait-box">
                            <div class="trait-label">Favoriet Voer</div>
                            <div class="trait-value">Meelwormen & Zaden</div>
                        </div>
                        <div class="trait-box">
                            <div class="trait-label">Herkenning</div>
                            <div class="trait-value">Oranjerode borst</div>
                        </div>
                    </div>
                </div>

                <div class="bird-image-box">
                    <img src="https://images.unsplash.com/photo-1544644181-1484b3fdfc62?auto=format&fit=crop&w=1000&q=80" alt="Roodborst">
                </div>
            </div>
        </section>

    </main>

    <script>
        // Clock Logic
        function updateClock() {
            const now = new Date();
            const hours = String(now.getHours()).padStart(2, '0');
            const minutes = String(now.getMinutes()).padStart(2, '0');
            const seconds = String(now.getSeconds()).padStart(2, '0');
            
            document.getElementById('clock-time').textContent = `${hours}:${minutes}:${seconds}`;
            
            const options = { weekday: 'long', day: 'numeric', month: 'short' };
            document.getElementById('clock-date').textContent = now.toLocaleDateString('nl-NL', options);
        }
        setInterval(updateClock, 1000);
        updateClock();

        // Automatic Slide Rotation System
        const slides = [
            { id: 'slide-1', title: 'SLIDE 1: LIVE 4K VOERSTATION' },
            { id: 'slide-2', title: 'SLIDE 2: LAATSTE WAARNEMINGEN' },
            { id: 'slide-3', title: 'SLIDE 3: VOGEL VAN DE DAG' }
        ];
        
        let currentSlideIndex = 0;
        const slideDuration = 15000; // 15 seconden per slide
        let progressInterval = null;
        let startTime = Date.now();

        function switchSlide() {
            // Unset active class from all
            document.querySelectorAll('.slide').forEach(slide => slide.classList.remove('active'));
            
            // Increment index
            currentSlideIndex = (currentSlideIndex + 1) % slides.length;
            
            // Set active class
            const current = slides[currentSlideIndex];
            document.getElementById(current.id).classList.add('active');
            document.getElementById('slide-name').textContent = current.title;

            // Reset progress bar
            startTime = Date.now();
        }

        function updateProgressBar() {
            const elapsedTime = Date.now() - startTime;
            const percentage = Math.min((elapsedTime / slideDuration) * 100, 100);
            document.getElementById('progress-bar').style.width = percentage + '%';

            if (elapsedTime >= slideDuration) {
                switchSlide();
            }
        }

        // Start kiosk loop
        startTime = Date.now();
        setInterval(updateProgressBar, 100);
    </script>
</body>
</html>

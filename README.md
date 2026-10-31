# E-remiza

<!DOCTYPE html>
<html lang="pl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>System E-Remiza</title>
    <style>
        :root { --bg-color: #0f172a; --card-color: #1e293b; --accent-red: #ef4444; --accent-green: #10b981; --text-color: #f8fafc; --input-bg: #334155; }
        body { font-family: 'Segoe UI', system-ui, sans-serif; background-color: var(--bg-color); color: var(--text-color); margin: 0; padding: 20px; display: flex; flex-direction: column; align-items: center; min-height: 100vh; }
        .container { width: 100%; max-width: 500px; text-align: center; }
        .card { background-color: var(--card-color); padding: 25px; border-radius: 15px; box-shadow: 0 10px 25px rgba(0,0,0,0.4); margin-top: 20px; }
        .hidden { display: none !important; }
        .btn { background-color: #3b82f6; color: white; border: none; padding: 12px 24px; font-size: 1rem; font-weight: bold; border-radius: 8px; cursor: pointer; width: 100%; margin: 10px 0; transition: transform 0.1s; }
        .btn:active { transform: scale(0.98); }
        .btn-red { background-color: var(--accent-red); }
        .btn-green { background-color: var(--accent-green); }
        .btn-outline { background: transparent; border: 2px solid #475569; color: #94a3b8; }
        input, textarea { width: 100%; padding: 10px; margin: 8px 0 16px 0; background-color: var(--input-bg); border: 1px solid #475569; border-radius: 6px; color: white; box-sizing: border-box; }
        label { display: block; text-align: left; font-size: 0.9rem; color: #94a3b8; }
        .status-badge { padding: 8px; border-radius: 6px; font-weight: bold; margin-bottom: 15px; }
        .status-active { background-color: rgba(16, 185, 129, 0.2); color: var(--accent-green); }
        .status-inactive { background-color: rgba(239, 68, 68, 0.2); color: var(--accent-red); }
        .alarm-alert { background-color: var(--accent-red); color: white; animation: blink 1s infinite alternate; padding: 20px; border-radius: 10px; font-size: 1.2rem; font-weight: bold; margin-top: 15px; }
        @keyframes blink { 0% { opacity: 0.6; box-shadow: 0 0 10px var(--accent-red); } 100% { opacity: 1; box-shadow: 0 0 30px var(--accent-red); } }
    </style>
</head>
<body>
    <div class="container">
        <h1>🚨 System E-Remiza</h1>
        <div id="screen-main" class="card">
            <h3>Wybierz panel operacyjny:</h3>
            <button class="btn" onclick="switchScreen('screen-login')">1 - Panel Dyżurnego</button>
            <button class="btn btn-outline" onclick="switchScreen('screen-remiza')">2 - Panel E-Remiza</button>
        </div>
        <div id="screen-login" class="card hidden">
            <h3>Logowanie Dyżurnego</h3>
            <label>Login:</label> <input type="text" id="login-user" value="TEST">
            <label>Hasło:</label> <input type="password" id="login-pass" value="TEST">
            <button class="btn btn-green" onclick="handleLogin()">Zaloguj się</button>
            <button class="btn btn-outline" onclick="switchScreen('screen-main')">Powrót</button>
        </div>
        <div id="screen-dyzurny" class="card hidden">
            <h3>Panel Dyżurnego - Karta Zdarzenia</h3>
            <label>Lokalizacja zdarzenia:</label> <input type="text" id="event-location" value="Chocznia, ul. Główna">
            <label>Opis / Szczegóły:</label> <textarea id="event-desc" rows="3">Zgłoszenie alarmowe</textarea>
            <button class="btn btn-red" onclick="sendAlarm()">🚨 WYŚLIJ ALARM DO REMIZY</button>
            <button class="btn btn-outline" onclick="switchScreen('screen-main')">Powrót</button>
        </div>
        <div id="screen-remiza" class="card hidden">
            <h3>Panel Odbiorczy E-Remiza</h3>
            <div id="remiza-status" class="status-badge status-inactive">Status: Nieaktywna</div>
            <button class="btn btn-green" id="btn-activate-remiza" onclick="toggleRemiza()">Aktywuj E-Remizę</button>
            <div id="alarm-box" class="hidden">
                <div class="alarm-alert">🔥 ALARM BOJOWY! 🔥</div>
                <div class="card" style="background: #020617; text-align: left; margin-top: 10px;">
                    <p><strong>Miejsce:</strong> <span id="alarm-display-loc"></span></p>
                    <p><strong>Opis:</strong> <span id="alarm-display-desc"></span></p>
                </div>
                <button class="btn btn-outline" onclick="silenceAlarm()">Wyłącz i wyczyść alarm</button>
            </div>
            <button class="btn btn-outline" onclick="switchScreen('screen-main')" style="margin-top: 20px;">Powrót</button>
        </div>
    </div>
    <script>
        const alarmSpeech = new SpeechSynthesisUtterance("Uwaga! Alarm bojowy dla jednostki e remiza!");
        alarmSpeech.lang = "pl-PL";
        alarmSpeech.rate = 1.0;

        let zapisaneZgłoszenie = null;
        let isRemizaActive = false;
        let speechInterval = null;

        function switchScreen(screenId) {
            document.getElementById('screen-main').classList.add('hidden');
            document.getElementById('screen-login').classList.add('hidden');
            document.getElementById('screen-dyzurny').classList.add('hidden');
            document.getElementById('screen-remiza').classList.add('hidden');
            document.getElementById(screenId).classList.remove('hidden');
            if (screenId === 'screen-remiza' && isRemizaActive && zapisaneZgłoszenie) { pokazIodtworzAlarm(); }
        }

        function handleLogin() { switchScreen('screen-dyzurny'); }

        function sendAlarm() {
            zapisaneZgłoszenie = {
                location: document.getElementById('event-location').value,
                desc: document.getElementById('event-desc').value
            };
            alert("Alarm wysłany!");
            switchScreen('screen-main');
        }

        function toggleRemiza() {
            const statusBadge = document.getElementById('remiza-status');
            const btn = document.getElementById('btn-activate-remiza');
            if (!isRemizaActive) {
                isRemizaActive = true;
                statusBadge.textContent = "Status: Aktywna (Czuwanie włączone)"; statusBadge.className = "status-badge status-active";
                btn.textContent = "Dezaktywuj E-Remizę"; btn.className = "btn btn-red";
                window.speechSynthesis.speak(new SpeechSynthesisUtterance("System aktywowany"));
                if (zapisaneZgłoszenie) { pokazIodtworzAlarm(); }
            } else {
                isRemizaActive = false;
                statusBadge.textContent = "Status: Nieaktywna"; statusBadge.className = "status-badge status-inactive";
                btn.textContent = "Aktywuj E-Remizę"; btn.className = "btn btn-green";
                silenceAlarm();
            }
        }

        function pokazIodtworzAlarm() {
            document.getElementById('alarm-display-loc').textContent = zapisaneZgłoszenie.location;
            document.getElementById('alarm-display-desc').textContent = zapisaneZgłoszenie.desc;
            document.getElementById('alarm-box').classList.remove('hidden');
            if(!speechInterval) {
                window.speechSynthesis.speak(alarmSpeech);
                speechInterval = setInterval(() => { window.speechSynthesis.speak(alarmSpeech); }, 4000);
            }
        }

        function silenceAlarm() {
            clearInterval(speechInterval); speechInterval = null; window.speechSynthesis.cancel();
            document.getElementById('alarm-box').classList.add('hidden'); zapisaneZgłoszenie = null;
        }
    </script>
</body>
</html>

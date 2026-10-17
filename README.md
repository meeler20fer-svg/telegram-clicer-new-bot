# telegram-clicer-new-bot
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>Telegram Tap Clicker</title>
    <!-- Подключаем SDK Telegram Web App -->
    <script src="https://telegram.org/js/telegram-web-app.js"></script>
    <style>
        * {
            box-sizing: border-box;
            user-select: none;
            -webkit-user-select: none;
        }
        body {
            margin: 0;
            padding: 0;
            background-color: #12131C;
            color: #ffffff;
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: space-between;
            height: 100vh;
            overflow: hidden;
        }
        .header {
            margin-top: 20px;
            text-align: center;
        }
        .score-container {
            font-size: 32px;
            font-weight: bold;
            display: flex;
            align-items: center;
            justify-content: center;
            gap: 8px;
        }
        .click-area {
            display: flex;
            justify-content: center;
            align-items: center;
            position: relative;
            width: 100%;
        }
        .tap-button {
            width: 200px;
            height: 200px;
            border-radius: 50%;
            background: radial-gradient(circle, #ff007f 0%, #7928ca 100%);
            border: 6px solid rgba(255, 255, 255, 0.2);
            box-shadow: 0 0 30px rgba(255, 0, 127, 0.5);
            color: #fff;
            font-size: 24px;
            font-weight: bold;
            cursor: pointer;
            outline: none;
            transition: transform 0.05s ease;
            display: flex;
            align-items: center;
            justify-content: center;
            text-shadow: 0 2px 4px rgba(0,0,0,0.5);
        }
        .tap-button:active {
            transform: scale(0.92);
        }
        .floating-text {
            position: absolute;
            font-size: 24px;
            font-weight: bold;
            color: #00ffcc;
            pointer-events: none;
            animation: floatUp 0.8s ease-out forwards;
        }
        @keyframes floatUp {
            0% {
                opacity: 1;
                transform: translateY(0) scale(1);
            }
            100% {
                opacity: 0;
                transform: translateY(-80px) scale(1.2);
            }
        }
        .upgrades-container {
            width: 100%;
            background: #1a1c29;
            border-top-left-radius: 20px;
            border-top-right-radius: 20px;
            padding: 20px;
            box-shadow: 0 -4px 15px rgba(0,0,0,0.3);
        }
        .upgrade-card {
            display: flex;
            justify-content: space-between;
            align-items: center;
            background: #252838;
            padding: 12px 16px;
            border-radius: 12px;
            margin-bottom: 10px;
        }
        .upgrade-info {
            display: flex;
            flex-direction: column;
        }
        .upgrade-title {
            font-weight: bold;
            font-size: 16px;
        }
        .upgrade-desc {
            font-size: 12px;
            color: #a0a0a0;
        }
        .buy-btn {
            background: #0088cc;
            border: none;
            color: white;
            padding: 10px 16px;
            border-radius: 8px;
            font-weight: bold;
            cursor: pointer;
        }
        .buy-btn:disabled {
            background: #444;
            color: #888;
            cursor: not-allowed;
        }
    </style>
</head>
<body>

    <div class="header">
        <div class="score-container">
            🪙 <span id="coins">0</span>
        </div>
        <div style="font-size: 14px; color: #a0a0a0; margin-top: 5px;">
            Сила клика: +<span id="click-power">1</span>
        </div>
    </div>

    <div class="click-area" id="clickArea">
        <button class="tap-button" id="tapBtn">ТАПАЙ!</button>
    </div>

    <div class="upgrades-container">
        <h3 style="margin-top: 0; margin-bottom: 12px;">Улучшения</h3>
        
        <div class="upgrade-card">
            <div class="upgrade-info">
                <span class="upgrade-title" id="upgradeName">Медная кнопка</span>
                <span class="upgrade-desc">+1 монета за клик</span>
            </div>
            <button class="buy-btn" id="buyUpgradeBtn">
                <span id="upgradeCost">10</span> 🪙
            </button>
        </div>
    </div>

    <script>
        // Инициализация Telegram WebApp
        const tg = window.Telegram?.WebApp;
        if (tg) {
            tg.expand(); // Разворачиваем на весь экран
        }

        // Данные игры
        let state = {
            coins: 0,
            clickPower: 1,
            upgradeLevel: 0,
            upgradeCost: 10
        };

        const buttonsConfig = [
            { name: "Медная кнопка", power: 1, color: "radial-gradient(circle, #d4af37 0%, #8a5a00 100%)" },
            { name: "Серебряная кнопка", power: 3, color: "radial-gradient(circle, #e0e0e0 0%, #757575 100%)" },
            { name: "Золотая кнопка", power: 5, color: "radial-gradient(circle, #ffd700 0%, #ff8c00 100%)" },
            { name: "Неоновая кнопка", power: 10, color: "radial-gradient(circle, #00f2fe 0%, #4facfe 100%)" },
            { name: "Алмазная кнопка", power: 25, color: "radial-gradient(circle, #a8ff78 0%, #78ffd6 100%)" }
        ];

        // Загрузка сохраненного прогресса
        function loadGame() {
            const saved = localStorage.getItem('clicker_save');
            if (saved) {
                state = JSON.parse(saved);
            }
            updateUI();
        }

        // Сохранение прогресса
        function saveGame() {
            localStorage.setItem('clicker_save', JSON.stringify(state));
        }

        // Элементы интерфейса
        const coinsEl = document.getElementById('coins');
        const clickPowerEl = document.getElementById('click-power');
        const tapBtn = document.getElementById('tapBtn');
        const clickArea = document.getElementById('clickArea');
        const buyUpgradeBtn = document.getElementById('buyUpgradeBtn');
        const upgradeNameEl = document.getElementById('upgradeName');
        const upgradeCostEl = document.getElementById('upgradeCost');

        // Обновление экрана
        function updateUI() {
            coinsEl.innerText = state.coins;
            clickPowerEl.innerText = state.clickPower;
            
            const currentConfig = buttonsConfig[Math.min(state.upgradeLevel, buttonsConfig.length - 1)];
            upgradeNameEl.innerText = currentConfig.name;
            upgradeCostEl.innerText = state.upgradeCost;
            
            // Заблокировать кнопку покупки, если не хватает денег
            buyUpgradeBtn.disabled = state.coins < state.upgradeCost;
        }

        // Логика клика
        tapBtn.addEventListener('click', (e) => {
            state.coins += state.clickPower;
            
            // Вибрация в Telegram (Haptic Feedback)
            if (tg && tg.HapticFeedback) {
                tg.HapticFeedback.impactOccurred('light');
            }

            // Анимация всплывающей цифры
            showFloatingNumber(e.clientX, e.clientY, `+${state.clickPower}`);

            saveGame();
            updateUI();
        });

        // Создание всплывающего текста при тапе
        function showFloatingNumber(x, y, text) {
            const el = document.createElement('div');
            el.className = 'floating-text';
            el.innerText = text;
            
            const rect = clickArea.getBoundingClientRect();
            el.style.left = `${x - rect.left - 10}px`;
            el.style.top = `${y - rect.top - 20}px`;

            clickArea.appendChild(el);

            setTimeout(() => {
                el.remove();
            }, 800);
        }

        // Покупка улучшения
        buyUpgradeBtn.addEventListener('click', () => {
            if (state.coins >= state.upgradeCost) {
                state.coins -= state.upgradeCost;
                
                const currentConfig = buttonsConfig[Math.min(state.upgradeLevel, buttonsConfig.length - 1)];
                state.clickPower += currentConfig.power;
                
                // Изменяем внешний вид кнопки при покупке
                tapBtn.style.background = currentConfig.color;

                state.upgradeLevel++;
                state.upgradeCost = Math.round(state.upgradeCost * 2.2);

                if (tg && tg.HapticFeedback) {
                    tg.HapticFeedback.notificationOccurred('success');
                }

                saveGame();
                updateUI();
            }
        });

        loadGame();
    </script>
</body>
</html>

#mafiaclub

<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Клуб «Чёрная Орхидея»</title>
    <style>
        :root {
            --bg-dark: #0a0a0a;
            --bg-card: #1a1a1a;
            --text-light: #f0f0f0;
            --text-medium: #b0b0b0;
            --accent-red: #ff4d4d;
            --accent-glow: rgba(255, 77, 77, 0.4);
            --border: #333;
        }

        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: var(--bg-dark);
            color: var(--text-light);
            line-height: 1.6;
            padding: 20px;
        }

        .container {
            max-width: 900px;
            margin: 0 auto;
        }

        header {
            text-align: center;
            margin-bottom: 40px;
            padding: 30px;
            background: linear-gradient(135deg, #1a1a1a 0%, #2a0000 100%);
            border: 1px solid var(--border);
            border-radius: 15px;
            box-shadow: 0 0 20px rgba(255, 77, 77, 0.2);
        }

        h1 {
            font-size: 2.5em;
            margin-bottom: 10px;
            background: linear-gradient(90deg, var(--accent-red), #ff9999);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
        }

        .subtitle {
            font-size: 1.2em;
            color: var(--text-medium);
            font-style: italic;
        }

        .leaderboard {
            display: grid;
            gap: 20px;
        }

        .member-card {
            background: var(--bg-card);
            border: 1px solid var(--border);
            border-radius: 12px;
            padding: 20px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            transition: transform 0.2s, box-shadow 0.2s;
            position: relative;
            overflow: hidden;
        }

        .member-card::before {
            content: '';
            position: absolute;
            top: 0;
            left: -100%;
            width: 100%;
            height: 100%;
            background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.1), transparent);
            transition: left 0.5s;
        }

        .member-card:hover::before {
            left: 100%;
        }

        .member-card:hover {
            transform: translateY(-5px);
            box-shadow: 0 10px 20px var(--accent-glow);
        }

        .member-info {
            display: flex;
            align-items: center;
            gap: 20px;
        }

        .avatar {
            width: 70px;
            height: 70px;
            border-radius: 50%;
            background-size: cover;
            background-position: center;
            border: 3px solid var(--accent-red);
            box-shadow: 0 0 10px var(--accent-glow);
        }

        .details {
            display: flex;
            flex-direction: column;
        }

        .name {
            font-weight: bold;
            font-size: 1.2em;
        }

        .role {
            font-size: 0.9em;
            color: var(--text-medium);
        }

        .rating-controls {
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: 10px;
        }

        .rating-display {
            font-size: 1.8em;
            font-weight: bold;
            color: var(--accent-red);
            min-width: 40px;
            text-align: center;
        }

        .btn {
            padding: 8px 16px;
            border: 1px solid var(--border);
            background: var(--bg-dark);
            color: var(--text-light);
            border-radius: 6px;
            cursor: pointer;
            font-weight: bold;
            transition: all 0.2s;
        }

        .btn:hover:not(:disabled) {
            background: var(--accent-red);
            color: var(--bg-dark);
            border-color: var(--accent-red);
        }

        .btn:disabled {
            opacity: 0.5;
            cursor: not-allowed;
        }

        .btn-decrease {
            order: 1;
        }

        .btn-increase {
            order: 3;
        }

        @media (max-width: 600px) {
            .member-card {
                flex-direction: column;
                gap: 20px;
                text-align: center;
            }
            .member-info {
                flex-direction: column;
            }
            .rating-controls {
                flex-direction: row;
                gap: 15px;
            }
            .btn-decrease { order: 0; }
            .btn-increase { order: 2; }
            .rating-display { order: 1; }
        }
    </style>
</head>
<body>

<div class="container">
    <header>
        <h1>Клуб «Чёрная Орхидея»</h1>
        <div class="subtitle">Совет Семьи — Рейтинг влияния</div>
    </header>

    <div class="leaderboard" id="leaderboard">
        <!-- Карточки участников будут сгенерированы здесь -->
    </div>
</div>

<script>
// Данные участников. Рейтинг можно менять здесь.
const members = [
    { id: 1, name: "Дон Витторе", role: "Крестный отец", rating: 9950, avatar: "https://i.pravatar.cc/150?img=1" },
    { id: 2, name: "Луиджи «Тень» Росси", role: "Консильери", rating: 8420, avatar: "https://i.pravatar.cc/150?img=2" },
    { id: 3, name: "Сальваторе", role: "Капореджиме", rating: 7100, avatar: "https://i.pravatar.cc/150?img=3" },
    { id: 4, name: "Марко «Скальпель»", role: "Адвокат", rating: 6550, avatar: "https://i.pravatar.cc/150?img=4" },
    { id: 5, name: "Энцо", role: "Уличный боец", rating: 5200, avatar: "https://i.pravatar.cc/150?img=5" }
];

// Функция для сохранения данных (на будущее, сейчас работает только в рамках сессии)
function saveData() {
    localStorage.setItem('mafiaClubData', JSON.stringify(members));
}

// Функция для сортировки и отрисовки
function renderBoard() {
    // Сортируем массив по рейтингу (от большего к меньшему)
    const sortedMembers = [...members].sort((a, b) => b.rating - a.rating);
    
    const container = document.getElementById('leaderboard');
    container.innerHTML = '';

    sortedMembers.forEach(member => {
        const card = document.createElement('div');
        card.className = 'member-card';
        card.innerHTML = `
            <div class="member-info">
                <div class="avatar" style="background-image: url('${member.avatar}')"></div>
                <div class="details">
                    <span class="name">${member.name}</span>
                    <span class="role">${member.role}</span>
                </div>
            </div>
            <div class="rating-controls">
                <button class="btn btn-decrease" onclick="changeRating(${member.id}, -100)" title="Понизить">−100</button>
                <div class="rating-display" id="rating-${member.id}">${member.rating}</div>
                <button class="btn btn-increase" onclick="changeRating(${member.id}, 100)" title="Повысить">+100</button>
            </div>
        `;
        container.appendChild(card);
    });
}

// Функция изменения рейтинга
function changeRating(id, amount) {
    const member = members.find(m => m.id === id);
    if (!member) return;

    member.rating += amount;
    // Ограничиваем рейтинг, чтобы он не ушел в минус
    if (member.rating < 0) member.rating = 0;
    
    renderBoard();
    saveData();
}

// Загрузка данных при старте (если они были сохранены ранее)
const savedData = localStorage.getItem('mafiaClubData');
if (savedData) {
    const parsed = JSON.parse(savedData);
    // Обновляем рейтинги в основном массиве
    parsed.forEach(saved => {
        const member = members.find(m => m.id === saved.id);
        if (member) {
            member.rating = saved.rating;
        }
    });
}

// Первичная отрисовка
renderBoard();
</script>

</body>
</html>

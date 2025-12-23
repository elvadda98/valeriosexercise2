<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Vocab Practice: Year-End Special</title>
    <style>
        :root {
            --primary-color: #2c3e50;
            --secondary-color: #3498db;
            --accent-color: #1abc9c;
            --light-bg: #f9fbfd;
            --dark-text: #2c3e50;
            --success-color: #27ae60;
            --warning-color: #f1c40f;
            --error-color: #e74c3c;
        }

        body { font-family: 'Segoe UI', Tahoma, sans-serif; background: var(--light-bg); color: var(--dark-text); margin: 0; padding-bottom: 50px; }
        header { background: var(--primary-color); color: white; padding: 25px; text-align: center; position: relative; }
        .score-display { position: absolute; top: 25px; right: 25px; background: rgba(255,255,255,0.1); padding: 5px 15px; border-radius: 20px; font-weight: bold; }
        .container { max-width: 950px; margin: 30px auto; padding: 0 20px; }
        
        .nav-buttons { display: flex; gap: 10px; margin-bottom: 25px; flex-wrap: wrap; }
        .nav-button { flex: 1; padding: 12px; border: none; border-radius: 8px; background: #dfe6e9; cursor: pointer; font-weight: bold; transition: 0.3s; }
        .nav-button.active { background: var(--secondary-color); color: white; }

        .exercise-section { background: white; padding: 30px; border-radius: 12px; box-shadow: 0 10px 30px rgba(0,0,0,0.05); }
        .word-bank { display: flex; flex-wrap: wrap; gap: 12px; background: #f1f4f8; padding: 20px; border-radius: 10px; margin-bottom: 20px; border: 2px dashed var(--secondary-color); }
        .word-bank-item { background: white; padding: 8px 15px; border-radius: 6px; font-weight: bold; box-shadow: 0 2px 5px rgba(0,0,0,0.1); color: var(--primary-color); }

        .vocab-item { display: flex; align-items: center; gap: 20px; padding: 15px 0; border-bottom: 1px solid #eee; }
        .word-cell { flex: 0 0 200px; font-weight: bold; color: var(--secondary-color); font-size: 1.1em; }

        .gap-sentence { margin: 20px 0; font-size: 1.1em; line-height: 1.8; }
        .gap-input { border: none; border-bottom: 2px solid var(--secondary-color); width: 180px; text-align: center; font-size: inherit; outline: none; background: transparent; }
        
        .matching-columns { display: grid; grid-template-columns: 1fr 1fr; gap: 40px; }
        .match-column { display: flex; flex-direction: column; gap: 10px; }
        .match-box { padding: 15px; border: 2px solid #ecf0f1; border-radius: 8px; cursor: pointer; transition: 0.2s; background: white; min-height: 50px; display: flex; align-items: center; justify-content: center; text-align: center; }
        .match-box.selected { border-color: var(--secondary-color); background: #ebf5fb; }
        .match-box.correct { background: var(--success-color); color: white; border-color: var(--success-color); cursor: default; }

        .status-green { color: var(--success-color); font-weight: bold; }
        .status-yellow { color: var(--warning-color); font-weight: bold; }
        .status-red { color: var(--error-color); font-weight: bold; }

        .speaker-btn, .mic-btn { background: #f1f4f8; border: none; border-radius: 50%; width: 40px; height: 40px; cursor: pointer; display: inline-flex; align-items: center; justify-content: center; font-size: 1.2em; }
        .action-btn { background: var(--accent-color); color: white; border: none; padding: 12px 30px; border-radius: 8px; cursor: pointer; font-weight: bold; margin-top: 20px; }
    </style>
</head>
<body>

<header>
    <h1>Vocabulary: New Year Prep</h1>
    <div class="score-display">Total Score: <span id="score-val">0</span></div>
</header>

<div class="container">
    <div class="nav-buttons">
        <button class="nav-button active" onclick="switchTab(0)">1. Study List</button>
        <button class="nav-button" onclick="switchTab(1)">2. Writing</button>
        <button class="nav-button" onclick="switchTab(2)">3. Matching</button>
        <button class="nav-button" onclick="switchTab(3)">4. Pronunciation</button>
    </div>

    <div id="tab-0" class="exercise-section">
        <h2>📚 Vocabulary List</h2>
        <div id="vocab-list"></div>
    </div>

    <div id="tab-1" class="exercise-section" style="display:none">
        <h2>✍️ Fill in the Gaps (Writing)</h2>
        <div id="writing-bank" class="word-bank"></div>
        <div id="writing-content"></div>
        <button class="action-btn" onclick="checkWriting()">Check Answers</button>
    </div>

    <div id="tab-2" class="exercise-section" style="display:none">
        <h2>🔗 Match Meaning (Left) to Word (Right)</h2>
        <div class="matching-columns">
            <div id="match-left" class="match-column"></div>
            <div id="match-right" class="match-column"></div>
        </div>
    </div>

    <div id="tab-3" class="exercise-section" style="display:none">
        <h2>🗣️ Pronunciation (Guess & Speak)</h2>
        <div id="pron-bank" class="word-bank"></div>
        <div id="pron-content"></div>
    </div>
</div>

<script>
    const data = [
        { word: "how long", def: "Used to ask about the duration of time.", sent: "___ does the flight to New York take?", pron: "___ have you been waiting?" },
        { word: "new Year Resolutions", def: "Promises you make to yourself for the start of the year.", sent: "One of my ___ is to go to the gym three times a week.", pron: "Do you usually keep your ___?" },
        { word: "eve", def: "The day or evening just before a major holiday.", sent: "We are going to a big party on New Year's ___.", pron: "I love the atmosphere on Christmas ___." },
        { word: "currently", def: "At the present time; now.", sent: "I am ___ living in London for work.", pron: "Are you ___ working on any new projects?" }
    ];

    let totalScore = 0;

    function shuffle(array) {
        let currentIndex = array.length, randomIndex;
        while (currentIndex != 0) {
            randomIndex = Math.floor(Math.random() * currentIndex);
            currentIndex--;
            [array[currentIndex], array[randomIndex]] = [array[randomIndex], array[currentIndex]];
        }
        return array;
    }

    function getEditDistance(a, b) {
        if (a.length === 0) return b.length; 
        if (b.length === 0) return a.length;
        var matrix = [];
        for (var i = 0; i <= b.length; i++) { matrix[i] = [i]; }
        for (var j = 0; j <= a.length; j++) { matrix[0][j] = j; }
        for (i = 1; i <= b.length; i++) {
            for (j = 1; j <= a.length; j++) {
                if (b.charAt(i - 1) == a.charAt(j - 1)) { matrix[i][j] = matrix[i-1][j-1]; } 
                else { matrix[i][j] = Math.min(matrix[i-1][j-1] + 1, Math.min(matrix[i][j-1] + 1, matrix[i-1][j] + 1)); }
            }
        }
        return matrix[b.length][a.length];
    }

    function speak(text) {
        const synth = window.speechSynthesis;
        const utterance = new SpeechSynthesisUtterance(text);
        utterance.rate = 0.9;
        synth.speak(utterance);
    }

    function switchTab(idx) {
        document.querySelectorAll('.exercise-section').forEach((s, i) => s.style.display = i === idx ? 'block' : 'none');
        document.querySelectorAll('.nav-button').forEach((b, i) => b.classList.toggle('active', i === idx));
    }

    function init() {
        const list = document.getElementById('vocab-list');
        list.innerHTML = "";
        data.forEach(item => {
            list.innerHTML += `<div class="vocab-item"><div class="word-cell">${item.word}</div><button class="speaker-btn" onclick="speak('${item.word}')">🔊</button><div>${item.def}</div></div>`;
        });

        const wordList = data.map(d => d.word);
        document.getElementById('writing-bank').innerHTML = shuffle([...wordList]).map(w => `<div class="word-bank-item">${w}</div>`).join('');
        document.getElementById('pron-bank').innerHTML = shuffle([...wordList]).map(w => `<div class="word-bank-item">${w}</div>`).join('');

        const wCont = document.getElementById('writing-content');
        wCont.innerHTML = "";
        data.forEach((item, i) => {
            wCont.innerHTML += `<div class="gap-sentence">${i+1}. ${item.sent.replace(item.word, '___').replace('___', `<input type="text" class="gap-input" id="w-in-${i}">`)} <div id="w-fb-${i}"></div></div>`;
        });

        const leftCol = document.getElementById('match-left');
        const rightCol = document.getElementById('match-right');
        const meanings = shuffle(data.map((d, i) => ({ t: d.def, id: i, type: 'd' })));
        const words = shuffle(data.map((d, i) => ({ t: d.word, id: i, type: 'w' })));
        meanings.forEach(obj => leftCol.innerHTML += `<div class="match-box" data-id="${obj.id}" data-type="${obj.type}" onclick="handleMatch(this)">${obj.t}</div>`);
        words.forEach(obj => rightCol.innerHTML += `<div class="match-box" data-id="${obj.id}" data-type="${obj.type}" onclick="handleMatch(this)">${obj.t}</div>`);

        const pCont = document.getElementById('pron-content');
        pCont.innerHTML = "";
        data.forEach((item, i) => {
            pCont.innerHTML += `<div class="gap-sentence">${item.pron.replace(item.word, '_________')} <button class="mic-btn" onclick="testSpeech('${item.word}', ${i})">🎤</button> <div id="p-fb-${i}" style="color:#888; font-size: 0.9em;">Guess the word & speak!</div></div>`;
        });
    }

    function checkWriting() {
        data.forEach((item, i) => {
            const input = document.getElementById(`w-in-${i}`);
            const fb = document.getElementById(`w-fb-${i}`);
            const val = input.value.trim().toLowerCase();
            const target = item.word.toLowerCase();
            const dist = getEditDistance(val, target);

            if (val === target) {
                fb.innerHTML = `<span class="status-green">✅ Correct!</span>`;
                totalScore += 1;
            } else if (dist <= 2) {
                fb.innerHTML = `<span class="status-yellow">⚠️ Close! Answer is "${item.word}"</span>`;
                totalScore += 0.5;
            } else {
                fb.innerHTML = `<span class="status-red">❌ Correct answer: "${item.word}"</span>`;
            }
        });
        updateDisplayScore();
    }

    let firstMatch = null;
    function handleMatch(el) {
        if (el.classList.contains('correct')) return;
        if (!firstMatch) {
            firstMatch = el; el.classList.add('selected');
        } else {
            if (firstMatch.dataset.id === el.dataset.id && firstMatch.dataset.type !== el.dataset.type) {
                firstMatch.classList.replace('selected', 'correct'); el.classList.add('correct');
                totalScore += 1;
            } else {
                firstMatch.classList.remove('selected');
            }
            firstMatch = null;
            updateDisplayScore();
        }
    }

    function testSpeech(word, idx) {
        const rec = new (window.SpeechRecognition || window.webkitSpeechRecognition)();
        rec.lang = 'en-US'; rec.start();
        document.getElementById(`p-fb-${idx}`).innerText = "Listening...";
        rec.onresult = (e) => {
            const heard = e.results[0][0].transcript.toLowerCase();
            const fb = document.getElementById(`p-fb-${idx}`);
            const target = word.toLowerCase();

            if (heard.includes(target)) {
                fb.innerHTML = `<span class="status-green">✅ Perfect! You said: "${heard}"</span>`;
                totalScore += 1;
            } else {
                fb.innerHTML = `<span class="status-red">❌ I heard "${heard}". The word was "${word}".</span>`;
            }
            updateDisplayScore();
        };
    }

    function updateDisplayScore() {
        document.getElementById('score-val').innerText = Math.floor(totalScore);
    }

    window.onload = init;
</script>
</body>
</html>

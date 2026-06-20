<!DOCTYPE html>
<html lang="el">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Για την Αλκμήνη!</title>
    <style>
        :root {
            --primary-color: #ff4d6d;
            --secondary-color: #ff85a2;
            --bg-color: #fff0f3;
            --text-color: #590d22;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: var(--bg-color);
            color: var(--text-color);
            margin: 0;
            padding: 0;
            display: flex;
            flex-direction: column;
            align-items: center;
            overflow-x: hidden;
        }

        /* Banner */
        .marquee-banner {
            background-color: var(--primary-color);
            color: white;
            padding: 10px 0;
            font-size: 1.2rem;
            font-weight: bold;
            width: 100%;
            overflow: hidden;
            white-space: nowrap;
            box-shadow: 0 4px 6px rgba(0,0,0,0.1);
        }

        .marquee-text {
            display: inline-block;
            animation: marquee 15s linear infinite;
        }

        @keyframes marquee {
            0% { transform: translate3d(100%, 0, 0); }
            100% { transform: translate3d(-100%, 0, 0); }
        }

        /* Main Container */
        .container {
            max-width: 600px;
            width: 90%;
            margin: 30px auto;
            text-align: center;
        }

        h1 {
            color: var(--primary-color);
            font-size: 2.5rem;
            text-shadow: 2px 2px 4px rgba(0,0,0,0.1);
            animation: heartbeat 1.5s infinite;
        }

        @keyframes heartbeat {
            0% { transform: scale(1); }
            20% { transform: scale(1.05); }
            40% { transform: scale(1); }
            60% { transform: scale(1.05); }
            100% { transform: scale(1); }
        }

        /* Card Style for Sections */
        .card {
            background: white;
            border-radius: 20px;
            padding: 25px;
            margin: 25px 0;
            box-shadow: 0 8px 16px rgba(255, 77, 109, 0.15);
            transition: transform 0.3s;
        }

        .card:hover {
            transform: translateY(-5px);
        }

        /* Buttons */
        .btn {
            background-color: var(--primary-color);
            color: white;
            border: none;
            padding: 12px 25px;
            font-size: 1.1rem;
            border-radius: 50px;
            cursor: pointer;
            font-weight: bold;
            transition: all 0.2s;
            box-shadow: 0 4px 15px rgba(255, 77, 109, 0.4);
        }

        .btn:hover {
            background-color: #ff758f;
            transform: scale(1.05);
        }

        .btn:active {
            transform: scale(0.95);
        }

        /* Compliment Box */
        #compliment-display {
            font-size: 1.3rem;
            font-style: italic;
            margin-top: 15px;
            min-height: 50px;
            color: #800f2f;
            font-weight: 600;
        }

        /* Quiz Choices */
        .quiz-options {
            display: flex;
            flex-direction: column;
            gap: 10px;
            margin-top: 15px;
        }

        .quiz-btn {
            background: #fff0f3;
            border: 2px solid var(--secondary-color);
            color: var(--text-color);
            padding: 10px;
            border-radius: 10px;
            cursor: pointer;
            font-weight: 600;
            transition: 0.2s;
        }

        .quiz-btn:hover {
            background: var(--secondary-color);
            color: white;
        }

        /* Slider / Meter */
        .slider {
            -webkit-appearance: none;
            width: 100%;
            height: 15px;
            border-radius: 10px;
            background: #ffe3e8;
            outline: none;
            margin: 20px 0;
        }

        .slider::-webkit-slider-thumb {
            -webkit-appearance: none;
            appearance: none;
            width: 25px;
            height: 25px;
            border-radius: 50%;
            background: var(--primary-color);
            cursor: pointer;
            box-shadow: 0 0 10px rgba(0,0,0,0.2);
        }

        #meter-result {
            font-size: 1.5rem;
            font-weight: bold;
            color: var(--primary-color);
        }

        /* Catch me Section */
        .catch-container {
            position: relative;
            height: 120px;
            margin-top: 20px;
        }

        #run-btn {
            position: absolute;
            left: 10%;
            transition: all 0.1s ease;
        }

        #stay-btn {
            position: absolute;
            right: 10%;
        }

        /* Floating Hearts */
        .heart {
            position: fixed;
            font-size: 24px;
            color: var(--primary-color);
            pointer-events: none;
            z-index: 999;
            animation: floatUp 2s linear forwards;
        }

        @keyframes floatUp {
            0% { transform: translateY(0) scale(1); opacity: 1; }
            100% { transform: translateY(-100px) scale(1.5); opacity: 0; }
        }
    </style>
</head>
<body>

    <div class="marquee-banner">
        <div class="marquee-text">
            ✨ ΕΚΤΑΚΤΟ ΑΝΑΚΟΙΝΩΘΕΝ: Η ΑΛΚΜΗΝΗ ΕΙΝΑΙ ΠΑΝΕΜΟΡΦΗ! ✨ ΠΑΓΚΟΣΜΙΑ ΔΙΑΠΙΣΤΩΣΗ ✨ ΚΑΝΕΙΣ ΔΕΝ ΜΠΟΡΕΙ ΝΑ ΤΟ ΑΡΝΗΘΕΙ ✨
        </div>
    </div>

    <div class="container">
        <h1>Αλκμήνη, είσαι πανέμορφη! ❤️</h1>
        <p>Αυτό το app φτιάχτηκε αποκλειστικά για να σου θυμίζει την αλήθεια.</p>

        <div class="card">
            <h3>✨ Η Γεννήτρια της Αλήθειας</h3>
            <p>Χρειάζεσαι μια δόση αντικειμενικότητας; Πάτα το κουμπί!</p>
            <button class="btn" onclick="generateCompliment(event)">Πάτα με!</button>
            <div id="compliment-display">Κάνε κλικ για να ξεκινήσεις...</div>
        </div>

        <div class="card">
            <h3>❓ Το Αντικειμενικό Κουίζ</h3>
            <p>Ποιο είναι το κυριότερο χαρακτηριστικό της Αλκμήνης;</p>
            <div class="quiz-options">
                <button class="quiz-btn" onclick="quizAnswer(false)">Α) Είναι απλά χαριτωμένη.</button>
                <button class="quiz-btn" onclick="quizAnswer(true)">Β) Η ομορφιά της μάλλον θα έπρεπε να είναι παράνομη, κλέβει τις εντυπώσεις!</button>
                <button class="quiz-btn" onclick="quizAnswer(false)">Γ) Έχει ωραίο στυλ, αλλά ως εκεί.</button>
            </div>
            <div id="quiz-feedback" style="margin-top:15px; font-weight:bold;"></div>
        </div>

        <div class="card">
            <h3>📊 Ομορφόμετρο</h3>
            <p>Σύρε τη μπάρα για να μετρήσουμε το επίπεδο ομορφιάς σου σήμερα:</p>
            <input type="range" min="1" max="5" value="1" class="slider" id="beauty-slider" oninput="updateMeter(this.value)">
            <div id="meter-result">Φόρτωση...</div>
        </div>

        <div class="card">
            <h3>🤔 Ώρα για ειλικρίνεια</h3>
            <p>Διάλεξε τι από τα δύο ισχύει:</p>
            <div class="catch-container">
                <button class="btn" id="run-btn" onmouseover="runAway()" onclick="runAway()">Είμαι μια απλή θνητή 🤷‍♀️</button>
                <button class="btn" id="stay-btn" onclick="celebrate(event)">Είμαι Θεά! 👑</button>
            </div>
        </div>
    </div>

    <script>
        // Compliments Array
        const compliments = [
            "Το χαμόγελό σου φωτίζει περισσότερο κι από τον ήλιο! ☀️",
            "Αν η ομορφιά ήταν χρόνος, θα ήσουν η αιωνιότητα. ⏳",
            "Δεν είσαι απλά όμορφη, είσαι η προσωποποίηση της κομψότητας! ✨",
            "Τα μάτια σου έχουν δικό τους μαγνητισμό, δεν υπάρχει άλλη εξήγηση. 👀💖",
            "Ακόμα και οι πιο όμορφες φωτογραφίες αδικούν την πραγματική σου γοητεία! 📸",
            "Η παρουσία σου και μόνο ομορφαίνει τον χώρο γύρω σου σε δευτερόλεπτα! 🌸",
            "Είσαι ο λόγος που η λέξη 'πανέμορφη' απέκτησε νόημα! 😍"
        ];

        function generateCompliment(e) {
            const randomIndex = Math.floor(Math.random() * compliments.length);
            document.getElementById('compliment-display').innerText = compliments[randomIndex];
            createHearts(e.clientX, e.clientY);
        }

        function quizAnswer(isCorrect) {
            const feedback = document.getElementById('quiz-feedback');
            if(isCorrect) {
                feedback.innerHTML = "Σωστά! Η επιστήμη και η αισθητική συμφωνούν απόλυτα! 🏆🎉";
                feedback.style.color = "green";
            } else {
                feedback.innerHTML = "Λάθος απάντηση! Ξαναπροσπάθησε, το σύστημα δεν δέχεται ψέματα. ❌😜";
                feedback.style.color = "var(--primary-color)";
            }
        }

        function updateMeter(val) {
            const result = document.getElementById('meter-result');
            const stages = {
                1: "Πανέμορφη 😍",
                2: "Εκθαμβωτική ✨",
                3: "Ακαταμάχητη 💖",
                4: "Έκπτωτος Άγγελος 👼",
                5: "👑 ΑΛΚΜΗΝΗ (Εκτός κλίμακας, κάηκε το σύστημα!) 🔥"
            };
            result.innerText = stages[val];
            
            if(val == 5) {
                // Trigger a burst of random hearts
                for(let i=0; i<15; i++) {
                    setTimeout(() => {
                        createHearts(Math.random() * window.innerWidth, Math.random() * window.innerHeight);
                    }, i * 100);
                }
            }
        }

        // Initialize meter
        updateMeter(1);

        // Runaway Button Logic
        function runAway() {
            const btn = document.getElementById('run-btn');
            // Move inside the container boundaries safely
            const newX = Math.random() * 60; // percentage
            const newY = Math.random() * 50; // pixels
            
            btn.style.left = newX + '%';
            btn.style.top = newY + 'px';
        }

        function celebrate(e) {
            alert("ΕΤΣΙ ΜΠΡΑΒΟ! Η αυτογνωσία είναι μεγάλη αρετή! 👑❤️");
            createHearts(e.clientX, e.clientY);
        }

        // Floating Hearts Effect
        function createHearts(x, y) {
            const heartCount = 6;
            for(let i=0; i<heartCount; i++) {
                const heart = document.createElement('div');
                heart.classList.add('heart');
                heart.innerText = '❤️';
                heart.style.left = (x ? x : window.innerWidth/2) + (Math.random() * 40 - 20) + 'px';
                heart.style.top = (y ? y : window.innerHeight/2) + (Math.random() * 40 - 20) + 'px';
                heart.style.animationDelay = (Math.random() * 0.3) + 's';
                document.body.appendChild(heart);
                
                setTimeout(() => {
                    heart.remove();
                }, 2000);
            }
        }
    </script>
</body>
</html>

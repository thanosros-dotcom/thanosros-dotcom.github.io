
<html lang="el">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Smart Weight Tracker v2</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        :root {
            --primary: #4f46e5;
            --danger: #ef4444;
            --background: #f8fafc;
            --surface: #ffffff;
            --text: #1e293b;
            --text-muted: #64748b;
        }

        body {
            font-family: 'Segoe UI', system-ui, sans-serif;
            background-color: var(--background);
            color: var(--text);
            margin: 0;
            padding: 20px;
        }

        .container { max-width: 1100px; margin: 0 auto; }
        header { text-align: center; margin-bottom: 30px; }
        h1 { margin-bottom: 5px; color: var(--primary); }
        .subtitle { color: var(--text-muted); margin-top: 0; }

        .grid { display: grid; grid-template-columns: 1fr; gap: 20px; }
        @media (min-width: 768px) { .grid { grid-template-columns: 1fr 2fr; } }

        .card {
            background: var(--surface);
            padding: 20px;
            border-radius: 12px;
            box-shadow: 0 4px 6px -1px rgb(0 0 0 / 0.05);
        }

        .form-group { margin-bottom: 15px; }
        label { display: block; margin-bottom: 5px; font-weight: 600; }
        input { width: 100%; padding: 8px; border: 1px solid #cbd5e1; border-radius: 6px; box-sizing: border-box; }
        
        button {
            width: 100%; background: var(--primary); color: white; border: none;
            padding: 10px; border-radius: 6px; font-weight: bold; cursor: pointer;
        }
        button:hover { filter: brightness(0.9); }

        .stats-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(140px, 1fr)); gap: 12px; margin-bottom: 20px; }
        .stat-box { background: #f1f5f9; padding: 12px; border-radius: 8px; text-align: center; }
        .stat-val { font-size: 1.3rem; font-weight: bold; color: var(--primary); }
        .stat-label { font-size: 0.8rem; color: var(--text-muted); }

        .table-container { max-height: 350px; overflow-y: auto; margin-top: 15px; }
        table { width: 100%; border-collapse: collapse; font-size: 0.85rem; }
        th, td { padding: 8px; text-align: left; border-bottom: 1px solid #e2e8f0; }
        th { background: #f8fafc; position: sticky; top: 0; }

        .btn-del {
            background: none; border: none; color: var(--danger); cursor: pointer;
            padding: 2px 6px; font-size: 0.8rem; width: auto; display: inline;
        }
        .btn-del:hover { text-decoration: underline; }

        .noise-high { color: #ef4444; font-weight: bold; }
        .noise-low { color: #3b82f6; font-weight: bold; }
        .noise-normal { color: #10b981; }
    </style>
</head>
<body>

<div class="container">
    <header>
        <h1>Smart Weight Tracker <span style="font-size:1rem; vertical-align:middle; color:#06b6d4;">v2 (Time-Aware)</span></h1>
        <p class="subtitle">Υπολογισμός True Trend με αυτόματη κάλυψη κενών ημερών</p>
    </header>

    <div class="grid">
        <!-- ΕΙΣΑΓΩΓΗ & ΙΣΤΟΡΙΚΟ -->
        <div>
            <div class="card">
                <h3>Νέα Καταγραφή</h3>
                <form id="weightForm">
                    <div class="form-group">
                        <label for="date">Ημερομηνία</label>
                        <input type="date" id="date" required>
                    </div>
                    <div class="form-group">
                        <label for="weight">Βάρος (kg)</label>
                        <input type="number" id="weight" step="0.1" placeholder="e.g. 78.5" required>
                    </div>
                    <button type="submit">Αποθήκευση</button>
                </form>
            </div>

            <div class="card" style="margin-top: 20px;">
                <div style="display:flex; justify-content:space-between; align-items:center;">
                    <h3>Ιστορικό</h3>
                    <button onclick="clearAllData()" style="background:var(--danger); width:auto; padding:4px 8px; font-size:0.75rem;">Διαγραφή Όλων</button>
                </div>
                <div class="table-container">
                    <table>
                        <thead>
                            <tr>
                                <th>Ημ/νία</th>
                                <th>Βάρος</th>
                                <th>True Trend</th>
                                <th>Θόρυβος</th>
                                <th></th>
                            </tr>
                        </thead>
                        <tbody id="historyTable"></tbody>
                    </table>
                </div>
            </div>
        </div>

        <!-- ΣΤΑΤΙΣΤΙΚΑ & ΓΡΑΦΗΜΑ -->
        <div class="card">
            <h3>Στατιστικά & Ανάλυση</h3>
            <div class="stats-grid">
                <div class="stat-box">
                    <div class="stat-val" id="statCurrent">-</div>
                    <div class="stat-label">Τελευταίο Βάρος</div>
                </div>
                <div class="stat-box">
                    <div class="stat-val" id="statTrend">-</div>
                    <div class="stat-label">True Trend</div>
                </div>
                <div class="stat-box">
                    <div class="stat-val" id="statWeeklyChange">-</div>
                    <div class="stat-label">Εβδομαδιαίο Trend</div>
                </div>
                <div class="stat-box">
                    <div class="stat-val" id="statNoise">-</div>
                    <div class="stat-label">Κατάσταση Θορύβου</div>
                </div>
            </div>

            <div style="position: relative; width:100%; height:380px;">
                <canvas id="weightChart"></canvas>
            </div>
        </div>
    </div>
</div>

<script>
    let weightData = JSON.parse(localStorage.getItem('weightDataV2')) || [];
    let myChart = null;

    document.getElementById('date').value = new Date().toISOString().substring(0, 10);

    document.getElementById('weightForm').addEventListener('submit', function(e) {
        e.preventDefault();
        const date = document.getElementById('date').value;
        const weight = parseFloat(document.getElementById('weight').value);

        const existingIndex = weightData.findIndex(d => d.date === date);
        if (existingIndex !== -1) {
            weightData[existingIndex].weight = weight;
        } else {
            weightData.push({ date, weight });
        }

        weightData.sort((a, b) => new Date(a.date) - new Date(b.date));
        saveAndRefresh();
        document.getElementById('weight').value = '';
    });

    function deleteEntry(date) {
        if (confirm(`Διαγραφή της καταγραφής για τις ${formatDate(date)}?`)) {
            weightData = weightData.filter(d => d.date !== date);
            saveAndRefresh();
        }
    }

    function clearAllData() {
        if (confirm('Προσοχή! Θα διαγραφούν μόνιμα όλες οι καταγραφές.')) {
            weightData = [];
            saveAndRefresh();
        }
    }

    function saveAndRefresh() {
        localStorage.setItem('weightDataV2', JSON.stringify(weightData));
        processDataAndRender();
    }

    // Η ΝΕΑ ΕΞΥΠΝΗ ΛΟΓΙΚΗ
    function processDataAndRender() {
        if (weightData.length === 0) {
            document.getElementById('historyTable').innerHTML = '';
            if(myChart) myChart.destroy();
            ['statCurrent', 'statTrend', 'statWeeklyChange', 'statNoise'].forEach(id => document.getElementById(id).innerText = '-');
            return;
        }

        // 1. Δημιουργία συνεχόμενου timeline (Linear Interpolation για τις κενές μέρες)
        let timeline = [];
        let firstDate = new Date(weightData[0].date);
        let lastDate = new Date(weightData[weightData.length - 1].date);
        
        // Map για γρήγορο lookup των πραγματικών καταγραφών
        let weightMap = {};
        weightData.forEach(d => weightMap[d.date] = d.weight);

        let currentLoopDate = new Date(firstDate);
        while (currentLoopDate <= lastDate) {
            let dateStr = currentLoopDate.toISOString().substring(0, 10);
            timeline.push({
                date: dateStr,
                actualWeight: weightMap[dateStr] || null, // null αν δεν ζυγίστηκες
                trend: 0,
                noise: 0
            });
            currentLoopDate.setDate(currentLoopDate.getDate() + 1);
        }

        // Γέμισμα των κενών (Interpolation) για τον υπολογισμό του EMA
        let lastKnownWeight = weightData[0].weight;
        let nextKnownWeight = weightData[0].weight;
        let nextKnownIndex = 0;

        for (let i = 0; i < timeline.length; i++) {
            if (timeline[i].actualWeight !== null) {
                lastKnownWeight = timeline[i].actualWeight;
            } else {
                // Βρες την επόμενη διαθέσιμη πραγματική μέτρηση για να κάνεις τη γραμμική παρεμβολή
                if (i > nextKnownIndex) {
                    for (let j = i; j < timeline.length; j++) {
                        if (timeline[j].actualWeight !== null) {
                            nextKnownWeight = timeline[j].actualWeight;
                            nextKnownIndex = j;
                            break;
                        }
                    }
                }
                // Αν είμαστε ανάμεσα σε δύο μετρήσεις, κάνε γραμμική εκτίμηση
                if (nextKnownIndex > i) {
                    let prevWeight = timeline[i-1].actualWeight || lastKnownWeight;
                    // Απλή αναλογία ημέρας
                    let steps = nextKnownIndex - (i - 1);
                    lastKnownWeight = prevWeight + (nextKnownWeight - prevWeight) / steps;
                }
            }
            timeline[i].interpolatedWeight = lastKnownWeight;
        }

        // 2. Υπολογισμός EMA πάνω στο πλήρες πλέον timeline (χωρίς χρονικά κενά)
        const alpha = 0.15; // 15% βάρος στη μέρα, 85% στο trend (ιδανικό για απόσβεση θορύβου)
        let currentTrend = timeline[0].interpolatedWeight;

        timeline.forEach((day, index) => {
            currentTrend = (day.interpolatedWeight * alpha) + (currentTrend * (1 - alpha));
            day.trend = Math.round(currentTrend * 100) / 100;
            
            if (day.actualWeight !== null) {
                day.noise = Math.round((day.actualWeight - day.trend) * 100) / 100;
            }
        });

        // 3. Ενημέρωση DOM (Πίνακας) - Δείχνουμε μόνο τις μέρες που ο χρήστης έβαλε όντως βάρος
        const tbody = document.getElementById('historyTable');
        tbody.innerHTML = '';
        
        // Φιλτράρουμε μόνο τις πραγματικές εγγραφές για το ιστορικό
        let realEntriesInTimeline = timeline.filter(t => t.actualWeight !== null);
        
        [...realEntriesInTimeline].reverse().forEach(entry => {
            let noiseClass = 'noise-normal';
            let noiseText = `${entry.noise > 0 ? '+' : ''}${entry.noise} kg`;
            if (entry.noise > 0.4) noiseClass = 'noise-high';
            else if (entry.noise < -0.4) noiseClass = 'noise-low';
            if (entry.noise === 0) noiseText = "0 kg";

            const row = `<tr>
                <td>${formatDate(entry.date)}</td>
                <td><b>${entry.actualWeight} kg</b></td>
                <td>${entry.trend} kg</td>
                <td class="${noiseClass}">${noiseText}</td>
                <td><button class="btn-del" onclick="deleteEntry('${entry.date}')">✕</button></td>
            </tr>`;
            tbody.innerHTML += row;
        });

        // 4. Στατιστικά (Τελευταία πραγματική μέρα)
        const latestReal = realEntriesInTimeline[realEntriesInTimeline.length - 1];
        document.getElementById('statCurrent').innerText = `${latestReal.actualWeight} kg`;
        document.getElementById('statTrend').innerText = `${latestReal.trend} kg`;

        // Εβδομαδιαία αλλαγή στο Trend (Κοιτάμε 7 ημέρες πίσω στο συνεχόμενο timeline)
        if (timeline.length >= 7) {
            let currentTrendVal = timeline[timeline.length - 1].trend;
            let pastTrendVal = timeline[timeline.length - 7].trend;
            let diff = Math.round((currentTrendVal - pastTrendVal) * 100) / 100;
            document.getElementById('statWeeklyChange').innerText = `${diff > 0 ? '+' : ''}${diff} kg`;
        } else {
            document.getElementById('statWeeklyChange').innerText = `-`;
        }

        // Κατάσταση θορύβου
        if (latestReal.noise > 0.4) {
            document.getElementById('statNoise').innerText = "Τεχνητά Αυξημένο";
            document.getElementById('statNoise').className = "stat-val noise-high";
        } else if (latestReal.noise < -0.4) {
            document.getElementById('statNoise').innerText = "Τεχνητά Μειωμένο";
            document.getElementById('statNoise').className = "stat-val noise-low";
        } else {
            document.getElementById('statNoise').innerText = "Σταθερό";
            document.getElementById('statNoise').className = "stat-val noise-normal";
        }

        // 5. Σχεδίαση Γραφήματος
        updateChart(timeline, realEntriesInTimeline);
    }

    function updateChart(fullTimeline, realEntries) {
        const ctx = document.getElementById('weightChart').getContext('2d');
        
        if (myChart) myChart.destroy();

        // Στο γράφημα δείχνουμε όλο το timeline για να είναι σωστό χρονικά, 
        // αλλά οι κουκκίδες του πραγματικού βάρους θα εμφανίζονται ΜΟΝΟ όταν υπάρχουν δεδομένα.
        const labels = fullTimeline.map(t => formatDate(t.date));
        const actualWeightsData = fullTimeline.map(t => t.actualWeight); // τα κενά θα είναι null και το Chart.js δεν θα τα ενώσει με γραμμή αν δεν θέλουμε
        const trendData = fullTimeline.map(t => t.trend);

        myChart = new Chart(ctx, {
            type: 'line',
            data: {
                labels: labels,
                datasets: [
                    {
                        label: 'Πραγματικό Βάρος (Κουκκίδες)',
                        data: actualWeightsData,
                        borderColor: '#94a3b8',
                        backgroundColor: '#64748b',
                        pointRadius: 4,
                        showLine: false, // Δεν ενώνει τα κενά με γραμμή για να μη λεκιάζει το γράφημα
                    },
                    {
                        label: 'True Trend (Συνεχές)',
                        data: trendData,
                        borderColor: '#4f46e5',
                        borderWidth: 3,
                        pointRadius: 0,
                        fill: false,
                        tension: 0.1
                    }
                ]
            },
            options: {
                responsive: true,
                maintainAspectRatio: false,
                plugins: { legend: { position: 'top' } },
                scales: { y: { ticks: { callback: v => v + ' kg' } } }
            }
        });
    }

    function formatDate(dateStr) {
        const d = new Date(dateStr);
        return d.toLocaleDateString('el-GR', { day: 'numeric', month: 'short' });
    }

    // Αρχικό τρέξιμο
    processDataAndRender();
</script>

</body>
</html>

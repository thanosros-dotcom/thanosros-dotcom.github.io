<!DOCTYPE html>
<html lang="el">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Smart Weight Tracker</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        :root {
            --primary: #4f46e5;
            --secondary: #06b6d4;
            --background: #f8fafc;
            --surface: #ffffff;
            --text: #1e293b;
            --text-muted: #64748b;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: var(--background);
            color: var(--text);
            margin: 0;
            padding: 20px;
        }

        .container {
            max-width: 1000px;
            margin: 0 auto;
        }

        header {
            text-align: center;
            margin-bottom: 30px;
        }

        h1 { margin-bottom: 5px; color: var(--primary); }
        p.subtitle { color: var(--text-muted); margin-top: 0; }

        .grid {
            display: grid;
            grid-template-columns: 1fr;
            gap: 20px;
        }

        @media (min-width: 768px) {
            .grid { grid-template-columns: 1fr 2fr; }
        }

        .card {
            background: var(--surface);
            padding: 20px;
            border-radius: 12px;
            box-shadow: 0 4px 6px -1px rgb(0 0 0 / 0.1);
        }

        .form-group {
            margin-bottom: 15px;
        }

        label {
            display: block;
            margin-bottom: 5px;
            font-weight: 600;
        }

        input[type="number"], input[type="date"] {
            width: 100%;
            padding: 8px;
            border: 1px solid #cbd5e1;
            border-radius: 6px;
            box-sizing: border-box;
        }

        button {
            width: 100%;
            background: var(--primary);
            color: white;
            border: none;
            padding: 10px;
            border-radius: 6px;
            font-weight: bold;
            cursor: pointer;
            transition: background 0.2s;
        }

        button:hover { background: #4338ca; }

        /* Stats Cards */
        .stats-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
            gap: 15px;
            margin-bottom: 20px;
        }

        .stat-box {
            background: #f1f5f9;
            padding: 15px;
            border-radius: 8px;
            text-align: center;
        }

        .stat-val {
            font-size: 1.5rem;
            font-weight: bold;
            color: var(--primary);
        }

        .stat-label {
            font-size: 0.85rem;
            color: var(--text-muted);
        }

        /* Table */
        .table-container {
            max-height: 300px;
            overflow-y: auto;
            margin-top: 15px;
        }

        table {
            width: 100%;
            border-collapse: collapse;
            font-size: 0.9rem;
        }

        th, td {
            padding: 10px;
            text-align: left;
            border-bottom: 1px solid #e2e8f0;
        }

        th { background: #f8fafc; }

        .noise-high { color: #ef4444; }
        .noise-low { color: #3b82f6; }
        .noise-normal { color: #10b981; }
    </style>
</head>
<body>

<div class="container">
    <header>
        <h1>Smart Weight Tracker</h1>
        <p class="subtitle">Δες το πραγματικό σου trend, πέρα από τον καθημερινό θόρυβο</p>
    </header>

    <div class="grid">
        <div class="space-y-4">
            <div class="card">
                <h3>Νέα Καταγραφή</h3>
                <form id="weightForm">
                    <div class="form-group">
                        <label for="date">Ημερομηνία</label>
                        <input type="date" id="date" required>
                    </div>
                    <div class="form-group">
                        <label for="weight">Βάρος (kg)</label>
                        <input type="number" id="weight" step="0.1" placeholder="π.χ. 78.5" required>
                    </div>
                    <button type="submit">Αποθήκευση</button>
                </form>
            </div>

            <div class="card" style="margin-top: 20px;">
                <h3>Ιστορικό</h3>
                <button onclick="clearData()" style="background: #ef4444; margin-bottom: 10px; font-size: 0.8rem; padding: 5px;">Καθαρισμός Όλων</button>
                <div class="table-container">
                    <table>
                        <thead>
                            <tr>
                                <th>Ημ/νία</th>
                                <th>Βάρος</th>
                                <th>True Trend</th>
                                <th>Θόρυβος</th>
                            </tr>
                        </thead>
                        <tbody id="historyTable"></tbody>
                    </table>
                </div>
            </div>
        </div>

        <div class="card">
            <h3>Στατιστικά & Ανάλυση</h3>
            <div class="stats-grid">
                <div class="stat-box">
                    <div class="stat-val" id="statCurrent">-</div>
                    <div class="stat-label">Τελευταίο Βάρος</div>
                </div>
                <div class="stat-box">
                    <div class="stat-val" id="statTrend">-</div>
                    <div class="stat-label">True Trend (Σήμερα)</div>
                </div>
                <div class="stat-box">
                    <div class="stat-val" id="statWeeklyChange">-</div>
                    <div class="stat-label">Εβδομαδιαία Μεταβολή</div>
                </div>
                <div class="stat-box">
                    <div class="stat-val" id="statNoise">-</div>
                    <div class="stat-label">Κατάσταση Θορύβου</div>
                </div>
            </div>

            <div style="position: relative; width:100%; height:350px;">
                <canvas id="weightChart"></canvas>
            </div>
        </div>
    </div>
</div>

<script>
    // Αρχικοποίηση δεδομένων από το LocalStorage
    let weightData = JSON.parse(localStorage.getItem('weightData')) || [];
    let myChart = null;

    // Θέτουμε ως default ημερομηνία τη σημερινή
    document.getElementById('date').valueToDate = new Date();
    document.getElementById('date').value = new Date().toISOString().substring(0, 10);

    // Φόρμα υποβολής
    document.getElementById('weightForm').addEventListener('submit', function(e) {
        e.preventDefault();
        const date = document.getElementById('date').value;
        const weight = parseFloat(document.getElementById('weight').value);

        // Έλεγχος αν υπάρχει ήδη η ημερομηνία για ενημέρωση, αλλιώς προσθήκη
        const existingIndex = weightData.findIndex(d => d.date === date);
        if (existingIndex !== -1) {
            weightData[existingIndex].weight = weight;
        } else {
            weightData.push({ date, weight });
        }

        // Ταξινόμηση ανά ημερομηνία
        weightData.sort((a, b) => new Date(a.date) - new Date(b.date));
        
        saveAndRefresh();
        document.getElementById('weight').value = '';
    });

    function saveAndRefresh() {
        calculateTrends();
        localStorage.setItem('weightData', JSON.stringify(weightData));
        updateDOM();
    }

    // Υπολογισμός True Trend με βάση τον Εκθετικό Κινητό Μέσο Όρο (EMA)
    function calculateTrends() {
        if (weightData.length === 0) return;

        // α = συντελεστής εξομάλυνσης (0.2 σημαίνει ότι δίνουμε 20% βάρος στη σημερινή μέρα και 80% στο ιστορικό trend)
        const alpha = 0.2; 
        let currentTrend = weightData[0].weight;

        weightData.forEach((entry, index) => {
            if (index === 0) {
                entry.trend = entry.weight;
                entry.noise = 0;
            } else {
                currentTrend = (entry.weight * alpha) + (currentTrend * (1 - alpha));
                entry.trend = Math.round(currentTrend * 100) / 100;
                // Θόρυβος = Πραγματικό βάρος - Trend (π.χ. +0.8kg λόγω αλατιού χθες)
                entry.noise = Math.round((entry.weight - entry.trend) * 100) / 100;
            }
        });
    }

    function updateDOM() {
        const tbody = document.getElementById('historyTable');
        tbody.innerHTML = '';
        
        if (weightData.length === 0) {
            if(myChart) myChart.destroy();
            return;
        }

        // Ενημέρωση Πίνακα (αντίστροφη σειρά για να φαίνονται τα πρόσφατα πάνω)
        [...weightData].reverse().forEach(entry => {
            let noiseClass = 'noise-normal';
            let noiseText = `${entry.noise > 0 ? '+' : ''}${entry.noise} kg`;
            
            if (entry.noise > 0.5) noiseClass = 'noise-high';
            else if (entry.noise < -0.5) noiseClass = 'noise-low';

            if (entry.noise === 0) noiseText = "0 kg";

            const row = `<tr>
                <td>${formatDate(entry.date)}</td>
                <td><b>${entry.weight} kg</b></td>
                <td>${entry.trend} kg</td>
                <td class="${noiseClass}">${noiseText}</td>
            </tr>`;
            tbody.innerHTML += row;
        });

        // Υπολογισμός Stats
        const latest = weightData[weightData.length - 1];
        document.getElementById('statCurrent').innerText = `${latest.weight} kg`;
        document.getElementById('statTrend').innerText = `${latest.trend} kg`;

        // Εβδομαδιαία πτώση (Σύγκριση με 7 καταγραφές πριν ή την παλαιότερη διαθέσιμη)
        if (weightData.length >= 2) {
            const indexSevenDaysAgo = Math.max(0, weightData.length - 7);
            const diff = Math.round((latest.trend - weightData[indexSevenDaysAgo].trend) * 100) / 100;
            document.getElementById('statWeeklyChange').innerText = `${diff > 0 ? '+' : ''}${diff} kg`;
        } else {
            document.getElementById('statWeeklyChange').innerText = `-`;
        }

        // Ερμηνεία θορύβου
        if (latest.noise > 0.5) {
            document.getElementById('statNoise').innerText = "Υψηλός (Κατακράτηση)";
            document.getElementById('statNoise').style.color = "#ef4444";
        } else if (latest.noise < -0.5) {
            document.getElementById('statNoise').innerText = "Χαμηλός (Αφυδάτωση)";
            document.getElementById('statNoise').style.color = "#3b82f6";
        } else {
            document.getElementById('statNoise').innerText = "Σταθερό Trend";
            document.getElementById('statNoise').style.color = "#10b981";
        }

        updateChart();
    }

    function updateChart() {
        const ctx = document.getElementById('weightChart').getContext('2d');
        
        const labels = weightData.map(d => formatDate(d.date));
        const weights = weightData.map(d => d.weight);
        const trends = weightData.map(d => d.trend);

        if (myChart) {
            myChart.destroy();
        }

        myChart = new Chart(ctx, {
            type: 'line',
            data: {
                labels: labels,
                datasets: [
                    {
                        label: 'Καθημερινό Βάρος (Με Θόρυβο)',
                        data: weights,
                        borderColor: '#cbd5e1',
                        borderWidth: 1.5,
                        pointRadius: 3,
                        fill: false,
                        borderDash: [5, 5]
                    },
                    {
                        label: 'True Trend (Εξομαλυμένο)',
                        data: trends,
                        borderColor: '#4f46e5',
                        borderWidth: 3,
                        pointRadius: 0,
                        fill: false,
                        tension: 0.2
                    }
                ]
            },
            options: {
                responsive: true,
                maintainAspectRatio: false,
                plugins: {
                    legend: { position: 'top' }
                },
                scales: {
                    y: {
                        ticks: { callback: value => value + ' kg' }
                    }
                }
            }
        });
    }

    function formatDate(dateStr) {
        const d = new Date(dateStr);
        return d.toLocaleDateString('el-GR', { day: 'numeric', month: 'short' });
    }

    function clearData() {
        if (confirm('Είσαι σίγουρος ότι θέλεις να διαγράψεις όλες τις καταγραφές;')) {
            weightData = [];
            saveAndRefresh();
        }
    }

    // Πρώτο τρέξιμο κατά το φορτωμα της σελίδας
    calculateTrends();
    updateDOM();
</script>

</body>
</html>

<!DOCTYPE html>
<html lang="el">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Weight Tracker</title>

<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

<style>
body{
    font-family: Arial,sans-serif;
    max-width:1000px;
    margin:auto;
    padding:20px;
    background:#f5f5f5;
}

.card{
    background:white;
    padding:20px;
    border-radius:10px;
    margin-bottom:20px;
    box-shadow:0 2px 5px rgba(0,0,0,0.1);
}

input,button{
    padding:10px;
    margin:5px;
}

button{
    cursor:pointer;
}

table{
    width:100%;
    border-collapse:collapse;
}

th,td{
    border:1px solid #ddd;
    padding:8px;
}

th{
    background:#eee;
}

.stats{
    display:flex;
    gap:20px;
    flex-wrap:wrap;
}

.stat{
    background:#fafafa;
    padding:15px;
    border-radius:8px;
    min-width:180px;
}
</style>
</head>
<body>

<h1>📉 Weight Tracker</h1>

<div class="card">

    <h3>Νέα Καταγραφή</h3>

    <input type="date" id="date">

    <input
        type="number"
        id="weight"
        step="0.1"
        placeholder="Βάρος (kg)"
    >

    <button onclick="addEntry()">Καταχώρηση</button>

</div>

<div class="card">

    <div class="stats">

        <div class="stat">
            <b>Τρέχον Βάρος</b>
            <div id="currentWeight">-</div>
        </div>

        <div class="stat">
            <b>7ήμερη Μεταβολή</b>
            <div id="weeklyChange">-</div>
        </div>

        <div class="stat">
            <b>Πρόβλεψη 30 ημερών</b>
            <div id="prediction">-</div>
        </div>

    </div>

</div>

<div class="card">
    <canvas id="chart"></canvas>
</div>

<div class="card">

    <h3>Καταγραφές</h3>

    <table>

        <thead>
            <tr>
                <th>Ημερομηνία</th>
                <th>Βάρος</th>
                <th></th>
            </tr>
        </thead>

        <tbody id="entriesTable"></tbody>

    </table>

</div>

<script>

let chart;

function getEntries(){
    return JSON.parse(localStorage.getItem("weights") || "[]");
}

function saveEntries(entries){
    localStorage.setItem("weights", JSON.stringify(entries));
}

function addEntry(){

    const date=document.getElementById("date").value;
    const weight=parseFloat(document.getElementById("weight").value);

    if(!date || !weight){
        alert("Συμπλήρωσε ημερομηνία και βάρος");
        return;
    }

    const entries=getEntries();

    entries.push({
        date,
        weight
    });

    entries.sort((a,b)=>
        new Date(a.date)-new Date(b.date)
    );

    saveEntries(entries);

    document.getElementById("weight").value="";

    render();
}

function deleteEntry(index){

    const entries=getEntries();

    entries.splice(index,1);

    saveEntries(entries);

    render();
}

function linearRegression(y){

    const n=y.length;

    let xSum=0;
    let ySum=0;
    let xySum=0;
    let xxSum=0;

    for(let i=0;i<n;i++){

        xSum+=i;
        ySum+=y[i];
        xySum+=i*y[i];
        xxSum+=i*i;

    }

    const slope=
        (n*xySum - xSum*ySum) /
        (n*xxSum - xSum*xSum);

    const intercept=
        (ySum - slope*xSum)/n;

    return {slope,intercept};
}

function render(){

    const entries=getEntries();

    const tbody=document.getElementById("entriesTable");

    tbody.innerHTML="";

    entries.forEach((e,index)=>{

        tbody.innerHTML+=`
        <tr>
            <td>${e.date}</td>
            <td>${e.weight}</td>
            <td>
                <button onclick="deleteEntry(${index})">
                Διαγραφή
                </button>
            </td>
        </tr>
        `;
    });

    if(entries.length===0){
        return;
    }

    const labels=entries.map(x=>x.date);

    const weights=entries.map(x=>x.weight);

    document.getElementById("currentWeight").innerText=
        weights[weights.length-1].toFixed(1)+" kg";

    if(weights.length>=2){

        const last=weights[weights.length-1];

        let weekAgo=weights[0];

        if(weights.length>=7){
            weekAgo=weights[weights.length-7];
        }

        const diff=last-weekAgo;

        document.getElementById("weeklyChange").innerText=
            diff.toFixed(1)+" kg";

    }

    const regression=linearRegression(weights);

    const trend=[];

    for(let i=0;i<weights.length;i++){

        trend.push(
            regression.intercept +
            regression.slope*i
        );

    }

    const futureLabels=[...labels];

    const forecast=[];

    for(let i=0;i<30;i++){

        const idx=weights.length+i;

        forecast.push(
            regression.intercept +
            regression.slope*idx
        );

        futureLabels.push(
            "Day +"+(i+1)
        );
    }

    const predictedWeight=
        forecast[forecast.length-1];

    document.getElementById("prediction").innerText=
        predictedWeight.toFixed(1)+" kg";

    if(chart){
        chart.destroy();
    }

    chart=new Chart(
        document.getElementById("chart"),
        {
            type:"line",
            data:{
                labels:futureLabels,
                datasets:[
                    {
                        label:"Βάρος",
                        data:[
                            ...weights,
                            ...Array(30).fill(null)
                        ],
                        borderWidth:2
                    },
                    {
                        label:"Trend",
                        data:[
                            ...trend,
                            ...forecast
                        ],
                        borderDash:[5,5],
                        borderWidth:2
                    }
                ]
            },
            options:{
                responsive:true
            }
        }
    );
}

document.getElementById("date").value =
    new Date().toISOString().split("T")[0];

render();

</script>

</body>
</html>

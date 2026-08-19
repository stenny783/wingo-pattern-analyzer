<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport"
      content="width=device-width, initial-scale=1.0">

<title>Wingo Pattern Analyzer V2</title>

<style>
* {
    box-sizing: border-box;
}

body {
    margin: 0;
    font-family: Arial, sans-serif;
    background: #f2f4f8;
    color: #222;
}

.header {
    background: #673ab7;
    color: white;
    padding: 22px 16px;
    text-align: center;
}

.header h1 {
    margin: 0;
    font-size: 24px;
}

.header p {
    margin: 7px 0 0;
    opacity: 0.9;
}

.container {
    padding: 15px;
    max-width: 600px;
    margin: auto;
}

.card {
    background: white;
    border-radius: 15px;
    padding: 18px;
    margin-bottom: 15px;
    box-shadow: 0 3px 10px rgba(0,0,0,0.08);
}

.card h2 {
    margin-top: 0;
    font-size: 19px;
}

input {
    width: 100%;
    padding: 15px;
    font-size: 20px;
    border: 1px solid #ccc;
    border-radius: 10px;
    text-align: center;
    margin-bottom: 10px;
}

button {
    width: 100%;
    padding: 14px;
    border: none;
    border-radius: 10px;
    font-size: 17px;
    font-weight: bold;
    margin-top: 8px;
    cursor: pointer;
}

.add-btn {
    background: #673ab7;
    color: white;
}

.predict-btn {
    background: #2196f3;
    color: white;
}

.clear-btn {
    background: #e53935;
    color: white;
}

.result-box {
    padding: 15px;
    border-radius: 12px;
    background: #f5f5f5;
    margin-top: 10px;
}

.big-result {
    font-size: 25px;
    font-weight: bold;
    text-align: center;
    margin: 10px 0;
}

.green {
    color: #16a34a;
}

.red {
    color: #dc2626;
}

.violet {
    color: #7c3aed;
}

.history {
    display: flex;
    flex-wrap: wrap;
    gap: 8px;
}

.number {
    width: 45px;
    height: 45px;
    border-radius: 50%;
    display: flex;
    justify-content: center;
    align-items: center;
    background: #eee;
    font-weight: bold;
    font-size: 18px;
}

.small {
    font-size: 14px;
    color: #666;
}

.warning {
    background: #fff3cd;
    border-left: 5px solid #ffb300;
    padding: 12px;
    border-radius: 8px;
    font-size: 14px;
}

.stat-row {
    display: flex;
    justify-content: space-between;
    padding: 8px 0;
    border-bottom: 1px solid #eee;
}

.hidden {
    display: none;
}
</style>
</head>

<body>

<div class="header">
    <h1>🤖 Wingo Pattern Analyzer V2</h1>
    <p>Pattern & statistical analysis</p>
</div>

<div class="container">

    <!-- ADD RESULT -->
    <div class="card">
        <h2>🎯 Add Wingo Result</h2>

        <input
            id="resultInput"
            type="number"
            min="0"
            max="9"
            placeholder="Enter number 0 - 9">

        <button class="add-btn" onclick="addResult()">
            ➕ Add Result
        </button>
    </div>

    <!-- NEXT ASSESSMENT -->
    <div class="card">
        <h2>🔮 Next Pattern Assessment</h2>

        <div id="assessment">
            <p class="small">
                Add some results first. The app will analyze your
                stored sequence.
            </p>
        </div>

        <button class="predict-btn" onclick="analyzePattern()">
            📊 Analyze Next Pattern
        </button>
    </div>

    <!-- HISTORY -->
    <div class="card">
        <h2>📜 Recent History</h2>

        <div id="history" class="history">
            <p class="small">No results added yet.</p>
        </div>
    </div>

    <!-- STATISTICS -->
    <div class="card">
        <h2>📈 Statistics</h2>

        <div id="statistics">
            <p class="small">No statistics available.</p>
        </div>
    </div>

    <!-- CLEAR -->
    <div class="card">
        <button class="clear-btn" onclick="clearData()">
            🗑️ Clear All Data
        </button>
    </div>

    <div class="warning">
        ⚠️ <b>Important:</b> This application performs
        statistical/pattern analysis only. It cannot guarantee
        the next Wingo result or winning outcome.
    </div>

</div>

<script>

let results = JSON.parse(
    localStorage.getItem("wingoResults") || "[]"
);

function saveResults() {
    localStorage.setItem(
        "wingoResults",
        JSON.stringify(results)
    );
}

function getColor(number) {

    number = Number(number);

    if (number === 0 || number === 5) {
        return "Violet";
    }

    if (
        number === 1 ||
        number === 3 ||
        number === 7 ||
        number === 9
    ) {
        return "Green";
    }

    return "Red";
}

function colorClass(color) {

    if (color === "Green") return "green";
    if (color === "Red") return "red";
    return "violet";
}

function addResult() {

    const input =
        document.getElementById("resultInput");

    const value = input.value.trim();

    if (
        value === "" ||
        Number(value) < 0 ||
        Number(value) > 9
    ) {
        alert("Please enter a number from 0 to 9.");
        return;
    }

    results.push(Number(value));

    /*
     * Keep the latest 100 results.
     */
    if (results.length > 100) {
        results.shift();
    }

    saveResults();

    input.value = "";

    updateHistory();
    updateStatistics();
    analyzePattern();
}

function updateHistory() {

    const history =
        document.getElementById("history");

    if (results.length === 0) {
        history.innerHTML =
            '<p class="small">No results added yet.</p>';
        return;
    }

    history.innerHTML = "";

    /*
     * Display newest first.
     */
    [...results]
        .reverse()
        .forEach(number => {

            const item =
                document.createElement("div");

            item.className = "number";

            item.textContent = number;

            history.appendChild(item);
        });
}

function countNumbers() {

    const counts = {};

    for (let i = 0; i <= 9; i++) {
        counts[i] = 0;
    }

    results.forEach(number => {
        counts[number]++;
    });

    return counts;
}

function countColors() {

    const counts = {
        Green: 0,
        Red: 0,
        Violet: 0
    };

    results.forEach(number => {
        counts[getColor(number)]++;
    });

    return counts;
}

function updateStatistics() {

    const statistics =
        document.getElementById("statistics");

    if (results.length === 0) {
        statistics.innerHTML =
            '<p class="small">No statistics available.</p>';
        return;
    }

    const numbers = countNumbers();
    const colors = countColors();

    let mostCommonNumber = 0;

    for (let i = 1; i <= 9; i++) {

        if (
            numbers[i] >
            numbers[mostCommonNumber]
        ) {
            mostCommonNumber = i;
        }
    }

    let mostCommonColor = "Green";

    if (colors.Red > colors[mostCommonColor]) {
        mostCommonColor = "Red";
    }

    if (colors.Violet > colors[mostCommonColor]) {
        mostCommonColor = "Violet";
    }

    statistics.innerHTML = `

        <div class="stat-row">
            <span>Total results</span>
            <b>${results.length}</b>
        </div>

        <div class="stat-row">
            <span>Green</span>
            <b>${colors.Green}</b>
        </div>

        <div class="stat-row">
            <span>Red</span>
            <b>${colors.Red}</b>
        </div>

        <div class="stat-row">
            <span>Violet</span>
            <b>${colors.Violet}</b>
        </div>

        <div class="stat-row">
            <span>Most frequent number</span>
            <b>${mostCommonNumber}</b>
        </div>

        <div class="stat-row">
            <span>Most frequent color</span>
            <b>${mostCommonColor}</b>
        </div>
    `;
}

function analyzePattern() {

    const assessment =
        document.getElementById("assessment");

    if (results.length < 3) {

        assessment.innerHTML = `
            <p>
                Add at least <b>3 results</b> before
                analyzing the pattern.
            </p>
        `;

        return;
    }

    const recent =
        results.slice(-20);

    const colorCounts = {
        Green: 0,
        Red: 0,
        Violet: 0
    };

    recent.forEach(number => {
        colorCounts[getColor(number)]++;
    });

    /*
     * Find the least frequent color
     * in the recent sequence.
     */
    let suggestedColor = "Green";

    if (
        colorCounts.Red <
        colorCounts[suggestedColor]
    ) {
        suggestedColor = "Red";
    }

    if (
        colorCounts.Violet <
        colorCounts[suggestedColor]
    ) {
        suggestedColor = "Violet";
    }

    /*
     * If there is a tie, use the color
     * that has not appeared most recently.
     */
    const lastColor =
        getColor(results[results.length - 1]);

    if (
        colorCounts.Green ===
        colorCounts.Red &&
        colorCounts.Red ===
        colorCounts.Violet
    ) {

        if (lastColor === "Green") {
            suggestedColor = "Red";
        } else {
            suggestedColor = "Green";
        }
    }

    /*
     * Number frequency.
     */
    const numberCounts = countNumbers();

    let leastNumber = 0;

    for (let i = 1; i <= 9; i++) {

        if (
            numberCounts[i] <
            numberCounts[leastNumber]
        ) {
            leastNumber = i;
        }
    }

    /*
     * Basic confidence calculation.
     */
    const highest =
        Math.max(
            colorCounts.Green,
            colorCounts.Red,
            colorCounts.Violet
        );

    const lowest =
        Math.min(
            colorCounts.Green,
            colorCounts.Red,
            colorCounts.Violet
        );

    let confidence = "Low";

    if (recent.length >= 10) {

        if (highest - lowest >= 4) {
            confidence = "Moderate";
        }

        if (highest - lowest >= 7) {
            confidence = "Higher";
        }
    }

    assessment.innerHTML = `

        <div class="result-box">

            <div class="big-result ${colorClass(suggestedColor)}">
                ${suggestedColor}
            </div>

            <p>
                🎯 <b>Next color assessment:</b>
                <span class="${colorClass(suggestedColor)}">
                    ${suggestedColor}
                </span>
            </p>

            <p>
                🔢 <b>Low-frequency number:</b>
                ${leastNumber}
            </p>

            <p>
                📊 <b>Recent results analyzed:</b>
                ${recent.length}
            </p>

            <p>
                📈 <b>Pattern confidence:</b>
                ${confidence}
            </p>

            <p class="small">
                Recent colors:
                Green ${colorCounts.Green},
                Red ${colorCounts.Red},
                Violet ${colorCounts.Violet}
            </p>

        </div>

        <div class="warning">
            ⚠️ This is a statistical assessment based
            on your entered history. It is not a guaranteed
            prediction of the next result.
        </div>
    `;
}

function clearData() {

    const confirmClear =
        confirm(
            "Are you sure you want to delete all stored results?"
        );

    if (!confirmClear) {
        return;
    }

    results = [];

    localStorage.removeItem("wingoResults");

    updateHistory();
    updateStatistics();
    analyzePattern();
}

/*
 * Load saved data when app opens.
 */
updateHistory();
updateStatistics();
analyzePattern();

</script>

</body>
</html>

<!DOCTYPE html>

<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Dashboard Tandon Air</title>

    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

    <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-app.js"></script>

    <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-database.js"></script>

    <style>
        body {
            font-family: Arial;
            background: #eef2f7;
            text-align: center;
        }

        header {
            background: #2c7be5;
            color: white;
            padding: 15px;
            font-size: 22px;
        }

        .container {
            display: flex;
            flex-wrap: wrap;
            justify-content: center;
        }

        .card {
            background: white;
            width: 280px;
            margin: 12px;
            padding: 20px;
            border-radius: 15px;
        }

        .value {
            font-size: 30px;
            color: #2c7be5;
        }

        button {
            padding: 10px;
            margin: 5px;
            border: none;
            border-radius: 8px;
            cursor: pointer;
        }

        .on {
            background: green;
            color: white;
        }

        .off {
            background: red;
            color: white;
        }

        .emg {
            background: orange;
            color: white;
        }

        .reset {
            background: gray;
            color: white;
        }
    </style>

</head>

<body>

    <header>🌱 Smart Greenhouse - Tandon Air</header>

    <div class="container">

        <div class="card">
            <h3>Level Air</h3>
            <div class="value" id="level">0%</div>
        </div>

        <div class="card">
            <h3>Debit Air</h3>
            <div class="value" id="flow">0 L/min</div>
        </div>

        <div class="card">
            <h3>Kualitas Air</h3>
            <div class="value" id="ppm">0 ppm</div>
        </div>

        <div class="card">
            <h3>Status Pompa</h3>
            <div id="status">OFF</div>

            <button class="on" onclick="pompaOn()">ON</button> <button class="off" onclick="pompaOff()">OFF</button><br>

            <button class="emg" onclick="emergencyOn()">EMERGENCY</button> <button class="reset" onclick="resetEmergency()">RESET</button>

            <div id="emergencyStatus">NORMAL</div>
        </div>

    </div>

    <h3>Grafik Level Air</h3>
    <canvas id="levelChart"></canvas>

    <h3>Grafik Debit Air</h3>
    <canvas id="flowChart"></canvas>

    <h3>Grafik Kualitas Air</h3>
    <canvas id="ppmChart"></canvas>

    <script>

        // ================= FIREBASE CONFIG =================
        var firebaseConfig = {
            apiKey: "AIzaSyDqHkPiJvYkYWECR-kBbuHtoqwcNSKuOQ4",
            authDomain: "smart-green-house-96fc7.firebaseapp.com",
            databaseURL: "https://smart-green-house-96fc7-default-rtdb.asia-southeast1.firebasedatabase.app",
            projectId: "smart-green-house-96fc7",
            storageBucket: "smart-green-house-96fc7.firebasestorage.app",
            messagingSenderId: "1086068447907",
            appId: "1:1086068447907:web:01cd8d0d0d863671643d16"
        };

        firebase.initializeApp(firebaseConfig);
        var db = firebase.database();

        // ================= AMBIL DATA FIREBASE =================
        db.ref("greenhouse").on("value", function (snapshot) {

            let data = snapshot.val();

            if (data) {
                level = data.level;
                flow = data.flow;
                ppm = data.ppm;

                pompa = data.pompa === "ON";
                emergency = data.emergency === "AKTIF";
            }

        });

        // ================= STATE =================
        let level = 0;
        let flow = 0;
        let ppm = 0;
        let pompa = false;
        let emergency = false;
        let lastDelete = 0;

        // ================= HISTORY =================
        let lastSave = 0;

        function simpanHistory() {

            let now = new Date();

            let waktu =
                now.getFullYear() + "-" +
                String(now.getMonth() + 1).padStart(2, '0') + "-" +
                String(now.getDate()).padStart(2, '0') + "_" +
                String(now.getHours()).padStart(2, '0') + ":" +
                String(now.getMinutes()).padStart(2, '0') + ":" +
                String(now.getSeconds()).padStart(2, '0');

            db.ref("history/" + waktu).set({
                level: level,
                flow: flow,
                ppm: ppm,
                pompa: pompa ? "ON" : "OFF"
            });

        }
        function hapusHistoryLama() {

            db.ref("history").once("value", function (snapshot) {

                snapshot.forEach(function (child) {

                    let key = child.key;

                    let waktuData = new Date(key.replace("_", " "));
                    let sekarang = new Date();

                    let selisih = (sekarang - waktuData) / 1000;

                    if (selisih > 60) { // > 60 detik
                        db.ref("history/" + key).remove();
                    }

                });

            });

        }

        // ================= DATA GRAFIK =================
        let labels = [];
        let levelData = [];
        let flowData = [];
        let ppmData = [];


        // ================= CHART =================
        const levelChart = new Chart(document.getElementById("levelChart"), {
            type: "line",
            data: { labels: labels, datasets: [{ label: "Level", data: levelData }] }
        });

        const flowChart = new Chart(document.getElementById("flowChart"), {
            type: "line",
            data: { labels: labels, datasets: [{ label: "Flow", data: flowData }] }
        });

        const ppmChart = new Chart(document.getElementById("ppmChart"), {
            type: "line",
            data: { labels: labels, datasets: [{ label: "PPM", data: ppmData }] }
        });

        // ================= KONTROL =================
        function pompaOn() { if (!emergency) pompa = true; }
        function pompaOff() { pompa = false; }
        function emergencyOn() { emergency = true; pompa = false; }
        function resetEmergency() { emergency = false; }

        // ================= SISTEM =================
        function updateSystem() {

            if (emergency) {
                pompa = false;
            }
            else if (pompa) {
                level += 5;
                flow += 2;
                ppm += 50;

                if (level >= 80) {
                    level = 80;
                    pompa = false;
                }
            }
            else {
                level -= 3;
                flow -= 1;
                ppm -= 30;

                if (level <= 20) {
                    pompa = true;
                }
            }

            // ===== UPDATE UI =====
            document.getElementById("level").innerText = level + "%";
            document.getElementById("flow").innerText = flow + " L/min";
            document.getElementById("ppm").innerText = ppm + " ppm";

            document.getElementById("status").innerText = pompa ? "ON" : "OFF";
            document.getElementById("status").style.color = pompa ? "green" : "red";

            document.getElementById("emergencyStatus").innerText =
                emergency ? "EMERGENCY AKTIF" : "NORMAL";

            // ===== GRAFIK =====
            let waktu = new Date().toLocaleTimeString();

            labels.push(waktu);
            levelData.push(level);
            flowData.push(flow);
            ppmData.push(ppm);

            if (labels.length > 10) {
                labels.shift();
                levelData.shift();
                flowData.shift();
                ppmData.shift();
            }

            levelChart.update();
            flowChart.update();
            ppmChart.update();

            // ===== FIREBASE REALTIME =====
            db.ref("greenhouse").set({
                level: level,
                flow: flow,
                ppm: ppm,
                pompa: pompa ? "ON" : "OFF",
                emergency: emergency ? "AKTIF" : "NORMAL"
            });

            // ===== HISTORY TIAP 10 DETIK =====
            let now = Date.now();

            if (now - lastSave > 10000) {
                simpanHistory();
                lastSave = now;
            }

            if (now - lastDelete > 30000) {
                hapusHistoryLama();
                lastDelete = now;
            }
        }

        // jalan tiap 2 detik
        setInterval(updateSystem, 5000);

    </script>

</body>
</html>

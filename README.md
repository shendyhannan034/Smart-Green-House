<!DOCTYPE html>

<html>
<head>
    <title>Simulasi Firebase</title>

```
<!-- Firebase -->
<script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-app.js"></script>
<script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-database.js"></script>
```

</head>

<body>

<h2>Simulasi Data Greenhouse</h2>

<p>Level Air: <span id="level">-</span></p>
<p>Debit Air: <span id="flow">-</span></p>

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

// ================= SIMULASI DATA =================
setInterval(() => {

    let level = Math.floor(Math.random() * 100);
    let flow = Math.floor(Math.random() * 20);

    // kirim ke Firebase
   db.ref("history").push({
    level: level,
    flow: flow,
    time: new Date().toLocaleTimeString()
});

}, 3000);

// ================= AMBIL DATA =================
db.ref("greenhouse").on("value", function(snapshot) {

    let data = snapshot.val();

    if (data) {
        document.getElementById("level").innerText = data.level;
        document.getElementById("flow").innerText = data.flow;
    }

});

</script>

</body>
</html>

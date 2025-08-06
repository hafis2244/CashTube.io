<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8">
  <title>V-Tube Dashboard</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <style>
    body {
      margin: 0;
      font-family: 'Segoe UI', sans-serif;
      background: #f0f0f0;
      color: #333;
    }

    header {
      background: #ff3c3c;
      color: white;
      padding: 20px;
      text-align: center;
    }

    .container {
      max-width: 800px;
      margin: auto;
      padding: 20px;
    }

    .card {
      background: white;
      padding: 20px;
      border-radius: 10px;
      box-shadow: 0 4px 8px rgba(0,0,0,0.1);
      margin-bottom: 20px;
    }

    h2 {
      margin-top: 0;
      color: #ff3c3c;
    }

    button {
      background: #ff3c3c;
      color: white;
      border: none;
      padding: 10px 20px;
      border-radius: 6px;
      cursor: pointer;
      margin-top: 10px;
    }

    button:hover {
      background: #e22e2e;
    }

    #notifikasiContainer p {
      background: #ffecec;
      padding: 8px;
      border-left: 4px solid #ff3c3c;
      margin: 5px 0;
      border-radius: 4px;
    }
  </style>
</head>
<body>

  <header>
    <h1>V-Tube Dashboard</h1>
    <p>Selamat datang, CASH!</p>
  </header>

  <div class="container">
    <div class="card">
      <h2>Status Akun</h2>
      <p><strong>Email:</strong> <span id="emailDisplay"></span></p>
      <p><strong>Saldo:</strong> Rp <span id="saldoDisplay"></span></p>
      <p><strong>Level:</strong> <span id="levelDisplay"></span></p>
      <button onclick="cekBonusMingguan()">🎁 Cek Bonus Mingguan</button>
    </div>

    <div class="card">
      <h2>Profil & Aktivitas</h2>
      <p><strong>Transaksi:</strong> <span id="jumlahTransaksi"></span> kali</p>
      <h3>Notifikasi Terbaru</h3>
      <div id="notifikasiContainer"></div>
    </div>
  </div>

  <script>
    const email = "cash@vtube.id";
    let semuaUser = JSON.parse(localStorage.getItem("dataUser")) || {};
    let user = semuaUser[email] || {
      nama: "CASH",
      saldo: 0,
      level: "Bronze",
      tanggalBonusTerakhir: null,
      riwayatTransaksi: [],
      notifikasi: []
    };

    function today() {
      const d = new Date();
      return d.toISOString().split("T")[0];
    }

    function simpanUser() {
      semuaUser[email] = user;
      localStorage.setItem("dataUser", JSON.stringify(semuaUser));
    }

    function hitungLevelUser() {
      let total = 0;
      for (let t of user.riwayatTransaksi) {
        if (t.jenis === "Deposit" || t.jenis === "Pembelian") {
          total += t.jumlah;
        }
      }
      if (total >= 2000000) {
        user.level = "Gold";
      } else if (total >= 500000) {
        user.level = "Silver";
      } else {
        user.level = "Bronze";
      }
      simpanUser();
    }

    function tambahNotifikasi(pesan) {
      user.notifikasi.push({ tanggal: today(), pesan });
      simpanUser();
    }

    function luckyDrawMingguan() {
      hitungLevelUser();
      let bonus = 0;
      if (user.level === "Gold") bonus = 100000;
      else if (user.level === "Silver") bonus = 50000;
      else bonus = 25000;

      user.saldo += bonus;
      user.tanggalBonusTerakhir = today();
      user.riwayatTransaksi.push({
        tanggal: today(),
        jenis: "Bonus Mingguan",
        jumlah: bonus
      });
      tambahNotifikasi(`🎁 Bonus Mingguan Rp ${bonus} diterima sebagai ${user.level}`);
      alert(`🎉 Kamu mendapat bonus Rp ${bonus} sebagai pengguna ${user.level}`);
      simpanUser();
      tampilkanDashboard();
    }

    function cekBonusMingguan() {
      const terakhir = user.tanggalBonusTerakhir;
      if (!terakhir || new Date(today()) - new Date(terakhir) >= 7 * 24 * 60 * 60 * 1000) {
        luckyDrawMingguan();
      } else {
        alert("⏳ Bonus mingguan belum tersedia. Coba lagi nanti.");
      }
    }

    function tampilkanDashboard() {
      document.getElementById("emailDisplay").textContent = email;
      document.getElementById("saldoDisplay").textContent = user.saldo;
      document.getElementById("levelDisplay").textContent = user.level;
      document.getElementById("jumlahTransaksi").textContent = user.riwayatTransaksi.length;

      const notif = document.getElementById("notifikasiContainer");
      notif.innerHTML = "";
      user.notifikasi.slice(-5).reverse().forEach(n => {
        const p = document.createElement("p");
        p.textContent = `${n.tanggal}: ${n.pesan}`;
        notif.appendChild(p);
      });
    }

    tampilkanDashboard();
  </script>

</body>
</html>

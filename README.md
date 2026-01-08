# Log-Analysis-Defense-Lab
Studi Kasus: Simulasi Serangan Brute Force dan Analisis Log pada Server Node.jsRole: Blue Team (Defender)

Log Analysis: Seni membaca jejak digital untuk menemukan pola serangan.
Failed Login: Indikasi Brute Force.
Suspicious Pattern: Kode injeksi atau pola bot.
Blue Team Tasks: Deteksi dini, analisa pola, dan respon cepat.
Tentu, ini versi **Super Detail**. Dokumen ini tidak hanya laporan, tapi juga **Panduan Langkah-demi-Langkah (Step-by-Step Guide** untuk orang lain yang ingin meniru Lab yang kamu buat.

Simpan ini sebagai `README.md` di repository GitHub kamu.

```markdown
# 🛡️ Log Analysis & Blue Team Defense Lab
**Studi Kasus Lengkap:** Deteksi, Simulasi, dan Mitigasi Serangan Brute Force pada Web Server.

Lab ini didesain untuk memahami bagaimana tim pertahanan (Blue Team) membaca log server, mendeteksi anomali, dan melakukan tindakan pencegahan terhadap serangan *brute force*.

---

## 📚 1. Konsep Teori (Theory)

Sebelum masuk ke praktek, kita pahami dulu konsep berdasarkan gambar referensi "Project #1 LOG ANALYSIS MINI LAB":

### A. Technical Skills (Log Analysis)
*   **Log Analysis:** Seni membedah catatan sistem (`/var/log/`, `Event Viewer`, atau file log aplikasi) untuk menemukan jejak digital.
*   **Failed Login:** Indikator utama peretasan. Pengguna normal mungkin salah password 1-2 kali. Kalau ada 10 kali dalam 1 detik, itu pasti bot.
*   **Suspicious Patterns:** Pola aneh seperti menggunakan tanda kutip `'` (SQL Injection), mencoba password umum (`123456`, `admin`), atau akses dari negara asing.

### B. Tools
*   **Node.js:** Server web yang akan kita buat.
*   **Axios:** Alat (HTTP Client) untuk mengirim request secara otomatis (mensimulasikan penyerang).

---

## 🏗️ 2. Persiapan Lab (Setup)

Buatlah struktur folder proyek sebagai berikut:

```text
LabLogNodeJS/
├── public/
│   └── index.html       (Halaman Login Palsu)
├── logs/
│   └── honey.log        (File Penyimpanan Log)
├── node_modules/       (Otomatis dibuat npm)
├── package.json        (Konfigurasi Proyek)
├── server.js           (Kode Server Target)
└── attacker.js         (Kode Penyerang)
```

### Langkah 1: Inisialisasi
Buka terminal di folder `LabLogNodeJS`, lalu jalankan:
```bash
npm init -y
npm install express axios
```
Pastikan folder `logs` sudah dibuat secara manual sebelum menjalankan server.

---

## 💻 3. Coding: Target Server (The Victim)

Tujuan server ini adalah menyediakan halaman login dan **mencatat segala yang terjadi**.

### File: `public/index.html`
Halaman login sederhana tanpa CSS rumit.
```html
<!DOCTYPE html>
<html>
<head><title>Login</title></head>
<body>
    <h3>Login Admin</h3>
    <form action="/login" method="POST">
        <input type="text" name="username" placeholder="Username" required>
        <input type="password" name="password" placeholder="Password" required>
        <button type="submit">LOGIN</button>
    </form>
</body>
</html>
```

### File: `server.js`
Kode utama server. Perhatikan bagian **Middleware Logging** yang menulis ke file `honey.log`.

```javascript
const express = require('express');
const fs = require('fs');
const path = require('path');

const app = express();

// 1. Setup Parser untuk membaca data Form HTML & JSON
app.use(express.json()); 
app.use(express.urlencoded({ extended: true })); 
app.use(express.static('public'));

// 2. MIDDLEWARE: Juru Tulis Log Otomatis
// Fungsi ini jalan SEBELUM setiap request
app.use((req, res, next) => {
    const waktu = new Date().toISOString();
    const ip = req.ip || req.connection.remoteAddress;
    const logEntry = `[${waktu}] [IP: ${ip}] ${req.method} ${req.url} Body: ${JSON.stringify(req.body)}\n`;

    // Tulis ke file log
    fs.appendFile(path.join(__dirname, 'logs', 'honey.log'), logEntry, (err) => {
        if (err) console.error('Gagal nulis log:', err);
    });

    console.log(logEntry.trim()); // Tampilkan di terminal juga
    next(); 
});

// 3. Route Login
app.post('/login', (req, res) => {
    // Kita pura-pura semua login GAGAL untuk simulasi
    res.send("Login Gagal! Username atau Password salah.");
});

app.listen(3000, () => {
    console.log('Server berjalan di http://localhost:3000');
});
```

---

## ⚔️ 4. Coding: Simulasi Serangan (The Attacker)

Kita akan membuat script `attacker.js` yang melakukan serangan **Brute Force** (tebak password menggunakan daftar kata).

### File: `attacker.js`

```javascript
const axios = require('axios');

const targetUrl = 'http://localhost:3000/login';

// Daftar Password "Kamus Serangan" (Dictionary Attack)
const passwordList = [
    '123456',
    'password',
    'admin',
    '12345678',
    'qwerty',
    'letmein',
    'monkey',
    'dragon',
    'sunshine',
    'iloveyou'
];

console.log(`[!] Serangan dimulai ke ${targetUrl}...`);
console.log(`[!] Mencoba ${passwordList.length} password pada user 'admin'...\n`);

// Looping otomatis
passwordList.forEach((pass, index) => {
    const payload = {
        username: 'admin',
        password: pass
    };

    // Kirim request dengan header form-encoded (agar server bisa baca)
    axios.post(targetUrl, payload, {
        headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
        transformRequest: [(data) => {
            return Object.keys(data).map(key => encodeURIComponent(key) + '=' + encodeURIComponent(data[key])).join('&');
        }]
    })
    .then(() => console.log(`[+] Sukses request ke-${index + 1} (${pass})`))
    .catch(() => console.log(`[-] Gagal request ke-${index + 1}`));
});
```

---

## 🔍 5. Eksekusi & Analisa Log (Execution)

### Langkah A: Jalankan Server
```bash
node server.js
```
*Biarkan terminal ini berjalan di latar belakang.*

### Langkah B: Jalankan Penyerang
Buka terminal baru:
```bash
node attacker.js
```

### Langkah C: Lihat Hasil Log
Buka file `logs/honey.log`. Anda akan melihat pola serangan seperti ini:

```text
[2026-01-08T05:24:37.552Z] [IP: ::1] POST /login Body: {"username":"admin","password":"123456"}
[2026-01-08T05:24:37.553Z] [IP: ::1] POST /login Body: {"username":"admin","password":"password"}
...
[2026-01-08T05:24:37.559Z] [IP: ::1] POST /login Body: {"username":"admin","password":"iloveyou"}
```

### Analisa Blue Team (Deteksi)
1.  **Kecepatan Tinggi:** Semua log terjadi pada jam yang sama detiknya (`05:24:37`). Tidak mungkin dilakukan manusia manual. Ini bot.
2.  **Pattern Fixed:** Username selalu `admin`, hanya password yang berubah. Ini indikasi **Credential Stuffing** atau **Dictionary Attack**.

---

## 🛡️ 6. Mitigasi (Solusi Pertahanan)

Berdasarkan log di atas, berikut adalah langkah-langkah mitigasi yang diterapkan:

### A. Strategi Obscurity (Ganti Username)
Mengubah username admin default (`admin`) menjadi sesuatu yang unik (`sys_admin_2024`).
*   *Kenapa?* Script penyerang hanya fokus menembak `admin`. Kalau user tidak ada, peluru mereka sia-sia.

### B. Firewall Blocking (Blokir IP)
Memblokir IP `::1` (atau IP eksternal jika real case).
*   *Kenapa?* Memutus koneksi fisik agar penyerang tidak bisa melanjutkan percobaan.

### C. Generic Error Message (Pesan Error Palsu)
Ubah respon server dari:
*"User tidak ditemukan"* atau *"Password salah"*
Menjadi:
*"Login Gagal! Username atau Password salah."*

*   *Kenapa?* Untuk mencegah **User Enumeration**. Hacker akan bingung apakah usernya benar tapi password salah, atau usernya tidak ada sama sekali.

---

## ✅ 7. Kesimpulan

Melalui lab ini, kita telah mempelajari siklus penuh keamanan siber:
1.  **Monitoring:** Server mencatat aktivitas.
2.  **Detection:** Blue Team mengenali pola aneh dari log.
3.  **Response:** Blue Team memblokir IP dan mengubah konfigurasi akun.
4.  **Hardening:** Menerapkan pesan error generik untuk menghalangi serangan lanjutan.

---
*Lab ini dibuat untuk tujuan edukasi keamanan siber.*
```

     

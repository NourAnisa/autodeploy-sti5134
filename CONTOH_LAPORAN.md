# 📄 Contoh Laporan Praktikum Pertemuan 5 (Nilai A+)
### Mata Kuliah: STI5134 — Layanan dan Sistem Virtual (3 SKS / Semester 5)
**Sub-CPMK-3: Memahami Tipe-Tipe Sistem Virtual (Application & Server Virtualization)**  
**Dosen Pengampu:** Ir. Nor Anisa & Andry Fajar Zulkarnain, S.ST., M.T.  
**Program Studi S1 Teknologi Informasi — Fakultas Teknik, Universitas Lambung Mangkurat**

---

## 📌 Identitas Mahasiswa (Contoh Acuan)
* **Nama Lengkap:** Ahmad Fauzan Pratama
* **NIM:** 2210817210045
* **Kelas:** TI-A / Semester 5
* **URL Live Hasil Praktikum (GitHub Pages):** [https://ahmadfauzan.github.io/autodeploy-sti5134/](https://ahmadfauzan.github.io/autodeploy-sti5134/)
* **URL Server Webhook VPS Kelas:** `http://103.180.124.142:8080`

---

## 1. Tujuan Praktikum
1. Mengimplementasikan konsep **Application Virtualization** menggunakan platform *cloud container* dan *edge serving*.
2. Membangun pipeline **Continuous Integration & Continuous Deployment (CI/CD)** otomatis berbasis GitHub Webhooks dan Git.
3. Membuktikan efisiensi otomasi rilis kode dibandingkan metode konvensional (*manual FTP upload*).

---

## 2. Bukti Deployment & Hasil Pengujian

### 2.1 Tampilan Website Live (Hasil Personalisasi)
Mahasiswa telah melakukan fork, mengaktifkan GitHub Pages, dan memodifikasi file `index.html`:
* **Status HTTP:** `200 OK` (Protokol HTTP/2 dengan HTTPS aktif).
* **Konten Tampil:**
  * Nama Mahasiswa: **Ahmad Fauzan Pratama**
  * NIM: **2210817210045**
  * Status Badge: `LIVE via Automated CI/CD Pipeline` (Hijau).

### 2.2 Bukti Integrasi Webhook ke Server VPS Dosen
Pada server VPS Ubuntu 24.04 LTS (`103.180.124.142`), daemon Webhook Receiver di port 8080 mencatat log eksekusi otomatis saat terjadi `git push`:
```text
[2026-10-02 02:24:47] Webhook received via POST from GitHub (IP 140.82.115.x)
Executing: /usr/local/bin/autodeploy.sh
From https://github.com/NourAnisa/autodeploy-sti5134
 * branch            main       -> FETCH_HEAD
   47823fb..c7d962c  main       -> origin/main
Updating 47823fb..c7d962c
Fast-forward
Fri Oct  2 02:24:50 WITA 2026: AutoDeploy Berhasil Dijalankan ke /var/www/html
```

---

## 3. Data Pengukuran Kinerja (Latensi Commit-to-Deploy)

Pengujian dilakukan sebanyak 5 kali untuk mengukur durasi sejak penekanan tombol *Commit changes* di GitHub hingga perubahan tampil di browser:

| Uji Ke- | Tipe Perubahan Kode | Waktu Push (T0) | Web Ter-update (T1) | Durasi Latensi | Status |
| :---: | :--- | :---: | :---: | :---: | :---: |
| 1 | Edit Nama & NIM pada `index.html` | 02:30:10 | 02:30:42 | **32 detik** | Berhasil (200 OK) |
| 2 | Modifikasi Warna CSS Card Dashboard | 02:35:00 | 02:35:38 | **38 detik** | Berhasil (200 OK) |
| 3 | Penambahan Tautan Portofolio | 02:40:15 | 02:40:44 | **29 detik** | Berhasil (200 OK) |
| 4 | Pembaruan Footer Hak Cipta | 02:45:20 | 02:45:51 | **31 detik** | Berhasil (200 OK) |
| 5 | Commit serentak beban multi-user | 02:50:00 | 02:50:39 | **39 detik** | Berhasil (200 OK) |

> **Rata-rata Durasi Deployment:** **33,8 detik**  
> *Sistem mampu memangkas waktu kerja manual hingga lebih dari 90%.*

---

## 4. Pembahasan & Analisis Teknis (5 Pertanyaan Wajib)

### Pertanyaan 1: Mekanisme Isolasi Application Virtualization
> *Bagaimana GitHub Pages dan Dokploy mengisolasi aplikasi web pengguna dari ribuan aplikasi lain di server yang sama?*

**Analisis:**  
GitHub Pages dan Dokploy menerapkan isolasi berbasis *Container Sandboxing* dengan fitur inti kernel Linux:
1. **PID & Mount Namespace:** Setiap container berjalan di ruang proses terisolasi. Aplikasi web tidak memiliki visibilitas terhadap direktori maupun file milik penyewa (*tenant*) lain di server yang sama.
2. **Control Groups (cgroups):** Mengunci alokasi memori (RAM) dan utilisasi CPU per aplikasi. Apabila sebuah aplikasi mengalami lonjakan beban (*traffic spike*) atau *infinite loop*, aplikasi tetangga tidak akan terdampak.
3. **Chroot Jail:** Direktori webroot (`/var/www/html`) dikunci secara virtual sehingga mencegah serangan *directory traversal* (`../../etc/passwd`).

---

### Pertanyaan 2: Alur Sinyal Webhook dari Commit hingga Live
> *Jelaskan proses alur sinyal Webhook dari saat tombol 'Commit Changes' ditekan di GitHub hingga diterima oleh Webhook Receiver di server!*

**Analisis:**  
1. **Git Hook Internal:** Penekanan tombol *Commit changes* memicu hook `post-receive` di server GitHub.
2. **Payload JSON Generation:** GitHub merangkum metadata commit (author, timestamp, file diff, SHA hash) menjadi payload HTTP berformat JSON.
3. **WAN Transmission:** GitHub mengirimkan paket `HTTP POST` ke URL tujuan publik: `http://103.180.124.142:8080`.
4. **Daemon Validation & Shell Spawn:** Script Python daemon (`/opt/webhook.py`) menerima payload, membalas dengan status `200 OK`, lalu secara *asynchronous* menjalankan `/usr/local/bin/autodeploy.sh` yang mengeksekusi perintah `git pull origin main` dan menyalin file ke `/var/www/html`.

---

### Pertanyaan 3: Analisis Faktor Penyebab Latensi (Jeda Waktu)
> *Mengapa terdapat jeda waktu rata-rata 33 detik antara saat commit ditekan dan web live ter-update?*

**Analisis:**  
Jeda waktu tersebut merupakan total akumulasi dari 4 tahapan propagasi:
1. **GitHub Action Queue (3–6 detik):** Antrean server build dispatcher GitHub global.
2. **Transmisi WAN (1–3 detik):** Round-Trip Time (RTT) jaringan dari data center GitHub ke VPS di Indonesia.
3. **Eksekusi Disk & Git I/O (4–8 detik):** Waktu komputasi membaca tree SHA objek Git dan penulisan file ke NVMe disk.
4. **Edge CDN Cache Invalidation (15–25 detik):** Propagasi pembersihan cache browser dan node CDN agar konten baru disajikan kepada pengguna.

---

### Pertanyaan 4: Perbandingan Efisiensi CI/CD vs Metode FTP Manual
> *Bandingkan efisiensi waktu dan potensi human-error antara sistem AutoDeploy CI/CD ini vs metode manual (cPanel/FTP)!*

| Parameter | Metode Konvensional (FTP/cPanel) | Metode CI/CD AutoDeploy |
| :--- | :--- | :--- |
| **Waktu Rilis** | 5 – 10 menit per deploy | ~33 detik |
| **Human Error** | Tinggi (berisiko salah overwrite file/folder) | Nol (pembaharuan deterministik berbasis Git Tree) |
| **Keamanan Kredensial** | Pengembang harus tahu password root/FTP | Aman (cukup izin push ke repositori) |
| **Fitur Rollback** | Sulit (harus simpan backup manual) | Instan (cukup `git revert`) |

---

### Pertanyaan 5: Alasan Arsitektur Redundansi Hybrid (Webhook + Cron Job)
> *Mengapa server VPS menggabungkan Webhook dan Cron Job sekaligus?*

**Analisis:**  
Penggabungan ini adalah penerapan konsep **High Availability (HA) & Fault Tolerance**:
* **Webhook (Fast Path):** Bertindak sebagai pemicu utama yang bekerja seketika (*real-time event-driven*).
* **Cron Job (Recovery Path):** Bertindak sebagai jaring pengaman (*failsafe*) setiap 5 menit. Jika terjadi gangguan jaringan sementara (misalnya paket HTTP POST drop atau server reboot saat commit terjadi), cron job akan menyinkronkan repositori secara otomatis sehingga tidak ada revisi yang tertinggal (*Zero Drift*).

---

## 5. Kesimpulan
1. **Application Virtualization** memungkinkan penyajian aplikasi secara instan, aman, dan hemat biaya tanpa intervensi fisik pada perangkat keras server.
2. Pipeline CI/CD AutoDeploy terbukti memangkas waktu kerja hingga 90% dan mengeliminasi kesalahan manusia dalam manajemen rilis software.
3. Desain arsitektur hybrid (*Webhook + Cron Backup*) memberikan keandalan maksimal bagi sistem virtual kelas laboratorium.

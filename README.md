# 🚀 Panduan Lengkap AutoDeploy CI/CD Pipeline & Application Virtualization
### Mata Kuliah: STI5134 — Layanan dan Sistem Virtual (Pertemuan 5 / Sub-CPMK-3)
**Program Studi S1 Teknologi Informasi — Fakultas Teknik, Universitas Lambung Mangkurat**  
**Dosen Pengampu:** Ir. Nor Anisa & Andry Fajar Zulkarnain, S.ST., M.T.

---

## 📌 Ringkasan Proyek

Repository ini adalah modul acuan dan template praktikum langsung untuk mempelajari **Application Virtualization** dan otomasi deployment berbasis **CI/CD (Continuous Integration & Continuous Deployment)**.

Sistem ini telah diuji dan berjalan secara penuh di infrastruktur server mandiri menggunakan:
- **Cloud Server VPS:** Ubuntu 24.04 LTS (1 Core vCPU, 1 GB RAM, 20 GB NVMe Disk)
- **Web Server:** Nginx (Serving web statis di `/var/www/html`)
- **PaaS & Container Manager:** Dokploy (Docker engine di port 3000)
- **Webhook Receiver Daemon:** Python HTTP Server (Port 8080 via Systemd)
- **Otomasi Deploy:** Script Bash (`autodeploy.sh`) + Git Pull + Cron Job Backup
- **Pemicu (Trigger):** GitHub Webhooks (Event `push` ke branch `main`)

> 📚 **DOKUMEN & PANDUAN PENTING MAHASISWA:**
> * 🐘 [**Panduan Upload & Deploy Framework (Laravel, React, Node.js, PHP)**](TUTORIAL_UPLOAD_FRAMEWORK.md) — *Cara aman upload proyek framework tanpa vendor & node_modules*
> * 📄 [**Contoh Laporan Praktikum Sesuai Standar RPS (Nilai A+)**](CONTOH_LAPORAN.md) — *Template dan data pengujian latensi commit-to-deploy*
> * ⚡ [**Tab GitHub Actions**](https://github.com/NourAnisa/autodeploy-sti5134/actions) — *Pantau proses build & deploy otomatis*

---

## 🗺️ Gambaran Sistem & Arsitektur CI/CD

```text
Kamu push kode ke GitHub (branch main)
        │
        ▼
GitHub Actions berjalan otomatis (file: .github/workflows/deploy.yml)
  • 🔍 1. Deteksi Framework (Laravel / PHP Composer / PHP Native / Node.js / HTML5)
  • 📦 2. Buat ZIP dari kode kamu (dotfiles disertakan, .env & vendor dikecualikan)
  • 🌐 3. Kirim trigger webhook & payload ke Server VPS (103.180.124.142:8080)
        │
        ▼
Server VPS Menerima Sinyal & Eksekusi Otomatis
  • Ekstrak / Git Pull kode terbaru ke /opt/sti5134
  • Sinkronisasi file ke web server Nginx (/var/www/html)
  • Failsafe redundansi: Cron Job setiap 5 menit (*/5)
        │
        ▼
Web kamu live di:
  • Server VPS Kelas: http://103.180.124.142
  • Dokploy Container: http://103.180.124.142:3000
  • GitHub Pages Mandiri: https://[username].github.io/autodeploy-sti5134/

Di GitHub Actions → tab Actions → job terbaru → Job Summary
kamu akan menemukan SEMUA informasi status deployment + link live secara lengkap!
```

---

## 🛠️ BAGIAN 1: Tutorial Membangun Server AutoDeploy Mandiri (Persis Praktikum Dosen)

Panduan ini mendokumentasikan langkah demi langkah pembuatan server dari nol hingga siap digunakan:

### Langkah 1: Penyediaan VPS (Virtual Private Server)
1. Gunakan penyedia VPS pilihan (contoh: *vpsmurah.co.id* paket Shared Xeon Rp 25.000/bulan).
2. Pilih spesifikasi minimal:
   - **vCPU:** 1 Core
   - **RAM:** 1 GB
   - **Storage:** 20 GB NVMe
   - **Sistem Operasi:** **Ubuntu 24.04 LTS** (Wajib 64-bit).
   - **Control Panel:** Dokploy (opsional) atau Clean OS.

### Langkah 2: Konfigurasi Port Forwarding (Jaringan & Akses)
Jika VPS Anda menggunakan IP Shared dengan Port Forwarding:
1. Buka menu **Jaringan & Akses** ➔ **Port Forwarding**.
2. Pastikan port berikut telah diarahkan ke IP Publik:
   - **SSH:** Port `22` (diarahkan ke port publik sistem, misal `20114`).
   - **Dokploy:** Port `3000` (TCP, label: `Dokploy`).
   - **Webhook Receiver:** Port `8080` (TCP, label: `Webhook`).

### Langkah 3: Remote Server via SSH
Buka terminal (PowerShell / Command Prompt / Terminal macOS/Linux) dan jalankan:
```bash
ssh -p 20114 root@103.180.124.142
```
*Masukkan password root VPS Anda.*

### Langkah 4: Instalasi Nginx dan Git
Perbarui repositori sistem dan pasang Nginx sebagai web server utama:
```bash
apt-get update -y
apt-get install -y nginx git python3
```

### Langkah 5: Clone Repository Proyek
Unduh repositori yang akan di-autodeploy ke dalam folder `/opt`:
```bash
rm -rf /opt/sti5134
git clone https://github.com/NourAnisa/autodeploy-sti5134 /opt/sti5134
cp -rT /opt/sti5134 /var/www/html
```

### Langkah 6: Membuat Script Shell AutoDeploy
Buat file eksekusi deployment di `/usr/local/bin/autodeploy.sh`:
```bash
cat << 'EOF' > /usr/local/bin/autodeploy.sh
#!/bin/bash
cd /opt/sti5134
git pull origin main 2>&1
cp -rT /opt/sti5134 /var/www/html
echo "$(date): AutoDeploy Berhasil Dijalankan" >> /var/log/autodeploy.log
EOF

chmod +x /usr/local/bin/autodeploy.sh
```

### Langkah 7: Membuat Daemon Webhook Receiver (Python)
Buat service penerima webhook yang bertugas mendengarkan sinyal POST dari GitHub di port `8080`:
```bash
cat << 'EOF' > /opt/webhook.py
#!/usr/bin/env python3
from http.server import HTTPServer, BaseHTTPRequestHandler
import subprocess, logging

logging.basicConfig(filename='/var/log/webhook.log', level=logging.INFO,
                    format='%(asctime)s %(message)s')

HTML_STATUS = """<!DOCTYPE html>
<html>
<head><title>AutoDeploy Webhook - STI5134</title></head>
<body style="font-family:sans-serif;text-align:center;padding:50px;background:#0f172a;color:#fff;">
    <h2>🚀 STI5134 AutoDeploy Webhook Receiver</h2>
    <p style="color:#34d399;">STATUS: AKTIF &amp; MENDENGARKAN EVENT POST GITHUB</p>
    <p>Repository: https://github.com/NourAnisa/autodeploy-sti5134</p>
</body>
</html>"""

class Handler(BaseHTTPRequestHandler):
    def do_GET(self):
        self.send_response(200)
        self.send_header('Content-Type', 'text/html; charset=utf-8')
        self.end_headers()
        self.wfile.write(HTML_STATUS.encode('utf-8'))

    def do_POST(self):
        length = int(self.headers.get('Content-Length', 0))
        self.rfile.read(length)
        logging.info("GitHub Webhook Triggered -> Menjalankan autodeploy.sh")
        subprocess.Popen(['/usr/local/bin/autodeploy.sh'])
        self.send_response(200)
        self.send_header('Content-Type', 'application/json')
        self.end_headers()
        self.wfile.write(b'{"status":"ok","message":"AutoDeploy triggered"}')

HTTPServer(('0.0.0.0', 8080), Handler).serve_forever()
EOF
```

Jadikan daemon sebagai systemd service agar otomatis hidup saat server reboot:
```bash
cat << 'EOF' > /etc/systemd/system/autodeploy-webhook.service
[Unit]
Description=AutoDeploy Webhook Receiver STI5134
After=network.target

[Service]
ExecStart=/usr/bin/python3 /opt/webhook.py
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl enable autodeploy-webhook
systemctl start autodeploy-webhook
```

### Langkah 8: Memasang Failsafe Cron Job
Sebagai cadangan jika koneksi webhook gagal, pasang cron job untuk auto-pull setiap 5 menit:
```bash
(crontab -l 2>/dev/null | grep -v autodeploy.sh; echo "*/5 * * * * /usr/local/bin/autodeploy.sh") | crontab -
```

### Langkah 9: Menghubungkan GitHub Webhooks
1. Buka repositori Anda di GitHub: `https://github.com/NourAnisa/autodeploy-sti5134`.
2. Klik tab **Settings** ➔ Pilih menu **Webhooks** di bilah kiri.
3. Klik tombol **Add webhook**.
4. Isi formulir konfigurasi:
   - **Payload URL:** `http://103.180.124.142:8080`
   - **Content type:** `application/json`
   - **Secret:** *(Kosongkan)*
   - **SSL verification:** Pilih *Disable (not recommended)* karena menggunakan HTTP port 8080.
   - **Which events would you like to trigger this webhook?:** Pilih *Just the push event*.
5. Klik **Add webhook**. GitHub akan mengirimkan ping payload perdana dengan indikator centang hijau (HTTP 200).

---

## 👨‍💻 BAGIAN 2: Tutorial Praktikum AutoDeploy (Untuk Mahasiswa)

Bagi mahasiswa yang mengikuti praktikum Pertemuan 5, terdapat 2 jalur implementasi yang dapat dilakukan:

### 🌟 JALUR UTAMA: Deploy Mandiri Gratis via GitHub Pages (3 Menit, 0 Rupiah)
Metode ini memanfaatkan infrastruktur *Serverless Edge / Application Virtualization* milik GitHub:

1. **Fork Repositori:**
   - Buka: [https://github.com/NourAnisa/autodeploy-sti5134](https://github.com/NourAnisa/autodeploy-sti5134)
   - Klik tombol **Fork** (kanan atas) ➔ Klik **Create fork**.
2. **Aktifkan Fitur AutoDeploy (GitHub Pages):**
   - Di repositori hasil fork Anda, buka menu **Settings** ➔ **Pages**.
   - Pada bagian **Build and deployment**:
     - *Source:* Deploy from a branch
     - *Branch:* Pilih `main` dan folder `/ (root)`
   - Klik **Save**.
3. **Akses Website Live Anda:**
   - Tunggu 30–60 detik. URL publik HTTPS Anda akan aktif di format:  
     `https://[username-github-anda].github.io/autodeploy-sti5134/`
4. **Uji Keajaiban CI/CD Pipeline (AutoDeploy):**
   - Buka file `index.html` langsung di GitHub.
   - Klik ikon pensil (**Edit this file**).
   - Cari teks `Ganti Nama Anda` dan ubah menjadi Nama Lengkap Anda.
   - Ganti `NIM: 2210817xxxxxx` dengan NIM asli Anda.
   - Gulir ke bawah, klik tombol hijau **Commit changes...** ➔ **Commit changes**.
   - Buka kembali URL web Anda dan lakukan *Hard Refresh* (`Ctrl + F5` atau `Cmd + Shift + R`).
   - **Hasil:** Website Anda berubah secara otomatis di internet tanpa perlu upload manual melalui FTP/File Manager!

---

### 🌐 JALUR SERVER LAB: Deploy Terpusat ke VPS Kampus (Dosen)
Bagi mahasiswa yang ditugaskan melakukan kontribusi ke server pusat kelas:
1. Buat branch baru atau lakukan Pull Request (PR) ke repositori dosen: `NourAnisa/autodeploy-sti5134`.
2. Saat Dosen melakukan merge PR ke branch `main`, GitHub secara otomatis memicu Webhook ke `http://103.180.124.142:8080`.
3. Server VPS kelas akan mengeksekusi `git pull` secara instan dalam waktu kurang dari 5 detik.
4. Perubahan seluruh mahasiswa langsung tampil di server kelas.

---

## 🐘 BAGIAN 2.5: Panduan Mahasiswa Deploy Framework (Laravel, React, Node.js, PHP)

> 📘 **Panduan Lengkap Langkah Demi Langkah:**  
> Buka file panduan khusus: [**`TUTORIAL_UPLOAD_FRAMEWORK.md`**](TUTORIAL_UPLOAD_FRAMEWORK.md) untuk panduan mendalam tentang konfigurasi `.gitignore`, `.env.example`, database SQLite/MySQL, dan trik membersihkan folder yang terlanjur ter-commit.

### 🚨 3 Aturan Emas Upload Framework ke GitHub:
1. **Dilarang keras meng-upload folder `vendor/` (Laravel) dan `node_modules/` (React/Node.js):**
   * Ukuran folder tersebut mencapai 200–500 MB dan berisi puluhan ribu file dependensi pihak ketiga.
   * Cukup upload `composer.json` atau `package.json`. Server cloud / PaaS akan otomatis mengunduhnya via `composer install` atau `npm install`.
2. **Dilarang meng-upload file `.env`:**
   * File `.env` memuat password database dan secret key lokal. Cukup upload template publiknya: **`.env.example`**.
3. **Konfigurasi Webroot:**
   * Pada Laravel, arahkan webroot server ke subfolder **`/public`**, bukan root project!

---

## 📖 BAGIAN 3: Bedah Konsep Teori (Kaitan dengan RPS Sub-CPMK-3)

### 1. Apa itu Application Virtualization?
Application Virtualization adalah teknik menyajikan dan menjalankan perangkat lunak dalam lingkungan eksekusi terisolasi tanpa bergantung secara kaku pada instalasi fisik sistem operasi host. 

Pada praktikum ini:
- Kode HTML/JS Anda tidak perlu tahu arsitektur hardware server secara fisik.
- Server Nginx menyajikan file dari *sandbox directory* yang terisolasi.
- Panel Dokploy mengelola service di atas container Docker terisolasi.

### 2. Apa Perbedaan Webhook (Push) vs Polling (Pull)?
| Parameter | Polling (Cron Job) | Webhook (Event-Driven) |
| :--- | :--- | :--- |
| **Mekanisme** | Server bertanya ke GitHub secara berkala: *"Ada update baru?"* | GitHub mengirim notifikasi langsung ke server saat peristiwa terjadi |
| **Efisiensi Sumber Daya** | Boros CPU & bandwidth jika tidak ada perubahan | Sangat efisien, proses hanya aktif saat ada transaksi push |
| **Latensi Waktu** | Ada jeda delay (misal: delay hingga 5 menit sesuai interval cron) | Real-time (instan dalam 1–5 detik setelah commit) |
| **Analogi Sederhana** | Anda menelepon kurir setiap 5 menit menanyakan paket | Kurir membunyikan bel rumah Anda tepat saat paket tiba |

### 3. CI/CD Pipeline
- **Continuous Integration (CI):** Mengintegrasikan perubahan kode ke repository utama secara berkesinambungan dan terverifikasi.
- **Continuous Deployment (CD):** Merilis setiap pembaruan kode ke lingkungan production (server live) secara otomatis tanpa keterlibatan manual operator.

---

## 📝 BAGIAN 4: Panduan Tugas & Analisis Laporan Praktikum

Sesuai ketentuan RPS Mata Kuliah **STI5134 (Sub-CPMK-3)**, setiap mahasiswa wajib menyusun **Laporan Praktikum Mandiri minimal 2 Halaman (Format PDF)** yang memuat:

1. **Bukti Deployment:**
   - Screenshot URL aktif GitHub Pages dengan Nama & NIM Anda terlihat jelas pada tampilan web.
   - Screenshot halaman riwayat commit di GitHub yang membuktikan adanya aktivitas perubahan file `index.html`.
2. **Analisis Teknis Pertanyaan Wajib:**
   - **Pertanyaan 1:** Jelaskan bagaimana GitHub Pages atau Dokploy mengisolasi aplikasi web Anda dari ribuan aplikasi pengguna lain di server yang sama!
   - **Pertanyaan 2:** Jelaskan proses alur sinyal Webhook dari saat tombol *Commit changes* ditekan di GitHub hingga diterima oleh Webhook Receiver di server!
   - **Pertanyaan 3:** Hitung durasi waktu (dalam detik) antara tombol *Commit changes* ditekan hingga halaman web live ter-update. Mengapa terjadi jeda waktu tersebut?
   - **Pertanyaan 4:** Bandingkan efisiensi waktu dan potensi *human-error* antara metode CI/CD AutoDeploy ini vs metode konvensional (upload manual via cPanel File Manager/FTP)!
   - **Pertanyaan 5:** Mengapa sistem di server VPS menggabungkan metode Webhook dan Cron Job sekaligus? Apa fungsi masing-masing dalam skenario kegagalan jaringan?

---

## 📄 Hak Cipta & Lisensi
Materi praktikum ini disusun untuk kepentingan perkuliahan **STI5134 Layanan dan Sistem Virtual**, Program Studi S1 Teknologi Informasi, Fakultas Teknik, Universitas Lambung Mangkurat.  
Dilisensikan di bawah [MIT License](LICENSE).

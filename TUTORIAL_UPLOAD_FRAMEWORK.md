# 📘 Panduan Mahasiswa: Cara Mengunggah & AutoDeploy Proyek Framework (Laravel, React/Vue/Node.js, PHP Native)
### Mata Kuliah: STI5134 — Layanan dan Sistem Virtual
**Program Studi S1 Teknologi Informasi — Fakultas Teknik, Universitas Lambung Mangkurat**  
**Dosen Pengampu:** Ir. Nor Anisa & Andry Fajar Zulkarnain, S.ST., M.T.

---

## 🚨 ATURAN EMAS PENGEMBANGAN FRAMEWORK (WAJIB DIBACA!)

Banyak mahasiswa pemula mengalami kegagalan saat pertama kali meng-upload proyek Laravel atau Node.js ke GitHub (misalnya proses upload macet berjam-jam atau ditolak GitHub). 

Patuhi **3 Aturan Emas** berikut sebelum melakukan `git push`:

| File / Folder | Status di Git | Alasan Teknis |
| :--- | :---: | :--- |
| **`vendor/`** (Laravel/PHP) | ❌ **HARAM DI-PUSH** | Berisi ribuan dependensi eksternal (100–300 MB). Cukup upload `composer.json`, server cloud yang akan mengunduhnya via `composer install`. |
| **`node_modules/`** (JS/React) | ❌ **HARAM DI-PUSH** | Berisi puluhan ribu file kecil (200–500 MB). Cukup upload `package.json`, server akan mengunduhnya via `npm install`. |
| **`.env`** (File Rahasia) | ❌ **HARAM DI-PUSH** | Memuat kata sandi database dan API key sensitif. Cukup upload template publiknya: **`.env.example`**. |
| **`storage/*.key`, `*.log`** | ❌ **HARAM DI-PUSH** | File runtime lokal yang tidak boleh menimpa server production. |

---

## 🐘 BAGIAN 1: Tutorial Deploy Proyek LARAVEL (PHP)

Jika Anda membuat tugas / proyek menggunakan **Laravel** (dari folder lokal misal `C:\xampp\htdocs\nama_proyek`):

### Langkah 1: Pastikan File `.gitignore` Sudah Benar
Buka VS Code di folder proyek Laravel Anda. Pastikan di root folder terdapat file **`.gitignore`** dengan isi minimal:
```text
/node_modules
/public/hot
/public/storage
/storage/*.key
/vendor
.env
.env.backup
.phpunit.result.cache
Homestead.json
Homestead.yaml
npm-debug.log
yarn-error.log
```

> 💡 **PENTING:** Jika folder `vendor/` atau `node_modules/` sempat terlanjur ter-commit sebelumnya, jalankan perintah ini di terminal proyek untuk menghapusnya dari Git tanpa menghapus file di laptop Anda:
> ```bash
> git rm -r --cached vendor node_modules
> git commit -m "chore: remove vendor and node_modules from git tracking"
> ```

---

### Langkah 2: Buat Template `.env.example`
Pastikan file `.env.example` tersedia agar server tahu variabel apa saja yang dibutuhkan:
```ini
APP_NAME=Laravel
APP_ENV=production
APP_KEY=
APP_DEBUG=false
APP_URL=http://localhost

DB_CONNECTION=sqlite
# Atau jika menggunakan MySQL:
# DB_CONNECTION=mysql
# DB_HOST=127.0.0.1
# DB_PORT=3306
# DB_DATABASE=nama_database
# DB_USERNAME=root
# DB_PASSWORD=
```

---

### Langkah 3: Inisialisasi Git dan Push ke GitHub
Buka terminal (PowerShell / Command Prompt / Git Bash) di folder proyek Laravel Anda:
```bash
# 1. Inisialisasi git jika belum
git init
git branch -M main

# 2. Tambahkan seluruh source code yang sudah disaring
git add .
git commit -m "feat: initial commit of my Laravel project"

# 3. Hubungkan ke repositori GitHub Anda
git remote add origin https://github.com/USERNAME-ANDA/NAMA-REPO-ANDA.git

# 4. Push ke GitHub
git push -u origin main
```
*Proses push akan berlangsung sangat cepat (kurang dari 15 detik) karena folder `vendor` dan `node_modules` sudah diabaikan!*

---

### Langkah 4: Cara Menjalankan Laravel di Server / Cloud (PaaS)

#### Opsi A: Menggunakan Dokploy PaaS (Server VPS Kampus)
Jika menggunakan panel container **Dokploy** di server kelas:
1. Buka dashboard Dokploy.
2. Klik **Add Application** ➔ Pilih **GitHub**.
3. Pilih repository Laravel Anda.
4. **Penting pada Build Type:** Pilih **`Nixpacks`**!
   * *Nixpacks secara otomatis mendeteksi bahwa ini adalah Laravel.*
   * *Nixpacks otomatis memasang PHP 8.2/8.3, Composer, menjalankan `composer install`, dan menyetel webroot ke subfolder `/public`!*
5. Di tab **Environment**, masukkan variabel penting:
   * `APP_KEY` (dihasilkan dari `php artisan key:generate --show`)
   * `APP_ENV=production`
   * `APP_DEBUG=false`
6. Klik **Deploy** ➔ Web Laravel Anda langsung live!

#### Opsi B: Menggunakan Render.com / Railway (Cloud Gratis)
1. Login ke [https://render.com](https://render.com) atau [https://railway.app](https://railway.app).
2. Pilih **New Web Service** ➔ Hubungkan repositori GitHub Anda.
3. Tentukan konfigurasi:
   * **Runtime:** PHP
   * **Build Command:** `composer install --no-dev --optimize-autoloader && php artisan config:cache && php artisan route:cache`
   * **Publish Directory / Root:** Subfolder `public/`
4. Tambahkan environment variable `APP_KEY`.

---

## ⚛️ BAGIAN 2: Tutorial Deploy Frontend Modern (React, Vue, Vite, Next.js)

Bagi mahasiswa yang menggunakan JavaScript framework modern:

### Langkah 1: Pastikan `.gitignore` Mengabaikan `node_modules`
File `.gitignore` wajib memuat:
```text
node_modules/
dist/
build/
.env
.env.local
.DS_Store
```

### Langkah 2: Konfigurasi Path Base (Khusus GitHub Pages)
Jika menggunakan **Vite (React / Vue)** dan ingin di-deploy gratis ke **GitHub Pages**:
Buka file `vite.config.js`, tambahkan baris `base`:
```javascript
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  base: './', // Wajib './' agar path aset CSS dan JS tidak 404 di GitHub Pages!
})
```

### Langkah 3: Build & Push
Jalankan di terminal lokal:
```bash
git add .
git commit -m "feat: prepare frontend project for autodeploy"
git push origin main
```

---

## 🌐 BAGIAN 3: Tutorial Deploy PHP Native / CodeIgniter

Bagi mahasiswa yang mengerjakan tugas menggunakan PHP Native atau CodeIgniter:

1. **Struktur Folder:**
   Pastikan file halaman utama bernama **`index.php`** dan diletakkan di root atau folder public.
2. **Koneksi Database Dinamis:**
   Gunakan variabel environment atau konfigurasi dinamis agar tidak error saat berpindah dari laptop ke server:
   ```php
   <?php
   $host = getenv('DB_HOST') ?: '127.0.0.1';
   $user = getenv('DB_USER') ?: 'root';
   $pass = getenv('DB_PASS') ?: '';
   $db   = getenv('DB_NAME') ?: 'nama_db';

   $conn = new mysqli($host, $user, $pass, $db);
   if ($conn->connect_error) {
       die("Koneksi gagal: " . $conn->connect_error);
   }
   ?>
   ```
3. Sertakan file database `.sql` di folder `database/` agar rekan tim atau dosen dapat melakukan *import* struktur tabel dengan mudah.

---

## 🛠️ TROUBLESHOOTING KENDALA UMUM MAHASISWA

| Gejala Error | Penyebab | Solusi |
| :--- | :--- | :--- |
| **Push Macet / Gagal File >100MB** | Folder `node_modules` atau `vendor` ikut ter-commit | Jalankan `git rm -r --cached vendor node_modules`, perbarui `.gitignore`, lalu commit ulang. |
| **Halaman Web 403 Forbidden** | Server membaca root folder, bukan subfolder `public/` | Pada Laravel, pastikan web server Nginx diarahkan ke `root /path/proyek/public;`. |
| **Halaman Web 500 Server Error** | `APP_KEY` belum dibuat atau permission folder storage terkunci | Jalankan `php artisan key:generate` dan berikan izin tulis: `chmod -R 775 storage bootstrap/cache`. |
| **Tampilan CSS/JS Rusak di GitHub Pages** | Path aset menggunakan path absolut (`/assets/`) | Pada Vite/Webpack, ubah base path menjadi relatif (`base: './'`). |

---

## 📄 Hak Cipta
Modul panduan ini disusun untuk Mata Kuliah **STI5134 Layanan dan Sistem Virtual**, Program Studi S1 Teknologi Informasi, Fakultas Teknik, Universitas Lambung Mangkurat.

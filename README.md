# 🚀 AutoDeploy Web — Layanan dan Sistem Virtual (STI5134)

> **Contoh Praktikum Pertemuan 5:** Penerapan *Application Virtualization & Cloud PaaS (Platform as a Service)* menggunakan GitHub dan Render.com.  
> Program Studi S1 Teknologi Informasi — Fakultas Teknik, Universitas Lambung Mangkurat.

---

## 🎯 Tujuan Pembelajaran (Sub-CPMK-3)
1. Memahami konsep **Application Virtualization / Containerization** (menjalankan aplikasi di atas infrastruktur cloud terisolasi tanpa instalasi server fisik).
2. Memahami alur **CI/CD (Continuous Integration & Continuous Deployment)** otomatis: `Git Push` ➔ `Webhook Trigger` ➔ `Cloud Build Container` ➔ `Live HTTPS Domain`.
3. Membuktikan kemudahan deployment modern berbasis Cloud PaaS tanpa biaya server.

---

## 🛠️ Cara Deploy dalam 3 Menit (Anti-Ribet)

### Langkah 1: Fork Repository Ini
1. Klik tombol **Fork** di pojok kanan atas halaman GitHub ini.
2. Beri nama repository (misal: `autodeploy-sti5134`).
3. Klik **Create fork**.

### Langkah 2: Hubungkan ke Render.com (100% Gratis)
1. Buka [https://render.com](https://render.com) di browser.
2. Klik **Sign In** ➔ Pilih **Sign in with GitHub**.
3. Di dashboard Render, klik tombol **New +** (di pojok kanan atas) ➔ Pilih **Static Site**.
4. Cari dan pilih repository yang baru saja Anda fork tadi ➔ Klik **Connect**.
5. Isi konfigurasi sederhana ini:
   - **Name:** Beri nama bebas (misal: `web-anisa-sti5134`)
   - **Branch:** `main`
   - **Build Command:** *(Biarkan kosong)*
   - **Publish Directory:** `.` *(Ketik satu titik saja)*
6. Gulir ke bawah, klik tombol biru **Create Static Site**.
7. Tunggu sekitar 30–60 detik hingga statusnya berubah menjadi **Live**!
8. Salin URL publik HTTPS Anda (contoh: `https://web-anisa-sti5134.onrender.com`).

---

## 🧪 Eksperimen Uji AutoDeploy (Magic Test)

Buktikan bahwa sistem ini bekerja secara otomatis:
1. Buka repository Anda di GitHub.
2. Klik file `index.html` ➔ Klik ikon pensil (✏️) untuk mengedit.
3. Ubah teks **"Ganti Nama Anda"** menjadi nama lengkap Anda, dan isi NIM Anda.
4. Klik tombol hijau **Commit changes...** ➔ **Commit changes**.
5. Buka kembali URL Render Anda dalam 1 menit, lalu refresh halaman browser Anda.
6. **Hasil:** Website otomatis ter-update di internet tanpa perlu Anda deploy ulang secara manual!

---

## 📚 Refleksi Analisis untuk Laporan (Sub-CPMK-3)
Dalam laporan praktikum, jawablah 3 pertanyaan analisis berikut:
1. **Bagaimana Render membedakan aplikasi Anda dari ribuan aplikasi pengguna lain di servernya?**  
   *(Analisis konsep isolasi Application Virtualization / Sandboxing)*.
2. **Apa yang terjadi saat tombol Commit Changes di-klik di GitHub?**  
   *(Jelaskan peran sinyal Webhook yang memicu container builder di Render)*.
3. **Mengapa website Anda langsung otomatis memiliki sertifikat SSL HTTPS tanpa perlu beli sertifikat SSL?**  
   *(Jelaskan peran Reverse Proxy dan Automated Certificate Authority di cloud)*.

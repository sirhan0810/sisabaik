# SisaBaik API (Chapter 7)

Backend REST API untuk platform SisaBaik yang dibangun menggunakan kombinasi Express.js dan sistem basis data relasional PostgreSQL. Proyek ini dikerjakan sebagai salah satu bagian dari materi perkuliahan Pengembangan Web di Program Studi Bisnis Digital, Universitas Negeri Padang.

## Prasyarat
- Node.js versi 22 atau versi yang lebih baru
- PostgreSQL lokal (telah menyiapkan database bernama `sisabaik_dev` beserta skema yang dibuat pada Chapter 6)
- npm

## Cara Menjalankan Proyek
1. Salin, buat, dan sesuaikan konfigurasi pada berkas `.env` di direktori utama berdasarkan contoh `.env.example`:

   ```env
   PORT=3000
   PGHOST=localhost
   PGPORT=5432
   PGDATABASE=sisabaik_dev
   PGUSER=postgres
   PGPASSWORD=GANTI DENGAN PASSWORD LOKAL
   ```

2. Unduh dan pasang seluruh pustaka (*dependencies*) yang dibutuhkan:

   ```bash
   npm install
   ```

3. Jalankan server aplikasi pada mode pengembangan:

   ```bash
   npm run dev
   ```

4. Akses peramban (*browser*) dan buka alamat `http://127.0.0.1:3000` untuk mulai memakai antarmuka pengujian.

## Pengujian API

- **Pengujian Mutasi dan Operasi CRUD (Secara Otomatis):**

   ```bash
   npm run api:test
   ```

- **Pengujian Beban & Konkurensi (Permintaan Bersamaan):**

   ```bash
   node scripts/uji-konkurensi.js
   ```

## Catatan Implementasi & Batasan

- **Identitas serta Kontrak Terbaru:** Proses pembuatan pesanan kini memanfaatkan atribut `emailPembeli` (merujuk pada data entitas `pengguna` yang aktif), sementara ID untuk penawaran maupun pesanan dikembalikan di dalam respons sebagai format string digit.

- **Validasi Data & Manajemen Transaksi:** Setiap proses perubahan data pesanan menerapkan mekanisme transaksi database yang bersifat atomik (`REPEATABLE READ`), penguncian baris eksklusif (`FOR UPDATE`), serta pengecekan aturan bisnis terkait pembatasan satu penyedia per transaksi pemesanan (`MIXED_PROVIDER`).

- **Sistem Keamanan & Autentikasi:** Saat ini, aplikasi masih beroperasi secara lokal menggunakan akun uji coba dan belum mengintegrasikan modul login atau otorisasi penuh secara menyeluruh.
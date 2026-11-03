# SisaBaik API

SisaBaik merupakan sebuah platform digital yang mempertemukan pihak penyedia makanan dengan masyarakat luas untuk meminimalisasi pembuangan bahan pangan dengan cara menjual kembali produk surplus yang layak konsumsi.

## Arsitektur Sistem dan Urutan Migrasi Database

Aplikasi ini menggunakan PostgreSQL sebagai sistem basis data utamanya. Berikut adalah tahapan eksekusi skrip migrasi serta pengisian data awal:

* `database/01_schema.sql` - Penyiapan struktur tabel dasar serta relasi antar-entitas.

* `database/02_seed.sql` - Penyediaan sekumpulan data awal (*sample data*) untuk kebutuhan pengujian.

* `database/03_queries.sql` sampai `06_...` - Berkas berisi sekumpulan kueri pendukung untuk operasi pengolahan data (CRUD) dan alur transaksi.

* `database/07_auth.sql` - Skrip penyesuaian basis data untuk mekanisme autentikasi, pengelolaan token keamanan, serta riwayat sesi pengguna.

## Layanan Registrasi Mandiri dan Hak Akses Pengguna

Platform ini memfasilitasi pendaftaran secara mandiri bagi 2 (dua) tipe peranan pengguna:

1. **Konsumen**: Pengguna biasa yang memiliki hak akses untuk menjelajahi katalog produk publik, membuat pesanan baru, dan melakukan transaksi pembelian.

2. **Penyedia**: Pelaku usaha kuliner yang mendaftarkan identitas usahanya untuk memasarkan makanan sisa produksi atau surplus.
   * **Proses Verifikasi Penyedia**: Setiap pendaftaran akun penyedia baru wajib melalui tahap pemeriksaan atau persetujuan terlebih dahulu sebelum diperbolehkan untuk menayangkan daftar penawaran di halaman katalog umum.

## Keamanan Sistem, Validasi Autentikasi, dan Pengakhiran Sesi (Logout)

* **Pengujian Fitur Autentikasi (`auth:test`)**: Jalankan instruksi berikut guna mengecek keandalan sistem autentikasi dan alur alokasi token:

  ```bash
  npm run auth:test
  ```

* **Dampak Pencabutan Seluruh Sesi (Global Logout)**: Ketika pengguna memilih untuk keluar dari seluruh sesi yang terhubung, sistem secara otomatis akan menghapus seluruh entitas token yang sedang aktif beserta data sesinya sesuai dengan struktur migrasi `07_auth.sql`. Hal ini membuat seluruh token JWT yang telah terbit sebelumnya seketika tidak dapat dipergunakan kembali di perangkat manapun.

## Pengaturan Variabel Lingkungan (Environment)

Daftar parameter lingkungan berikut wajib disiapkan di dalam berkas `.env` (tanpa perlu menyertakan nilai kredensial atau kunci rahasia):

* `PORT`

* `DATABASE_URL`

* `JWT_SECRET`

* `NODE_ENV`

## Langkah Menjalankan Server dan Halaman Web

1. Pasang seluruh modul pustaka yang dibutuhkan:

   ```bash
   npm install
   ```

2. Operasikan server dalam lingkungan pengembangan:

   ```bash
   npm run dev
   ```

3. Buka peramban (*browser*) untuk mengakses antarmuka statis pada lokasi `public/index.html` atau halaman alur autentikasi pada `public/auth.html`.
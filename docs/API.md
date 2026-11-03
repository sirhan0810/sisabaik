# Dokumentasi API SisaBaik

Dokumen ini memuat rincian mengenai *endpoint*, spesifikasi muatan data (*payload*), mekanisme keamanan, hingga regulasi otorisasi data pada API SisaBaik.

## Jalur Pendaftaran (Registrasi)

Sistem membedakan mekanisme pembuatan akun menjadi dua alur independen sesuai kebutuhan data profil pengguna:

1. **Pendaftaran Akun Konsumen**

   * `POST /api/auth/register/consumer`
   * Membuat identitas akun baru untuk pembeli atau pelanggan umum.

2. **Pendaftaran Akun Penyedia**

   * `POST /api/auth/register/provider`
   * Memproses pendaftaran mitra penyedia produk makanan dengan melampirkan rincian profil unit usaha.

## Parameter Profil Usaha dan Struktur Body Bisnis

Komposisi *payload* pada pendaftaran mitra penyedia maupun saat pembaharuan profil usaha mencakup entitas data berikut:

* `business_name` (string, wajib): Identitas resmi atau nama komersial entitas usaha.

* `business_category` (string, wajib): Bidang/kategori usaha kuliner (misalnya: Toko Roti/Bakery, Rumah Makan/Restoran, Penyedia Katering).

* `address` (string, wajib): Alamat lengkap lokasi operasional toko/usaha.

* `operational_hours` (string/object): Rentang waktu/jam layanan penyerahan makanan surplus.

## Kode Status Kesalahan (Error HTTP)

Umpan balik atas kegagalan proses mengacu pada konvensi kode status HTTP standar:

* `400 Bad Request`: Format isi data (*payload*) tidak valid atau gagal dalam verifikasi input.

* `401 Unauthorized`: Kredensial token tidak sah, telah kadaluwarsa, atau tidak dikirimkan dalam *header*.

* `403 Forbidden`: Otorisasi hak akses tidak mencukupi (contohnya: akun penyedia masih dalam tahap peninjauan status).

* `404 Not Found`: Alamat *endpoint* atau entitas data yang dimaksud tidak ditemukan.

* `429 Too Many Requests`: Frekuensi permintaan melampaui batas ambang yang diizinkan (*rate limit*).

* `500 Internal Server Error`: Terjadi hambatan tak terduga pada sistem server atau sistem basis data.

## Pembatasan Frekuensi Akses (Rate Limiting) Bertingkat

Sistem mengimplementasikan kontrol pembatasan laju permintaan sesuai tingkat sensitivitas *endpoint*:

* **Endpoint Publik / Umum**: Menerapkan ambang batas standar untuk keperluan eksplorasi katalog produk.

* **Endpoint Autentikasi & Transaksi**: Menerapkan batas ambang yang jauh lebih ketat guna menanggulangi potensi serangan *brute-force* serta penyalahgunaan pemrosesan transaksi.

## Otorisasi Lingkup Pesanan (Order Scopes)

Aksesibilitas pengelolaan transaksi disesuaikan menurut hirarki peran serta kepemilikan data:

* Konsumen hanya diizinkan untuk melihat, menargetkan transaksi baru, dan membatalkan pesanan yang terdaftar atas nama mereka sendiri.

* Penyedia memiliki akses terbatas hanya pada pengelolaan pesanan yang masuk ke inventaris toko atau unit bisnis mereka.

## Ketentuan Katalog Publik

* Halaman katalog umum (`GET /api/catalog` atau rute setara) secara eksklusif hanya menyajikan daftar produk dari **penyedia yang telah mengantongi status verifikasi (verified/reviewed)**.

* Daftar penawaran dari penyedia yang masih memegang status peninjauan (*pending review*) secara otomatis disembunyikan dari katalog umum.
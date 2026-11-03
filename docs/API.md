# Dokumentasi Kontrak REST API - SisaBaik Chapter 7

Rangkuman ini menyajikan penjelasan mengenai *endpoint*, parameter *query*, kolom (*field*) wajib, logika bisnis, serta kualifikasi kode status respons yang diterapkan pada layanan REST API SisaBaik.

## Alamat Utama (Base URL)
`http://127.0.0.1:3000/api`

---

## Daftar Endpoint yang Tersedia

### 1. Cek Ketersediaan Layanan (Health Check)
- **GET `/health`**

  * **Deskripsi:** Memeriksa status keaktifan proses server serta memastikan koneksi ke basis data PostgreSQL berjalan normal.
  * **Parameter Query:** Tidak membutuhkan parameter.
  * **Contoh Respons Berhasil (`200 OK`):**
    ```json
    {
      "success": true,
      "data": {
        "status": "ok",
        "database": "connected",
        "waktu": "2026-10-12T00:00:00.000Z"
      }
    }
    ```

---

### 2. Pengelolaan Penawaran (Offers)
- **GET `/penawaran`**

  * **Deskripsi:** Menampilkan seluruh koleksi penawaran yang tersedia, lengkap dengan opsi penyaringan dan pencarian data.
  * **Parameter Query yang Dapat Digunakan:**
    * `q` (string): Kata kunci untuk pencarian berdasarkan judul atau deskripsi produk.
    * `kategori` (string): Penyaringan berdasarkan kategori barang.
    * `status` (string): Penyaringan menurut status kondisi penawaran.
    * `tersedia` (boolean/string): Menyaring item yang jumlah stoknya masih mencukupi.
    * `page` (integer): Urutan halaman data (paginasi).
    * `limit` (integer): Batas maksimum item per halaman.

- **GET `/penawaran/:id`**

  * **Deskripsi:** Mengambil informasi detail dari satu entitas penawaran tertentu berdasarkan nilai ID.
  * **Parameter URL:** `:id` (Pengenal unik penawaran).

- **POST `/penawaran`**

  * **Deskripsi:** Menambahkan item penawaran baru ke dalam sistem (wajib menyertakan header `Content-Type: application/json`).
  * **Atribut Wajib (Body Request):**
    * `penyediaId` (string/integer): ID identitas pihak penyedia/mitra pemilik produk.
    * `judul` (string): Nama atau deskripsi singkat penawaran produk.
    * `harga` (number): Harga per unit produk.
    * `stok` (integer): Ketersediaan jumlah unit awal.

- **PUT `/penawaran/:id`**

  * **Deskripsi:** Memperbarui data pada penawaran yang diizinkan untuk diubah (tidak berlaku untuk ID dan keterkaitan penyedia).
  * **Atribut Wajib / Dapat Diubah (Body Request):**
    * `judul` (string)
    * `harga` (number)
    * `stok` (integer)
    * `status` (string)

- **DELETE `/penawaran/:id`**

  * **Deskripsi:** Menghapus data penawaran. Proses penghapusan akan dibatalkan oleh sistem apabila penawaran bersangkutan telah memiliki catatan transaksi/pesanan sebelumnya.
  * **Respons Berhasil:** `204 No Content` (tanpa memuat *body*).
  * **Respons Ditolak:** `409 Conflict` (apabila data penawaran memiliki riwayat transaksi).

---

### 3. Pemrosesan Pesanan (Orders) & Ketentuan Bisnis
- **GET `/pesanan`**

  * **Deskripsi:** Menampilkan ringkasan daftar transaksi pesanan secara bertahap (paginasi).
  * **Parameter Query yang Diterima:** `page`, `limit`.

- **GET `/pesanan/:id`**

  * **Deskripsi:** Menampilkan rincian menyeluruh dari suatu transaksi pesanan (mencakup data umum transaksi beserta daftar item di dalamnya).
  * **Parameter URL:** `:id` (Pengenal unik transaksi pesanan).

- **POST `/pesanan`**

  * **Deskripsi:** Membuat entitas pesanan baru secara atomik melalui mekanisme transaksi database, lengkap dengan peninjauan kuantitas stok dan pembatasan asal penyedia.
  * **Atribut Wajib (Body Request):**
    * `emailPembeli` (string): Alamat surat elektronik akun pembeli yang sudah terdaftar dan aktif dalam database.
    * `items` (array of objects): Rincian produk yang dipesan, di mana setiap objek wajib mencantumkan:
      * `penawaranId` (string/integer)
      * `kuantitas` (integer)
  * **Ketentuan Bisnis Utama (Aturan Tunggal Penyedia / Single Provider Rule):**
    * Layanan memberlakukan verifikasi ketat sehingga seluruh item di dalam satu kali proses pemesanan **harus berasal dari mitra penyedia (*merchant*) yang sama**. Jika ditemukan kombinasi produk dari penyedia berbeda, pemrosesan akan langsung digagalkan dengan tanggapan error `MIXED_PROVIDER` (`409 Conflict`).
  * **Contoh Format Body Request:**
    ```json
    {
      "emailPembeli": "siti.pembeli@example.test",
      "items": [
        {
          "penawaranId": "7",
          "kuantitas": 1
        }
      ]
    }
    ```

---

## Format Standar Respons & Klasifikasi Kode Status Error

- **Struktur Respons Sukses:**

  ```json
  {
    "success": true,
    "data": { ... }
  }
  ```

- **Struktur Respons Gagal:**

  ```json
  {
    "success": false,
    "error": {
      "code": "KODE_ERROR",
      "message": "Pesan deskripsi kesalahan..."
    }
  }
  ```

- **Katalog Kode Status HTTP yang Diterapkan:**

    * `200 OK`: Permintaan membaca atau memperbarui data berhasil diproses.
    * `201 Created`: Pembuatan entitas data baru berhasil (disertai keluaran data yang baru dibuat).
    * `204 No Content`: Operasi penghapusan data berhasil dilakukan tanpa mengembalikan isi respons (*body*).
    * `400 Bad Request`: Pemeriksaan entri gagal, terdapat parameter wajib yang kosong, atau struktur JSON tidak valid.
    * `404 Not Found`: Lokasi endpoint atau atribut ID data yang dicari tidak ditemukan dalam basis data.
    * `409 Conflict`: Terjadi pelanggaran aturan bisnis (seperti stok habis saat transaksi, pembatalan penghapusan penawaran yang berriwayat, atau ketidaksesuaian aturan penyedia tunggal / *mixed provider*).
    * `413 Payload Too Large`: Ukuran muatan (*payload*) data permintaan melebihi batas batas yang diizinkan server.
    * `415 Unsupported Media Type`: Format header `Content-Type` tidak sesuai (bukan `application/json` pada pengiriman metode mutasi).
    * `500 Internal Server Error`: Terjadi kendala internal pada server atau terputusnya koneksi ke sistem basis data.
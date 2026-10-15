Developed by Shofi Salsabila (23367077)

# SisaBaik API

SisaBaik API adalah layanan back-end berbasis Node.js dan Express yang diintegrasikan dengan basis data relasional PostgreSQL. Aplikasi ini dirancang untuk mendukung sistem SisaBaik dalam mengelola data penawaran surplus makanan, pencarian, filter, paginasi, serta pemesanan atomik melalui transaksi basis data.

## Capaian Chapter 7
- Membangun REST API menggunakan Node.js dan Express dengan pemisahan lapisan (Router, Controller, Repository, Service, dan Transaksi).
- Mengintegrasikan skema basis data relasional PostgreSQL (tabel `pengguna`, `penyedia`, `penawaran`, `pesanan`, `detail_pesanan`, dan `pengambilan`).
- Menerapkan kueri terparameterisasi untuk operasi CRUD, pencarian teks, dan filter data.
- Menerapkan paginasi dengan metadata lengkap (`jumlah`, `page`, `limit`, `totalPages`).
- Mengimplementasikan transaksi atomik pada pembuatan pesanan (`BEGIN`, `SELECT ... FOR UPDATE`, snapshot harga, validasi stok, dan `ROLLBACK` jika gagal).
- Menyediakan penanganan kesalahan terpusat (*error handler*) yang memetakan kode error database dan validasi menjadi respons JSON yang konsisten.

## Endpoint Utama
- **GET /api/health**: Memeriksa kesehatan server dan koneksi basis data.
- **GET /api/penawaran**: Mendapatkan daftar penawaran dengan dukungan query `q`, `kategori`, `status`, `tersedia`, `page`, dan `limit`.
- **GET /api/penawaran/:id**: Mendapatkan detail satu penawaran berdasarkan ID.
- **POST /api/penawaran**: Membuat penawaran baru (memerlukan data JSON tervalidasi).
- **PUT /api/penawaran/:id**: Memperbarui atribut penawaran yang dapat diubah.
- **DELETE /api/penawaran/:id**: Menghapus penawaran yang belum memiliki riwayat pesanan.
- **GET /api/pesanan**: Mendapatkan daftar ringkasan pesanan terpaginasi.
- **GET /api/pesanan/:id**: Mendapatkan detail pesanan beserta item dan snapshot harganya.
- **POST /api/pesanan**: Membuat pesanan baru secara atomik dengan penguncian stok dan snapshot harga.

## Menjalankan Proyek
1. Pastikan PostgreSQL aktif dan basis data `sisabaik_dev` sudah tersedia dari Chapter 6.
2. Salin `.env.example` menjadi `.env`, lalu sesuaikan konfigurasi basis data (`PGPASSWORD`, dll.).
3. Jalankan perintah instalasi dependensi:
   ```bash
   npm install
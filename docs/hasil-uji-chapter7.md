# Laporan Hasil Uji - Chapter 7

Nama: Shofi Salsabila
NIM: 23367077
Database: sisabaik_dev (PostgreSQL)

## Ringkasan Hasil Pengujian

- **Health Check dan GET Koleksi (200):** Server Express dan database PostgreSQL berhasil terhubung dengan baik. Data berhasil dikembalikan menggunakan format envelope standar beserta informasi paginasi yang akurat.
- **Validasi Input dan Error Handling (400, 415, 413):** Berbagai pengujian negatif seperti ID tidak valid, format JSON yang rusak, kesalahan header `Content-Type`, ukuran payload di atas 20 KB, hingga field asing berhasil ditolak dengan kode status HTTP yang sesuai tanpa mengganggu database.
- **CRUD Penawaran:**
  - `POST` berhasil membuat data penawaran baru (201) lengkap dengan kembalian header `Location`.
  - `PUT` berhasil memperbarui data secara utuh dengan tetap menjaga aturan agar `penyedia_id` dan waktu pembuatan bersifat tetap (*immutable*).
  - `DELETE` berhasil menghapus penawaran yang belum memiliki riwayat (204) dan secara otomatis menolak penghapusan penawaran yang sudah terikat dengan riwayat pesanan (409) demi menjaga integritas data.
- **Pencarian, Filter, dan Paginasi:** 
  - Kombinasi filter (`q`, `kategori`, `status`, `tersedia`) dan paginasi berjalan lancar. 
  - Query pencarian teks sudah aman dari celah SQL Injection berkat penggunaan *parameterized query*.
  - Akses halaman di luar batas yang wajar tetap mengembalikan respons sukses dengan data kosong tanpa merusak metadata total data.
- **Transaksi Pemesanan Atomik:**
  - Pemesanan valid berhasil dibuat (201) dengan menerapkan penguncian baris (`FOR UPDATE`), pengurangan stok otomatis, serta penyimpanan snapshot harga yang akurat.
  - Skenario error seperti pesanan lintas penyedia atau stok yang tidak mencukupi berhasil dideteksi dan dibatalkan (409) dengan mekanisme *rollback* sehingga tidak meninggalkan data parsial di database.
- **Konkurensi Stok dan Persistensi:**
  - Uji coba dua *request* secara bersamaan pada penawaran dengan sisa stok 1 menghasilkan tepat satu transaksi sukses (201) dan satu penolakan (409) dengan sisa stok akhir menjadi 0.
  - Data pesanan dan snapshot harga terbukti aman serta persisten meskipun server dilakukan *restart* secara manual.

## Kesimpulan Pengujian

Seluruh rangkaian pengujian dasar melalui `npm run api:test` dan pengujian konkurensi stok lewat `node scripts/uji-konkurensi.js` berhasil dijalankan dengan hasil tuntas tanpa kendala. Mekanisme transaksi, penguncian stok, dan pencatatan snapshot harga pada SisaBaik API telah berjalan sesuai dengan spesifikasi yang diharapkan.
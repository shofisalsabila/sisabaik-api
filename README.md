Developed by Shofi Salsabila (23367077)

# SisaBaik API

SisaBaik API adalah layanan back-end berbasis Node.js dan Express yang diintegrasikan dengan basis data relasional PostgreSQL. Aplikasi ini dirancang untuk mendukung sistem SisaBaik dalam mengelola autentikasi pengguna, otorisasi berbasis peran (RBAC), kepemilikan data, penawaran surplus makanan, serta pemesanan atomik melalui transaksi basis data.

## Capaian Chapter 8
- **Migrasi Autentikasi:** Menambahkan kolom `kata_sandi_hash` dan `versi_token` pada tabel `pengguna` melalui migrasi `database/07_auth.sql` tanpa menghapus data lama.
- **Pendaftaran & Login Mandiri:** Menerapkan pendaftaran mandiri untuk peran pembeli dan penyedia dengan enkripsi kata sandi menggunakan `bcryptjs` (faktor kerja 12) serta penerbitan token JWT berstandar HS256.
- **Peninjauan Penyedia:** Akun penyedia baru memerlukan peninjauan operasional oleh pengelola melalui script CLI (`npm run penyedia:verifikasi`) sebelum mendapatkan izin penuh untuk mengelola penawaran.
- **Otorisasi & Kepemilikan Data:** Melindungi route menggunakan middleware verifikasi token, pemeriksaan peran (`wajibPeran`), verifikasi status penyedia terverifikasi, serta pembatasan akses data berdasarkan kepemilikan (*ownership scope*) pada SQL.
- **Pencabutan Sesi (Logout):** Menambahkan mekanisme pencabutan token lama di seluruh sesi melalui peningkatan nilai `versi_token` di basis data.
- **Keamanan & Pembatas Laju (*Rate Limit*):** Mengonfigurasi pembatas laju terpisah untuk endpoint login (`20 req / 15 menit`) dan pendaftaran (`50 req / 15 menit`) menggunakan `express-rate-limit`, serta pengamanan header HTTP menggunakan `helmet`.

## Endpoint Utama Chapter 8
- **Auth & Akun:**
  - `POST /api/auth/registrasi`: Pendaftaran mandiri akun pembeli.
  - `POST /api/auth/registrasi-penyedia`: Pendaftaran mandiri akun penyedia dan profil usaha berstatus menunggu.
  - `POST /api/auth/login`: Autentikasi kredensial dan penerbitan JWT `accessToken`.
  - `GET /api/auth/saya`: Mendapatkan informasi identitas aman dari basis data (memerlukan token).
  - `POST /api/auth/logout`: Mencabut seluruh sesi token aktif akun (memerlukan token).
- **Penawaran (Dengan Pembatasan Peran & Kepemilikan):**
  - `GET /api/penawaran`: Katalog publik (hanya menampilkan penawaran aktif, stok positif, belum kedaluwarsa, dan dari penyedia terverifikasi).
  - `GET /api/penawaran/milik-saya`: Menampilkan daftar dan metadata penawaran khusus milik penyedia yang sedang login.
  - `POST /api/penawaran`: Membuat penawaran baru (khusus penyedia terverifikasi).
  - `PUT /api/penawaran/:id`: Memperbarui penawaran (khusus penyedia pemilik).
  - `DELETE /api/penawaran/:id`: Menghapus penawaran milik sendiri.
- **Pesanan (Dengan Cakupan Pembeli & Penyedia):**
  - `GET /api/pesanan`: Mendapatkan daftar pesanan terpaginasi sesuai cakupan akun (pembeli pemilik atau penyedia terkait).
  - `GET /api/pesanan/:id`: Mendapatkan detail pesanan beserta item dan snapshot harga.
  - `POST /api/pesanan`: Membuat pesanan baru secara atomik (khusus pembeli).
- **Kesehatan Sistem:**
  - `GET /api/health`: Memeriksa kesehatan server dan koneksi basis data.

## Cara Menjalankan Proyek
1. Pastikan PostgreSQL lokal aktif dan basis data `sisabaik_dev` beserta skema sebelumnya sudah tersedia.
2. Terapkan migrasi basis data Chapter 8 menggunakan Query Tool pgAdmin atau psql dengan menjalankan file `database/07_auth.sql`.
3. Salin `.env.example` menjadi `.env`, lalu lengkapi konfigurasi basis data (`PGPASSWORD`) serta buat kunci rahasia token dengan mengisi `JWT_SECRET` (64 digit hex acak):
   ```bash
   node -e "console.log(require('node:crypto').randomBytes(32).toString('hex'))"
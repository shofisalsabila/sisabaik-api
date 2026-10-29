# Laporan Hasil Uji - Chapter 8 (Autentikasi & Otorisasi)

- **Tanggal Pengujian:** 2026-10-29
- **Nama Database:** `sisabaik_dev`
- **Commit Terkait:** `feat: add authentication and ownership checks` & `test: verify role and ownership access`
- **Fixture / Akun Uji yang Digunakan:** Akun manual (Rina Baru, Budi Baru, Siti Baru, Andi Baru) & Script Uji Otomatis (`npm run auth:test`).

## Ringkasan Hasil Pengujian & Bukti Aktual
1. **GET Pesanan Tanpa Token:** Mengembalikan status `401 AUTH REQUIRED`. Tidak ada data privat yang bocor.
2. **Token Rusak / Invalid:** Mengembalikan status `401 TOKEN_INVALID`. Tidak ada mutasi data.
3. **Uji Peran (Role Forbidden):** Pembeli mencoba `POST /api/penawaran` atau penyedia mencoba `POST /api/pesanan` menghasilkan status `403 ROLE_FORBIDDEN` (Bukti penolakan peran berhasil).
4. **Uji Kepemilikan (Ownership):** 
   - Mengakses/memodifikasi penawaran Rina menggunakan token Budi menghasilkan status `404` (Data Rina tetap utuh).
   - Mengakses pesanan Siti menggunakan token Andi menghasilkan status `404` (Data pesanan tidak dibuka).
5. **Pencabutan Sesi (Logout):** Memanggil endpoint `/api/auth/saya` menggunakan token lama setelah melakukan *logout* menghasilkan status `401 SESSION_REVOKED` (Bukti perubahan versi token / pencabutan sesi di semua tab/sesi lama).
6. **Batas Laju (Rate Limit):** Percobaan login berlebih menghasilkan status `429 AUTH_RATE_LIMIT` dengan header `Retry-After`.

## Status Pengujian Otomatis
- **Script:** `npm run auth:test`
- **Hasil:** PASS (44 pemeriksaan HTTP dan *assertion* kepemilikan/snapshot berhasil dilewati).
- **Catatan Keamanan:** Tidak ada token JWT, *hash* password, maupun password mentah yang dicetak ke dalam laporan atau log pengujian.
# Dokumentasi Kontrak API - SisaBaik Chapter 8

Dokumentasi ini mencakup spesifikasi endpoint, metode HTTP, aturan otorisasi, format payload, pembatas laju (rate limit), serta penanganan error untuk layanan SisaBaik API Chapter 8.

---

## 1. Autentikasi dan Akun (/api/auth)

* **POST /api/auth/registrasi**
  * **Akses:** Publik (Dibatasi oleh pembatas laju registrasi)
  * **Body Request:** namaLengkap, email, kataSandi
  * **Respons Sukses (201 Created):** Mengembalikan data akun pembeli baru tanpa menyertakan password, hash, atau token

* **POST /api/auth/registrasi-penyedia**
  * **Akses:** Publik (Dibatasi oleh pembatas laju registrasi)
  * **Body Request:** namaLengkap, email, kataSandi, namaUsaha, alamat, nomorTelepon
  * **Respons Sukses (201 Created):** Membuat akun dan profil usaha berstatus menunggu dalam satu transaksi

* **POST /api/auth/login**
  * **Akses:** Publik (Dibatasi oleh pembatas laju login)
  * **Body Request:** email, kataSandi
  * **Respons Sukses (200 OK):** Mengembalikan objek accessToken, tokenType (Bearer), expiresIn, serta data profil pengguna

* **GET /api/auth/saya**
  * **Akses:** Terlindungi (Memerlukan token yang valid pada header Authorization: Bearer <token>)
  * **Respons Sukses (200 OK):** Mengembalikan identitas aman pengguna yang sedang login dari basis data

* **POST /api/auth/logout**
  * **Akses:** Terlindungi (Memerlukan token valid, dengan body {})
  * **Respons Sukses (204 No Content):** Mencabut seluruh sesi token lama milik akun dengan menaikkan versi token di basis data

---

## 2. Penawaran (/api/penawaran)

* **GET /api/penawaran**
  * **Akses:** Publik
  * **Keterangan:** Katalog publik yang hanya menampilkan penawaran dengan status aktif, stok positif (> 0), belum kedaluwarsa, dari penyedia yang aktif, serta status verifikasi terverifikasi. Mendukung query parameter q, kategori, page, dan limit

* **GET /api/penawaran/:id**
  * **Akses:** Publik
  * **Keterangan:** Menampilkan detail penawaran jika tersedia; mengembalikan 404 jika tidak terlihat dalam katalog publik

* **GET /api/penawaran/milik-saya**
  * **Akses:** Penyedia terverifikasi
  * **Keterangan:** Menampilkan daftar dan metadata seluruh status penawaran yang hanya dimiliki oleh penyedia yang sedang login

* **POST /api/penawaran**
  * **Akses:** Penyedia terverifikasi
  * **Keterangan:** Membuat penawaran baru. Field penyediaId ditentukan otomatis oleh server berdasarkan identitas token akun yang login

* **PUT /api/penawaran/:id**
  * **Akses:** Penyedia terverifikasi dan pemilik sah penawaran
  * **Keterangan:** Memperbarui penawaran milik sendiri. Seluruh field bisnis yang dapat diedit wajib dikirimkan

* **DELETE /api/penawaran/:id**
  * **Akses:** Penyedia terverifikasi dan pemilik sah penawaran
  * **Keterangan:** Menghapus penawaran. Konflik penghapusan data bersejarah akan ditolak dengan status 409

---

## 3. Pesanan (/api/pesanan)

* **GET /api/pesanan**
  * **Akses:** Pembeli atau penyedia terverifikasi
  * **Keterangan:** Menampilkan daftar pesanan terpaginasi. Pembeli hanya melihat pesanan miliknya sendiri, sedangkan penyedia hanya melihat pesanan yang ditujukan kepada profil usahanya

* **GET /api/pesanan/:id**
  * **Akses:** Pembeli pemilik atau penyedia terkait yang terverifikasi
  * **Keterangan:** Menampilkan detail pesanan beserta item dan snapshot harga

* **POST /api/pesanan**
  * **Akses:** Pembeli
  * **Keterangan:** Membuat pesanan baru secara atomik berdasarkan items (berisi penawaranId dan kuantitas). Field pembeliId ditentukan otomatis oleh server dari token autentikasi

---

## 4. Kesehatan Sistem (/api/health)
* **GET /api/health**
  * **Akses:** Publik
  * **Respons (200 OK):** Memastikan server berjalan dan koneksi basis data dapat diakses

---

## 5. Batas Laju (Rate Limit)
* **Login (/api/auth/login):** Dibatasi maksimal 20 request gagal per 15 menit per alamat IP. Mengembalikan kode error AUTH_RATE_LIMIT disertai header Retry-After jika terlampaui
* **Registrasi (/api/auth/registrasi & /api/auth/registrasi-penyedia):** Berbagi batas bersama maksimal 50 request (berhasil maupun gagal) per 15 menit. Mengembalikan kode error REGISTRATION_RATE_LIMIT disertai header Retry-After

---

## 6. Standar Status Kode Error
* **400**: Kontrak body, field, atau parameter tidak sesuai
* **401**: Login gagal, token tidak ada, rusak, kedaluwarsa (TOKEN_EXPIRED), atau dicabut (SESSION_REVOKED)
* **403**: Identitas valid tetapi peran atau profil usaha tidak diizinkan (ROLE_FORBIDDEN, PROVIDER_NOT_VERIFIED)
* **404**: Resource tidak ditemukan atau berada di luar cakupan akses pengguna (OFFER_NOT_FOUND, ORDER_NOT_FOUND)
* **409**: Konflik data, seperti konflik pendaftaran, kekurangan stok (INSUFFICIENT_STOCK), campuran penyedia (MIXED_PROVIDER), atau penghapusan data bersejarah
* **413 / 415**: Payload terlalu besar atau tipe konten tidak didukung (Content-Type selain application/json pada mutasi)
* **429**: Batas laju tercapai (AUTH_RATE_LIMIT / REGISTRATION_RATE_LIMIT)
* **500 / 503**: Kesalahan internal server atau basis data
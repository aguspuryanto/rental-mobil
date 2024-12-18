Membangun Aplikasi Rental Mobil
Tech: Laravel 11, Bootstrap 5

Fitur:
REST API untuk mendukung integrasi lintas platform.
Laravel Breeze untuk autentikasi pengguna yang cepat dan efisien.
Spatie Role Permission untuk pengelolaan peran dan izin secara fleksibel.
Sanctum untuk melindungi API dengan manajemen token yang andal.
Midtrans Payment Gateway untuk memproses pembayaran online secara aman dan mudah.

Produk dan Layanan
Sewa Mobil (Guest)
Sewakan Mobil (Host)

TREVO adalah aplikasi marketplace sewa mobil P2P lepas kunci atau dengan supir yang mempertemukan penyewa (guest) dengan pemilik (host). Host TREVO tersebar di beberapa kota untuk Sewa Mobil Jakarta, Bekasi, Depok, Bogor, Tangerang, Bali, Jogja, Surabaya, Medan, serta belasan lokasi lainnya.

https://stories.trevo.id/untung-banyak-program-referral/?_ga=2.252778945.915013323.1734498320-207576451.1734498319

gambaran modul dan fungsionalitas yang perlu dipertimbangkan:

Modul Utama
Autentikasi Pengguna

Registrasi dan login menggunakan Laravel Breeze.
Pengelolaan role pengguna: Guest dan Host (dengan Spatie Role Permission).
Reset dan perubahan kata sandi.
REST API

Endpoint untuk data mobil, pencarian, booking, dan pembayaran.
Proteksi endpoint menggunakan Sanctum.
Manajemen Mobil

Guest: Lihat daftar mobil yang tersedia untuk disewa.
Host: Tambah, edit, atau hapus mobil yang disewakan (dengan gambar, deskripsi, dan tarif).
Booking dan Pembayaran

Sistem pencarian mobil berdasarkan lokasi, tanggal, dan harga.
Integrasi Midtrans untuk proses pembayaran.
Sistem notifikasi setelah pembayaran berhasil.
Dashboard

Guest: Riwayat penyewaan, detail booking, dan status pembayaran.
Host: Statistik mobil yang disewakan, pendapatan, dan ulasan penyewa.
Ulasan dan Rating

Guest dapat memberikan ulasan dan rating untuk Host dan mobil.
Admin Panel

Manajemen data pengguna, mobil, dan transaksi.
Laporan pendapatan platform dan aktivitas pengguna.

-- Struktur Database untuk Aplikasi Rental Mobil

-- Tabel untuk pengguna (Guest dan Host)
CREATE TABLE users (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL,
    role ENUM('guest', 'host', 'admin') DEFAULT 'guest',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- Tabel untuk mobil yang disewakan oleh Host
CREATE TABLE cars (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT UNSIGNED NOT NULL,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    price_per_day DECIMAL(10, 2) NOT NULL,
    location VARCHAR(255),
    status ENUM('available', 'booked', 'unavailable') DEFAULT 'available',
    image_url VARCHAR(255),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

-- Tabel untuk booking mobil oleh Guest
CREATE TABLE bookings (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT UNSIGNED NOT NULL,
    car_id BIGINT UNSIGNED NOT NULL,
    start_date DATE NOT NULL,
    end_date DATE NOT NULL,
    total_price DECIMAL(10, 2) NOT NULL,
    payment_status ENUM('pending', 'paid', 'failed') DEFAULT 'pending',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (car_id) REFERENCES cars(id) ON DELETE CASCADE
);

-- Tabel untuk pembayaran melalui Midtrans
CREATE TABLE payments (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    booking_id BIGINT UNSIGNED NOT NULL,
    transaction_id VARCHAR(255) UNIQUE NOT NULL,
    payment_method VARCHAR(50),
    amount DECIMAL(10, 2) NOT NULL,
    status ENUM('pending', 'success', 'failed') DEFAULT 'pending',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (booking_id) REFERENCES bookings(id) ON DELETE CASCADE
);

-- Tabel untuk ulasan dan rating
CREATE TABLE reviews (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    booking_id BIGINT UNSIGNED NOT NULL,
    rating TINYINT UNSIGNED CHECK(rating BETWEEN 1 AND 5),
    comment TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (booking_id) REFERENCES bookings(id) ON DELETE CASCADE
);

REST API
Autentikasi dan Pengguna
POST /api/register
Registrasi pengguna baru (Guest/Host).
Body: name, email, password, role
Response: Token autentikasi.

POST /api/login
Login pengguna.
Body: email, password
Response: Token autentikasi.

GET /api/profile
Detail profil pengguna yang sedang login.
Header: Token Bearer.
Response: user_id, name, email, role.

Manajemen Mobil (Host)
POST /api/cars
Tambahkan mobil untuk disewakan.
Body: name, description, price_per_day, location, image_url.
Response: Detail mobil yang ditambahkan.

GET /api/cars
Daftar semua mobil (filter berdasarkan lokasi, harga, status).
Query Params: location, price_range.
Response: Array data mobil.

GET /api/cars/{id}
Detail mobil berdasarkan ID.
Response: Data lengkap mobil.

PUT /api/cars/{id}
Update informasi mobil.
Body: Field yang ingin diubah.
Response: Detail mobil yang diperbarui.

DELETE /api/cars/{id}
Hapus mobil yang disewakan.
Response: Status berhasil atau gagal.

Booking dan Pembayaran
POST /api/bookings
Booking mobil.
Body: car_id, start_date, end_date.
Response: Detail booking dan total harga.

GET /api/bookings
Daftar booking pengguna (Guest).
Response: Array data booking.

GET /api/bookings/{id}
Detail booking tertentu.
Response: Data lengkap booking.

POST /api/payments
Proses pembayaran booking via Midtrans.
Body: booking_id, payment_method.
Response: URL pembayaran Midtrans.

GET /api/payments/{id}
Status pembayaran.
Response: Status pembayaran (pending/success/failed).

Ulasan
POST /api/reviews
Tambahkan ulasan untuk booking yang selesai.
Body: booking_id, rating, comment.
Response: Detail ulasan.

GET /api/reviews/{car_id}
Lihat ulasan untuk mobil tertentu.
Response: Array ulasan dan rating.

==
https://maxschmitt.me/posts/next-js-api-proxy
==

# Langkah-langkah Membuat Proyek Laravel 11 dengan Bootstrap 5

# 1. Instal Laravel 11
composer create-project laravel/laravel rental-mobil "^11.0"

# 2. Masuk ke direktori proyek
cd rental-mobil

# 3. Instal Laravel Breeze untuk autentikasi pengguna
composer require laravel/breeze --dev

# 4. Instal Breeze scaffolding dengan Bootstrap 5
#php artisan breeze:install bootstrap
php artisan breeze:install blade
php artisan migrate
npm install
npm run dev
php artisan serve

# 5. Jalankan migrasi database
php artisan migrate

# 6. Instal dependensi frontend
npm install

# 7. Build aset dengan Vite
npm run dev

# 8. Setup Spatie Role Permission
# Instal library Spatie Role Permission
composer require spatie/laravel-permission

# Publikasikan file konfigurasi dan migrasi Spatie
php artisan vendor:publish --provider="Spatie\Permission\PermissionServiceProvider"

# Jalankan migrasi untuk tabel roles dan permissions
php artisan migrate

# 9. Instal Sanctum untuk API security
composer require laravel/sanctum

# Publikasikan file konfigurasi Sanctum
php artisan vendor:publish --provider="Laravel\Sanctum\SanctumServiceProvider"

# Jalankan migrasi untuk tabel personal_access_tokens
php artisan migrate

# 10. Setup Midtrans Payment Gateway
# Tambahkan library Midtrans jika diperlukan
composer require midtrans/midtrans-php

# 11. Konfigurasi .env untuk database, Sanctum, dan Midtrans
cp .env.example .env

# Edit file .env sesuai kebutuhan:
# - Database credentials (DB_DATABASE, DB_USERNAME, DB_PASSWORD)
# - Sanctum domain (SANCTUM_STATEFUL_DOMAINS)
# - Midtrans API credentials (MIDTRANS_SERVER_KEY, MIDTRANS_CLIENT_KEY)

# 12. Jalankan server lokal
php artisan serve

==

ERP
FITUR PRODUK
STOK BARANG
-Kelola Stok
-Laporan Stok
-Kelola Stok Gudang
-Pengelolaan Barang
-Pengelolaan Harga
-Kelola Keuntungan
-Pengelolaan Toko
-Sistem Pemesanan Online
-Pengelolaan Pesanan
-Barcode
-No. Seri/Lot
-WMS(Warehouse Management System)

PRODUKSI
PENJUALAN
PEMBELIAN
AKUNTANSI
PENGGAJIAN
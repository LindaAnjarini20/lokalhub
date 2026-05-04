# 🛒 Platform LOKAL

> **Platform Digital Berbasis Mobile untuk Optimalisasi Sirkulasi Ekonomi Lokal**  
> Menghubungkan konsumen dengan UMKM lokal dalam ekosistem ekonomi digital tertutup (*closed-loop*).

![Version](https://img.shields.io/badge/versi-1.1.0-blue)
![Status](https://img.shields.io/badge/status-Draft%20Akhir-yellow)
![Platform](https://img.shields.io/badge/platform-Android%20%7C%20iOS-green)
![License](https://img.shields.io/badge/lisensi-Internal-lightgrey)

---

## 📋 Daftar Isi

- [Tentang Proyek](#tentang-proyek)
- [Fitur Utama](#fitur-utama)
- [Arsitektur Sistem](#arsitektur-sistem)
- [Tech Stack](#tech-stack)
- [Persyaratan Sistem](#persyaratan-sistem)
- [Instalasi & Menjalankan](#instalasi--menjalankan)
- [API Reference](#api-reference)
- [Struktur Database](#struktur-database)
- [Keamanan & Kepatuhan](#keamanan--kepatuhan)
- [Non-Functional Requirements](#non-functional-requirements)
- [Tim Pengembang](#tim-pengembang)
- [Roadmap](#roadmap)
- [Lisensi](#lisensi)

---

## Tentang Proyek

**Platform LOKAL** adalah aplikasi mobile ekosistem ekonomi digital yang dirancang untuk mengoptimalkan sirkulasi ekonomi lokal di **Kota Bandung dan sekitarnya** (Kab. Bandung, Kab. Bandung Barat, Kota Cimahi).

Platform ini menghubungkan konsumen dengan UMKM lokal menggunakan:
- 🗺️ **Peta interaktif** berbasis lokasi
- 🪙 **Lokal Coin** — sistem insentif token reward internal
- 🤖 **Analitik harga berbasis ML** untuk membantu UMKM menentukan harga kompetitif

**Tujuan utama:**
- Mengurangi kebocoran ekonomi lokal
- Memberi akses digital bagi UMKM yang selama ini tidak terjangkau platform besar
- Membangun ekosistem ekonomi komunitas yang berkelanjutan

---

## Fitur Utama

| Kode | Fitur | Prioritas |
|------|-------|-----------|
| F-01 | **Autentikasi & Manajemen Pengguna** — Registrasi multi-peran, login OTP SMS, JWT RS256 | 🔴 Tinggi |
| F-02 | **Peta Pasar & Katalog Produk** — Peta interaktif UMKM radius 0.5–10 km, CRUD produk | 🔴 Tinggi |
| F-03 | **Transaksi & Pembayaran** — Keranjang multi-UMKM, checkout via Midtrans (GoPay, OVO, DANA, VA, QRIS) | 🔴 Tinggi |
| F-04 | **Lokal Coin & Insentif** — Reward 2%/transaksi, diskon maks. 20%, kadaluwarsa 6 bulan | 🟡 Sedang |
| F-05 | **Rekomendasi Harga ML** — Analisis harga produk serupa radius 5 km secara asinkron | 🟡 Sedang |
| F-06 | **Dashboard Analitik UMKM** — Grafik penjualan, produk terlaris, pendapatan bersih, real-time | 🔴 Tinggi |
| F-07 | **Notifikasi & Otomasi** — Push notification & SMS via n8n untuk event transaksi | 🔴 Tinggi |

### Detail Fitur

**🔐 Autentikasi (F-01)**
- Registrasi 3 peran: Konsumen, UMKM, Produsen
- OTP 6 digit via SMS (Twilio), berlaku 5 menit
- Maks. 5 percobaan OTP salah → blokir 15 menit
- JWT RS256: access token 24 jam, refresh token 30 hari
- Verifikasi dokumen NIB/SIUP oleh admin untuk akun UMKM
- Bonus 50 Lokal Coin untuk pengguna baru

**🗺️ Peta Pasar & Produk (F-02)**
- Google Maps SDK dengan marker setiap UMKM aktif
- Radius pencarian 0.5–10 km (default: 5 km)
- Filter produk: nama, kategori, harga, jarak, rating
- UMKM dapat kelola produk: maks. 5 foto/produk, maks. 2 MB/foto
- Pagination: 20 item/halaman

**🛒 Transaksi & Pembayaran (F-03)**
- Keranjang belanja multi-UMKM dengan validasi stok real-time
- Metode pembayaran: GoPay, OVO, DANA, Virtual Account Bank, QRIS
- Link/QR pembayaran kadaluwarsa 30 menit
- Webhook Midtrans divalidasi dengan SHA-512
- Riwayat transaksi + dukungan alur refund

**🪙 Lokal Coin (F-04)**
- Kredit otomatis 2% dari nilai transaksi yang selesai
- +5 koin untuk setiap ulasan valid
- Maks. 20% nilai transaksi dapat dibayar dengan Lokal Coin
- Tidak dapat dikonversi ke mata uang fiat
- Hangus otomatis setelah 6 bulan; notifikasi peringatan 30 hari sebelumnya

**🤖 Rekomendasi Harga ML (F-05)**
- Analisis asinkron saat UMKM menambah produk baru
- Respons: harga saran, rentang harga, jumlah produk serupa
- Batas waktu 3 detik dengan graceful degradation

**📊 Dashboard Analitik UMKM (F-06)**
- Grafik penjualan: harian, mingguan, bulanan
- Metrik: total pendapatan, pertumbuhan %, jumlah order, produk terlaris, rating, pelanggan baru
- Data diperbarui maks. 5 menit setelah transaksi
- Filter rentang tanggal kustom

**🔔 Notifikasi & Otomasi (F-07)**
- Notifikasi: konfirmasi pesanan, pembayaran berhasil/gagal, update status pengiriman
- Peringatan stok menipis (< 10 unit) untuk UMKM
- Preferensi notifikasi dapat diatur pengguna
- Semua alur diimplementasikan via n8n workflow engine

---

## Arsitektur Sistem

```
┌─────────────────────────────────────────────────────────┐
│                    CLIENT LAYER                         │
│         Flutter 3.19 (Android API 24+ / iOS 14+)        │
│   [Auth] [Market Map] [Checkout] [Wallet] [Dashboard]   │
└─────────────────────┬───────────────────────────────────┘
                      │ HTTPS / TLS 1.2+
┌─────────────────────▼───────────────────────────────────┐
│                  API GATEWAY LAYER                      │
│          Nginx 1.25 — Reverse Proxy + Rate Limit        │
└──────┬──────────────┬──────────────┬────────────────────┘
       │              │              │
┌──────▼──────┐ ┌─────▼──────┐ ┌───▼────────────────────┐
│  Laravel 11 │ │    n8n     │ │  ML Service             │
│  API Backend│ │  Workflow  │ │  Python 3.11 / FastAPI  │
│  PHP 8.3    │ │  Engine    │ │  scikit-learn           │
└──────┬──────┘ └─────┬──────┘ └───────────────────────┘
       │              │
┌──────▼──────────────▼───────────────────────────────────┐
│                    DATA LAYER                           │
│   MySQL 8.0  │  Redis 7.2  │  MinIO (S3-compatible)    │
└─────────────────────────────────────────────────────────┘
                      │
┌─────────────────────▼───────────────────────────────────┐
│                EXTERNAL SERVICES                        │
│   Midtrans (Payment) │ Twilio (SMS) │ Google Maps API   │
└─────────────────────────────────────────────────────────┘
```

---

## Tech Stack

### Mobile (Client)
| Teknologi | Versi | Fungsi |
|-----------|-------|--------|
| Flutter | 3.19 | Framework mobile cross-platform |
| Riverpod | Latest | State management |
| Dio | Latest | HTTP client |

### Backend
| Teknologi | Versi | Fungsi |
|-----------|-------|--------|
| PHP | 8.3 | Runtime backend |
| Laravel | 11 | API framework |
| Python | 3.11 | ML Service runtime |
| FastAPI | Latest | ML Service API framework |
| scikit-learn | Latest | Machine learning |
| n8n | Self-hosted | Workflow automation |

### Infrastructure
| Teknologi | Versi | Fungsi |
|-----------|-------|--------|
| Nginx | 1.25 | Reverse proxy & rate limiting |
| MySQL | 8.0 | Database utama |
| Redis | 7.2-alpine | Cache & session (OTP) |
| MinIO | Latest | Object storage (S3-compatible) |
| Docker | Latest | Containerisasi |

### External Services
| Layanan | Fungsi |
|---------|--------|
| Midtrans | Payment gateway (SNAP API) |
| Twilio | SMS Gateway (OTP) |
| Google Maps Platform | Maps SDK, Geocoding, Distance Matrix |
| FCM / APNs | Push notification |

---

## Persyaratan Sistem

### Mobile
- **Android:** API 24+ (Android 7.0 Nougat ke atas)
- **iOS:** iOS 14.0 ke atas
- **Koneksi internet:** Minimum 3G/HSPA

### Server (Production)
- **OS:** Ubuntu 22.04 LTS
- **CPU:** 8 vCPU
- **RAM:** 16 GB
- **Lokasi:** VPS Indonesia (wajib — UU PDP)

---

## Instalasi & Menjalankan

### Prerequisites

Pastikan sudah terinstall:
- [Docker](https://docs.docker.com/get-docker/) & Docker Compose
- [Flutter SDK 3.19+](https://docs.flutter.dev/get-started/install)
- [PHP 8.3](https://www.php.net/) & [Composer](https://getcomposer.org/)
- [Python 3.11+](https://www.python.org/)
- [Node.js](https://nodejs.org/) (untuk n8n)

---

### 1. Clone Repository

```bash
git clone https://github.com/your-org/platform-lokal.git
cd platform-lokal
```

### 2. Setup Backend (Laravel 11)

```bash
cd backend

# Install dependencies
composer install

# Salin dan konfigurasi environment
cp .env.example .env
php artisan key:generate

# Generate JWT key pair (RS256)
php artisan jwt:generate-keys

# Edit .env dengan kredensial database, Midtrans, Twilio, Google Maps
nano .env
```

**Konfigurasi `.env` penting:**
```env
APP_URL=https://api.lokal.id

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=lokal_db
DB_USERNAME=your_db_user
DB_PASSWORD=your_db_password

REDIS_HOST=127.0.0.1
REDIS_PORT=6379

MIDTRANS_SERVER_KEY=your_midtrans_server_key
MIDTRANS_CLIENT_KEY=your_midtrans_client_key

TWILIO_SID=your_twilio_sid
TWILIO_AUTH_TOKEN=your_twilio_auth_token
TWILIO_FROM=+1xxxxxxxxxx

GOOGLE_MAPS_KEY=your_google_maps_api_key
```

```bash
# Jalankan migrasi dan seeder
php artisan migrate --seed

# Jalankan server (development)
php artisan serve
```

### 3. Setup ML Service (Python/FastAPI)

```bash
cd ml-service

# Buat virtual environment
python -m venv venv
source venv/bin/activate  # Linux/Mac
# atau: venv\Scripts\activate  # Windows

# Install dependencies
pip install -r requirements.txt

# Jalankan service
uvicorn main:app --host 0.0.0.0 --port 8000
```

### 4. Setup dengan Docker Compose (Rekomendasi)

```bash
# Di root folder proyek
cp .env.example .env
# Edit .env sesuai konfigurasi

docker-compose up -d
```

### 5. Setup n8n Workflow Engine

```bash
docker run -it --rm \
  --name n8n \
  -p 5678:5678 \
  -v ~/.n8n:/home/node/.n8n \
  n8nio/n8n
```

Import workflow template dari folder `/n8n-workflows/` melalui UI n8n di `http://localhost:5678`.

### 6. Setup Mobile (Flutter)

```bash
cd mobile

# Install dependencies
flutter pub get

# Salin dan konfigurasi environment
cp lib/config/env.example.dart lib/config/env.dart

# Jalankan di device/emulator
flutter run

# Build APK (Android)
flutter build apk --release

# Build IPA (iOS)
flutter build ios --release
```

---

## API Reference

**Base URL:** `https://api.lokal.id/v1`

**Autentikasi:** Bearer Token (JWT RS256)

### Endpoint Utama

| Method | Endpoint | Deskripsi | Auth |
|--------|----------|-----------|------|
| `POST` | `/auth/request-otp` | Kirim OTP ke nomor HP | Public |
| `POST` | `/auth/verify-otp` | Verifikasi OTP & dapatkan JWT | Public |
| `POST` | `/auth/refresh` | Refresh access token | Public |
| `DELETE` | `/auth/logout` | Invalidasi token aktif | Bearer |
| `GET/PATCH` | `/users/me` | Ambil / perbarui profil pengguna | Bearer |
| `GET` | `/products` | Daftar produk dengan filter lokasi | Bearer |
| `POST` | `/products` | Tambah produk baru | Bearer+UMKM |
| `GET/PATCH/DELETE` | `/products/{id}` | Detail / update / hapus produk | Bearer |
| `GET` | `/umkm/nearby` | Daftar UMKM terdekat untuk peta | Bearer |
| `POST` | `/orders` | Buat pesanan dari keranjang | Bearer |
| `GET` | `/orders` | Riwayat pesanan pengguna | Bearer |
| `PATCH` | `/orders/{id}/status` | Update status pesanan | Bearer+UMKM |
| `POST` | `/orders/{id}/review` | Berikan ulasan produk | Bearer |
| `GET` | `/wallet/balance` | Saldo & ringkasan Lokal Coin | Bearer |
| `GET` | `/wallet/history` | Riwayat transaksi Lokal Coin | Bearer |
| `GET` | `/umkm/analytics/summary` | Ringkasan performa penjualan | Bearer+UMKM |
| `GET/PATCH` | `/notifications` | Daftar / tandai baca notifikasi | Bearer |

> 📄 Dokumentasi lengkap tersedia dalam format **OpenAPI 3.0** di `/docs/openapi.yaml`  
> 🧪 Postman Collection tersedia di `/docs/lokal-api.postman_collection.json`

### Rate Limiting

| Endpoint | Limit |
|----------|-------|
| OTP / Login | 5 request/menit per IP |
| Endpoint umum | 100 request/menit per token |

---

## Struktur Database

- **Engine:** MySQL 8.0
- **Koordinat UMKM:** Tipe data `POINT` dengan indeks `SPATIAL` (query geospasial < 200ms)
- **Atribut produk dinamis:** Tipe data `JSON`
- **Backup otomatis:** Setiap hari pukul 02.00 WIB, retensi 30 hari

Lihat diagram ERD lengkap di [`/docs/erd.png`](./docs/erd.png).

---

## Keamanan & Kepatuhan

### Keamanan Teknis
- **HTTPS/TLS 1.2+** untuk semua komunikasi client-server
- **JWT RS256** disimpan di Android Keystore / iOS Keychain
- **Rate limiting** ketat pada endpoint OTP (5/menit per IP)
- **AES-256** untuk enkripsi data sensitif (nomor HP, alamat) di database
- **SHA-512** untuk validasi webhook Midtrans
- **Idempotency** pada webhook untuk mencegah double processing
- **Audit log immutable** untuk seluruh transaksi keuangan
- Proteksi terhadap **OWASP Top 10**

### Kepatuhan Regulasi
- ✅ **UU PDP No. 27/2022** — Data pribadi disimpan di server Indonesia
- ✅ **OJK/Bank Indonesia** — Seluruh transaksi keuangan melalui Midtrans (berizin)
- ✅ Platform berperan sebagai **marketplace** (bukan lembaga keuangan)
- ✅ Syarat & Ketentuan + Kebijakan Privasi disetujui saat registrasi
- ✅ Data pengguna tidak digunakan untuk iklan pihak ketiga tanpa persetujuan

---

## Non-Functional Requirements

| Kategori | Target |
|----------|--------|
| Response time API kritikal | < 500ms pada 1.000 concurrent request |
| Response time ML Service | < 3 detik |
| Render peta 100+ marker | < 2 detik di perangkat kelas menengah |
| Query geospasial | < 200ms |
| Concurrent users (pilot) | 10.000 pengguna |
| Uptime | ≥ 99.5%/bulan |
| Unit test coverage | Minimum 80% |
| Waktu onboarding pengguna baru | ≤ 5 menit |
| Maintenance window | 00.00–04.00 WIB |

---

## Tim Pengembang

Dikembangkan oleh tim mahasiswa **Program Studi Sistem Informasi**  
**Fakultas Ilmu Komputer dan Sistem Informasi**  
**Universitas Kebangsaan Republik Indonesia** — Tahun 2026

| Nama | NPM |
|------|-----|
| Linda Anjarini | 20241320058 |
| Kiara Evi Nurdiati Putri Rahmatillah | 20241320067 |
| Najwa Alifah | 20241320077 |
| Ikhsan | 20241320083 |
| Naufal Al Farros | 20241320091 |
| Ikbal Maulana Aspahni | 20241320053 |
| Fito Zulhian Jabatami | 20241320074 |

---

## Roadmap

| Versi | Target | Fitur |
|-------|--------|-------|
| **v1.1.0** ✅ | April 2026 | MVP - Fitur inti (F-01 s.d. F-07) |
| **v1.2.0** | Q3 2026 | Integrasi logistik (JNE, SiCepat, AnterAja), video tutorial onboarding UMKM |
| **v1.5.0** | Q4 2026 | Mekanisme penyelesaian sengketa konsumen-UMKM |
| **v2.0.0** | 2026 | Multi-bahasa (Inggris, Sunda), ekspansi wilayah |

---

## Lisensi

Dokumen ini bersifat **Internal - Dokumen Perancangan**.  
Versi SRS: **1.1.0 - April 2026** | Status: **Draft Akhir**  
Klasifikasi: Internal Tim Pengembang Platform LOKAL - Bandung, Jawa Barat, Indonesia.

---

<div align="center">
  <strong>Platform LOKAL</strong> — Membangun Ekosistem Ekonomi Komunitas yang Berkelanjutan 🌱
</div>

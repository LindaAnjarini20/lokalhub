# Diagram Deployment - LokalHub

## Gambaran Umum
Dokumen ini menjelaskan arsitektur deployment LokalHub untuk environment produksi dan pengembangan menggunakan Docker dan Docker Compose. Semua layanan backend (API, database, cache, reverse proxy) dikontainerisasi agar mudah di-deploy dan diskalakan.

## Komponen Infrastruktur
- Nginx (Reverse Proxy) - menerima trafik HTTPS, meneruskan ke container Laravel API.
- Laravel API - aplikasi backend (container PHP-FPM + codebase Laravel).
- Queue Worker - menjalankan jobs/queue (Laravel Horizon atau worker artisan queue:work).
- Database - PostgreSQL atau MySQL (container terpisah, data persistent volume).
- Redis - cache dan queue backend (container terpisah).
- Storage Volume - shared volume untuk file uploads (atau gunakan S3 untuk production).
- Optional: WebSocket server (socket.io atau laravel-websockets) untuk real-time chat.
- Monitoring (Prometheus/Grafana) dan Logging (ELK) sebagai layanan terpisah.

## Topologi Deployment (Sederhana)

```
Internet
  |
  v
[Load Balancer / CDN]
  |
  v
[Nginx Reverse Proxy (SSL Termination)]
  |
  +-------------------------------+
  |                               |
  v                               v
[Laravel API Container]       [WebSocket Container (opsional)]
  |                               |
  v                               v
[Redis Container]             [Redis Container]
[Postgres Container]
[File Storage Volume] (shared)
```

## Docker Compose (Contoh minimal)

```yaml
version: '3.8'
services:
  nginx:
    image: nginx:stable
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/conf.d:/etc/nginx/conf.d
      - ./storage:/var/www/storage
    depends_on:
      - app

  app:
    build:
      context: ./backend
      dockerfile: Dockerfile
    image: lokalhub_app:latest
    working_dir: /var/www
    volumes:
      - ./backend:/var/www
      - ./storage:/var/www/storage
    environment:
      - APP_ENV=production
      - DB_HOST=db
      - REDIS_HOST=redis
    depends_on:
      - db
      - redis

  queue:
    image: lokalhub_app:latest
    command: php artisan queue:work --sleep=3 --tries=3
    depends_on:
      - app
      - redis

  db:
    image: postgres:13
    environment:
      POSTGRES_DB: lokalhub
      POSTGRES_USER: lokalhub
      POSTGRES_PASSWORD: secret
    volumes:
      - db_data:/var/lib/postgresql/data

  redis:
    image: redis:6
    volumes:
      - redis_data:/data

volumes:
  db_data:
  redis_data:
```

## Praktik Keamanan dan Operasional
- Gunakan HTTPS (TLS) di reverse proxy (sertifikat dari Let's Encrypt atau CA internal).
- Jangan menyimpan credential sensitif dalam repository; gunakan secrets manager atau environment variables yang aman.
- Backup database secara berkala dan simpan snapshot di cloud storage.
- Gunakan health checks dan auto-restart policy untuk container.
- Pisahkan environment (development/staging/production) dengan konfigurasi yang berbeda.

## Catatan Deploy ke Hosting
- Backend container dapat di-deploy ke VPS atau layanan container (DigitalOcean App Platform, AWS ECS, GCP Cloud Run).
- Untuk production, pertimbangkan menggunakan managed database dan object storage (RDS, S3/GCS) agar lebih handal.
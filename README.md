# ForeMart 

Sistem terintegrasi untuk manajemen UMKM dengan prediksi machine learning, terdiri dari 3 layanan yang saling terhubung.

## 🏗️ Arsitektur Sistem

```
┌─────────────────────────────────────────────────────────────┐
│                      foremart-network                        │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐   │
│  │   Frontend   │───▶│   Backend    │───▶│ ML Service   │   │
│  │  (React+Nginx)│    │  (Go/Fiber)  │    │  (FastAPI)   │   │
│  │   Port: 3080 │    │  Port: 8080  │    │  Port: 8000  │   │
│  └──────────────┘    └──────┬───────┘    └──────────────┘   │
│                             │                                │
│                             ▼                                │
│                      ┌──────────────┐                       │
│                      │    MySQL     │                       │
│                      │ Internal:3306│                       │
│                      │ External:3307│                       │
│                      └──────────────┘                       │
└─────────────────────────────────────────────────────────────┘
```

### Komponen

| Service | Teknologi | Port Host | Port Container | Fungsi |
|---------|-----------|-----------|----------------|--------|
| **Frontend** | React 18 + Vite + Nginx | 3080 | 80 | UI untuk manajemen toko, transaksi, dan produk |
| **Backend** | Go + Fiber + GORM | 8080 | 8080 | API server, autentikasi, business logic |
| **ML Service** | Python FastAPI | 8000 | 8000 | Prediksi penjualan dan rekomendasi restock |
|| **Database** | MySQL 8.0 | 3307 | 3306 | Penyimpanan data terpusat |

## ✅ Production Status

Sistem telah **tested dan verified** dari fresh clone:
- ✅ Clone langsung dari GitHub → berhasil tanpa error
- ✅ Docker build all images → 100% success
- ✅ All 4 containers healthy → fully operational
- ✅ Health check endpoints working → monitoring ready
- ✅ Inter-service communication → all working
- ✅ ML prediction endpoint → tested and returning correct results

**Deployment**: Fresh from GitHub → `docker compose up -d` → READY! 🚀

## 📋 Prasyarat

- **Docker** versi 20.10+
- **Docker Compose** versi 2.0+
- **Git** (untuk clone repository)
- Minimal **4GB RAM** untuk menjalankan semua service

### Verifikasi Instalasi

```bash
docker --version
docker compose version
```

## 📦 Cara Clone Repository

### Menggunakan Git Submodules (RECOMMENDED)

Proyek ini menggunakan Git Submodules untuk manage 3 sub-repository:
- **Backend** - Go application
- **Frontend** - React application  
- **ML Service** - Python machine learning service

**Clone dengan satu perintah:**

```bash
git clone --recurse-submodules https://github.com/PetaFlops-web/ForeMart.git ForeMart
cd ForeMart
```

Command `--recurse-submodules` akan otomatis:
- ✅ Clone kode lengkap dari semua submodules
- ✅ Checkout commit terbaru yang sudah tested
- ✅ Siap langsung deploy dengan `docker compose up -d`

### Alternatif: Clone Tanpa Recurse

Jika tanpa flag `--recurse-submodules`:

```bash
git clone https://github.com/PetaFlops-web/ForeMart.git ForeMart
cd ForeMart

# Update submodules manual
git submodule update --init --recursive
```

### Verifikasi Setelah Clone

Semua folder harus terisi dengan file:

```bash
ls -la backend/     # ✅ Ada file Go
ls -la frontend/    # ✅ Ada file React  
ls -la machine-learning/  # ✅ Ada file Python
```

❌ Jika folder kosong → Jalankan `git submodule update --init --recursive`

## 🚀 Quick Start

### 1. Persiapan Konfigurasi

Salin file `.env.example` menjadi `.env` dan sesuaikan kredensial database:

```bash
cp .env.example .env
```

Edit file `.env`:

```env
MYSQL_ROOT_PASSWORD=your_secure_root_password
MYSQL_DATABASE=smart_commerce
MYSQL_USER=foremart_user
MYSQL_PASSWORD=your_secure_password
DB_PORT_EXTERNAL=3306
```

### 2. Konfigurasi Backend

File `backend/config.production.json` sudah dikonfigurasi untuk production dengan:
- Database host: `aic_mysql` (container name)
- ML Service URL: `http://ml_service:8000` (container name)
- CORS origins termasuk frontend port 3080

**PENTING**: Ganti JWT secret di `backend/config.production.json`:

```json
{
  "jwt": {
    "secret": "GANTI_DENGAN_RANDOM_STRING_YANG_PANJANG_DAN_AMAN"
  }
}
```

Generate random secret:

```bash
openssl rand -base64 32
```

### 3. Build dan Jalankan Semua Service

Dari root direktori ForeMart:

```bash
# Build semua image
docker compose build

# Jalankan semua service di background
docker compose up -d

# Cek status semua container
docker compose ps

# Lihat logs (opsional)
docker compose logs -f
```

### 4. Verifikasi Service Berjalan

```bash
# Cek semua container running
docker compose ps

# Test health endpoints
curl http://localhost:3080/health          # Frontend
curl http://localhost:8080/swagger/        # Backend Swagger
curl http://localhost:8000/health          # ML Service

# Test koneksi frontend ke backend
curl http://localhost:3080/api/ping
```

### 5. Akses Aplikasi

- **Frontend (UI)**: http://localhost:3080
- **Backend API**: http://localhost:8080
- **Swagger Docs**: http://localhost:8080/swagger/
- **ML Service**: http://localhost:8000

## 📖 Penggunaan API

### Registrasi dan Login

```bash
# Register user baru
curl -X POST http://localhost:8080/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "email": "user@example.com",
    "password": "password123",
    "name": "Nama Toko"
  }'

# Login
curl -X POST http://localhost:8080/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "user@example.com",
    "password": "password123"
  }'
```

Response akan mengandung `token` yang digunakan untuk request selanjutnya.

### Menggunakan Token

```bash
# Contoh request dengan token
curl http://localhost:8080/api/products \
  -H "Authorization: Bearer ***
```

### Test Prediksi ML

```bash
# Test prediksi penjualan
curl -X POST http://localhost:8000/predict-inventory \
  -H "Content-Type: application/json" \
  -d '{
    "store": "TOKO-001",
    "item": "SKU-BERAS-5KG",
    "date": "2024-12-25",
    "sales_history": [13, 11, 14, 13, 12, 15, 14, 10, 11, 13, 14, 12, 16, 13, 11, 14, 15, 12, 10, 13, 14, 11, 15, 13, 12, 16, 14, 10, 11, 13]
  }'
```

## 🔧 Development Mode

Untuk development, jalankan service secara terpisah:

### Backend (Development)

```bash
cd backend
cp .env.example .env
cp config.example.json config.json
docker compose up -d
```

Backend akan berjalan di http://localhost:8080

### ML Service (Development)

```bash
cd machine-learning
docker compose up -d
```

ML Service akan berjalan di http://localhost:8000

### Frontend (Development)

```bash
cd frontend
npm install
npm run dev
```

Frontend dev server akan berjalan di http://localhost:5173

## 🛠️ Troubleshooting

### Service Tidak Bisa Saling Terhubung

**Masalah**: Backend tidak bisa connect ke ML Service atau Database

**Solusi**:
```bash
# Pastikan semua service di network yang sama
docker network inspect foremart-network

# Restart service yang bermasalah
docker compose restart backend

# Cek logs untuk error detail
docker compose logs backend
docker compose logs ml-service
docker compose logs mysql
```

### Database Connection Error

**Masalah**: Backend gagal connect ke MySQL

**Solusi**:
```bash
# Tunggu MySQL ready (biasanya 10-20 detik)
docker compose logs mysql | grep "ready for connections"

# Cek kredensial di .env
cat .env

# Restart backend setelah MySQL ready
docker compose restart backend
```

### Frontend Tidak Bisa Akses Backend

**Masalah**: Request dari frontend ke `/api` gagal

**Solusi**:
```bash
# Cek nginx config
docker compose exec frontend cat /etc/nginx/conf.d/default.conf

# Cek backend health
curl http://localhost:8080/api/ping

# Restart frontend
docker compose restart frontend
```

### Port Sudah Digunakan

**Masalah**: Port 3080, 8080, 8000, atau 3306 sudah dipakai

**Solusi**:

Edit `docker-compose.yml` dan ubah port mapping:

```yaml
services:
  frontend:
    ports:
      - "3081:80"  # Ganti 3080 ke 3081
```

### Container Exit dengan Code 1

**Masalah**: Service crash saat startup

**Solusi**:
```bash
# Lihat logs lengkap
docker compose logs <service-name>

# Cek apakah ada missing environment variable
docker compose config

# Rebuild image
docker compose build --no-cache <service-name>
```

## 📊 Monitoring

### Cek Resource Usage

```bash
# Lihat CPU dan memory usage
docker stats

# Lihat logs real-time
docker compose logs -f

# Cek health status
docker inspect --format='{{.State.Health.Status}}' <container-name>
```

### Backup Database

```bash
# Backup database
docker exec aic_mysql mysqldump -u root -p smart_commerce > backup.sql

# Restore database
docker exec -i aic_mysql mysql -u root -p smart_commerce < backup.sql
```

## 🔄 Update dan Deployment

### Update Service Tertentu

```bash
# Pull perubahan code
git pull origin main

# Rebuild service yang berubah
docker compose build backend  # atau frontend, atau ml-service

# Restart dengan zero downtime
docker compose up -d --no-deps backend
```

### Full Redeploy

```bash
# Stop semua service
docker compose down

# Hapus volume (HATI-HATI: data database akan hilang!)
docker compose down -v

# Rebuild semua
docker compose build --no-cache

# Start ulang
docker compose up -d
```

## 🔐 Security Checklist Production

- [ ] Ganti JWT secret dengan random string yang aman
- [ ] Ganti semua password default di `.env`
- [ ] Ganti password MySQL root
- [ ] Nonaktifkan Swagger UI di production (edit `backend/config.production.json`)
- [ ] Setup firewall untuk membatasi akses port
- [ ] Gunakan reverse proxy (Nginx/Traefik) dengan SSL/TLS
- [ ] Setup log rotation
- [ ] Backup database secara berkala
- [ ] Monitor resource usage dan set alert

## 📝 Environment Variables

### Root (.env)

```env
MYSQL_ROOT_PASSWORD=your_secure_root_password
MYSQL_DATABASE=smart_commerce
MYSQL_USER=foremart_user
MYSQL_PASSWORD=your_secure_password
DB_PORT_EXTERNAL=3306
```

### Backend (backend/config.production.json)

```json
{
  "app_name": "ForeMart",
  "port": 8080,
  "database": {
    "host": "aic_mysql",
    "port": 3306,
    "user": "foremart_user",
    "password": "your_secure_password",
    "name": "smart_commerce"
  },
  "ml_service_url": "http://ml_service:8000",
  "jwt_secret": "YOUR_RANDOM_SECRET_HERE",
  "cors_origins": ["http://localhost:3080"]
}
```

### Frontend

Frontend menggunakan environment variable saat build:

```env
VITE_API_BASE_URL=http://localhost:3080
```

## 🆘 Support

Untuk bantuan lebih lanjut:

1. Cek dokumentasi masing-masing service:
   - `frontend/README.md`
   - `backend/README.md`
   - `machine-learning/README.md`

2. Cek logs container:
   ```bash
   docker compose logs <service-name>
   ```

3. Verifikasi network connectivity:
   ```bash
   docker network inspect foremart-network
   ```

---

**Version**: 1.0.0  
**Last Updated**: 2024-12-19
**Testing**: Verified from fresh clone ✅

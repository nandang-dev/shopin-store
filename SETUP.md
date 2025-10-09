# 🚀 Panduan Setup HiyoRi E-Commerce

Panduan lengkap untuk setup project HiyoRi E-Commerce di lokal environment Anda.

## 📋 Prerequisites

Pastikan Anda sudah install:
- **Node.js** versi 18.x atau lebih tinggi
- **npm** atau **yarn**
- **Git**
- Akun **Supabase** (gratis)
- Akun **Stripe** (gratis untuk testing)
- Akun **AWS** (untuk S3 storage)

## 🛠️ Langkah-langkah Setup

### 1️⃣ Clone & Install Dependencies

```bash
# Jika belum clone, clone dulu repository
git clone <repository-url>
cd HiyoRi-Ecommerce

# Install dependencies
npm install
```

### 2️⃣ Setup Supabase

#### a. Buat Project Baru di Supabase

1. Kunjungi [supabase.com](https://supabase.com)
2. Login atau daftar akun baru
3. Klik **"New Project"**
4. Isi detail project:
   - **Name**: HiyoRi-Ecommerce
   - **Database Password**: Buat password yang kuat (SIMPAN PASSWORD INI!)
   - **Region**: Pilih yang terdekat dengan lokasi Anda
5. Tunggu sampai project selesai dibuat (~2 menit)

#### b. Dapatkan Supabase Credentials

1. Buka project Anda di Supabase Dashboard
2. Pergi ke **Settings → API**
3. Copy credentials berikut:
   - **Project URL** → `NEXT_PUBLIC_SUPABASE_URL`
   - **anon/public key** → `NEXT_PUBLIC_SUPABASE_ANON_KEY`
   - **service_role key** → `DATABASE_SERVICE_ROLE`
   - **Project Reference ID** (dari URL) → `NEXT_PUBLIC_SUPABASE_PROJECT_REF`

4. Pergi ke **Settings → Database**
5. Copy **Connection String** (URI mode):
   - Format: `postgresql://postgres:[YOUR-PASSWORD]@db.[REF].supabase.co:5432/postgres`
   - Ganti `[YOUR-PASSWORD]` dengan password database Anda
   - Simpan sebagai → `DATABASE_URL`

#### c. Enable Authentication Providers

1. Pergi ke **Authentication → Providers**
2. Enable **Email** provider (sudah default)
3. (Opsional) Enable **Google OAuth**:
   - Ikuti instruksi untuk setup Google OAuth
   - Tambahkan credentials dari Google Cloud Console

### 3️⃣ Setup Stripe

#### a. Buat Akun Stripe

1. Kunjungi [stripe.com](https://stripe.com)
2. Daftar akun baru atau login
3. Aktifkan **Test Mode** (toggle di kanan atas)

#### b. Dapatkan API Keys

1. Pergi ke **Developers → API keys**
2. Copy credentials:
   - **Publishable key** → `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY`
   - **Secret key** → `STRIPE_SECRET_KEY`

#### c. Setup Webhook (untuk local development)

1. Install Stripe CLI:
```bash
brew install stripe/stripe-cli/stripe
# atau download dari https://stripe.com/docs/stripe-cli
```

2. Login ke Stripe CLI:
```bash
stripe login
```

3. Untuk mendapatkan webhook secret (jalankan nanti saat development):
```bash
stripe listen --forward-to localhost:3000/api/webhook
```
   - Copy **webhook signing secret** → `STRIPE_WEBHOOK_SECERT_KEY`

### 4️⃣ Setup AWS S3

#### a. Buat S3 Bucket

1. Login ke [AWS Console](https://console.aws.amazon.com)
2. Pergi ke **S3**
3. Klik **Create bucket**
4. Konfigurasi:
   - **Bucket name**: `hiyori-media-uploads` (atau nama lain yang unik)
   - **Region**: Pilih region (contoh: `us-west-2`)
   - **Block Public Access**: Uncheck (karena kita perlu akses public untuk gambar)
5. Klik **Create bucket**

#### b. Setup CORS Policy

1. Buka bucket yang baru dibuat
2. Pergi ke **Permissions → CORS**
3. Tambahkan konfigurasi berikut:

```json
[
    {
        "AllowedHeaders": ["*"],
        "AllowedMethods": ["GET", "PUT", "POST", "DELETE"],
        "AllowedOrigins": ["http://localhost:3000", "https://your-production-domain.com"],
        "ExposeHeaders": []
    }
]
```

#### c. Buat IAM User untuk API Access

1. Pergi ke **IAM → Users**
2. Klik **Add users**
3. Username: `hiyori-s3-user`
4. Access type: **Programmatic access**
5. Permissions: **Attach existing policies directly**
   - Pilih **AmazonS3FullAccess** (atau buat custom policy yang lebih restrictive)
6. Selesaikan proses
7. **SIMPAN** credentials:
   - **Access Key ID** → `S3_ACCESS_KEY_ID`
   - **Secret Access Key** → `S3_SECRET_ACCESS_KEY`

### 5️⃣ Setup Environment Variables

1. Copy file `.env.example` ke `.env.local`:
```bash
cp .env.example .env.local
```

2. Buka `.env.local` dan isi semua nilai dengan credentials yang sudah Anda dapatkan:

```env
NEXT_PUBLIC_SITE_URL=http://localhost:3000

# Supabase
NEXT_PUBLIC_SUPABASE_URL=https://xxxxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJxxxx...
NEXT_PUBLIC_SUPABASE_PROJECT_REF=xxxxx
DATABASE_URL=postgresql://postgres:your-password@db.xxxxx.supabase.co:5432/postgres
DATABASE_SERVICE_ROLE=eyJxxxx...

# Stripe
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_test_xxxx...
STRIPE_SECRET_KEY=sk_test_xxxx...
STRIPE_WEBHOOK_SECERT_KEY=whsec_xxxx...

# AWS S3
S3_ACCESS_KEY_ID=AKIAxxxx...
S3_SECRET_ACCESS_KEY=xxxx...
NEXT_PUBLIC_S3_BUCKET=your-bucket-name
NEXT_PUBLIC_S3_REGION=us-west-2
```

### 6️⃣ Setup Database Schema

Setelah environment variables sudah disetup:

```bash
# Generate migration files (jika belum ada)
npm run db:generate

# Push schema ke database Supabase
npm run db:push
```

### 7️⃣ (Opsional) Seed Database dengan Data Demo

Untuk mengisi database dengan data contoh:

```bash
npm run db:seed
```

### 8️⃣ Setup Row Level Security di Supabase

1. Buka Supabase Dashboard → **SQL Editor**
2. Jalankan query untuk setup RLS policies (file SQL akan tersedia di `src/lib/supabase/`)

### 9️⃣ Jalankan Development Server

```bash
npm run dev
```

Buka browser dan akses: **http://localhost:3000**

## 🎯 Testing Setup

### Test Authentication
1. Buka `/sign-up`
2. Buat akun baru
3. Cek email untuk verifikasi

### Test Admin Access
1. Login dengan akun yang sudah dibuat
2. Update `is_admin` di Supabase Database untuk user Anda:
   ```sql
   UPDATE profiles 
   SET is_admin = true 
   WHERE email = 'your-email@example.com';
   ```
3. Akses `/admin` untuk melihat CMS

### Test Payment (Stripe Test Mode)
1. Tambahkan produk ke cart
2. Checkout
3. Gunakan test card number Stripe:
   - **Card Number**: `4242 4242 4242 4242`
   - **Expiry**: Tanggal masa depan
   - **CVC**: 123
   - **ZIP**: 12345

### Test Media Upload
1. Login sebagai admin
2. Pergi ke `/admin/medias`
3. Upload gambar test

## 🔧 Troubleshooting

### Error: "DATABASE_URL is missing"
- Pastikan file `.env.local` sudah dibuat dan berisi `DATABASE_URL`
- Restart development server setelah menambahkan env vars

### Error: "Supabase client error"
- Cek apakah `NEXT_PUBLIC_SUPABASE_URL` dan `NEXT_PUBLIC_SUPABASE_ANON_KEY` benar
- Pastikan Supabase project Anda sudah fully initialized

### Error: "S3 upload failed"
- Cek credentials S3
- Pastikan CORS policy sudah ditambahkan ke bucket
- Verifikasi region bucket sesuai dengan `NEXT_PUBLIC_S3_REGION`

### Error: "Stripe webhook failed"
- Untuk local development, jalankan `stripe listen --forward-to localhost:3000/api/webhook`
- Copy webhook secret yang diberikan ke `STRIPE_WEBHOOK_SECERT_KEY`

## 📚 Useful Commands

```bash
# Development
npm run dev                 # Start dev server
npm run build              # Build for production
npm start                  # Start production server

# Database
npm run db:generate        # Generate migrations
npm run db:push           # Push schema to DB
npm run db:studio         # Open Drizzle Studio
npm run db:seed           # Seed database

# Code Quality
npm run lint              # Run ESLint
npm run format            # Format code with Prettier
npm test                  # Run tests

# GraphQL
npm run codegen           # Generate GraphQL types
npm run codegen:watch     # Watch mode for codegen
```

## 🎉 Setup Complete!

Selamat! Project Anda sekarang sudah siap untuk development.

### Next Steps:
1. Baca dokumentasi di `/docs/project-structure.md`
2. Explore features yang sudah ada
3. Mulai customize sesuai kebutuhan Anda

## 📞 Butuh Bantuan?

- **GitHub Issues**: [Create an issue](https://github.com/clonglam/HIYORI-master/issues)
- **Documentation**: Check `/docs` folder
- **Community**: Join our Discord (if available)

Happy Coding! 🚀


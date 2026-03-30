# 🚀 Dongtube — Panduan Setup Environment

## Langkah 1 — Salin file .env

```bash
cp .env.example .env
```

Buka `.env` lalu isi semua variabel sesuai panduan di bawah.

---

## Langkah 2 — Security Keys (WAJIB)

### TOKEN_SECRET
Dipakai untuk sign JWT token login user. **Wajib diganti** di production — kalau kosong, server pakai nilai default yang mudah ditebak.

```bash
# Generate di terminal:
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
```

### CRON_SECRET
Dipakai untuk mengamankan endpoint `/api/cron/run`. Tanpa ini, endpoint cron **diblokir total**.

```bash
node -e "console.log(require('crypto').randomBytes(20).toString('hex'))"
```

### ADMIN_PASS & ADMIN_PATH
- `ADMIN_PASS` → password untuk login ke panel admin (`/admin`)
- `ADMIN_PATH` → bisa diubah dari `admin` ke string lain agar URL admin tidak mudah ditebak, misal `ADMIN_PATH=rahasia99`

---

## Langkah 3 — Database

### Opsi A: Supabase (Rekomendasi)

1. Buka [supabase.com](https://supabase.com) → **New Project**
2. Setelah project dibuat, buka **Settings → API**
3. Copy **Project URL** → isi `SUPABASE_URL`
4. Copy **service_role** key (bukan anon!) → isi `SUPABASE_SERVICE_KEY`
5. Buka **SQL Editor**, jalankan query ini:

```sql
CREATE TABLE IF NOT EXISTS kv_store (
  key        TEXT PRIMARY KEY,
  value      JSONB NOT NULL DEFAULT '{}',
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX IF NOT EXISTS idx_kv_store_prefix
  ON kv_store (key text_pattern_ops);
```

Set di `.env`:
```
DATABASE=supabase
SUPABASE_URL=https://xxxx.supabase.co
SUPABASE_SERVICE_KEY=eyJ...
```

### Opsi B: GitHub Database (Tanpa Supabase)

1. Buat repo **private** baru di GitHub (misal: `dongtube-db`)
2. Buat GitHub Personal Access Token dengan scope **repo** (full)
3. Set di `.env`:

```
DATABASE=github
GH_DB_TOKEN=ghp_xxx
GH_DB_OWNER=username_kamu
GH_DB_REPO=dongtube-db
GH_DB_BRANCH=main
```

> ⚠️ GitHub DB lebih lambat dari Supabase karena setiap read/write = API call. Cocok untuk trafik kecil.

---

## Langkah 4 — GitHub CDN (WAJIB)

CDN dipakai untuk menyimpan gambar produk, file upload, dan aset lainnya.

1. Buat repo **public** baru di GitHub (misal: `dongtube-cdn`)
2. Buat **Personal Access Token** → [github.com/settings/tokens](https://github.com/settings/tokens)
   - Scope yang dibutuhkan: **repo** (full control)
3. Set di `.env`:

```
GH_TOKEN=ghp_xxx
GH_OWNER=username_kamu
GH_REPO=dongtube-cdn
GH_BRANCH=main
GH_PRIVATE=false
```

**Multi-repo CDN** (jika satu repo mau penuh — GitHub limit 1GB per repo):
```
GH_REPO=cdn1,cdn2,cdn3
```
Server akan otomatis berpindah ke repo berikutnya saat yang aktif hampir penuh.

**CDN Kedua (opsional fallback):**
```
GH_TOKEN2=ghp_yyy
GH_OWNER2=username_kedua
GH_REPO2=dongtube-cdn2
```

---

## Langkah 5 — Pakasir QRIS Payment

1. Daftar di [pakasir.com](https://pakasir.com)
2. Buat toko → catat **slug** (subdomain toko kamu)
3. Buka **Pengaturan → API** → copy API Key
4. Set di `.env`:

```
PAKASIR_SLUG=nama-tokomu
PAKASIR_APIKEY=pak_xxx
```

---

## Langkah 6 — Pterodactyl Panel (untuk produk hosting)

> Skip langkah ini jika tidak jual produk panel.

1. Buka panel Pterodactyl kamu → **Admin → Application API**
2. Buat API key baru → copy key → isi `PTERO_APIKEY`
3. Buka **Account → API Credentials** → buat → isi `PTERO_CAPIKEY`
4. Cek ID Egg, Nest, dan Location di Admin panel:
   - **Admin → Nests → [Nest] → Eggs** → lihat ID di URL
   - **Admin → Locations** → lihat ID

```
PTERO_DOMAIN=https://panel.domainmu.com
PTERO_APIKEY=ptla_xxx
PTERO_CAPIKEY=ptlc_xxx
PTERO_EGG=15
PTERO_NEST=5
PTERO_LOCATION=1
```

---

## Langkah 7 — RumahOTP (untuk produk OTP)

> Skip langkah ini jika tidak jual nomor OTP.

1. Daftar di [rumahotp.com](https://rumahotp.com)
2. Buka **Dashboard → API Key** → copy key
3. Set di `.env`:

```
RUMAHOTP_APIKEY=xxx
```

---

## Langkah 8 — Google Login (opsional)

1. Buka [console.cloud.google.com](https://console.cloud.google.com)
2. Buat project baru → **APIs & Services → Credentials**
3. **Create Credentials → OAuth 2.0 Client ID → Web Application**
4. Tambahkan ke **Authorized JavaScript origins**: `https://domainmu.com`
5. Copy **Client ID** → isi `GOOGLE_CLIENT_ID`

```
GOOGLE_CLIENT_ID=xxxxxxx.apps.googleusercontent.com
```

---

## Deploy ke Vercel

1. Push project ke GitHub
2. Import di [vercel.com](https://vercel.com)
3. Di **Settings → Environment Variables**, tambahkan semua variabel dari `.env` satu per satu
   - **Jangan** upload file `.env` langsung — masukkan via UI Vercel
4. Tambahkan juga di **Settings → Cron Jobs**:
   ```
   Path: /api/cron/run
   Schedule: 0 * * * *   (setiap jam)
   ```
   Set header `Authorization: Bearer <CRON_SECRET>` atau gunakan query `?secret=<CRON_SECRET>`

> ✅ Vercel akan inject env vars secara otomatis saat build & runtime.

---

## Jalankan Lokal

```bash
# Install dependencies
npm install

# Jalankan server
node index.js

# Atau dengan nodemon (auto-restart saat file berubah)
npm install -g nodemon
nodemon index.js
```

Server berjalan di `http://localhost:3000`

---

## Checklist Minimal (harus ada sebelum go-live)

- [ ] `TOKEN_SECRET` — diisi string acak kuat
- [ ] `CRON_SECRET` — diisi string acak
- [ ] `ADMIN_PASS` — ganti dari default `admin123`
- [ ] `DATABASE` + Supabase **atau** GitHub DB
- [ ] `GH_TOKEN` + `GH_OWNER` + `GH_REPO` (CDN)
- [ ] `PAKASIR_SLUG` + `PAKASIR_APIKEY` (payment)
- [ ] `STORE_NAME` + `STORE_WA`
- [ ] SQL table `kv_store` sudah dibuat di Supabase (jika pakai Supabase)

---

## Urutan Prioritas Setup

```
TOKEN_SECRET  ──►  DATABASE  ──►  GH CDN  ──►  PAKASIR  ──►  PTERO/OTP  ──►  GOOGLE
   (wajib)         (wajib)       (wajib)      (wajib)       (opsional)     (opsional)
```

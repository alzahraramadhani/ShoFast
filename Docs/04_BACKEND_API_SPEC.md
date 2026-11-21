# 04 — Spesifikasi Backend API, FFmpeg Anti-Duplikat & Algoritma Jadwal Acak (FastAPI)

Backend **ShoFast** dibangun menggunakan **Python 3.11+ (FastAPI)** yang berjalan di dalam container Docker (bisa di PC Windows lokal atau VPS Linux 24 jam). Backend bertanggung jawab untuk mengekstrak video TikTok tanpa watermark, memproses anti-duplikat FFmpeg, meresolve metadata produk Shopee secara aman, menghasilkan caption AI unik via `gemini-2.5-flash`, menghitung jadwal acak terstratifikasi, serta mem-parsing CSV laporan komisi Shopee Affiliate.

---

## 1. Struktur Folder Backend (`backend/`)

```text
backend/
├── Dockerfile
├── docker-compose.yml
├── requirements.txt
├── .env.example
└── app/
    ├── main.py                    # Entrypoint FastAPI, CORS, API Key Middleware, Scheduler cleanup
    ├── config.py                  # Pydantic BaseSettings (.env loader)
    ├── models/
    │   └── schemas.py             # Pydantic request & response schemas
    ├── routers/
    │   ├── tiktok.py              # /api/v1/tiktok/* & /api/v1/videos/download/*
    │   ├── shopee.py              # /api/v1/shopee/resolve-product
    │   ├── ai.py                  # /api/v1/ai/generate-caption
    │   ├── queue.py               # /api/v1/queue/single & /api/v1/queue/batch
    │   └── commission.py          # /api/v1/commission/import-csv
    └── services/
        ├── supabase_client.py     # Singleton Supabase Admin Client (Service Role)
        ├── tiktok_downloader.py   # Wrapper yt-dlp + thumbnail extractor
        ├── ffmpeg_processor.py    # Anti-duplikat FFmpeg pipeline + SHA256 hash
        ├── shopee_parser.py       # Safe redirect resolver & URL/OG metadata parser
        ├── ai_caption.py          # Modular LLM Provider (Default: Gemini 2.5 Flash)
        ├── scheduler.py           # Stratified Random Windowing + Cooldown Anti-Bot
        └── csv_importer.py        # Parser CSV resmi Shopee Affiliate
```

---

## 2. Daftar Variabel Lingkungan (`.env`)

```env
APP_ENV=development
API_SECRET_KEY=ganti_dengan_kunci_rahasia_panjang_kamu
BASE_PUBLIC_URL=http://192.168.1.10:8000

SUPABASE_URL=https://xxxx.supabase.co
SUPABASE_SERVICE_ROLE_KEY=eyJhbGciOi...

LLM_PROVIDER=gemini
GEMINI_API_KEY=AIzaSy...
GEMINI_MODEL=gemini-2.5-flash
OPENAI_API_KEY=
GROQ_API_KEY=

TEMP_VIDEO_DIR=/tmp/shofast_cache
VIDEO_CACHE_TTL_HOURS=24
```

---

## 3. Spesifikasi Endpoint REST API

Seluruh endpoint (kecuali `GET /health` dan `GET /api/v1/videos/download/{task_id}` yang menggunakan token/ID acak) wajib menyertakan header `X-API-Key: <API_SECRET_KEY>`.

### 3.1. `POST /api/v1/tiktok/fetch`
Mengunduh video TikTok tanpa watermark menggunakan `yt-dlp`, menjalankan proses FFmpeg anti-duplikat ke `/tmp/shofast_cache/{task_id}.mp4`, dan mengembalikan metadata lengkap untuk preview di **Content Studio Page**.

- **Request Body**:
```json
{
  "tiktok_url": "https://vt.tiktok.com/ZS2x9qLM7/"
}
```
- **Response (`200 OK`)**:
```json
{
  "task_id": "sf_9a8b7c6d5e4f",
  "tiktok_video_id": "7384910293847561",
  "author": "@glowup.skincare",
  "title": "Viral Skincare Glowing Fast Review",
  "duration_sec": 32,
  "size_bytes": 14889779,
  "resolution": "1080x1920",
  "thumbnail_url": "https://p16-sign-sg.tiktokcdn.com/...",
  "download_url": "http://192.168.1.10:8000/api/v1/videos/download/sf_9a8b7c6d5e4f",
  "file_hash_sha256": "e3b0c44298fc1c149afbf4c8996fb924..."
}
```

### 3.2. `GET /api/v1/videos/download/{task_id}`
Melakukan streaming file MP4 hasil FFmpeg ke perangkat Android (mendukung HTTP `Range` request untuk resume download).
- **Query Param Opsional**: `?cleanup=false` (file tetap ada selama masa TTL 24 jam atau sampai Android mengonfirmasi selesai download lewat `DELETE /api/v1/videos/cache/{task_id}`).

### 3.3. `POST /api/v1/shopee/resolve-product`
Menguraikan link produk Shopee (pendek maupun panjang) **secara aman tanpa menembak internal private API Shopee** (hanya melakukan HTTP `HEAD`/`GET` redirect resolution untuk memperoleh URL kanonik, mengekstrak `shop_id`, `item_id`, serta nama produk dari URL slug / HTML OpenGraph meta tag jika tersedia, dengan fallback nama bersih dari slug URL).

- **Request Body**:
```json
{
  "shopee_url": "https://s.shopee.co.id/7zX9abc123"
}
```
- **Response (`200 OK`)**:
```json
{
  "shopee_url": "https://s.shopee.co.id/7zX9abc123",
  "resolved_url": "https://shopee.co.id/Serum-Acne-Whitening-20ml-Niacinamide-i.123456.789012345",
  "shop_id": "123456",
  "item_id": "789012345",
  "product_name": "Serum Acne Whitening 20ml Niacinamide",
  "price": 49000,
  "original_price": 99000,
  "commission_rate": 15.0,
  "thumbnail_url": "https://down-id.img.susercontent.com/..."
}
```
> [!NOTE]
> Jika Shopee memblokir scraping OpenGraph HTML pada IP VPS tertentu (HTTP 403), parser **tidak boleh crash dan tidak boleh melakukan bypass agresif**. Sebagai gantinya, parser tetap mengekstrak `product_name`, `shop_id`, dan `item_id` secara bersih dari URL hasil redirect (`-i.{shop_id}.{item_id}`), yang sudah cukup akurat bagi Accessibility Service untuk mencari produk di Shopee Video!

### 3.4. `POST /api/v1/ai/generate-caption`
Menghasilkan 1 set caption Shopee Video (`[HOOK]`, body, CTA) + array `#Hashtag` menggunakan `gemini-2.5-flash`.

- **Request Body**:
```json
{
  "keywords": "Serum Jerawat, Ampuh 7 Hari, Hemat 50%",
  "video_title": "Viral Skincare Glowing Fast Review",
  "product_names": ["Serum Acne Whitening 20ml - Niacinamide"],
  "style": "racun shopee, singkat, persuasif, CTA keranjang kuning"
}
```
- **Response (`200 OK`)**:
```json
{
  "caption_text": "Parah sih, serum ini beneran bikin muka kinclong tanpa breakout! 😱✨\n\nBeli sekarang mumpung lagi promo hemat 50% di keranjang bawah! 👇",
  "hashtags": ["#RacunShopee", "#ShopeeHaul", "#SkincareViral", "#SerumJerawat"]
}
```

### 3.5. `POST /api/v1/queue/batch`
Endpoint utama **Batch Ingestion Page**. Memproses banyak baris sekaligus:
1. Parse setiap baris `Link TikTok | LinkShopee1, LinkShopee2` (validasi maksimal 6 produk/baris).
2. Hitung timestamp jadwal untuk tiap item menggunakan **`StratifiedRandomScheduler`** sesuai `daily_quota`, `window_start`, dan `window_end` yang dipilih user.
3. Secara paralel terbatas (async semaphore = 3 agar tidak kena rate limit):
   - Fetch & FFmpeg proses video TikTok.
   - Resolve semua link produk Shopee di baris tersebut.
   - Generate **caption & hashtag unik per video** via Gemini AI (berdasarkan nama produk baris itu + `global_keywords` sebagai gaya).
4. Simpan seluruh item ke `batch_jobs`, `queue_items`, dan `queue_item_products` di Supabase.

- **Request Body**:
```json
{
  "raw_lines": [
    "https://vt.tiktok.com/xyz123 | https://s.shopee.co.id/aaa111",
    "https://vt.tiktok.com/xyz456 | https://s.shopee.co.id/bbb222, https://s.shopee.co.id/ccc333"
  ],
  "daily_quota": 5,
  "window_start": "09:00",
  "window_end": "17:00",
  "min_cooldown_minutes": 45,
  "global_keywords": "Promo gajian, wajib checkout keranjang"
}
```
- **Response (`201 Created`)**:
```json
{
  "batch_id": "b1a2c3d4-...",
  "created_count": 2,
  "failed_lines": [],
  "scheduled_summary": [
    {
      "queue_item_id": "q1...",
      "video_title": "...",
      "scheduled_at": "2026-11-22T09:37:18+07:00",
      "product_count": 1
    },
    {
      "queue_item_id": "q2...",
      "video_title": "...",
      "scheduled_at": "2026-11-22T11:14:52+07:00",
      "product_count": 2
    }
  ]
}
```

### 3.6. `POST /api/v1/commission/import-csv`
Menerima upload file `.csv` (`multipart/form-data`) laporan komisi resmi yang diunduh user dari Portal Shopee Affiliate, mem-parsing kolom tanggal, klik, pesanan, estimasi komisi, dan top produk, lalu menyimpannya dengan aman ke `commission_snapshots` & `commission_top_products`.

---

## 4. Implementasi Pipeline Anti-Duplikat FFmpeg

Tujuan pipeline ini adalah menghapus seluruh sidik jari TikTok (encoder tag, MD5 hash, metadata stream) tanpa merusak ketajaman visual 1080p:

```python
# app/services/ffmpeg_processor.py
import hashlib
import random
import subprocess
from pathlib import Path

def process_anti_duplicate(input_path: Path, output_path: Path) -> str:
    """
    Membersihkan metadata TikTok dan memberikan mutasi mikro imperceptible
    agar hash SHA-256 dan fingerprint frame 100% unik untuk Shopee Video.
    """
    # Variasi mikro acak yang tidak terlihat mata manusia
    gamma_shift = round(random.uniform(0.99, 1.02), 3)
    contrast_shift = round(random.uniform(0.99, 1.02), 3)
    tempo_audio = round(random.uniform(0.995, 1.005), 4)
    crf_value = random.choice([21, 22, 23])

    vf_filter = (
        f"crop=iw-2:ih-2:1:1,"               # Pangkas 1px pinggir (ubah dimensi matriks halus)
        f"pad=iw+2:ih+2:1:1:color=black,"    # Kembalikan dimensi awal
        f"eq=gamma={gamma_shift}:contrast={contrast_shift}"
    )
    af_filter = f"atempo={tempo_audio}"

    cmd = [
        "ffmpeg", "-y",
        "-i", str(input_path),
        "-map_metadata", "-1",               # Hapus seluruh EXIF, encoder, & tag TikTok
        "-vf", vf_filter,
        "-af", af_filter,
        "-c:v", "libx264",
        "-preset", "fast",
        "-crf", str(crf_value),
        "-pix_fmt", "yuv420p",
        "-c:a", "aac",
        "-b:a", "128k",
        "-movflags", "+faststart",
        str(output_path)
    ]

    subprocess.run(cmd, check=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE)

    # Hitung hash baru sebagai verifikasi keunikan
    sha256 = hashlib.sha256()
    with open(output_path, "rb") as f:
        for chunk in iter(lambda: f.read(65536), b""):
            sha256.update(chunk)
    return sha256.hexdigest()
```

---

## 5. Algoritma Penjadwalan Acak Terstratifikasi (`Stratified Random Windowing`)

Sesuai permintaan khusus pengguna: jika user memilih **5 video/hari** dan menentukan rentang jam **09:00 hingga 17:00 WIB**, jadwal **tidak boleh** berada di jam-jam kaku yang identik tiap hari. Sistem membagi jendela waktu ke dalam `N` *bucket* seimbang, menghindari slot waktu yang sudah lewat atau terisi di hari tersebut, menjaga cooldown minimum (misal ≥ 45 menit), dan mengambil menit + detik acak:

```python
# app/services/scheduler.py
import random
from datetime import datetime, time, timedelta
from zoneinfo import ZoneInfo

WIB = ZoneInfo("Asia/Jakarta")

def generate_random_window_slots(
    item_count: int,
    daily_quota: int,
    window_start: time,
    window_end: time,
    existing_scheduled_timestamps: list[datetime],
    min_cooldown_minutes: int = 45,
    now_wib: datetime | None = None,
) -> list[datetime]:
    """
    Menghasilkan daftar timestamp acak yang tersebar merata di dalam
    rentang [window_start, window_end] per hari, dengan jarak aman anti-bot.
    """
    if now_wib is None:
        now_wib = datetime.now(WIB)

    results: list[datetime] = []
    occupied = sorted(ts.astimezone(WIB) for ts in existing_scheduled_timestamps)
    current_date = now_wib.date()

    # Hitung total menit rentang aktif per hari
    start_dt_sample = datetime.combine(current_date, window_start, tzinfo=WIB)
    end_dt_sample = datetime.combine(current_date, window_end, tzinfo=WIB)
    if end_dt_sample <= start_dt_sample:
        raise ValueError("Jam selesai harus lebih besar dari jam mulai.")

    total_window_minutes = int((end_dt_sample - start_dt_sample).total_seconds() // 60)
    bucket_minutes = total_window_minutes / daily_quota

    # Pastikan jitter padding tidak melanggar batas bucket
    safe_margin = min(10, int(bucket_minutes * 0.15))

    while len(results) < item_count:
        day_start = datetime.combine(current_date, window_start, tzinfo=WIB)

        for bucket_idx in range(daily_quota):
            if len(results) >= item_count:
                break

            b_start_min = int(bucket_idx * bucket_minutes) + safe_margin
            b_end_min = int((bucket_idx + 1) * bucket_minutes) - safe_margin
            if b_end_min <= b_start_min:
                b_start_min = int(bucket_idx * bucket_minutes)
                b_end_min = int((bucket_idx + 1) * bucket_minutes)

            # Coba maksimal 15 kali dalam bucket untuk cari slot bebas cooldown
            chosen_slot = None
            for _ in range(15):
                rand_min = random.randint(b_start_min, max(b_start_min, b_end_min))
                rand_sec = random.randint(5, 54)
                candidate = day_start + timedelta(minutes=rand_min, seconds=rand_sec)

                # Lewati jika waktu sudah berlalu (+ buffer 10 menit untuk download)
                if candidate <= now_wib + timedelta(minutes=10):
                    continue

                # Cek cooldown terhadap semua slot yang sudah terisi
                conflict = any(
                    abs((candidate - occ).total_seconds()) < (min_cooldown_minutes * 60)
                    for occ in occupied
                )
                if not conflict:
                    chosen_slot = candidate
                    break

            if chosen_slot is not None:
                results.append(chosen_slot)
                occupied.append(chosen_slot)

        # Spill-over ke hari berikutnya jika kuota hari ini penuh atau jam sudah lewat
        current_date += timedelta(days=1)

    return results
```

### Contoh Hasil Penjadwalan Nyata (5 Video/Hari, Rentang 09:00–17:00 WIB):
- **Hari ke-1**: `09:24:18`, `11:08:41`, `12:51:09`, `14:37:52`, `16:19:33`
- **Hari ke-2**: `09:46:11`, `10:53:27`, `13:14:04`, `15:02:49`, `16:42:15`
Pola setiap hari selalu berbeda di menit dan detik yang acak namun tetap berada di rentang `09:00–17:00 WIB` dan terjaga jarak amannya (`>= 45 menit`).

---

## 6. Spesifikasi `Dockerfile` & `docker-compose.yml`

```dockerfile
# backend/Dockerfile
FROM python:3.11-slim

# Install FFmpeg & dependensi sistem
RUN apt-get update && apt-get install -y --no-install-recommends \
    ffmpeg \
    curl \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY app/ ./app/

RUN mkdir -p /tmp/shofast_cache

EXPOSE 8000
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

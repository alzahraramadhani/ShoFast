# 07 — Struktur Proyek & Roadmap Pengembangan Bertahap (ShoFast)

Dokumen ini menjadi peta jalan (*roadmap*) eksekusi koding untuk membangun **ShoFast** secara terstruktur, terverifikasi di setiap fase, dan mudah dikelola baik secara lokal maupun cloud.

---

## 1. Struktur Monorepo yang Direkomendasikan (`c:\ShoFast\`)

```text
c:\ShoFast\
├── AGENTS.md                         # Instruksi utama & aturan anti-bot untuk AI coding agent
├── Docs/                             # Seluruh spesifikasi & referensi visual Figma
│   ├── Design Figma/                 # 5 screenshot referensi UI
│   ├── 01_PRD_AND_FEATURES.md
│   ├── 02_SYSTEM_ARCHITECTURE.md
│   ├── 03_DATABASE_SCHEMA.md
│   ├── 04_BACKEND_API_SPEC.md
│   ├── 05_ANDROID_ACCESSIBILITY_ENGINE.md
│   ├── 06_UI_UX_DESIGN_SYSTEM.md
│   └── 07_DEVELOPMENT_ROADMAP.md
├── backend/                          # Python FastAPI Server + Docker
│   ├── Dockerfile
│   ├── docker-compose.yml
│   ├── requirements.txt
│   ├── .env.example
│   ├── app/
│   └── tests/                        # Pytest unit test (scheduler, ffmpeg, csv parser)
└── android/                          # Native Android Project (Kotlin + Jetpack Compose)
    ├── build.gradle.kts
    ├── settings.gradle.kts
    └── app/
        └── src/main/
            ├── AndroidManifest.xml
            ├── res/
            └── java/com/shofast/app/
                ├── data/             # Supabase repo, FastAPI client, DataStore, MediaStore helper
                ├── di/               # Hilt Modules
                ├── domain/           # Models & UseCases
                ├── service/          # ShoFastAccessibilityService, StateMachine, Overlay, AlarmReceiver
                └── ui/               # Theme, Navigation, Home, Studio, Batch, Queue, Commission
```

---

## 2. Urutan Fase Coding & Kriteria Verifikasi

### Fase 1: Setup Database Supabase & Infrastruktur Backend
1. Buat proyek di Supabase, jalankan seluruh skrip SQL pada [`Docs/03_DATABASE_SCHEMA.md`](file:///c:/ShoFast/Docs/03_DATABASE_SCHEMA.md), dan simpan URL serta `service_role` / `anon` key.
2. Inisialisasi folder `backend/` lengkap dengan `requirements.txt` (`fastapi`, `uvicorn`, `yt-dlp`, `google-genai`, `supabase`, `httpx`, `beautifulsoup4`, `python-multipart`, `pytest`) dan `Dockerfile`.
3. **Verifikasi Fase 1**:
   - Container/server FastAPI berjalan di port `8000` dan `GET /health` mengembalikan `{"status": "ok"}`.
   - Koneksi ke Supabase berhasil diuji.

### Fase 2: Layanan Inti Backend (Media, AI, Shopee Parser, Scheduler, CSV)
1. Implementasi `tiktok_downloader.py` + `ffmpeg_processor.py` (uji unduh link `vt.tiktok.com` dan verifikasi metadata asli terhapus via `ffprobe` serta hash SHA-256 selalu unik).
2. Implementasi `shopee_parser.py` (aman tanpa scraping private endpoint; resolusi shortlink -> extract `shop_id`, `item_id`, `product_name`).
3. Implementasi `ai_caption.py` dengan `gemini-2.5-flash` (uji output format JSON ketat agar `caption_text` dan `hashtags` rapi).
4. Implementasi `scheduler.py` (*Stratified Random Windowing*) dan `csv_importer.py`.
5. **Verifikasi Fase 2**:
   - Jalankan `pytest backend/tests/` untuk memverifikasi algoritma penjadwalan acak (memastikan seluruh slot berada dalam `09:00–17:00 WIB`, menjaga `min_cooldown_minutes >= 45`, dan spillover multi-hari berfungsi benar).
   - Uji seluruh endpoint melalui Swagger UI (`/docs`) atau skrip uji API.

### Fase 3: Fondasi Proyek Android & Design System UI (5 Layar)
1. Buat proyek Android Jetpack Compose + Hilt + `supabase-kt` + Ktor + WorkManager.
2. Bangun `Theme`, `Color`, `Type`, `ShoFastTopBar`, dan `FloatingBottomNavBar` sesuai [`Docs/06_UI_UX_DESIGN_SYSTEM.md`](file:///c:/ShoFast/Docs/06_UI_UX_DESIGN_SYSTEM.md).
3. Implementasi 5 layar utama (`HomeScreen`, `StudioScreen`, `BatchScreen`, `QueueScreen`, `CommissionScreen`) terhubung dengan ViewModel dan data nyata dari Supabase & FastAPI, plus layar/dialog Pengaturan URL Server Backend.
4. **Verifikasi Fase 3**:
   - `./gradlew assembleDebug` sukses tanpa error.
   - Uji alur Studio (Fetch TikTok -> Generate AI Caption -> Tambah 1–6 Produk -> Simpan ke Antrean) dan alur Batch Ingestion (input multi-baris -> jadwal acak terisi di Supabase).

### Fase 4: Pre-Downloader Video & MediaStore Gallery Sync
1. Implementasi `VideoPreDownloadWorker` (`WorkManager`) untuk mengunduh file MP4 dari FastAPI ke `Movies/ShoFast/`.
2. Implementasi `MediaStoreHelper` untuk menyimpan video dan memicu `MediaScannerConnection.scanFile()` tepat sebelum eksekusi agar video pasti muncul di urutan #1 Galeri Shopee.
3. Implementasi penghapusan otomatis file lokal saat status menjadi `completed`.
4. **Verifikasi Fase 4**:
   - File terunduh ke `Movies/ShoFast/` pada perangkat uji (atau emulator), muncul paling atas saat membuka aplikasi Galeri sistem, dan terhapus saat fungsi cleanup dipanggil.

### Fase 5: Mesin Otomasi `ShoFastAccessibilityService`, Overlay & WakeLock
1. Bangun `AutopilotForegroundService`, `OverlayCountdownManager` (10 detik countdown + tombol Tunda 5 Menit/Batal + Mini HUD), dan `ExactAlarmReceiver`.
2. Bangun `ShoFastAccessibilityService` dan State Machine (`STATE_1` sampai `STATE_7`) lengkap dengan `Humanizer` (jitter delay & gesture offset) dan Watchdog Timeout.
3. Implementasi Auto-Retry maksimal 2x (jeda 5 menit + jitter) dan deteksi protektif terhadap kata kunci Captcha/Login.
4. **Verifikasi Fase 5 (Uji Lapangan di HP Asli dengan Akun Shopee)**:
   - Lakukan inspeksi hierarki node aplikasi Shopee (`com.shopee.id`) versi terbaru menggunakan `adb shell uiautomator dump` atau logger internal `ShoFastAccessibilityService` saat pengembangan untuk memastikan seluruh pencocokan teks/view di `STATE_1`–`STATE_7` akurat 100%.
   - Uji skenario **Postingan Langsung Sekarang** dengan 1 produk, kemudian uji skenario **Batch 3 produk keranjang sekaligus** hingga video terunggah dan file lokal terhapus otomatis.

### Fase 6: Pemolesan Akhir, Import CSV & Uji Tahan Banting
1. Uji fitur `Import CSV Shopee` di Halaman Komisi dengan sampel CSV laporan Shopee Affiliate Indonesia.
2. Uji skenario layar mati (`WakeLock`), skenario klik `Tunda 5 Menit` pada Overlay, dan skenario kegagalan jaringan (verifikasi Auto-Retry & pencatatan di tab `Gagal`).
3. Pastikan dokumentasi dan konfigurasi produksi (`docker-compose.yml`, release APK keystore) rapi dan siap dipakai harian.

# AGENTS.md — Panduan Utama Proyek ShoFast

File ini adalah referensi konteks utama bagi setiap AI Coding Agent atau developer yang mengerjakan repositori **ShoFast** (`c:\ShoFast`).

---

## 1. Deskripsi Proyek
**ShoFast** adalah sistem otomasi **Shopee Affiliate Video** untuk penggunaan pribadi/internal yang mengunduh video TikTok tanpa watermark, membersihkan sidik jari media (anti-duplikat), menghasilkan caption AI unik, menjadwalkan waktu posting acak terstratifikasi, dan mengunggah video + menyematkan 1–6 keranjang produk affiliate secara otomatis ke aplikasi Shopee Android (`com.shopee.id`).

---

## 2. Arsitektur & Tech Stack (Sudah Final — Jangan Ubah Tanpa Persetujuan)
- **Mobile App & Executor (`android/`)**: Native Android **Kotlin + Jetpack Compose** (Material 3, Hilt, Coroutines, `supabase-kt`, Ktor, WorkManager, AlarmManager, MediaStore, `AccessibilityService`, `SYSTEM_ALERT_WINDOW` Overlay, `PowerManager.WakeLock`).
- **Backend Server (`backend/`)**: **Python 3.11+ (FastAPI)** + **yt-dlp** + **FFmpeg** + **Google Gemini API (`gemini-2.5-flash`)**, dikemas dengan **Docker** agar fleksibel dijalankan di PC Windows lokal maupun VPS Linux 24 jam.
- **Database & Realtime**: **Supabase** (PostgreSQL + Auth single-user/internal + Realtime subscription).
- **Video Storage Lifecycle**: Tidak menggunakan Cloud Object Storage permanen. Video hasil FFmpeg disimpan sementara di cache server (`/tmp/shofast_cache`), diunduh HP ke `Movies/ShoFast/`, dan **otomatis dihapus** segera setelah upload ke Shopee Video berhasil.

---

## 3. Aturan Ketat Keamanan Anti-Bot (CRITICAL)
1. **JANGAN PERNAH** mengimplementasikan scraping atau pemanggilan Private/Internal API Shopee menggunakan cookie/session WebView untuk upload video maupun sinkronisasi komisi. Hal ini melanggar keamanan Shopee (`af-ac-enc-dat` / `x-sap-ri`) dan membahayakan akun pengguna.
2. **JANGAN PERNAH** menggunakan `AccessibilityService` untuk membaca/men-scrape halaman Laporan Komisi Affiliate di aplikasi Shopee. Halaman Komisi di ShoFast hanya membaca file **Import CSV Resmi Shopee Affiliate** dan **Statistik Produktivitas Posting** lokal/Supabase.
3. **Jadwal Penuploadan Wajib Acak**: Selalu gunakan algoritma *Stratified Random Windowing* (`app/services/scheduler.py`) di dalam rentang jam pilihan user (misal `N` video/hari antara `09:00–17:00 WIB` dengan cooldown minimum 45 menit) — jangan pernah menjadwalkan di jam dan menit statis yang sama tiap hari.
4. **Humanized Automation**: Setiap aksi pada `ShoFastAccessibilityService` wajib melewati `Humanizer.humanDelay()` (1.8–4.2 detik acak) dan ketukan koordinat ber-jitter (`±12px`). Jika menemukan layar Captcha atau verifikasi login di Shopee, **hentikan otomasi segera** dan beri tahu pengguna.

---

## 4. Peta Dokumentasi Spesifikasi (`Docs/`)
Sebelum menulis kode pada modul mana pun, baca dokumen spesifikasi terkait di bawah ini:

1. **Desain Visual Referensi (Figma)**: [`Docs/Design Figma/`](file:///c:/ShoFast/Docs/Design%20Figma) (`Home Page.jpg`, `Studio Page.jpg`, `Batch Page.jpg`, `Antrean Page.jpg`, `Komisi Page.jpg`)
2. **Kebutuhan Produk & 5 Halaman**: [`Docs/01_PRD_AND_FEATURES.md`](file:///c:/ShoFast/Docs/01_PRD_AND_FEATURES.md)
3. **Arsitektur Sistem & Siklus File**: [`Docs/02_SYSTEM_ARCHITECTURE.md`](file:///c:/ShoFast/Docs/02_SYSTEM_ARCHITECTURE.md)
4. **Skema Database & SQL DDL Supabase**: [`Docs/03_DATABASE_SCHEMA.md`](file:///c:/ShoFast/Docs/03_DATABASE_SCHEMA.md)
5. **Spesifikasi Backend API, FFmpeg & Scheduler**: [`Docs/04_BACKEND_API_SPEC.md`](file:///c:/ShoFast/Docs/04_BACKEND_API_SPEC.md)
6. **Mesin Accessibility Service Android**: [`Docs/05_ANDROID_ACCESSIBILITY_ENGINE.md`](file:///c:/ShoFast/Docs/05_ANDROID_ACCESSIBILITY_ENGINE.md)
7. **Design System & Komponen Jetpack Compose**: [`Docs/06_UI_UX_DESIGN_SYSTEM.md`](file:///c:/ShoFast/Docs/06_UI_UX_DESIGN_SYSTEM.md)
8. **Struktur Monorepo & Roadmap Coding**: [`Docs/07_DEVELOPMENT_ROADMAP.md`](file:///c:/ShoFast/Docs/07_DEVELOPMENT_ROADMAP.md)

---

## 5. Aturan Wajib Git, Version Control & Auto-Push (CRITICAL)

Repositori GitHub resmi proyek ini berada di: `https://github.com/alzahraramadhani/ShoFast` (branch utama: `main`). Setiap AI Coding Agent **WAJIB** mematuhi protokol berikut di setiap tugas:

### 5.1. Wajib Auto-Commit & Direct Push ke `main` di Setiap Perubahan
Setiap kali selesai membuat, mengubah, atau memperbaiki kode/konfigurasi/dokumentasi (dan setelah verifikasi/test terkait lolos):
1. Periksa status & diff staged dengan `git status` dan pastikan **tidak ada** file terlarang (`.env`, API key, `.jks`/`.keystore`, file video `.mp4`, atau `.csv` laporan komisi pribadi).
2. Stage seluruh perubahan relevan (`git add ...`).
3. Buat commit dengan format **Conventional Commits** yang jelas (lihat poin 5.2).
4. **WAJIB langsung push ke remote**:
   ```bash
   git push origin main
   ```
   *(Jangan biarkan perubahan berhenti hanya di lokal tanpa ter-push, kecuali user secara eksplisit meminta bekerja di branch fitur sementara).*

### 5.2. Format Commit Standar Industri (Conventional Commits)
Workflow `.github/workflows/repo-guard.yml` memvalidasi setiap commit secara otomatis. Wajib gunakan pola:
```text
<type>(<scope>): <subject>
```
- **`<type>`**: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert`
- **`<scope>`**: `backend`, `ffmpeg`, `scheduler`, `ai`, `shopee`, `tiktok`, `android`, `accessibility`, `ui`, `overlay`, `mediastore`, `supabase`, `db`, `ci`, `repo`, `spec`, `readme`
- **`<subject>`**: Kalimat imperatif bahasa Inggris ringkas, huruf kecil di awal, tanpa titik di akhir (minimal 3 karakter).

**Contoh Commit Valid:**
- `feat(scheduler): add stratified random windowing with 45m cooldown`
- `feat(accessibility): implement multi-product search loop in shopee video`
- `fix(ffmpeg): strip tiktok metadata and apply imperceptible hash mutation`
- `ci(workflow): update android build matrix for gradle cache`
- `docs(readme): document local tailscale backend setup`

### 5.3. Kolaborasi Developer & Branching (GitHub Flow)
- Untuk kolaborasi paralel oleh developer (**Al Zahra Ramadhani** & **Dalvin Widya Purnama**), gunakan konvensi branch `feat/*`, `fix/*`, `docs/*`, `ci/*`, `refactor/*`, `style/*` dan Pull Request template di [`.github/PULL_REQUEST_TEMPLATE.md`](file:///c:/ShoFast/.github/PULL_REQUEST_TEMPLATE.md).
- Rilis versi stabil menggunakan tag SemVer (`vX.Y.Z`) untuk memicu `.github/workflows/release.yml`.


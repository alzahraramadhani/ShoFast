## Ringkasan Perubahan
<!-- Jelaskan perubahan yang dilakukan secara singkat dan jelas -->

## Jenis Perubahan
- [ ] `feat`: Fitur baru
- [ ] `fix`: Perbaikan bug
- [ ] `docs`: Dokumentasi
- [ ] `refactor` / `style`: Refactoring atau penyesuaian UI Figma
- [ ] `ci` / `build` / `chore`: CI/CD, dependensi, atau konfigurasi repo

## Cakupan Modul
- [ ] `backend/` (Python FastAPI / FFmpeg / Gemini / Scheduler)
- [ ] `android/` (Kotlin / Jetpack Compose / AccessibilityService)
- [ ] `Docs/` (Spesifikasi & Skema Supabase)
- [ ] `.github/` (Workflows & Template)

## Daftar Periksa Kepatuhan & Pengujian
- [ ] Tidak ada file rahasia (`.env`, API key, `.jks`, `.mp4`, `.csv` pribadi) yang ikut ter-commit.
- [ ] Mematuhi aturan Anti-Bot di `AGENTS.md` (tidak ada private API scraping Shopee & jadwal menggunakan random windowing).
- [ ] Sudah diuji secara lokal (`pytest` / `./gradlew testDebugUnitTest` / verifikasi manual).

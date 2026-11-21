# Panduan Kontribusi & Version Control ShoFast

Terima kasih telah berkontribusi pada pengembangan **ShoFast**! Dokumen ini menjelaskan standar alur kerja Git, format commit industri (**Conventional Commits**), rilis SemVer, dan kepatuhan anti-bot yang wajib dipatuhi.

---

## 1. Strategi Branching (GitHub Flow)

Repositori ini menggunakan **GitHub Flow**:
- **`main`**: Branch utama yang selalu stabil, bersih, dan siap dijalankan/dirilis.
- **Branch Fitur / Perbaikan (untuk kolaborasi manual atau Pull Request)**:
  - `feat/<nama-fitur>` — Fitur baru (misal: `feat/ffmpeg-anti-duplicate`, `feat/accessibility-cart-tagger`)
  - `fix/<nama-bug>` — Perbaikan bug (misal: `fix/shopee-gallery-picker`)
  - `docs/<topik>` — Pembaruan dokumentasi (misal: `docs/api-spec-update`)
  - `ci/<topik>` — Konfigurasi CI/CD & otomasi repo (misal: `ci/android-gradle-cache`)
  - `refactor/<topik>` — Refactoring kode tanpa mengubah perilaku fitur
  - `style/<topik>` — Perbaikan UI/Compose layout atau formatting kode

> **Catatan**: Untuk AI coding agent pada proyek ini, aturan *auto-commit & direct push* ke `main` diatur khusus di [`AGENTS.md`](AGENTS.md). Untuk kolaborasi paralel oleh developer manusia (**Al Zahra Ramadhani** & **Dalvin Widya Purnama**), gunakan branch fitur + Pull Request jika mengerjakan modul yang bersinggungan.

---

## 2. Standar Pesan Commit (Conventional Commits v1.0.0)

Setiap commit **wajib** mengikuti format berikut (divalidasi otomatis oleh `.github/workflows/repo-guard.yml`):

```text
<type>(<scope>): <subject>

[optional body]

[optional footer(s)]
```

### Daftar `<type>` yang Valid:
- `feat` : Penambahan fitur atau fungsionalitas baru
- `fix` : Perbaikan bug
- `docs` : Perubahan dokumentasi (`README.md`, `AGENTS.md`, `Docs/*`)
- `style` : Perubahan visual UI/Compose atau formatting kode tanpa ubah logika
- `refactor` : Restrukturisasi kode internal tanpa tambah fitur/ubah bug
- `perf` : Peningkatan performa (misal optimasi FFmpeg atau Coroutines)
- `test` : Penambahan atau pembaruan unit/integration test
- `build` : Perubahan sistem build, dependensi (`requirements.txt`, `build.gradle.kts`, `Dockerfile`)
- `ci` : Perubahan konfigurasi CI/CD GitHub Actions (`.github/workflows/*`)
- `chore` : Tugas pemeliharaan repo (`.gitignore`, lisensi, cleanup)
- `revert` : Mengembalikan commit sebelumnya

### Daftar `<scope>` yang Direkomendasikan:
- `backend`, `ffmpeg`, `scheduler`, `ai`, `shopee`, `tiktok`
- `android`, `accessibility`, `ui`, `overlay`, `mediastore`
- `supabase`, `db`, `ci`, `repo`, `spec`, `readme`

### Contoh Commit yang Benar:
```bash
git commit -m "feat(scheduler): implement stratified random windowing algorithm"
git commit -m "feat(accessibility): add multi-product search loop up to 6 items"
git commit -m "fix(ffmpeg): strip tiktok encoder metadata tags properly"
git commit -m "docs(readme): update docker deployment instructions"
```

---

## 3. Rilis & Versioning (Semantic Versioning)

Rilis resmi ditandai dengan Git Tag format `vMAJOR.MINOR.PATCH`:
```bash
git tag -a v0.1.0 -m "chore(release): initial MVP release"
git push origin v0.1.0
```
Push tag `v*.*.*` akan memicu workflow `.github/workflows/release.yml` secara otomatis untuk membangun image Docker backend ke GHCR, meng-compile APK Android, dan menerbitkan GitHub Release.

---

## 4. Aturan Ketat Sebelum Commit & PR
1. **DILARANG** menyertakan `.env`, API Key (Gemini/Supabase), keystore (`.jks`), file `.mp4` uji coba, atau `.csv` laporan komisi pribadi.
2. **Wajib Mematuhi Aturan Anti-Bot**: Tidak boleh ada kode scraping private API Shopee atau scraping halaman komisi via Accessibility Service (lihat `AGENTS.md` Bagian 3).
3. Pastikan test lokal (`pytest` untuk `backend/` atau `./gradlew testDebugUnitTest` untuk `android/`) lolos sebelum melakukan push.

# 06 — Design System & Spesifikasi UI/UX Jetpack Compose (ShoFast)

Dokumen ini memetakan seluruh token desain visual (warna, tipografi, bentuk, bayangan) dan hierarki komponen Jetpack Compose yang diekstrak secara akurat dari 5 desain Figma di [`Docs/Design Figma/`](file:///c:/ShoFast/Docs/Design%20Figma), beserta penyesuaian fitur keamanan anti-bot yang telah disepakati.

---

## 1. Design Tokens (Warna, Tipografi & Bentuk)

### 1.1. Palet Warna (`Color.kt`)
Diekstrak langsung dari tampilan visual 5 halaman Figma:

| Token | Kode Hex | Penggunaan |
|---|---|---|
| `ShoFastRed` | `#D92D20` | Tombol CTA utama (`Fetch Video`, `SIMPAN KE ANTREAN`, `PROSES & BUAT ANTREAN MASAL`), badge merah, aksen chart aktif. |
| `ShoFastRedSoft` | `#FFF1F2` | Latar tag hashtag (`#RacunShopee`), badge persentase kenaikan (`↑ +14%`), badge tombol `Reset Form`. |
| `ShoFastRedMuted` | `#FEE2E2` | Batang chart tidak aktif, latar icon container merah lembut. |
| `BackgroundCream` | `#F8F7F6` | Latar belakang kanvas utama aplikasi (putih hangat/krem sangat lembut). |
| `SurfaceWhite` | `#FFFFFF` | Latar kartu utama (`Ringkasan Hari Ini`, `Jadwal Berikutnya`, form step cards). |
| `InputBackground` | `#F5F4F2` | Latar kolom input teks, kotak editor kode link masal, dan preview caption. |
| `NavDarkPill` | `#1A1A1A` | Kapsul navigasi melayang di bawah (*Floating Bottom Nav*) & tombol sekunder gelap (`Generate Caption`, `Jalankan Manual`). |
| `StatusGreen` | `#16A34A` | Teks status `Aktif`, `Siap Unduh`, ikon centang aktivitas berhasil. |
| `StatusGreenBg` | `#DCFCE7` | Latar pill status hijau (`Aktif`, `• Siap Tayang`, jumlah tab `Berhasil`). |
| `TextPrimary` | `#111827` | Judul halaman, nominal angka komisi besar, judul video. |
| `TextSecondary` | `#6B7280` | Subjudul, label form, timestamp aktivitas, metadata ukuran video. |
| `BorderSubtle` | `#E5E7EB` | Garis tepi tipis (`1.dp`) pada kartu putih & tombol sekunder. |

### 1.2. Bentuk & Elevasi (`Shape.kt`)
- **Card Utama**: `RoundedCornerShape(20.dp)` dengan border `1.dp` (`#F1EFEA`) dan elevasi halus `2.dp`.
- **Input Field & Kotak Preview**: `RoundedCornerShape(14.dp)`.
- **Tombol CTA Utama**: Tinggi `52.dp`, `RoundedCornerShape(14.dp)`, teks uppercase tebal atau title case sesuai Figma.
- **Badge / Pill / Filter Tab**: `RoundedCornerShape(50)` (Kapsul penuh).
- **Floating Bottom Nav Bar**: Mengambang dengan margin bawah `16.dp` dan horizontal `24.dp`, tinggi `64.dp`, sudut `RoundedCornerShape(32.dp)`, latar `#1A1A1A`.

### 1.3. Tipografi (`Type.kt`)
Menggunakan font sans-serif modern yang bersih (**Plus Jakarta Sans** atau **Inter** yang dibundel di `res/font/`):
- **Brand Title (`ShoFast`)**: `20.sp`, Bold (`700`).
- **Page Header**: `22.sp`, Bold (`700`), tracking `-0.3.sp`.
- **Big Metric (`Rp 3.450.000` / `Rp 185.500`)**: `30.sp` – `32.sp`, ExtraBold (`800`), `tabular-nums`.
- **Section Title**: `16.sp`, SemiBold (`600`).
- **Card Title**: `14.sp` – `15.sp`, SemiBold (`600`).
- **Body / Caption**: `13.sp` – `14.sp`, Regular (`400`).
- **Micro Label / Badge**: `11.sp`, Medium/SemiBold (`500`–`600`).

---

## 2. Pemetaan Komponen Jetpack Compose per Layar

### 2.1. `ShoFastTopBar` & `FloatingBottomNavBar` (Global Shell)
- **`ShoFastTopBar`**: Logo `ShoFast` + ikon petir merah-putih di kiri; ikon lonceng notifikasi (dengan titik indikator merah jika ada error baru) + avatar lingkaran (bisa diklik untuk membuka dialog **Pengaturan Server FastAPI & Preferensi Default**) di kanan.
- **`FloatingBottomNavBar`**: 4 item (`Home`, `Studio`, `Antrean` + red badge counter dinamis, `Komisi`). Disembunyikan saat berada di sub-halaman `BatchScreen`.

### 2.2. `HomeScreen` ([`Home Page.jpg`](file:///c:/ShoFast/Docs/Design%20Figma/Home%20Page.jpg))
1. `AccessibilityStatusCard`: Mengamati state `AccessibilityService` secara real-time via `LifecycleEventEffect(ON_RESUME)`.
2. `TodaySummaryCard`:
   - Kartu atas: Menampilkan `Estimasi Komisi` (jika ada data CSV) atau `Video Terposting Hari Ini` (tab switcher kecil atau mode otomatis) + mini bar chart jam (`10:00` .. `Nanti`).
   - 2 kartu bawah: `TOTAL KLIK` & `PESANAN SAH` (atau metrik produktivitas posting bila belum impor CSV).
3. `NextScheduleCard`: Menampilkan antrean `scheduled`/`ready` terdekat dengan hitung mundur menit (`Dalam 45 Menit`), tombol hitam `▷ Jalankan Manual` dan tombol putih `✎ Edit Post`.
4. `QuickActionsRow`: Kartu gradien merah `Buat Post Baru` (ke Studio) & kartu putih `Batch Ingestion` (ke Batch).
5. `RecentActivitySection`: List 2 item terbaru dari `automation_logs` + link `Semua Log`.

### 2.3. `StudioScreen` ([`Studio Page.jpg`](file:///c:/ShoFast/Docs/Design%20Figma/Studio%20Page.jpg))
- Header `Content Studio` + pill `Reset Form`.
- `StepBadge(number = 1..4)` (lingkaran pink lembut dengan angka merah) di setiap kartu langkah:
  1. `TikTokSourceStepCard`: Input URL + tombol paste + CTA `Fetch Video (Tanpa Watermark)` + `CleanHdPreviewCard` (thumbnail durasi, badge `Watermark Removed (Clean HD)`, author, ukuran/resolusi, link `Pratinjau Full`).
  2. `CaptionHashtagStepCard`: Input kata kunci + tombol gelap `✨ Generate Caption & Hashtag` + box preview editable + tombol `Salin` + `FlowRow` chips hashtag + `+ Tambah Tag`.
  3. `AffiliateProductsStepCard`: Daftar produk (1 s/d 6 item) + tombol dashed `+ Tambah Produk Lain` (membuka BottomSheet input URL produk Shopee).
  4. `PostingScheduleStepCard`: Date-Time Picker `WIB` + Switch `Posting Langsung Sekarang`.
- CTA Bawah: `SIMPAN KE ANTREAN POSTING`.

### 2.4. `BatchScreen` ([`Batch Page.jpg`](file:///c:/ShoFast/Docs/Design%20Figma/Batch%20Page.jpg))
- Top bar lokal dengan tombol `←` kembali, judul `Batch Ingestion`, dan `Reset Form`.
- `BulkLinkInputCard`:
  - Badge dinamis `• N Pasang Terdeteksi` (merah lembut jika valid > 0).
  - Chip format `Link TikTok | Link Produk Shopee` (catatan: mendukung multi-produk dipisah koma setelah `|`).
  - Editor teks bernomor baris di sisi kiri (`01`, `02`, `03`...) dengan font monospace (`JetBrains Mono` / `FontFamily.Monospace`).
  - Footer `Bersihkan` & `Isi Sampel Link`.
- `RandomWindowScheduleCard` *(Ditingkatkan sesuai kebutuhan Anti-Bot Acak)*:
  - Header `Jadwal Acak Anti-Bot` + label `WIB Timezone`.
  - **Pilihan Target Harian**: Radio card `3 Video / Hari` (Rekomendasi Aman), `5 Video / Hari`, dan `Kustom` (input angka `1–20 Video / Hari`).
  - **Rentang Jam Upload Acak**: Dua picker berdampingan **`Dari Jam [09:00]`** s/d **`Sampai Jam [17:00 WIB]`**.
  - Info helper: *"Sistem mengacak menit & detik secara merata di dalam rentang jam pilihanmu dengan jarak aman min. 45 menit antar upload."*
- `SmartBatchCaptionCard`:
  - Input kata kunci/gaya bahasa global + tombol `Generate Caption & Hashtag` (untuk melihat contoh preview).
  - Penjelasan visual: *"AI otomatis membuat caption & hashtag unik untuk setiap video sesuai nama produknya."*
- CTA Bawah: `PROSES & BUAT ANTREAN MASAL`.

### 2.5. `QueueScreen` ([`Antrean Page.jpg`](file:///c:/ShoFast/Docs/Design%20Figma/Antrean%20Page.jpg))
- Header `Manajemen Antrean` + `Total N Video`.
- `LazyRow` tab filter pill (`Terjadwal`, `Diproses`, `Berhasil`, `Gagal`) lengkap dengan badge angka berwarna kontras.
- `LazyColumn` kartu video: thumbnail `84.dp x 104.dp` bersudut membulat `14.dp` dengan badge durasi gelap di pojok kanan bawah dan ikon play di kiri atas, judul max 2 baris, ikon jam + waktu WIB, pill status + pill jumlah produk, dan menu titik tiga (`⋮`).
- `ExtendedFloatingActionButton` merah `+ Tambah Video` di kanan bawah (di atas floating bottom nav).

### 2.6. `CommissionScreen` ([`Komisi Page.jpg`](file:///c:/ShoFast/Docs/Design%20Figma/Komisi%20Page.jpg))
- Header `Komisi Affiliate` + tombol aman **`Import CSV Shopee`** (membuka Android `ActivityResultContracts.GetContent("text/csv")` untuk memilih file CSV laporan resmi Shopee Affiliate, tanpa risiko deteksi bot sama sekali).
- Tab periode: `Hari Ini`, `7 Hari`, `30 Hari`, `Kustom`.
- Kartu `Total Estimasi Komisi` + persentase + pesanan & klik.
- Kartu `Grafik Performa Harian` (implementasi Custom Canvas Compose Bar Chart dengan garis putus-putus horizontal, batang membulat penuh `RoundedCornerShape(50)`, dan tooltip gelap mengambang di atas batang yang diketuk).
- `TopProductsSection` (`Produk Paling Menghasilkan`) dengan nomor peringkat (`1` berlatar merah, `2..n` berlatar hitam), thumbnail, pesanan & `CR %`, serta nominal komisi merah di kanan.
- Jika belum ada CSV yang diimpor, tampilkan *Empty State* ramah dengan panduan singkat cara unduh CSV resmi Shopee Affiliate + tombol langsung `Import CSV Sekarang` serta tetap menampilkan **Ringkasan Produktivitas Autopilot** (Video Terposting & Keranjang Tersemat).

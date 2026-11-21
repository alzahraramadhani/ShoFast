# 01 — Product Requirements Document (PRD) & Spesifikasi Fitur ShoFast

Dokumen ini mendefinisikan visi produk, batasan keamanan anti-bot, serta spesifikasi fungsional lengkap dari **ShoFast** berdasarkan 5 layar desain Figma pada folder [`Docs/Design Figma`](file:///c:/ShoFast/Docs/Design%20Figma):
- [`Home Page.jpg`](file:///c:/ShoFast/Docs/Design%20Figma/Home%20Page.jpg)
- [`Studio Page.jpg`](file:///c:/ShoFast/Docs/Design%20Figma/Studio%20Page.jpg)
- [`Batch Page.jpg`](file:///c:/ShoFast/Docs/Design%20Figma/Batch%20Page.jpg)
- [`Antrean Page.jpg`](file:///c:/ShoFast/Docs/Design%20Figma/Antrean%20Page.jpg)
- [`Komisi Page.jpg`](file:///c:/ShoFast/Docs/Design%20Figma/Komisi%20Page.jpg)

---

## 1. Visi & Ringkasan Eksekutif

**ShoFast** adalah sistem otomasi konten **Shopee Affiliate Video** untuk penggunaan **pribadi / internal** (satu perangkat atau beberapa perangkat milik pengguna sendiri). Aplikasi ini mengotomatiskan seluruh alur kerja affiliator dari hulu ke hilir:
1. Mengunduh video TikTok **tanpa watermark** dalam kualitas bersih (Clean HD MP4).
2. Membersihkan metadata & menerapkan mutasi mikro **anti-duplikat** via FFmpeg agar video dianggap 100% original oleh algoritma Shopee Video.
3. Membuat **caption & hashtag persuasif bergaya "racun Shopee"** secara otomatis dengan AI (`gemini-2.5-flash`) yang unik untuk setiap video.
4. Menjadwalkan posting secara **acak terstratifikasi (Anti-Bot Random Windowing)** di dalam rentang jam pilihan pengguna (misal: `5 video/hari` diacak antara `09:00 - 17:00 WIB`).
5. Mengeksekusi upload ke aplikasi resmi **Shopee Android (`com.shopee.id`)** dan menyematkan **1–6 produk keranjang affiliate** secara otomatis menggunakan Android **Accessibility Service** dengan perilaku menyerupai manusia.

---

## 2. Prinsip Utama Keamanan Anti-Bot (Wajib Dipatuhi)

Berdasarkan analisis mendalam terhadap sistem deteksi bot Shopee (`af-ac-enc-dat`, `x-sap-ri`, TLS fingerprinting, dan pelacakan pola waktu):
1. **DILARANG menembak Internal API Shopee secara langsung** menggunakan cookie WebView/session hijacking untuk upload maupun sinkronisasi komisi, karena berisiko tinggi memicu Captcha berulang, logout paksa, hingga pembatasan/banned akun (`F01`/`D01`).
2. **DILARANG melakukan scraping layar halaman Komisi Affiliate Shopee** via Accessibility Service, karena menambah jejak interaksi non-manusia pada WebView sensitif Shopee.
3. **Pengecekan Komisi Real-Time** dilakukan secara manual oleh pengguna langsung di aplikasi Shopee. Sebagai gantinya, fitur **Komisi & Ringkasan** di ShoFast ditenagai secara **100% aman** oleh:
   - **Import File CSV Resmi** dari portal/ekspor laporan Shopee Affiliate.
   - **Statistik Produktivitas Autopilot** (total video tayang, jumlah keranjang tersemat, tingkat keberhasilan posting).
4. **Jadwal Upload Tidak Pernah Kaku/Robotik**: Sistem menggunakan algoritma *Stratified Random Windowing* dengan variasi menit & detik acak serta jeda cooldown minimum antar video.
5. **Humanized Interaction**: Setiap ketukan, pengetikan, dan transisi layar pada Accessibility Service memiliki jeda acak (1.8–4.2 detik) dan variasi koordinat sentuhan (`±12px`).

---

## 3. Spesifikasi Fungsional Per Halaman (5 Layar)

### 3.1. Home Page (`HomeScreen`)
Referensi visual: [`Home Page.jpg`](file:///c:/ShoFast/Docs/Design%20Figma/Home%20Page.jpg)

| Komponen UI | Spesifikasi Fungsional |
|---|---|
| **Header Bar** | Logo ShoFast (ikon petir "VOLT"), judul `ShoFast`, ikon lonceng notifikasi (menampilkan riwayat peringatan kegagalan/error), dan avatar profil pengguna. |
| **Accessibility Service Status Banner** | Menampilkan status real-time apakah `ShoFastAccessibilityService` aktif (`Aktif` hijau) atau mati (`Nonaktif` merah). Tombol `Cek Izin >` membuka dialog/halaman pemeriksaan izin (Accessibility, Overlay `SYSTEM_ALERT_WINDOW`, Exact Alarm, dan Battery Optimization). |
| **Ringkasan Hari Ini** | - Kartu utama menampilkan **Estimasi Komisi** (dari data Import CSV resmi terakhir, atau metrik **Video Terposting Hari Ini** bila belum ada CSV yang diimpor) + grafik batang mini per jam.<br>- Dua kartu ringkasan di bawahnya: **Total Klik** & **Pesanan Sah** (atau **Keranjang Tersemat** & **Tingkat Sukses Autopilot %**). Link `Detail` mengarahkan ke tab Komisi. |
| **Jadwal Berikutnya** | Menampilkan 1 item antrean paling dekat dari status `scheduled` / `ready`:<br>- Pill waktu merah (`20:00 WIB (Dalam 45 Menit)` dihitung dinamis dari `scheduled_at`).<br>- Pill status (`• Siap Tayang` jika video MP4 sudah terunduh di HP, atau `• Mengunduh` jika masih diproses).<br>- Thumbnail, judul video, jumlah produk disematkan, dan nomor urut antrean.<br>- Tombol **`Jalankan Manual`**: Memicu countdown overlay 10 detik dan langsung mengeksekusi upload video ini.<br>- Tombol **`Edit Post`**: Membuka item di Studio Page dalam mode edit. |
| **Aksi Cepat** | - Tombol merah **`Buat Post Baru`**: Navigasi ke tab `Studio`.<br>- Tombol putih **`Batch Ingestion`**: Navigasi ke halaman `Batch`. |
| **Aktivitas Terakhir** | Menampilkan 2–3 log terbaru dari tabel `automation_logs` (misal: *"Kipas Angin Leher Portable — Berhasil diunggah ke Shopee Video • 1 Keranjang Tersemat"*). Link `Semua Log` membuka sheet log lengkap. |
| **Floating Bottom Navigation** | Pill melayang gelap (`#1A1A1A`) dengan 4 menu: `Home`, `Studio`, `Antrean` (badge merah jumlah antrean `Terjadwal`), dan `Komisi`. |

---

### 3.2. Content Studio Page (`StudioScreen`)
Referensi visual: [`Studio Page.jpg`](file:///c:/ShoFast/Docs/Design%20Figma/Studio%20Page.jpg)

Digunakan untuk membuat 1 postingan Shopee Video secara mendetail (single-post mode) atau mengedit post yang sudah dijadwalkan.

1. **Langkah 1 — Sumber Video TikTok**:
   - Input teks URL TikTok (mendukung `vt.tiktok.com/...`, `vm.tiktok.com/...`, dan URL penuh `tiktok.com/@user/video/...`) lengkap dengan tombol *Paste from Clipboard*.
   - Tombol **`Fetch Video (Tanpa Watermark)`**: Memanggil endpoint backend `POST /api/v1/tiktok/fetch`.
   - Setelah sukses, tampilkan **Card Pratinjau Video**: thumbnail dengan tombol *Play* (membuka `Pratinjau Full` video player internal), badge `Watermark Removed (Clean HD)`, judul/deskripsi asli TikTok, `@author`, ukuran file (MB), resolusi (MP4 1080p), dan indikator `✓ Siap Unduh`.
2. **Langkah 2 — Caption & Hashtag**:
   - Input `Masukkan Kata Kunci` (opsional, sebagai konteks gaya/keunggulan produk, misal: *Serum Jerawat, Ampuh 7 Hari, Hemat 50%*).
   - Tombol **`Generate Caption & Hashtag`**: Memanggil endpoint `POST /api/v1/ai/generate-caption` dengan konteks judul TikTok + daftar produk Shopee yang sudah ditambahkan.
   - Kotak pratinjau caption yang dapat diedit langsung oleh pengguna (`[HOOK] ...`, body, CTA) + tombol `Salin` + chip hashtag yang bisa ditambah (`+ Tambah Tag`) atau dihapus.
3. **Langkah 3 — Produk Affiliate (1 s/d 6 Produk)**:
   - Mendukung penambahan **1 hingga 6 tautan produk Shopee** per video (batas aman & optimal tampilan keranjang Shopee Video).
   - Tombol `+ Tambah Produk Lain` membuka modal input link Shopee (`https://shopee.co.id/...` atau `https://s.shopee.co.id/...`).
   - Backend mengekstrak metadata produk (nama, harga, harga coret, estimasi komisi jika tersedia, thumbnail) dan menampilkannya dalam daftar kartu produk dengan tombol hapus (ikon tempat sampah).
4. **Langkah 4 — Waktu Posting**:
   - Dropdown/Picker tanggal & jam posting (dengan zona waktu `WIB`).
   - Toggle **`Posting Langsung Sekarang`**: Jika aktif, video langsung didownload ke HP dan memicu proses upload sesaat setelah tombol simpan ditekan (mengabaikan penjadwalan antrean).
5. **Tombol Utama**: **`SIMPAN KE ANTREAN POSTING`** (atau `POSTING SEKARANG` jika toggle aktif). Tombol `Reset Form` di pojok kanan atas membersihkan seluruh input.

---

### 3.3. Batch Ingestion Page (`BatchScreen`)
Referensi visual: [`Batch Page.jpg`](file:///c:/ShoFast/Docs/Design%20Figma/Batch%20Page.jpg)

Digunakan untuk mengimpor puluhan pasang video & produk secara masal dalam sekali kerja, dilengkapi penjadwalan acak anti-bot.

1. **Bagian 1 — Input Link Masal**:
   - Editor teks bernomor baris (`01`, `02`, `03`, ...) dengan format baku per baris:
     `Link TikTok | Link Produk Shopee 1, Link Produk Shopee 2` (mendukung 1–6 link Shopee dipisah koma `,` setelah pemisah pipe `|`).
   - Badge real-time di kanan atas menghitung baris valid: misal `• 3 Pasang Terdeteksi` (atau badge error jika format baris tidak valid).
   - Tombol bantuan `Bersihkan` dan `Isi Sampel Link`.
2. **Bagian 2 — Penjadwalan Acak Anti-Bot (`Rentang Jam & Kuota Harian`)**:
   - Berbeda dari interval kaku di desain awal, bagian ini mengimplementasikan **Algoritma Stratified Random Windowing** agar jadwal posting tidak pernah terdeteksi sebagai pola bot:
     - **Pilihan Target Video per Hari**:
       - `3 Video / Hari` (Rekomendasi Aman)
       - `5 Video / Hari` (Mode Aktif)
       - `Kustom`: Pengguna bebas memasukkan angka `N Video / Hari` (misal: 2, 4, 7, 10 video/hari).
     - **Pemilih Rentang Jam Upload (`WIB Timezone`)**:
       - `Jam Mulai`: misal `09:00 WIB`
       - `Jam Selesai`: misal `17:00 WIB`
     - **Pratinjau Distribusi Acak**: Sistem otomatis membagi rentang waktu `[Jam Mulai .. Jam Selesai]` ke dalam `N` jendela waktu dan mengambil menit & detik **secara acak** dengan jeda aman antar posting (cooldown minimum). Jika batch berisi 15 video dengan target 5 video/hari, otomatis dibagi ke 3 hari berturut-turut dengan jam acak berbeda setiap hari.
3. **Bagian 3 — Smart Per-Video AI Caption & Hashtag**:
   - Input `Masukkan Kata Kunci / Gaya Bahasa` (opsional, berlaku sebagai arahan gaya/CTA global untuk batch tersebut).
   - Saat proses batch dijalankan, **FastAPI + Gemini AI** membuatkan **caption & hashtag yang UNIK untuk masing-masing baris video** berdasarkan *Nama Produk Shopee* dan *Judul Asli TikTok* baris tersebut, sehingga tidak terjadi duplikasi teks caption antarpengunggahan.
   - Kotak `Pratinjau Caption Shopee Video` menampilkan contoh hasil generasi untuk baris pertama sebagai preview cepat.
4. **Tombol Utama**: **`PROSES & BUAT ANTREAN MASAL`**. Menampilkan progress indikator parsing -> resolusi produk -> generate caption unik -> download/persiapan antrean.

---

### 3.4. Manajemen Antrean Page (`QueueScreen`)
Referensi visual: [`Antrean Page.jpg`](file:///c:/ShoFast/Docs/Design%20Figma/Antrean%20Page.jpg)

1. **Header**: Judul `Manajemen Antrean` dan total penghitung `Total N Video`.
2. **Tab Filter Horizontal** dengan badge penghitung:
   - `Terjadwal` (mencakup status `scheduled`, `downloading`, `ready`)
   - `Diproses` (status `processing` — sedang dieksekusi oleh Accessibility Service)
   - `Berhasil` (status `completed` — sudah terunggah & file lokal sudah dihapus)
   - `Gagal` (status `failed` — gagal setelah auto-retry maksimal 2x)
3. **Kartu Video Antrean**:
   - Thumbnail video + durasi di pojok kanan bawah (`0:45`).
   - Judul video, jadwal tayang (`Hari Ini, 20:00 WIB` / `Besok, 11:24 WIB`), badge status (`• Siap Tayang`, `• Terjadwal`, `• Menunggu Retry (1/2)`, `• Gagal`), serta badge jumlah produk (`2 Produk`).
   - Menu titik tiga (`⋮`):
     - `Jalankan Sekarang` (untuk item `Terjadwal`)
     - `Edit di Studio`
     - `Coba Ulang (Retry)` (untuk item `Gagal`)
     - `Hapus dari Antrean` (sekaligus menghapus file MP4 lokal jika ada)
4. **Floating Action Button**: `+ Tambah Video` merah di kanan bawah (mengarah ke `Studio` atau pilihan `Studio / Batch`).

---

### 3.5. Komisi & Performa Page (`CommissionScreen`)
Referensi visual: [`Komisi Page.jpg`](file:///c:/ShoFast/Docs/Design%20Figma/Komisi%20Page.jpg)

Sesuai kebijakan **100% Anti-Bot Safe**:
- Tombol pojok kanan atas pada desain Figma (`Sinkron Shopee`) diganti fungsinya menjadi **`Import CSV Shopee`** untuk mengimpor file laporan `.csv` resmi yang diunduh pengguna dari portal Shopee Affiliate (tanpa login/scraping bot ke Shopee).
- Dilengkapi mode tampilan tab atau sub-ringkasan **Statistik Produktivitas Posting** (data lokal/Supabase murni yang selalu real-time tanpa perlu CSV).
- **Filter Periode**: `Hari Ini`, `7 Hari`, `30 Hari`, `Kustom`.
- **Kartu Ringkasan Pendapatan**: Total Estimasi Komisi (contoh: `Rp 3.450.000`), persentase pertumbuhan (`↗ +22%`), total pesanan sah, dan total klik.
- **Grafik Performa Harian**: Bar chart vertikal dengan tooltip interaktif saat batang tanggal disentuh (`Rp 215.000`).
- **Produk Paling Menghasilkan**: Daftar peringkat produk dengan thumbnail, jumlah pesanan, Conversion Rate (`CR %`), dan nilai komisi.

---

## 4. Alur Eksekusi Autopilot & Penanganan Kegagalan

1. **Pre-Download Terjadwal**: Aplikasi Android men-download file MP4 yang sudah melewati proses FFmpeg dari FastAPI ke folder lokal `Movies/ShoFast/` segera setelah masuk antrean (atau paling lambat 30 menit sebelum `scheduled_at`).
2. **Saat Waktu Jadwal Tiba (Mode Hybrid WakeLock + Overlay)**:
   - `AlarmManager` (`setExactAndAllowWhileIdle`) membangunkan sistem.
   - Jika layar mati/terkunci tanpa PIN, Android menyalakan layar via `PowerManager.WakeLock`.
   - Muncul **Floating Overlay Countdown 10 Detik**: *"ShoFast Autopilot akan mengunggah 'Raket Nyamuk...' dalam 10 dtk"* dengan tombol **`[Tunda 5 Menit]`** dan **`[Batalkan]`**.
   - Jika pengguna sedang memakai HP dan menekan *Tunda 5 Menit*, jadwal digeser +5 menit (+ jitter acak 15–45 detik).
   - Jika hitungan mundur habis, `ShoFastAccessibilityService` masuk ke State Machine upload di aplikasi `com.shopee.id`, menampilkan **Mini Floating HUD** langkah demi langkah di sudut layar.
3. **Auto-Cleanup**: Segera setelah konfirmasi upload selesai di Shopee Video, status berubah menjadi `completed`, log tersimpan di Supabase, dan file MP4 lokal di `Movies/ShoFast/` serta cache server **otomatis dihapus** agar ruang penyimpanan HP dan server tetap lega.
4. **Auto-Retry Cerdas**: Jika eksekusi gagal (misal jaringan terputus, dialog promo Shopee muncul tiba-tiba, atau timeout state > 25 detik), sistem menutup Shopee dengan aman, mencatat `error_message`, dan menjadwalkan ulang otomatis dalam **5 menit + jitter acak** (maksimal **2x percobaan ulang**) sebelum memindahkan item ke tab `Gagal`.

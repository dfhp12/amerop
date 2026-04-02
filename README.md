# Forecast Amerop

Aplikasi web monitoring dan forecasting anggaran untuk **Direktorat Jenderal Amerika dan Eropa (Ditjen Amerop)**. Berjalan sepenuhnya di browser — tidak butuh server, tidak butuh instalasi.

---

## Gambaran Umum

Forecast Amerop menyatukan data pagu, realisasi SAKTI, Pertanggungjawaban (PJ), dan forecast kegiatan ke dalam satu dashboard. Data ditarik langsung dari Google Sheets yang sudah ada; tidak ada database terpisah. Semua forecast dan snapshot SAKTI tersimpan di `localStorage` browser.

**Unit yang dicakup:**

| Kode Unit | Label |
|---|---|
| Dit KSIA Amerop | Output AEB.001 |
| Dit Amerika I | Output AEC.001 |
| Dit Amerika II | Output AEC.002 |
| Dit Eropa I | Output AEC.003 |
| Dit Eropa II | Output AEC.004 |
| Setditjen Amerop | EBA.962, EBA.994, EBA.Z08, EBA.Z24 |
| Direktif Presiden | FAN.ZZ1 |

---

## Sumber Data

| Sumber | Format | Keterangan |
|---|---|---|
| **FA CSV** | Google Sheets (published CSV) | Data pagu & realisasi SAKTI per output/kegiatan/akun. Diperbarui setiap kali sheet di-publish ulang. |
| **PJ Sheets** | 7 Google Sheets (gviz CSV) | Satu sheet per unit (Amerika I, Amerika II, Eropa I, Eropa II, KSIA, Setditjen, Jaldis). Dibaca langsung dari browser via URL publik. |
| **Forecast** | `localStorage` browser | Entri forecast yang dibuat manual di tab Forecast. Tersimpan di perangkat pengguna. |
| **Riwayat SAKTI** | `localStorage` browser | Snapshot data pagu & realisasi yang disimpan secara manual untuk perbandingan antar waktu. |
| **Embedded Fallback** | Hard-coded di `index.html` | Data cadangan yang ditampilkan jika FA CSV gagal dimuat (misal: tidak ada koneksi internet). |

---

## Fitur & Tab

### 1. Ringkasan
Halaman utama. Menampilkan kartu metrik: Total Pagu, Realisasi SAKTI, PJ Submitted, Forecast, dan Sisa. Di bawahnya ada grafik PJ bulanan dan tabel per direktorat (Pagu / SAKTI / PJ / Forecast / Serapan).

### 2. Detail Anggaran
Tabel hierarkis: Output → Kegiatan → Akun (MAK). Setiap baris menampilkan Pagu, Realisasi, dan Sisa. Dapat difilter per direktorat.

### 3. Rekonsiliasi
Mencocokkan data PJ yang sudah di-submit dengan realisasi SAKTI. Berguna untuk mengidentifikasi gap antara yang sudah dipertanggungjawabkan dan yang sudah tercatat di SAKTI.

### 4. RPD (Rencana Penarikan Dana)
Menampilkan rencana dan realisasi penarikan dana per bulan dalam format tabel. Berguna untuk memantau kepatuhan terhadap RPD yang sudah disepakati.

### 5. Tracker PJ
Menampilkan semua entri kegiatan dari sheet PJ masing-masing unit. Dilengkapi filter per unit, bulan, dan status. Entri bisa ditandai "Sudah PJ" langsung dari tampilan ini. Mendukung ekspor ke CSV.

### 6. Forecast
Buat dan kelola entri forecast untuk kegiatan yang belum terealisasi. Setiap entri bisa dikaitkan dengan satu atau lebih MAK beserta nominalnya. Total forecast muncul di dashboard Ringkasan. Mendukung ekspor ke CSV.

### 7. Pengaturan
- **Ekspor Data** — Download seluruh data PJ atau Forecast ke file CSV.
- **Riwayat SAKTI** — Simpan snapshot kondisi pagu & realisasi saat ini, lalu bandingkan dua snapshot untuk melihat perubahan.

---

## Cara Menjalankan

Buka `index.html` langsung di browser — tidak perlu web server. Pastikan terhubung ke internet agar data dari Google Sheets dapat dimuat. Jika offline, aplikasi akan menggunakan data fallback yang ter-embed di file HTML.

---

## Konfigurasi (untuk Developer)

Semua konfigurasi ada di bagian `// ========== CONFIG ==========` di dalam `index.html`.

```js
// URL Apps Script backend (untuk operasi tulis, jika digunakan)
var SCRIPT_URL = 'https://script.google.com/macros/s/...';

// URL Google Sheet FA (pagu & realisasi SAKTI) — harus di-publish sebagai CSV
const FA_CSV = 'https://docs.google.com/spreadsheets/d/e/.../pub?output=csv';

// Sheet PJ per unit — harus bisa diakses publik
const PJ_SHEETS = {
  pj1: { id: '...', unit: 'Dit Amerika I',   type: 'kegiatan' },
  pj2: { id: '...', unit: 'Dit Amerika II',  type: 'kegiatan' },
  pj3: { id: '...', unit: 'Dit Eropa I',     type: 'kegiatan' },
  pj4: { id: '...', unit: 'Dit Eropa II',    type: 'kegiatan' },
  pj5: { id: '...', unit: 'Dit KSIA Amerop', type: 'kegiatan' },
  pj6: { id: '...', unit: 'Setditjen',       type: 'kegiatan' },
  pj7: { id: '...',  unit: '',               type: 'jaldis'   },
};
```

Untuk memperbarui data embedded fallback, ganti isi array `B_EMBED` di baris berikutnya dengan data terbaru dari SAKTI.

---

## Script Import: `import_sementara.gs`

Script Google Apps Script ini digunakan untuk memindahkan data dari spreadsheet **Sementara** ke spreadsheet **Amerop Forecasts 2026**. Cara pakai:

1. Buka spreadsheet **Amerop Forecasts 2026** di Google Sheets.
2. Buka **Extensions → Apps Script**.
3. Paste isi `import_sementara.gs` ke editor.
4. Jalankan fungsi `importFromSementara()`.
5. Script akan membaca semua baris dari sheet Sementara, melewati duplikat (berdasarkan nama kegiatan), menormalkan format tanggal, lalu menambahkan baris baru ke sheet Amerop.

Log hasil import (berapa yang ditambah, dilewati, atau blank) akan tampil di Apps Script Logger dan sebagai pop-up alert.

---

## Catatan Teknis

- Tidak ada backend yang dibutuhkan untuk operasi baca. Semua data dibaca langsung dari URL publik Google Sheets.
- Forecast dan snapshot SAKTI tersimpan di `localStorage` browser. Data ini **tidak sinkron antar perangkat** dan **akan hilang** jika browser storage dibersihkan.
- Normalizer tanggal di script import mendukung berbagai format: ISO, `DD/MM/YYYY`, `W{n}/{m}/{y}`, nama bulan dalam Bahasa Indonesia dan Inggris, rentang tanggal, dan format minggu ("Minggu 3 Juni").

---

---

# FAQ

**T: Data tidak muncul / halaman kosong. Apa yang salah?**

J: Kemungkinan besar browser memblokir request ke Google Sheets karena CORS atau koneksi internet bermasalah. Coba:
1. Pastikan terhubung ke internet.
2. Buka `index.html` langsung di browser (bukan dari dalam folder ZIP yang belum diekstrak).
3. Jika pakai browser berbasis Chromium, coba jalankan dengan flag `--disable-web-security` untuk pengujian lokal — atau upload file ke hosting sederhana (GitHub Pages, dll).
4. Jika semua gagal, aplikasi akan menampilkan data embedded fallback secara otomatis.

---

**T: Data SAKTI sudah diperbarui di Google Sheets tapi di aplikasi masih lama.**

J: FA CSV di-cache oleh browser. Hard-refresh (`Ctrl+Shift+R` atau `Cmd+Shift+R`) biasanya cukup. Jika masih tidak berubah, pastikan sheet FA sudah di-publish ulang ke web (`File → Share → Publish to web`) setelah perubahan data.

---

**T: Forecast yang saya buat hilang setelah ganti browser / perangkat.**

J: Forecast disimpan di `localStorage` browser yang aktif, bukan di server atau Google Sheets. Data ini bersifat lokal per perangkat. Untuk backup, gunakan fitur **Download Forecast CSV** di tab Pengaturan, lalu simpan filenya.

---

**T: Bagaimana cara menambah unit/direktorat baru?**

J: Edit bagian `const DIR` dan `const PJ_SHEETS` di `index.html`. Tambahkan entri baru dengan ID sheet Google Sheets-nya dan label unit. Pastikan sheet tersebut bisa diakses publik (Anyone with the link → Viewer).

---

**T: Sheet PJ tidak terbaca. Angka PJ semua nol.**

J: Ada beberapa kemungkinan penyebab:
- Sheet belum di-set ke akses publik ("Anyone with the link can view").
- ID Sheet di konfigurasi `PJ_SHEETS` salah.
- Nama kolom di sheet tidak sesuai dengan yang diharapkan parser (cek header baris pertama di sheet).
Buka browser Developer Tools (F12 → Console) untuk melihat error spesifik saat data dimuat.

---

**T: Apa perbedaan kolom SAKTI, PJ, dan Forecast di dashboard?**

J: Ketiganya berbeda sumber dan makna:
- **SAKTI** — realisasi resmi yang tercatat di sistem SAKTI (pemerintah). Angka ini dari FA CSV.
- **PJ** (Pertanggungjawaban) — total yang sudah diajukan oleh unit masing-masing melalui sheet PJ mereka. Bisa berbeda dengan SAKTI karena timing.
- **Forecast** — estimasi pengeluaran yang belum terealisasi, dibuat manual di tab Forecast. Menunjukkan proyeksi sisa anggaran yang akan terserap.

---

**T: Apa fungsi "Simpan Snapshot" di Pengaturan?**

J: Snapshot merekam kondisi data pagu dan realisasi pada satu titik waktu. Dengan menyimpan beberapa snapshot (misal: sebelum dan sesudah revisi pagu), Anda bisa membandingkan keduanya untuk melihat perubahan angka secara spesifik — berguna untuk rekap revisi atau laporan bulanan.

---

**T: Script `import_sementara.gs` bisa dijalankan berkali-kali tanpa duplikasi?**

J: Ya. Script memeriksa nama kegiatan yang sudah ada di sheet Amerop sebelum menambahkan baris baru. Baris dengan nama kegiatan yang sudah ada (case-insensitive) akan dilewati dan dicatat sebagai "SKIP (dupe)" di log.

---

**T: Apakah aplikasi ini aman dipakai untuk data anggaran sensitif?**

J: Aplikasi ini membaca data dari Google Sheets yang sudah di-set ke akses publik (Anyone with link). Artinya siapa pun yang punya link Google Sheets tersebut bisa membaca datanya — terlepas dari aplikasi ini. Jika data bersifat sensitif, pertimbangkan untuk membatasi akses sheet dan mengintegrasikan autentikasi Google (memerlukan perubahan pada Apps Script backend).

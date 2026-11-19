# BDI Pusat Kendali (web)

Antarmuka web untuk kerangka `bdi-pusat-kendali`. Dibuat hanya dengan **HTML, CSS, dan JavaScript biasa**:
tanpa framework, tanpa Python, tanpa proses build, tanpa koneksi internet.

## Cara membuka

1. Ekstrak zip.
2. Klik dua kali `index.html` (Chrome, Edge, Firefox, atau Safari versi terbaru).

Bisa juga diunggah apa adanya ke hosting statis mana pun (Netlify, GitHub Pages, cPanel, dll.).

## Isi halaman

| Halaman | Isi |
|---|---|
| Hari ini | Pengingat KAK hari ini, batch terlambat/segera, jendela pemasaran terendah, RAB di bawah target |
| Tracker KAK | Papan 7 tahap (belum mulai sampai dipublikasikan); tombol di kartu memajukan tahap |
| Batch 2027 | Tabel 83 batch: filter bulan/tier/tahap/status, urutkan, unduh CSV jalur mundur |
| Detail batch | Ringkasan (tanggal pasti, metode, tempat, pendaftar, progres KAK), Jalur KAK (garis waktu + dasar tiap tanggal), KAK v3 (25 butir), RAB |
| Kalender | Tampilan bulan Agu 2026 sampai Des 2027: hari pertama pelatihan, tenggat KAK, libur; unduh `.ics` |
| RAB | Form seperti Excel (6 bagian), panel Cek Target: margin, peserta impas, peserta minimal untuk target, harga minimum, sensitivitas |
| Pengaturan | Aturan jalur mundur (hari kerja), program mahal H-70, nilai awal RAB, data lembaga, libur, cadangan/pulihkan data |

**Tanggal acuan** di kanan atas bisa diganti untuk simulasi ("bagaimana kalau hari ini 8 Des 2026?").

## Struktur folder

```
index.html            kerangka halaman
css/style.css         seluruh tampilan
js/data.js            data awal: 32 program, 83 batch, 30 libur, aturan, template KAK v3, kerangka RAB, 3 RAB contoh
js/domain.js          logika inti (kalender hari kerja, jalur mundur KAK, pengingat, RAB, .ics)
js/store.js           penyimpanan di browser (localStorage)
js/ui.js              format tanggal/rupiah, label, dialog, notifikasi
js/pages/*.js         satu file per halaman
js/app.js             navigasi dan tanggal acuan
tools/build-data.mjs  membuat ulang js/data.js dari db/seed (opsional, butuh Node)
```

## Kebenaran hitungan

`js/domain.js` adalah port langsung dari `src/domain/*.ts` di kerangka. Sebelum diserahkan sudah diuji:

- dibandingkan dengan implementasi TypeScript asli: setiap hari kalender 2026-08-01 s.d. 2027-12-31, jalur mundur untuk setiap tanggal pelatihan 2027 (3 variasi aturan), 34.760 kombinasi tahap batch x tanggal hari ini (termasuk pengingat), dan 3.000 RAB acak. Semua sama persis;
- nilai emas: 11 tanggal jalur mundur (`kak_golden.json`) dan 3 RAB asli (PICU & NICU B13 total 71.078.625 margin 43,4%; PONEK B1 87.007.125 / 42,0%; BTCLS Lira B2 13.735.000 / 60,8%);
- uji di browser (Chrome) untuk semua halaman dan aksi utama, layar desktop, tablet, dan HP, tanpa error console.

Satu perbaikan dibanding versi TypeScript: di file `.ics`, titik koma sekarang benar-benar di-escape (`\;`) sesuai RFC 5545.

## Yang perlu diketahui

- **Data tersimpan di browser masing-masing** (localStorage). Belum ada server, login, atau berbagi data antar pengguna. Pakai *Unduh cadangan* / *Pulihkan dari cadangan* di Pengaturan untuk memindahkan data. Untuk dipakai bersama satu tim, langkah berikutnya adalah backend sesuai `db/schema.sql` dan `docs/api/openapi.yaml` di kerangka; fungsi di `js/domain.js` bisa dipakai ulang di Node.
- Tanggal pelatihan 2027 baru tingkat bulan. Batch tanpa tanggal pasti dihitung dari tanggal 1 (paling ketat) dan diberi label "perkiraan".
- Hanya janji layanan Kemenkes 14 hari kerja yang bersumber resmi. Durasi lain (persiapan, penyusunan, TTD, buffer revisi) adalah asumsi BDI dan bisa diubah di Pengaturan.
- Libur 2027 diambil dari artikel Kemensetneg atas SKB 3 Menteri, belum dicocokkan dengan PDF SKB asli. Hitungan di luar 1 Agu 2026 s.d. 31 Des 2027 sengaja ditolak.
- Belum ada: konversi otomatis file KAK lama ke v3 dan ekspor DOCX, integrasi WhatsApp/email, impor data peserta.

## Memperbarui data awal

`js/data.js` dibuat dari folder `db/seed` kerangka dengan skrip Node (JavaScript, bukan Python). Bila seed berubah, misalnya tanggal pasti diisi di `batches_2027_rencana.csv`:

```bash
node tools/build-data.mjs ../bdi-pusat-kendali js/data.js
```

Tanggal pasti juga bisa langsung diisi lewat halaman Detail batch tanpa mengubah file.

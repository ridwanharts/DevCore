# 🧰 DevCore

> Kumpulan tools developer berbasis web — gratis, tanpa login, tanpa server.
> Dibangun sebagai **proyek belajar JavaScript**: dari DOM, fetch API, dan state
> management manual, sampai deploy ke production.

Dokumen ini adalah PRD (Product Requirements Document) DevCore. Isinya hidup:
diperbarui setiap fase selesai. **Belum ada kode aplikasi di repo ini** — PRD dulu,
kode menyusul per fase.

---

## 1. Visi & Tujuan

**Visi:** satu tempat untuk tools developer kecil yang sering dipakai
sehari-hari — buka browser, langsung pakai, tanpa install apa pun.

**Tujuan:**

1. **Belajar membangun project web dengan JavaScript** — ini tujuan utama.
   Setiap tools adalah latihan: manipulasi DOM, event handling, fetch API,
   parsing data, dan mengelola state tanpa framework.
2. Menyediakan tools yang benar-benar berguna, bukan sekadar demo.
3. Menjaga proyek tetap gratis dan privat: semua proses berjalan di browser
   pengguna, tidak ada data yang dikirim ke server DevCore.

**Prinsip desain:**

- Client-side only — tidak ada backend, tidak ada database, tidak ada akun.
- Satu tools = satu modul mandiri, mudah ditambah tanpa merusak tools lain.
- Cepat dan ringan — situs statis, tanpa loading yang tidak perlu.
- Jujur soal keterbatasan (misalnya CORS di Mini Postman, lihat §2.2).

---

## 2. Daftar Tools

### 2.1 MVP

#### 🔍 JSON Viewer — terinspirasi [jsonviewer.stack.hu](https://jsonviewer.stack.hu/)

- **Input:** tempel (paste) JSON, upload file `.json`, dan contoh bawaan.
- **Format & Minify:** rapikan JSON (indentasi 2 spasi) dan padatkan kembali.
- **Validasi:** JSON tidak valid harus menampilkan pesan error yang jelas
  (posisi baris/kolom bila memungkinkan), bukan sekadar "error".
- **Tree View:** tampilkan JSON sebagai pohon yang bisa di-expand/collapse,
  per node maupun semua sekaligus.
- **Search/Filter:** cari key atau value di dalam struktur.
- **Utilitas:** salin hasil, unduh sebagai `.json`, dan klik node untuk
  menyalin JSON path-nya (misalnya `data.users[0].name`).
- **Statistik:** ukuran input, jumlah key, dan kedalaman maksimal struktur.
- **Ketahanan:** JSON besar harus tetap ditangani dengan baik
  (peringatan performa bila melewati batas wajar, mis. > 1 MB).

#### 📬 Mini Postman — HTTP client sederhana

- **Request Builder:**
  - Method: GET, POST, PUT, PATCH, DELETE, HEAD, OPTIONS.
  - URL, query params (tabel key–value dengan checkbox aktif/nonaktif).
  - Headers (tabel key–value).
  - Body: raw JSON (dengan validasi) untuk method yang mendukung body.
  - Auth: Bearer Token dan Basic Auth.
- **Kirim & Respons (via `fetch`):**
  - Status code + status text, waktu respons (ms), dan ukuran body.
  - Tab respons: body terformat (JSON pretty) / raw text, dan headers respons.
- **Riwayat:** request terakhir tersimpan otomatis di `localStorage`
  (maksimal 50 entri), bisa diklik untuk dimuat ulang dan dihapus.
- **Collections (opsional, Fase 2 akhir):** simpan request bernama agar bisa
  dipakai ulang.
- **⚠️ Batasan yang harus dipahami:** browser memblokir request ke API yang
  tidak mengizinkan CORS. Ini keterbatasan browser, bukan bug DevCore.
  Solusi lanjutannya ada di Fase 4 (Cloudflare Worker sebagai proxy).

### 2.2 Backlog (Fase 3+)

| Tools | Deskripsi singkat |
|---|---|
| Base64 Encode/Decode | Encode/decode teks dan file kecil |
| JWT Decoder | Decode header & payload JWT (tanpa verifikasi signature — beri label yang jelas) |
| UUID Generator | Generate UUID v4, tunggal maupun massal |
| Timestamp Converter | Konversi epoch ↔ tanggal (default zona Asia/Jakarta) |
| Regex Tester | Uji pola regex dengan highlight hasil match dan grup |
| Color Tools | Konversi HEX/RGB/HSL + pembuat palet |
| Markdown Preview | Tulis Markdown, lihat hasil render langsung |
| Text Tools | Ubah case, hitung kata/karakter, diff teks sederhana |
| URL Encoder/Decoder | Encode/decode URL dan query string |

---

## 3. Tech Stack (Keputusan)

| Lapisan | Pilihan | Alasan |
|---|---|---|
| Build tool | **Vite** | Dev server cepat, HMR, output situs statis; standar industri dengan konfigurasi minimal |
| Bahasa | **JavaScript (ES Modules), tanpa framework** | Tujuan proyek ini adalah belajar fundamental: DOM, event, fetch, dan state manual. React sengaja **tidak** dipakai di sini — React sudah dipakai di proyek lain (Edu Fia Map); DevCore melatih dasarnya |
| Styling | **CSS native + design token WanStudio Chunky** | Belajar layout CSS (Flexbox/Grid) tanpa abstraksi. Token warna, shadow keras 4px, dan radius chunky diambil dari `theme.json` WanStudio agar konsisten dengan produk lain |
| Routing | **Hash router sederhana** (`#/json-viewer`, `#/mini-postman`) | Satu file router, tanpa reload halaman; sekaligus latihan konsep routing manual |
| Penyimpanan | **`localStorage`** | Riwayat request, pengaturan, dan tema — tanpa backend |
| Kualitas kode | ESLint + Prettier (akhir Fase 1) | Diaktifkan setelah fondasi jalan, supaya tidak menghambat belajar di awal |
| Testing | Manual di MVP; Vitest opsional di fase lanjut | Prioritas awal adalah fitur jalan dan konsep paham |

**Bahasa antarmuka:** UI tools memakai Bahasa Inggris (konvensi umum tools
developer), sedangkan dokumentasi proyek ini memakai Bahasa Indonesia.

---

## 4. Arsitektur

- **Static SPA:** satu `index.html`, router hash, tiap tools satu halaman.
- **Struktur folder (rencana):**

```text
DevCore/
├── index.html
├── src/
│   ├── main.js            # router + layout utama
│   ├── styles/            # tokens.css, base.css, components.css
│   ├── lib/               # utilitas bersama (dom, storage, toast, ...)
│   └── tools/
│       ├── json-viewer/   # modul mandiri per tools
│       └── mini-postman/
└── public/                # aset statis (favicon, og-image)
```

- **Aturan modul:** setiap tools punya `init()`/`destroy()` sendiri dan tidak
  boleh bergantung pada tools lain. Menambah tools baru = menambah satu
  folder + satu entri di router dan daftar tools.
- **Tanpa backend:** tidak ada API key, tidak ada environment variable,
  tidak ada tracking/analytics di MVP.

---

## 5. Desain & UX

- Tema **WanStudio Chunky**: biru `#1c91ec` dkk, shadow keras, radius chunky,
  emoji sebagai ikon utama.
- Layout: sidebar daftar tools di desktop, navigasi atas di mobile.
- Responsif untuk layar kecil; keyboard-friendly untuk aksi utama.
- Pesan error ramah dan spesifik; empty state jelas (misalnya JSON Viewer
  kosong menampilkan contoh, bukan halaman blank).
- Dark mode: **bukan** prioritas MVP — masuk backlog Fase 4.

---

## 6. Rencana Deploy — Cloudflare Pages

**Kenapa Cloudflare Pages:** gratis, deploy otomatis dari GitHub, preview
deployment untuk setiap pull request, dan cepat (CDN global). Cocok untuk
situs statis seperti DevCore.

**Langkah-langkah:**

1. Push repo ke GitHub (branch `main`).
2. Cloudflare Dashboard → **Workers & Pages** → **Create** → **Pages** →
   **Connect to Git** → pilih repo `ridwanharts/DevCore`.
3. Build settings:
   - Framework preset: **Vite**
   - Build command: `npm run build`
   - Output directory: `dist`
   - Root directory: (kosong)
4. **Deploy** → DevCore langsung live di domain `*.pages.dev`.
5. **(Opsional) Custom domain:** `devcore.wanstudio.my.id`
   - Tambahkan CNAME di DNS Cloudflare yang mengarah ke project Pages.
   - Lalu di Pages → **Custom domains** → tambahkan domain (SSL otomatis).
6. Setelah itu: setiap push ke `main` = deploy produksi otomatis,
   setiap pull request = URL preview tersendiri.

**Catatan keamanan:** jangan pernah commit secret apa pun ke repo ini.
Di MVP tidak ada environment variable yang dibutuhkan sama sekali.

---

## 7. Roadmap Belajar

| Fase | Fokus | Kriteria selesai |
|---|---|---|
| **Fase 0 — Setup** | Scaffold Vite (vanilla), struktur folder, token CSS, layout + router hash, halaman home berisi daftar tools | Jalan di lokal, ter-deploy di Pages |
| **Fase 1 — JSON Viewer** | Parser + validasi, format/minify, tree view, search, copy/download | Semua fitur §2.1 (JSON Viewer) jalan |
| **Fase 2 — Mini Postman** | Request builder, kirim via fetch, tampilkan respons, riwayat localStorage | Semua fitur §2.1 (Mini Postman) jalan |
| **Fase 3 — Tools backlog** | 2–3 tools kecil per iterasi (Base64, JWT, UUID, Timestamp, …) | Tiap tools mengikuti pola modul mandiri §4 |
| **Fase 4 — Lanjutan** | Cloudflare Worker sebagai proxy Mini Postman (belajar serverless), dark mode, PWA/offline (opsional) | Diputuskan saat Fase 3 selesai |

Setiap fase selesai jika: fitur jalan di lokal **dan** ter-deploy di Pages,
lalu tabel status di §8 diperbarui.

---

## 8. Status Tools

| Tools | Status |
|---|---|
| JSON Viewer | 📋 Direncanakan (Fase 1) |
| Mini Postman | 📋 Direncanakan (Fase 2) |
| Base64, JWT Decoder, UUID, Timestamp, Regex, Color, Markdown, Text, URL | 📋 Backlog (Fase 3+) |

---

## 9. Non-Goals (yang sengaja TIDAK dibuat)

- Tidak ada login, akun, atau sinkronisasi cloud.
- Tidak ada backend atau database sendiri.
- Tidak ada monetisasi, iklan, atau analytics di MVP.
- Bukan pengganti Postman penuh (tidak ada test runner, mock server,
  maupun kolaborasi tim).

---

## 10. Menjalankan Proyek (untuk nanti, saat kode mulai ditulis)

```bash
npm install   # install dependensi
npm run dev   # server pengembangan (Vite)
npm run build # build production ke folder dist/
npm run preview # pratinjau hasil build secara lokal
```

---

*PRD ini ditulis sebelum satu baris kode pun dibuat — sesuai tujuan proyek:
belajar membangun project web JavaScript dari nol, satu tools dalam satu waktu.* 🧰

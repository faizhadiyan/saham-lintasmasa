# Lintasmasa Saham

Website data pasar saham Indonesia — statis, di-build ulang tiap hari setelah pasar tutup.
Bagian dari proyek **lintasmasa.id**.

- Live: https://saham-lintasmasa.vercel.app (custom domain menyusul)
- Repo: https://github.com/faizhadiyan/saham-lintasmasa
- 940 emiten IDX, screener, halaman detail per emiten (grafik 1 tahun + MA20/MA50),
  indeks & sektoral, dashboard valuasi.

## Struktur

```
tools/
  extract_universe.py  # universe 940 emiten dari Wikipedia -> data/universe.csv (sekali saja / saat IPO baru)
  fetch_all.py         # quotes + fundamental + histori 1 thn via yfinance (multi-thread)
  build_snapshot.py    # raw -> data/snapshots/YYYY-MM-DD.json (agregat pasar, sektoral, top lists, IHSG 2 thn)
  build.py             # snapshot -> dist/ (HTML statis siap deploy)
data/
  universe.csv         # kode, nama, tgl listing, jml saham, papan, sektor
  raw/quotes_YYYY-MM-DD.json
  history/<KODE>.json  # OHLCV harian 1 tahun per emiten
  snapshots/YYYY-MM-DD.json
dist-vercel/          # output build (DIST_DIR) -> upload langsung ke Vercel via tools/vercel_upload.py
dist/                 # output lama -> push GitHub (jalur belum aktif, project belum git-linked)
```

## Workflow update harian (setelah pasar tutup, ~16.30 WIB)

```bash
cd ~/workspace/lintasmasa/web-saham
D=$(date +%F)
.venv/bin/python tools/fetch_all.py $D        # ~6-10 menit untuk 940 emiten
.venv/bin/python tools/build_snapshot.py $D   # agregat + IHSG
DIST_DIR=$PWD/dist-vercel .venv/bin/python tools/build.py $D  # generate dist-vercel/
python3 tools/vercel_upload.py               # upload file -> Vercel (paralel)
python3 tools/vercel_deploy.py               # create deployment ke production
# verifikasi: homepage memuat "Penutupan $D"
```

Catatan: deploy langsung via Vercel API (project belum git-linked,
koneksi GitHub mentok 403 saat pembuatan). Jalur push GitHub -> auto-deploy
belum aktif; jangan pakai sampai git-link terpasang.

Semua langkah di atas bisa dijalankan berurutan; kalau fetch gagal di tengah,
jalankan ulang (file per emiten ditimpa, idempotent).

## Catatan data

- Harga/fundamental: Yahoo Finance (`<KODE>.JK`), tertunda. PER/PBV trailing.
- Market cap = harga × jumlah saham beredar (Wikipedia). Lebih konsisten antar emiten
  daripada marketCap Yahoo yang kadang basi.
- Emiten yang 404 di Yahoo (delisting/suspensi) otomatis tidak masuk snapshot.
- Halaman valuasi (`/valuasi/`) memakai data riset statis (diupdate manual per kuartal),
  bukan dari snapshot harian.

## Roadmap

- [ ] Analisis AI per emiten (slot "Analisis" sudah disiapkan di halaman detail)
- [ ] Konstituen LQ45 / IDX30
- [ ] Kalender IPO & dividen
- [ ] Custom domain saham.lintasmasa.id

## Deploy Vercel (LIVE 2026-10-13)
- URL: https://saham-lintasmasa.vercel.app
- Project: saham-lintasmasa (prj_dOO2qQTXkiXXK15sVtW5upIbwH8c)
- Cara deploy: `python3 tools/vercel_upload.py` lalu `python3 tools/vercel_deploy.py`
- Catatan: history di-inline ke HTML (var H), screener data per huruf. Jangan kembalikan ke bundle per huruf (file >48KB gagal upload via MCP).

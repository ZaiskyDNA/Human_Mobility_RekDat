# Analisis Mobilitas DIY: Dampak Peristiwa dan Pola Komuter

**Penulis:** Muhammad Zakiyyuddin Abdul Adhiim  
**NIM:** 24/545668/TK/60719  
**Mata kuliah:** Rekayasa Data, Universitas Gadjah Mada

---

## Deskripsi

Proyek ini menganalisis data mobilitas perangkat seluler teragregasi (dataset *JogjaPulse*) di Daerah Istimewa Yogyakarta (DIY) dari 1 Desember 2021 sampai 30 April 2022. Periode tersebut mencakup libur Natal dan Tahun Baru (Nataru), gelombang Omicron, Ramadan 1443 H, dan masa menjelang mudik Lebaran.

Seluruh alur rekayasa data dikerjakan di satu notebook, `analisis_mobilitas_jogja.ipynb`:

1. **Pengambilan data**: membaca data dari `data/`. Jika file belum ada, notebook mengunduhnya dari API Zenodo dan memverifikasi checksum MD5.
2. **Pemahaman data**: kamus data, *grain* tiap tabel, dan pengecekan kunci wilayah.
3. **Pemeriksaan kualitas data**: duplikat, nilai kosong, rentang waktu, kelengkapan, efek k-anonim, dan deteksi masalah panel perangkat.
4. **Pembersihan dan transformasi**: pembagian segmen panel, kalender peristiwa, join ke batas wilayah, dan pembentukan tabel analisis (sebagian dengan DuckDB).
5. **Analisis dan visualisasi**: pola dasar, dampak peristiwa (RQ1), serta pola komuter dan arus pergerakan (RQ2).
6. **Keterbatasan, kesimpulan, dan referensi.**

## Pertanyaan Penelitian

- **RQ1 (utama):** Bagaimana sebaran kehadiran antarkapanewon dan pola waktunya bergeser saat Nataru, gelombang Omicron, Ramadan, dan menjelang mudik Lebaran, dibanding periode acuan di segmen panel yang sama? Kapanewon mana yang pangsanya paling berubah?
- **RQ2 (pelengkap):** Kapanewon mana yang berperan sebagai pusat aktivitas dan mana yang sebagai kawasan tempat tinggal, dan ke mana arus pergerakan utamanya?

Semula RQ1 bertujuan mengukur besar perubahan mobilitas total. Rumusannya diubah setelah pemeriksaan kualitas, karena jumlah perangkat yang teramati berubah drastis selama periode data (lihat bagian Metode).

## Data

| File | Isi | Sumber |
|---|---|---|
| `data/presence_kapanewon_hourly.csv` | Jumlah perangkat berbeda (`n_maids`) per kapanewon per jam, 78 kapanewon x 151 hari x 24 jam | JogjaPulse, Zenodo (CC BY 4.0) |
| `data/od_home_visits_kapanewon_monthly.csv` | Arus asal (rumah) ke tujuan (kunjungan siang) antarkapanewon per bulan | JogjaPulse, Zenodo (CC BY 4.0) |
| `data/kapanewon_diy.geojson` | Batas wilayah 78 kapanewon DIY | [aaronsanger/idn-geojson](https://github.com/aaronsanger/idn-geojson) (`districts/34_*.geojson`), turunan dari [cahyadsn/wilayah_boundaries](https://github.com/cahyadsn/wilayah_boundaries) |
| Kalender peristiwa | Rentang tanggal Nataru, Omicron, Ramadan, pra-mudik beserta sumbernya | Disusun manual di notebook |

Catatan data:

- Semua waktu dalam WIB (UTC+7).
- Baris dengan kurang dari 5 perangkat dihapus oleh penyusun dataset (k-anonim, k = 5). Baris yang hilang berarti "kurang dari 5", bukan nol, sehingga tidak diisi 0.
- `n_maids` adalah jumlah perangkat **berbeda** dan tidak aditif, sehingga tidak dijumlahkan lintas jam.
- Batas wilayah digabung ke data memakai kode BPS (`34.xx.xx`). Ke-78 kode kapanewon di GeoJSON cocok dengan kode dan nama di dataset JogjaPulse, tanpa satu pun yang tidak cocok. Join memakai kode, bukan nama, karena nama JETIS dipakai di Bantul (34.02.09) dan di Kota Yogyakarta (34.71.02). Poligon dari sumber ini sudah disederhanakan dan bukan data resmi, jadi hanya dipakai untuk peta dan centroid.

## Metode

### Masalah panel perangkat

Pemeriksaan kualitas menemukan bahwa rata-rata perangkat yang teramati per jam turun dari 8.663 (Desember 2021) menjadi 1.442 (16 Februari - 30 April 2022), atau turun 83%. Penurunannya terjadi mendadak pada 1 Januari dan 16 Februari 2022. Perubahan ini berasal dari komposisi sampel penyedia data, bukan dari perilaku penduduk. Konsekuensinya bagi analisis:

1. **Tiga segmen panel:** S1 (1-31 Des), S2 (1 Jan-15 Feb), S3 (16 Feb-30 Apr). Tanggal 8 Maret 2022 dikeluarkan karena anomali.
2. **Pangsa, bukan angka mentah:** indeks harian kapanewon adalah rata-rata `n_maids` per jam. Perbandingan antarwaktu memakai pangsa kapanewon terhadap total.
3. **Himpunan kapanewon tetap:** pangsa hanya dihitung pada kapanewon dengan slot jam terisi >= 80% di setiap bulan dalam jendela perbandingan. Syarat ini mencegah pangsa kawasan kota naik secara artifisial karena supresi di pedesaan.
   - Set A (Des-Feb): 66 kapanewon, untuk Nataru dan Omicron.
   - Set B (Mar-Apr): 26 kapanewon, untuk Ramadan dan pra-mudik.
4. **Baseline per segmen:** median pangsa per kapanewon per hari-dalam-minggu, mengikuti metode Google COVID-19 Community Mobility Reports.

| Event | Periode | Acuan |
|---|---|---|
| Nataru | 24-31 Des 2021 | 6-19 Des 2021 |
| Omicron | 1-28 Feb 2022 | 6-19 Des 2021 (hanya pangsa) |
| Ramadan | 3-24 Apr 2022 | Maret 2022 (tanpa 3 dan 8 Mar) |
| Pra-mudik | 25-30 Apr 2022 | Maret 2022 (tanpa 3 dan 8 Mar) |

### Tabel analisis

| Tabel | Isi |
|---|---|
| `daily_kpw` | Indeks kehadiran harian per kapanewon (dibentuk dengan DuckDB) |
| `baseline_change` | Pangsa, baseline, dan % perubahan pangsa per kapanewon per hari event |
| `commuter_idx` | Kehadiran siang (09-16), malam (00-05), dan rasio siang/malam per kapanewon per segmen (median hari kerja) |
| `od` / `arus` | Arus asal-tujuan Des-Feb, proporsi per asal, inflow, outflow, dan indeks net inflow `(in - out) / (in + out)` |

Untuk memastikan tidak ada data yang hilang diam-diam, total `n_maids` dicek sama sebelum dan sesudah agregasi, dan jumlah baris dicek sama sebelum dan sesudah join wilayah.

## Hasil Utama

1. **Nataru hampir tidak menggeser sebaran kehadiran.** Pangsa kabupaten/kota berubah antara -1,5% (Sleman) dan +3,7% (Gunungkidul), dan indeks mentah DIY -0,8% terhadap acuan.
2. **Lonjakan pangsa Kota Yogyakarta saat Omicron tidak dapat ditafsirkan sebagai efek pandemi.** Median perubahan pangsa harian Kota sepanjang Februari +76,5%, tetapi kenaikan ini sudah +69,3% sebelum titik putus 16 Februari, dan pangsa mingguan Kota naik bertahap dari 9,8% menjadi 21,8%, searah dengan menyusutnya panel.
3. **Ramadan memunculkan sinyal perilaku paling jelas.** Di 26 kapanewon perkotaan, bagian kehadiran pukul 02-04 terhadap total harian naik 82% dibanding Maret (waktu sahur), sedangkan bagian pagi dan saat berbuka turun.
4. **Menjelang mudik**, pangsa Kota Yogyakarta turun 22,1% dan pangsa Sleman naik 13,5% dibanding Maret (Set B).
5. **Kota Yogyakarta adalah tujuan aktivitas siang menurut data OD.** Median indeks net inflow kapanewon Kota +0,33, sedangkan keempat kabupaten negatif. Arus terbesar berpusat di Depok-Ngaglik-Ngemplak (Sleman). Depok terlibat di 9 dari 15 arus teratas, dan arus terbesar adalah Ngaglik ke Depok (rata-rata 1.120 perangkat/bulan).
6. **Validasi silang tidak mendukung rasio siang/malam sebagai ukuran pusat aktivitas.** Spearman rho = -0,25 (p = 0,04) pada 64 kapanewon, dan -0,16 (p = 0,32, tidak signifikan) pada 39 kapanewon dengan jam malam lengkap. Rasio tertinggi justru di Playen (4,5) dan terendah di Gondomanan (1,4), sehingga rasio ini lebih mencerminkan aktivitas perangkat di malam hari.

## Figure

Semua gambar dihasilkan oleh notebook dan disimpan di `figures/` (PNG, 200 dpi, 16:9).

| File | Isi |
|---|---|
| `01_tren_harian.png` | Indeks kehadiran harian DIY, rata-rata bergerak 7 hari, periode event, dan titik putus panel |
| `02_heatmap_hari_jam.png` | Heatmap hari dalam minggu x jam (Desember) |
| `03_perubahan_per_event.png` | Median % perubahan pangsa per event per kabupaten/kota |
| `04_peta_perubahan_event.png` | Peta % perubahan pangsa per kapanewon untuk keempat event |
| `05_profil_jam_ramadan.png` | Profil per jam: acuan Maret vs Ramadan |
| `06_peta_rasio_siang_malam.png` | Peta rasio siang/malam per kapanewon |
| `07_peta_arus_od.png` | 15 arus antar-kapanewon terbesar |
| `08_scatter_validasi.png` | Rasio siang/malam vs indeks net inflow OD |

## Keterbatasan

- Angka adalah *presence* perangkat dari SDK aplikasi, bukan jumlah penduduk, dan representativitasnya berbeda antarwilayah.
- Panel perangkat tidak stabil, sehingga perubahan total mobilitas DIY tidak dapat diukur dan efek Omicron tidak dapat dipisahkan dari perubahan sampel.
- Supresi lebih berat di pedesaan dan di bulan akhir. Hasil Ramadan dan pra-mudik hanya berlaku untuk 26 kapanewon perkotaan, dan Kulon Progo tidak terwakili.
- Data OD menghitung pengunjung unik per bulan, bukan jumlah perjalanan, dan hanya Desember-Februari yang dipakai.
- Semua hasil bersifat deskriptif dan tidak menunjukkan hubungan sebab-akibat.

## Struktur Repo

```
.
├── README.md
├── requirements.txt
├── .gitignore
├── analisis_mobilitas_jogja.ipynb
├── data/
│   ├── presence_kapanewon_hourly.csv
│   ├── od_home_visits_kapanewon_monthly.csv
│   └── kapanewon_diy.geojson
└── figures/
    └── 01_ ... 08_*.png
```

## Cara Menjalankan

```bash
git clone https://github.com/ZaiskyDNA/Human_Mobility_RekDat.git
cd Human_Mobility_RekDat
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
jupyter notebook analisis_mobilitas_jogja.ipynb
```

Lalu pilih **Kernel > Restart & Run All**. Data sudah tersedia di `data/`, jadi notebook bisa langsung dijalankan tanpa unduhan tambahan. Library yang dipakai: pandas, numpy, geopandas, matplotlib, seaborn, scipy, requests, dan duckdb.

## Referensi

1. Widyawan, Pratama, A. R., Ariansyah, A., Farda, N. M., Pramana, S., & Miller, O. (2026). *JogjaPulse: An Aggregated Spatio-Temporal Mobility Dataset for Yogyakarta, Indonesia* (v1) [Dataset]. Zenodo. https://doi.org/10.5281/zenodo.23157460 (lisensi CC BY 4.0)
2. Google LLC. *COVID-19 Community Mobility Reports*: metodologi baseline. https://www.google.com/covid19/mobility/
3. cahyadsn. *wilayah_boundaries*. https://github.com/cahyadsn/wilayah_boundaries
4. aaronsanger. *idn-geojson*. https://github.com/aaronsanger/idn-geojson

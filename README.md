# Analisis Mobilitas DIY: Dampak Peristiwa dan Pola Komuter

Tugas mata kuliah Rekayasa Data, Universitas Gadjah Mada.

Analisis data mobilitas perangkat seluler teragregasi (JogjaPulse) di Daerah Istimewa Yogyakarta, 1 Desember 2021 - 30 April 2022. Seluruh proses ada di `analisis_mobilitas_jogja.ipynb`: pengambilan data, pemeriksaan kualitas, transformasi, analisis, dan visualisasi.

## Pertanyaan

- **RQ1:** Bagaimana sebaran kehadiran antarkapanewon dan pola waktunya bergeser saat Nataru, gelombang Omicron, Ramadan, dan menjelang mudik Lebaran dibanding periode acuan di segmen panel yang sama?
- **RQ2:** Kapanewon mana yang berperan sebagai pusat aktivitas dan mana yang sebagai kawasan tempat tinggal, dan ke mana arus pergerakan utamanya?

## Data

| File | Sumber |
|---|---|
| `data/presence_kapanewon_hourly.csv` | JogjaPulse, Zenodo (CC BY 4.0) |
| `data/od_home_visits_kapanewon_monthly.csv` | JogjaPulse, Zenodo (CC BY 4.0) |
| `data/kapanewon_diy.geojson` | [aaronsanger/idn-geojson](https://github.com/aaronsanger/idn-geojson) (`districts/34_*.geojson`), turunan dari [cahyadsn/wilayah_boundaries](https://github.com/cahyadsn/wilayah_boundaries) |

Batas wilayah digabung ke data memakai kode BPS (`34.xx.xx`). Ke-78 kode kapanewon di GeoJSON cocok dengan kode dan nama di dataset JogjaPulse, tanpa satu pun yang tidak cocok. Join memakai kode, bukan nama, karena nama JETIS dipakai di Bantul dan di Kota Yogyakarta. Poligon dari sumber ini sudah disederhanakan dan bukan data resmi, jadi hanya dipakai untuk peta dan centroid.

Kalender peristiwa disusun manual di dalam notebook.

Sitasi dataset:

> Widyawan, Pratama, A. R., Ariansyah, A., Farda, N. M., Pramana, S., & Miller, O. (2026). *JogjaPulse: An Aggregated Spatio-Temporal Mobility Dataset for Yogyakarta, Indonesia* (v1) [Dataset]. Zenodo. https://doi.org/10.5281/zenodo.23157460

## Cara menjalankan

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
jupyter notebook analisis_mobilitas_jogja.ipynb
```

Data sudah tersedia di `data/`. Jika file tidak ada, notebook mengunduhnya dari Zenodo dan memverifikasi MD5. Gambar hasil disimpan di `figures/`.

## Ringkasan temuan

- Jumlah perangkat yang teramati turun sekitar 83% selama periode data, dengan titik putus pada 1 Januari dan 16 Februari 2022. Karena itu analisis memakai pangsa kapanewon pada himpunan kapanewon tetap dan membandingkan setiap event dengan acuan di segmen panel yang sama.
- Nataru hampir tidak menggeser sebaran kehadiran: pangsa kabupaten/kota berubah antara -1,5% dan +3,7%.
- Lonjakan pangsa Kota Yogyakarta saat Omicron (+76%) berimpit dengan penyusutan panel, sehingga tidak dapat ditafsirkan sebagai efek pandemi.
- Saat Ramadan, bagian kehadiran pukul 02-04 terhadap total harian naik 82% dibanding Maret (waktu sahur), diukur di 26 kapanewon perkotaan.
- Menurut data OD, Kota Yogyakarta adalah tujuan aktivitas siang (median indeks net inflow +0,33), sedangkan arus terbesar berpusat di Depok-Ngaglik-Ngemplak. Rasio siang/malam dari data presence tidak mendukung temuan ini (Spearman rho = -0,25 pada 64 kapanewon; -0,16 dan tidak signifikan pada 39 kapanewon dengan jam malam lengkap).

## Penulis

Nama: Muhammad Zakiyyuddin Abdul Adhiim
NIM: 24/545668/TK/60719

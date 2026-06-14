# ✈️ Big Data Final Project — Flight Delay Analytics

**Mata Kuliah:** Big Data | **Semester:** Genap 2025/2026  
**Topik:** Analitik Transportasi & Keterlambatan Penerbangan Domestik AS

---

## 📋 Deskripsi Project

Project ini mengimplementasikan pipeline **ETL** (Extract–Transform–Load) dan **ELT** (Extract–Load–Transform) untuk menganalisis pola keterlambatan penerbangan domestik Amerika Serikat. Data diolah dari dua sumber (data penerbangan dan data bandara global), ditransformasi, lalu dimuat ke **Google BigQuery** sebagai data warehouse untuk kemudian divisualisasikan dalam dashboard analitik interaktif.

### 🎯 Tujuan Analitik
- Mengidentifikasi maskapai dengan rata-rata keterlambatan tertinggi
- Menganalisis pola keterlambatan berdasarkan waktu (jam, hari, bulan, musim)
- Membandingkan keterlambatan antar rute dan kategori jarak penerbangan
- Menghitung on-time rate yang akurat (exclude penerbangan cancelled)

---

## 📁 Struktur Direktori

```
bigdata_final_project/
├── etl_pipeline/
│   └── etl_pipeline_flight_FINAL.ipynb    # Pipeline ETL lengkap
├── elt_pipeline/
│   └── elt_pipeline_flight_FINAL.ipynb    # Pipeline ELT lengkap
├── raw/                                    # Data mentah hasil extract (auto-generated)
│   ├── raw_flights.csv
│   └── raw_airports.csv
├── datalake/                               # Data lake ELT (auto-generated)
│   ├── lake_flights.csv
│   └── lake_airports.csv
├── cleaned/                                # Data bersih hasil transform ETL (auto-generated)
│   └── cleaned_flights_merged.csv
├── warehouse/                              # SQLite backup lokal (auto-generated)
│   └── flight_warehouse.db
├── dashboard/                              # File konfigurasi/export dashboard
├── architecture_diagram.pdf               # Diagram arsitektur sistem
├── Dokumentasi_Database_Warehouse.docx    # Dokumentasi lengkap database & SQL
└── README.md                              # File ini
```

---

## 🗂️ Dataset
<img width="1421" height="734" alt="image" src="https://github.com/user-attachments/assets/4ab2c5f8-7fcf-4d1c-911c-2fecbcf84392" />

<img width="1371" height="723" alt="image" src="https://github.com/user-attachments/assets/c0bb8954-6a56-4057-93e4-31bbae72ce39" />



| Sumber | File | Ukuran | Deskripsi |
|--------|------|--------|-----------|
| https://www.kaggle.com/datasets/patrickzel/flight-delay-and-cancellation-dataset-2019-2023 | `flights_sample_3m.csv` | ~585 MB | Data keterlambatan penerbangan AS (3 juta baris, diambil 150.000) |
| https://www.kaggle.com/datasets/samvelkoch/global-airports-iata-icao-timezone-geo | `airports.csv` | ~670 KB | Data bandara global (IATA, koordinat, timezone) |

### Spesifikasi Dataset Setelah ETL

| Parameter | Nilai |
|-----------|-------|
| Jumlah baris | ≥ 100.000 baris |
| Jumlah kolom | 21+ kolom |
| Kolom numerik | `dep_delay`, `arr_delay`, `distance`, `air_time` |
| Kolom kategorikal | `airline`, `origin`, `dest`, `delay_status` |
| Kolom tanggal/waktu | `fl_date`, `flight_year`, `flight_month` |
| Fitur baru (feature engineering) | 5 fitur: `delay_status`, `is_weekend`, `flight_year`, `flight_month`, `day_of_week` |

---

## ⚙️ Prasyarat

### Akun & Layanan
- [x] Google Account dengan akses Google Colab
- [x] Google Drive (minimal 2 GB ruang kosong)
- [x] Google Cloud Platform project dengan BigQuery diaktifkan
- [x] Colab runtime: **GPU (T4)** — diperlukan untuk cuDF

### Library Python
Library diinstal otomatis saat menjalankan cell pertama notebook. Daftar dependensi utama:

```
cudf-cu12          # GPU DataFrame (NVIDIA RAPIDS)
pandas             # CPU DataFrame fallback
numpy              # Komputasi numerik
scikit-learn       # MinMaxScaler (ETL)
pandas-gbq         # Upload ke BigQuery
google-cloud-bigquery  # BigQuery client
db-dtypes          # Tipe data BigQuery
sqlalchemy         # Koneksi database lokal
```

---

## 🚀 Cara Menjalankan

### Langkah 0 — Persiapan Awal

1. **Upload dataset** ke Google Drive di folder `TUBES BIG DATA`:
   ```
   MyDrive/
   └── TUBES BIG DATA/
       ├── flights_sample_3m.csv   ← upload di sini
       └── airports.csv            ← upload di sini
   ```

2. **Buat GCP Project** dan aktifkan BigQuery API di [console.cloud.google.com](https://console.cloud.google.com)

3. **Catat Project ID** GCP kamu (contoh: `tubes-bigdata-498212`)

---

### Langkah 1 — Jalankan Pipeline ETL

> File: `etl_pipeline/etl_pipeline_flight_FINAL.ipynb`

1. Buka file di **Google Colab** (klik kanan → Open with Colab)
2. Pastikan runtime: `Runtime → Change runtime type → GPU (T4)`
3. Edit **Cell 3** — sesuaikan konfigurasi:
   ```python
   BASE_DIR    = '/content/drive/MyDrive/TUBES BIG DATA'  # ← sesuaikan jika perlu
   GCP_PROJECT = 'YOUR_GCP_PROJECT_ID'                    # ← ganti dengan project ID kamu
   BQ_DATASET  = 'etl_flight_analytics'
   ROW_LIMIT   = 150_000
   ```
4. Jalankan semua cell secara berurutan: `Runtime → Run all`
5. Saat muncul popup autentikasi Google, klik **Allow**

**Output yang dihasilkan:**
- `raw/raw_flights.csv` dan `raw/raw_airports.csv` — data mentah
- `cleaned/cleaned_flights_merged.csv` — data bersih
- `warehouse/flight_warehouse.db` — SQLite backup
- BigQuery tables: `fact_flights`, `dim_airline`, `dim_airport`, `dim_date`, `etl_transformed`
- `etl_log.txt` — log proses lengkap

---

### Langkah 2 — Jalankan Pipeline ELT

> File: `elt_pipeline/elt_pipeline_flight_FINAL.ipynb`

1. Buka file di **Google Colab**
2. Pastikan runtime: `Runtime → Change runtime type → GPU (T4)`
3. Edit **Cell 3** — sesuaikan konfigurasi:
   ```python
   BASE_DIR    = '/content/drive/MyDrive/TUBES BIG DATA'  # ← sesuaikan jika perlu
   GCP_PROJECT = 'YOUR_GCP_PROJECT_ID'                    # ← ganti dengan project ID kamu
   BQ_DATASET  = 'elt_flight_analytics'
   ROW_LIMIT   = 150_000
   ```
4. Jalankan semua cell: `Runtime → Run all`
5. Saat muncul popup autentikasi, klik **Allow**

**Output yang dihasilkan:**
- `datalake/lake_flights.csv` dan `datalake/lake_airports.csv` — data lake
- BigQuery tables: `raw_flights`, `raw_airports`, `elt_cleaned_flights`, `elt_cleaned_airports`, `elt_transformed`, `elt_metrics`
- `elt_log.txt` — log proses lengkap

---

### Langkah 3 — Buka Dashboard

> Tool: **Looker Studio** / **Power BI** / **Apache Superset**

1. Buka [Looker Studio](https://lookerstudio.google.com) (gratis, langsung connect BigQuery)
2. Klik **Create → Data Source → BigQuery**
3. Pilih project: `tubes-bigdata-498212` → dataset: `etl_flight_analytics`
4. Gunakan tabel `etl_transformed` atau `fact_flights` sebagai sumber utama
5. Import file konfigurasi dari folder `dashboard/` (jika tersedia)

---

## 🏗️ Arsitektur Sistem

```
┌─────────────────────────────────────────────────────┐
│                   SUMBER DATA                        │
│  flights_sample_3m.csv    airports.csv               │
└────────────┬────────────────────┬───────────────────┘
             │                    │
    ┌────────▼────────┐  ┌───────▼────────┐
    │   PIPELINE ETL  │  │  PIPELINE ELT  │
    │                 │  │                │
    │ Extract         │  │ Extract        │
    │    ↓            │  │    ↓           │
    │ Transform       │  │ Load (raw)     │
    │  (Python)       │  │    ↓           │
    │    ↓            │  │ Transform      │
    │ Load            │  │  (SQL BQ)      │
    └────────┬────────┘  └───────┬────────┘
             │                   │
    ┌────────▼────────────────────▼────────┐
    │           DATA WAREHOUSE             │
    │         Google BigQuery              │
    │                                      │
    │  ETL: etl_flight_analytics           │
    │  ├── fact_flights (149K+ baris)      │
    │  ├── dim_airline                     │
    │  ├── dim_airport                     │
    │  └── dim_date                        │
    │                                      │
    │  ELT: elt_flight_analytics           │
    │  ├── raw_flights / raw_airports      │
    │  ├── elt_cleaned_flights / airports  │
    │  ├── elt_transformed                 │
    │  └── elt_metrics                     │
    └────────────────┬─────────────────────┘
                     │
    ┌────────────────▼─────────────────────┐
    │         DASHBOARD ANALITIK           │
    │     Looker Studio / Power BI         │
    │  • KPI delay & on-time rate          │
    │  • Tren waktu (bulan/tahun)          │
    │  • Perbandingan maskapai & rute      │
    │  • Filter interaktif                 │
    └──────────────────────────────────────┘
```

---

## 🔄 Ringkasan Tahapan Pipeline

### ETL Pipeline

| Tahap | Deskripsi | Output |
|-------|-----------|--------|
| **Extract** | Baca 150K baris dari 2 CSV menggunakan cuDF | `raw/` folder |
| **Transform — Cleaning** | Hapus duplikat, handle missing value, outlier hard-cap [-120, 1440 menit] | DataFrame bersih |
| **Transform — Standarisasi** | Normalisasi MinMax, Label Encoding, snake_case kolom | Kolom `_norm`, `_enc` |
| **Transform — Enrichment** | JOIN flights + airports (origin & dest), buat 5 fitur baru | `cleaned_flights_merged.csv` |
| **Transform — Validasi** | 6 aturan kualitas data (uniqueness, null, range, dtype, referential, distribusi) | Log validasi |
| **Load** | Upload Star Schema ke BigQuery + SQLite backup | 4 tabel BigQuery |

### ELT Pipeline

| Tahap | Deskripsi | Output |
|-------|-----------|--------|
| **Extract** | Baca 150K baris dari 2 CSV menggunakan cuDF | `datalake/` folder |
| **Load (Raw)** | Upload data mentah langsung ke BigQuery tanpa transformasi | `raw_flights`, `raw_airports` |
| **Transform — Cleaning SQL** | `CREATE OR REPLACE TABLE` dengan `ROW_NUMBER()`, `PARSE_DATE()`, CAST, hard-cap | `elt_cleaned_*` |
| **Transform — Merge & FE SQL** | `LEFT JOIN` airports (2x), buat 6 fitur via SQL (`EXTRACT`, `CASE WHEN`) | `elt_transformed` |
| **Transform — Agregasi SQL** | `GROUP BY` per maskapai/bulan/negara, hitung `on_time_rate_pct` yang benar | `elt_metrics` |
| **Validasi SQL** | 6 aturan kualitas via BigQuery SQL (`INFORMATION_SCHEMA`, `COUNTIF`) | Log validasi |

---

## 📊 Skema Database

### ETL — Star Schema

```
fact_flights (149K+ baris)
├── airline_code   FK → dim_airline.airline_code
├── origin_code    FK → dim_airport.iata_code
├── dest_code      FK → dim_airport.iata_code
├── date_id        FK → dim_date.date_id
├── dep_delay, arr_delay, distance, cancelled, air_time
├── dep_delay_norm, arr_delay_norm   (MinMax normalized)
├── delay_status                     (on_time/minor/moderate/severe/cancelled)
└── is_weekend, flight_month, origin_city, dest_city, ...
```

### ELT — Pipeline Bertahap BigQuery

```
raw_flights → elt_cleaned_flights → elt_transformed → elt_metrics
raw_airports → elt_cleaned_airports ↗
```

---

## 📝 Sistem Logging

Kedua pipeline menggunakan **logging manual** (`write_custom_log`) dengan mekanisme `f.flush()` untuk memastikan log tersimpan langsung ke Google Drive tanpa buffering.

Format log:
```
[2026-06-09 13:35:06] [ETL] [EXTRACT_SOURCE1] [SUCCESS] - Sukses mengekstrak data penerbangan.
   > Nama Sumber Data: flights_sample_3m.csv
   > Jumlah Baris Extracted: 150,000
   > Ukuran File Asli: 585.69 MB
   > Waktu Eksekusi: 5.86 detik
```

File log tersimpan di:
- ETL: `TUBES BIG DATA/etl_log.txt`
- ELT: `TUBES BIG DATA/elt_log.txt`

---

## ✅ Validasi Kualitas Data (6 Aturan)

| # | Aturan | Metode |
|---|--------|--------|
| 1 | **Uniqueness Check** | Cek duplikat composite key (fl_date, airline, origin, dest) |
| 2 | **Null Check** | Cek NULL pada kolom kritis (airline, origin, dest, fl_date) |
| 3 | **Range Check** | dep_delay harus dalam rentang [-120, 1440] menit (hanya non-cancelled) |
| 4 | **Datatype Consistency** | fl_date harus bertipe DATE/datetime, numerik harus FLOAT |
| 5 | **Referential Integrity** | Cek persentase match origin ke tabel airports |
| 6 | **Distribusi Data** | Mean dep_delay harus dalam rentang (-10, 60) menit, std > 5 |

---
## Dashboard
<img width="903" height="678" alt="Screenshot 2026-06-14 205314" src="https://github.com/user-attachments/assets/d2542bf8-25b6-45bc-9561-482156437f4b" />
<img width="903" height="682" alt="Screenshot 2026-06-14 205259" src="https://github.com/user-attachments/assets/5da8a2ce-6726-4655-884f-fcb63229bc56" />
<img width="905" height="680" alt="Screenshot 2026-06-14 205226" src="https://github.com/user-attachments/assets/deeaf819-51c1-4736-894a-32feb0bbe603" />
<img width="902" height="677" alt="Screenshot 2026-06-14 205208" src="https://github.com/user-attachments/assets/ac4d1837-c143-47c4-a05f-33332a00dc78" />
|https://datastudio.google.com/reporting/1947150e-21ea-4d2f-8deb-be0fb10b5814|


---
## Kesimpulan

Berdasarkan hasil implementasi dan evaluasi kedua pipeline, dapat disimpulkan bahwa kedua pendekatan menghasilkan kualitas data yang identik — sama-sama lulus 5 dari 6 dimensi validasi dengan referential integrity 99,9%. Namun dari sisi hasil analitik, ETL menghasilkan output yang lebih terstruktur dan konsisten karena data telah dibersihkan, dinormalisasi, dan diorganisasi dalam star schema sebelum masuk ke warehouse, sehingga nilai-nilai seperti on-time rate (66,04%), rata-rata delay (10,09 menit), dan distribusi status delay yang dihasilkan lebih dapat dipercaya. Sementara ELT menghasilkan nilai yang sedikit berbeda pada jumlah rekaman akhir (148.843 vs 149.801) akibat perbedaan strategi deduplikasi, meskipun selisihnya tidak signifikan (0,08%). Dengan demikian, dari perspektif kualitas dan keandalan hasil, ETL lebih unggul karena transformasi yang ketat sebelum pemuatan data menjamin konsistensi output analitik yang lebih baik.


---
## 🐛 Troubleshooting

| Masalah | Solusi |
|---------|--------|
| `etl_log.txt` berukuran 0 bytes | Gunakan `write_custom_log()` (sudah diimplementasikan), bukan `logging.basicConfig` |
| `KeyError: column not found` | Jalankan cell diagnostik untuk cek nama kolom aktual: `print(df.columns.tolist())` |
| cuDF tidak tersedia | Pastikan runtime Colab menggunakan GPU (T4). Jika tidak ada, notebook akan fallback ke pandas otomatis |
| BigQuery authentication error | Jalankan ulang cell autentikasi, pastikan akun Google memiliki akses ke GCP project |
| Memory error di Colab | Kurangi `ROW_LIMIT` dari 150.000 ke 100.000 |
| `on_time_rate` ~1% (tidak wajar) | Bug fillna cancelled — sudah diperbaiki di FIXED version. Pastikan menggunakan `etl_pipeline_flight_FINAL.ipynb` |

---

## 👥 Tim
||
| Nama | NIM |
|------|-----|
| _Anom Nur Maulid_ | _1103223193_ |
| _Muhammad Rayhan Ananta Kabalmay_ | _1103223077_ |


---

> **Catatan:** Seluruh kode dapat dijalankan ulang dari awal (*reproducible*). Pastikan dataset telah diupload ke Google Drive sebelum menjalankan pipeline.

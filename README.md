# 🏛️ Sistem Case-Based Reasoning (CBR)
### Analisis Putusan Pengadilan – Perkara Perdata Agama

> **Mata Kuliah:** Penalaran Komputer — UMM Informatika 2025/2026  
> **Domain:** Perdata Agama (Gugatan Cerai, Harta Bersama, Waris, dll.)  
> **Sumber Data:** [Direktori Putusan Mahkamah Agung RI](https://putusan3.mahkamahagung.go.id)

---

## 📑 Daftar Isi

- [Deskripsi Proyek](#-deskripsi-proyek)
- [Struktur Folder](#-struktur-folder)
- [Persyaratan Sistem](#-persyaratan-sistem)
- [Instalasi](#-instalasi)
- [Konfigurasi Google Drive (Colab)](#️-konfigurasi-google-drive-colab)
- [Cara Menjalankan Pipeline](#-cara-menjalankan-pipeline)
- [Output yang Dihasilkan](#-output-yang-dihasilkan)
- [Alur Siklus CBR](#-alur-siklus-cbr)

---

## 📌 Deskripsi Proyek

Proyek ini mengimplementasikan siklus **Case-Based Reasoning (CBR)** lengkap berbasis Python untuk mendukung analisis putusan pengadilan agama. Sistem ini mampu:

1. **Membangun Case Base** — scraping & preprocessing ≥ 30 dokumen putusan PDF
2. **Case Representation** — ekstraksi metadata & feature engineering ke format CSV/JSON
3. **Case Retrieval** — TF-IDF + SVM / Naive Bayes + Cosine Similarity
4. **Case Solution Reuse** — prediksi amar putusan dari top-k kasus termirip
5. **Model Evaluation** — Accuracy, Precision, Recall, F1-score + error analysis

---

## 📁 Struktur Folder

```
CBR_Perdata_Agama/
│
├── CBR_Perdata_Agama.ipynb     ← Notebook utama (semua tahap CBR)
├── README.md                   ← Dokumentasi ini
│
├── data/
│   ├── pdf/                    ← Letakkan 30+ file PDF putusan di sini
│   ├── raw/                    ← Teks bersih hasil ekstraksi (case_001.txt, ...)
│   ├── processed/
│   │   ├── cases.csv           ← Dataset representasi kasus
│   │   ├── cases.json          ← Dataset dalam format JSON
│   │   └── cases_features.csv  ← Dataset + fitur tambahan
│   ├── eval/
│   │   ├── queries.json        ← Query evaluasi + ground truth
│   │   ├── retrieval_metrics.csv
│   │   └── prediction_metrics.csv
│   └── results/
│       ├── predictions.csv     ← Hasil prediksi solusi
│       └── evaluation_chart.png
│
└── logs/
    └── cleaning.log            ← Log proses pembersihan teks
```

---

## 💻 Persyaratan Sistem

| Komponen | Versi Minimum |
|----------|--------------|
| Python   | 3.8+         |
| Jupyter Notebook / Google Colab | — |
| RAM      | 4 GB (8 GB direkomendasikan) |

### Library yang Dibutuhkan

```
pdfminer.six
beautifulsoup4
requests
scikit-learn
pandas
numpy
matplotlib
seaborn
openpyxl
```

---

## ⚙️ Instalasi

### Opsi A — Google Colab (Direkomendasikan)

Tidak perlu instalasi manual. Buka notebook di Colab, lalu jalankan sel **"Instalasi & Import Library"** — semua dependensi akan terinstall otomatis.

```python
# Sel ini sudah ada di notebook, cukup jalankan
packages = ['pdfminer.six', 'beautifulsoup4', 'requests',
            'scikit-learn', 'pandas', 'numpy',
            'matplotlib', 'seaborn', 'openpyxl']
```

### Opsi B — Jupyter Lokal

**1. Clone repository**
```bash
git clone https://github.com/<username>/<repo-name>.git
cd <repo-name>
```

**2. Buat virtual environment (opsional tapi direkomendasikan)**
```bash
python -m venv venv

# Windows
venv\Scripts\activate

# macOS / Linux
source venv/bin/activate
```

**3. Install dependensi**
```bash
pip install pdfminer.six beautifulsoup4 requests scikit-learn pandas numpy matplotlib seaborn openpyxl
```

**4. Jalankan Jupyter**
```bash
jupyter notebook CBR_Perdata_Agama.ipynb
```

---

## ☁️ Konfigurasi Google Drive (Colab)

Sebelum menjalankan sel manapun, **wajib jalankan sel Patch Google Drive terlebih dahulu**.

### Langkah 1 — Tentukan path folder Drive Anda

Buka notebook, cari sel bertanda:
```
╔══════════════════════════════════╗
║   PATCH GOOGLE DRIVE             ║
╚══════════════════════════════════╝
```

Ubah baris berikut sesuai lokasi folder di Google Drive Anda:

```python
# Ganti path ini sesuai kebutuhan Anda
GDRIVE_BASE = '/content/drive/MyDrive/CBR_Perdata_Agama'

# Contoh lain:
# GDRIVE_BASE = '/content/drive/MyDrive/Kuliah/Semester6/CBR_PA'
```

### Langkah 2 — Jalankan sel Patch Drive

Sel ini akan otomatis:
- Mount Google Drive ke `/content/drive`
- Membuat struktur folder proyek di Drive
- Mengarahkan semua path output ke Google Drive

### Langkah 3 — Upload PDF Putusan

Setelah patch berhasil dijalankan, upload minimal **30 file PDF putusan** ke:
```
/content/drive/MyDrive/CBR_Perdata_Agama/data/pdf/
```

Nama file bebas, contoh:
```
001_cerai_gugat.pdf
002_harta_bersama.pdf
003_waris.pdf
...
```

> **Catatan:** Jika menjalankan secara lokal (bukan Colab), folder simulasi Drive akan dibuat otomatis di `./gdrive_mock/CBR_Perdata_Agama/`.

---

## ▶️ Cara Menjalankan Pipeline

Jalankan sel-sel notebook **secara berurutan dari atas ke bawah**. Berikut urutan dan penjelasan tiap tahap:

---

### 🔹 Tahap 0 — Persiapan (Wajib dijalankan pertama)

| Urutan | Nama Sel | Fungsi |
|--------|----------|--------|
| 1 | **Patch Google Drive** | Mount Drive & buat struktur folder |
| 2 | **Instalasi & Import Library** | Install & import semua dependensi |
| 3 | **Inisialisasi Struktur Folder & Logging** | Validasi path & setup logging |

---

### 🔹 Tahap 1 — Membangun Case Base

**Prasyarat:** File PDF putusan sudah ada di folder `data/pdf/`

```
Jalankan sel: "Proses Semua File PDF"
```

Sel ini akan:
- Mengekstrak teks dari setiap file PDF
- Membersihkan teks (hapus header/footer, normalisasi karakter)
- Memvalidasi kelengkapan teks (minimal 80% isi tersedia)
- Menyimpan hasil ke `data/raw/case_001.txt`, `case_002.txt`, dst.

**Output:** File `.txt` bersih per putusan di folder `data/raw/`

---

### 🔹 Tahap 2 — Case Representation

```
Jalankan sel: "Proses Case Representation"
Jalankan sel: "Tampilkan Preview Dataset"  (Feature Engineering)
```

Sel ini akan:
- Mengekstrak metadata: nomor perkara, tanggal, jenis perkara, pasal, pihak
- Mengekstrak ringkasan fakta & amar putusan
- Membuat fitur tambahan (`has_cerai`, `has_waris`, `word_count`, dll.)
- Menyimpan ke `cases.csv`, `cases.json`, dan `cases_features.csv`

**Output:** `data/processed/cases_features.csv`

---

### 🔹 Tahap 3 — Case Retrieval

```
Jalankan sel: "TF-IDF Vectorization"
Jalankan sel: "Train/Test Split"
Jalankan sel: "Model 1: SVM"
Jalankan sel: "Model 2: Naive Bayes"
Jalankan sel: "Fungsi Retrieval (Cosine Similarity)"
Jalankan sel: "Buat File Queries untuk Evaluasi"
```

Proses yang terjadi:
- Teks digabung dan direpresentasikan sebagai vektor TF-IDF (5.000 fitur, bigram)
- Data dibagi 80:20 (train:test) dengan stratified sampling
- Dua model dilatih: **SVM (linear kernel)** dan **Naive Bayes**
- Fungsi `retrieve(query, k=5)` tersedia untuk mencari kasus termirip via cosine similarity
- 10 query evaluasi dibuat otomatis dari dataset dan disimpan ke `queries.json`

**Contoh penggunaan fungsi retrieval:**
```python
hasil = retrieve("penggugat menggugat cerai karena perselisihan terus menerus", k=5)
# → [(case_id, similarity_score), ...]
```

**Output:** Model terlatih + `data/eval/queries.json`

---

### 🔹 Tahap 4 — Case Solution Reuse

```
Jalankan sel: "Demo Manual: 5 Kasus Baru"
```

Proses yang terjadi:
- Fungsi `predict_outcome(query, k=5, method='weighted')` dipanggil untuk 5 kasus demo
- Metode `weighted similarity` digunakan untuk memilih solusi terbaik dari top-k
- Hasil prediksi disimpan ke `predictions.csv`

**Contoh penggunaan:**
```python
hasil = predict_outcome(
    "penggugat meminta pembagian harta bersama berupa rumah dan kendaraan",
    k=5,
    method='weighted'   # atau 'majority'
)
print(hasil['predicted_label'])
print(hasil['predicted_solution'])
```

**Output:** `data/results/predictions.csv`

---

### 🔹 Tahap 5 — Model Evaluation

```
Jalankan sel: "Jalankan Evaluasi Retrieval"
Jalankan sel: "Evaluasi Klasifikasi (SVM vs Naive Bayes)"
Jalankan sel: "Visualisasi Perbandingan Model"
Jalankan sel: "Analisis Kegagalan (Error Analysis)"
```

Hasil evaluasi meliputi:
- **Hit@5** — persentase query yang ground truth-nya masuk top-5 retrieval
- **Accuracy, Precision, Recall, F1-score** — SVM vs Naive Bayes
- **Confusion matrix** SVM
- **Pie chart** distribusi jenis perkara
- **Error analysis** — kasus yang salah prediksi beserta rekomendasinya

**Output:** `data/eval/retrieval_metrics.csv`, `data/results/evaluation_chart.png`

---

### 🔹 Tahap Opsional — Revisi & Retain

```
Jalankan sel: "Fungsi Revisi & Retain"
```

Menambahkan kasus baru yang sudah terbukti solusinya ke dalam case base:

```python
retain_case({
    'no_perkara'   : '999/Pdt.G/2024/PA.XXX',
    'jenis_perkara': 'cerai gugat',
    'text_full'    : 'deskripsi lengkap kasus baru...',
    'amar_putusan' : 'mengabulkan gugatan penggugat...'
})
```

---

### 🔹 Sinkronisasi ke Google Drive

```
Jalankan sel: "Sinkronisasi Semua Output ke Google Drive"
```

Menyalin semua file output (CSV, JSON, PNG, log) ke Google Drive secara otomatis.

---

## 📊 Output yang Dihasilkan

| File | Lokasi | Isi |
|------|--------|-----|
| `case_XXX.txt` | `data/raw/` | Teks bersih per putusan |
| `cases.csv` | `data/processed/` | Dataset representasi kasus |
| `cases_features.csv` | `data/processed/` | Dataset + fitur tambahan |
| `queries.json` | `data/eval/` | Query evaluasi + ground truth |
| `retrieval_metrics.csv` | `data/eval/` | Metrik SVM & Naive Bayes |
| `prediction_metrics.csv` | `data/eval/` | Hasil evaluasi retrieval |
| `predictions.csv` | `data/results/` | Prediksi 5 kasus demo |
| `evaluation_chart.png` | `data/results/` | Visualisasi perbandingan model |
| `cleaning.log` | `logs/` | Log proses pembersihan teks |

---

## 🔄 Alur Siklus CBR

```
PDF Putusan (≥30 dokumen)
        │
        ▼
┌─────────────────┐
│  TAHAP 1        │  Ekstraksi teks → Cleaning → data/raw/*.txt
│  Case Base      │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  TAHAP 2        │  Metadata + Feature Engineering → cases.csv / cases.json
│  Representation │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  TAHAP 3        │  TF-IDF → SVM / Naive Bayes → retrieve(query, k=5)
│  Retrieval      │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  TAHAP 4        │  Top-k cases → Weighted Similarity → predict_outcome()
│  Solution Reuse │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  TAHAP 5        │  Accuracy / Precision / Recall / F1 / Error Analysis
│  Evaluation     │
└─────────────────┘
```

---

## 📝 Catatan Penting

- **Urutan sel wajib diikuti** — beberapa sel bergantung pada variabel dari sel sebelumnya
- **Patch Google Drive harus dijalankan pertama** sebelum sel apapun
- Jika dataset < 2 kasus per label, stratified split akan dinonaktifkan otomatis
- Fungsi `retain_case()` bersifat opsional namun meningkatkan performa seiring waktu
- Semua log aktivitas tersimpan di `logs/cleaning.log`
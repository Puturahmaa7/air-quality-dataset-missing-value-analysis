# Air Quality Dataset Missing Value Imputation Evaluation

## Deskripsi Proyek

Proyek ini bertujuan untuk mengevaluasi dan menentukan metode penanganan *missing values* pada Beijing Multi-Site Air-Quality Dataset. Analisis berfokus pada tujuh variabel, yaitu `PM2.5`, `PM10`, `DEWP`, `TEMP`, `PRES`, `RAIN`, dan `WSPM`.

Evaluasi dilakukan menggunakan simulasi *artificial missingness* terhadap data yang memiliki nilai lengkap (*ground truth*). Beberapa metode imputasi dibandingkan berdasarkan kategori panjang gap untuk mengetahui metode yang memiliki performa dan konsistensi terbaik.

Selain membandingkan akurasi imputasi, evaluasi juga mempertimbangkan kondisi *co-missing*, hasil trial konfirmasi, serta ketersediaan gap aktual sebagai dasar kekuatan dukungan data dalam penyusunan rekomendasi.

## Dataset

Dataset yang digunakan adalah **Air-Quality-Dataset**, yang berisi pengukuran kualitas udara dan kondisi meteorologi dari 12 stasiun pemantauan di Beijing, Tiongkok.

- Periode pengamatan: 1 Maret 2013 – 28 Februari 2017
- Jumlah stasiun: 12
- Interval pengamatan: Per jam (*hourly*)
- Jumlah observasi setelah penggabungan: 420.768 baris
- Format: CSV

Dataset terdiri dari 12 file CSV berdasarkan masing-masing stasiun pemantauan dan disimpan di dalam folder `data/`.

## Struktur Repository

Seluruh proses eksperimen, evaluasi, dan penyusunan rekomendasi dilakukan dalam satu notebook utama.

```text
air-quality-dataset-missing-value-analysis/
│
├── data/
│   └── PRSA_Data_*.csv
│
├── Missing_Value_Imputation_Evaluation_and_Recommendation.ipynb
└── README.md
```

## Ruang Lingkup Analisis

### 1. Data Loading

- Membaca 12 file CSV dari folder `data/`.
- Menggabungkan seluruh data stasiun menjadi satu DataFrame (`air_df`).
- Memeriksa jumlah observasi dari setiap stasiun.

### 2. Konfigurasi Simulasi dan Evaluasi

- Menentukan tujuh variabel target yang dievaluasi.
- Menentukan fitur meteorologi sebagai variabel pendukung.
- Menetapkan enam kategori panjang gap:
  - 1–3 jam
  - 4–6 jam
  - 7–12 jam
  - 13–24 jam
  - 25–72 jam
  - Lebih dari 72 jam
- Menentukan random seed untuk fase seleksi dan konfirmasi.
- Menetapkan dua skenario simulasi, yaitu `independent` dan `co-missing`.

### 3. Persiapan Data Ground Truth

- Menggabungkan informasi tahun, bulan, hari, dan jam menjadi timestamp.
- Mengurutkan data berdasarkan stasiun dan waktu.
- Memeriksa duplikasi berdasarkan kombinasi stasiun dan timestamp.
- Membentuk data *complete-case* untuk setiap variabel target.
- Menyimpan nilai asli sebagai acuan evaluasi imputasi.

### 4. Identifikasi dan Kategorisasi Gap Aktual

- Mengidentifikasi blok *missing values* yang terjadi secara berurutan pada setiap stasiun.
- Memastikan gap memiliki nilai pengamatan sebelum dan sesudahnya.
- Menghitung panjang setiap gap aktual.
- Mengelompokkan gap berdasarkan enam kategori durasi.
- Merangkum jumlah gap dan total jam kehilangan data untuk setiap variabel.

### 5. Simulasi Artificial Missingness

Membentuk blok *missing values* buatan pada data *ground truth* berdasarkan distribusi panjang gap aktual.

Dua skenario yang digunakan:

- `independent`: simulasi kehilangan data pada satu stasiun tanpa menambahkan pola kehilangan data dari stasiun lain.
- `co-missing`: simulasi kehilangan data dengan mempertimbangkan pola kehilangan data secara bersamaan pada stasiun lain berdasarkan template gap aktual.

### 6. Evaluasi Metode Imputasi

Membandingkan enam metode imputasi pada blok missing values buatan yang sama:

1. Linear Interpolation
2. Conditional Median
3. KNN Imputer
4. Neighbor Station
5. Interp + NC (level)
6. Interp + NC (anomaly)

Evaluasi performa menggunakan beberapa metrik:

- **RMSE (Root Mean Squared Error):** Mengukur besarnya kesalahan imputasi dengan memberikan penalti lebih besar pada kesalahan yang tinggi.
- **MAE (Mean Absolute Error):** Mengukur rata-rata nilai absolut kesalahan imputasi.
- **Bias:** Mengukur kecenderungan hasil imputasi lebih tinggi atau lebih rendah dari nilai aslinya.
- **Standard Deviation Ratio:** Membandingkan variasi hasil imputasi terhadap variasi nilai asli.
- **Fallback Rate:** Mengukur frekuensi penggunaan metode pengganti ketika metode utama tidak dapat memberikan hasil imputasi.

### 7. Seleksi dan Konfirmasi Metode

Evaluasi dibagi menjadi dua fase menggunakan kelompok random seed yang berbeda.

- **Fase seleksi:** Memilih metode berdasarkan rata-rata RMSE dan mengidentifikasi metode terbaik kedua.
- **Fase konfirmasi:** Menguji metode terpilih menggunakan seed berbeda dan membandingkan performanya dengan Linear Interpolation sebagai metode referensi.

Analisis juga mempertimbangkan jumlah kemenangan (*wins*), selisih relatif RMSE, dan konsistensi hasil antartrial.

### 8. Analisis Dampak Co-Missing

- Membandingkan performa metode imputasi pada skenario `independent` dan `co-missing`.
- Menganalisis perubahan RMSE akibat perbedaan kondisi kehilangan data.
- Memeriksa proporsi kehilangan data pada variabel lain selama gap target berlangsung.
- Membandingkan pola *co-missing* aktual dengan hasil simulasi.

### 9. Penyusunan Rekomendasi Metode Imputasi

Menyusun aturan penanganan *missing values* berdasarkan kombinasi variabel target dan kategori panjang gap.

Rekomendasi mempertimbangkan:

- Performa metode pada fase konfirmasi.
- Perbandingan dengan metode referensi.
- Konsistensi hasil antartrial.
- Ketersediaan gap aktual.
- Penggunaan *replacement* dalam simulasi.

Hasil keputusan memisahkan dua aspek:

**Status keputusan metode:**
- `didukung`
- `batas`
- `tidak ada gap aktual`

**Kekuatan dukungan data:**
- `memadai`
- `terbatas`
- `lemah`

Hasil akhir disajikan dalam tabel keputusan dan ringkasan metode rekomendasi berdasarkan variabel serta kategori panjang gap.

## Metode Evaluasi

| Komponen | Konfigurasi |
|---|---|
| Variabel target | 7 variabel |
| Skenario simulasi | 2 skenario |
| Metode imputasi | 6 metode |
| Kategori panjang gap | 6 kategori |
| Fase seleksi | 5 random seed |
| Fase konfirmasi | 5 random seed |
| Metode referensi | Linear Interpolation |
| Ambang peningkatan RMSE relatif | 2% |
| Minimum trial konsistensi | 3 trial |

## Teknologi dan Library

Proyek ini menggunakan Python dan beberapa library berikut:

- **Pandas** – pengolahan dan manipulasi data.
- **NumPy** – komputasi numerik.
- **SciPy** – fungsi dan analisis statistik.
- **Scikit-learn** – preprocessing, KNN Imputer, dan evaluasi performa imputasi.

## Cara Menjalankan

### 1. Clone Repository

```bash
git clone https://github.com/Puturahmaa7/air-quality-dataset-missing-value-analysis.git
```

### 2. Masuk ke Direktori Repository

```bash
cd air-quality-dataset-missing-value-analysis
```

### 3. Instal Library

```bash
pip install numpy pandas scipy scikit-learn jupyter
```

### 4. Pastikan Dataset Tersedia

Pastikan seluruh 12 file CSV berada di dalam folder `data/` dengan format nama:

```text
PRSA_Data_*.csv
```

### 5. Jalankan Jupyter Notebook

```bash
jupyter notebook
```

Buka notebook:

`Missing_Value_Imputation_Evaluation_and_Recommendation.ipynb`

Jalankan seluruh cell secara berurutan dari awal hingga akhir.

## Batasan Analisis

- Evaluasi menggunakan data *complete-case*, sehingga tidak sepenuhnya merepresentasikan seluruh kondisi dataset asli.
- Lokasi gap buatan dipilih secara acak berdasarkan distribusi gap aktual.
- Jumlah blok simulasi tidak selalu merepresentasikan jumlah gap aktual unik karena dapat menggunakan *replacement*.
- Fase seleksi dan konfirmasi menggunakan data dasar serta kumpulan gap aktual yang sama, sehingga belum setara dengan validasi pada dataset eksternal.
- Hasil perbandingan metode merupakan evaluasi eksploratif dan bukan bukti signifikansi statistik.
- Perubahan performa pada skenario *co-missing* tidak secara otomatis membuktikan hubungan sebab-akibat.
- Performa metode pada gap asli yang nilai sebenarnya tidak diketahui belum dapat diverifikasi secara langsung.
- Rekomendasi untuk variabel dengan jumlah gap aktual terbatas perlu diinterpretasikan berdasarkan tingkat dukungan datanya.

## Hasil Akhir

Proyek ini menghasilkan evaluasi komparatif enam metode imputasi untuk tujuh variabel target berdasarkan enam kategori panjang gap dan dua skenario simulasi.

Hasil evaluasi digunakan untuk menyusun rekomendasi metode penanganan *missing values* dengan mempertimbangkan performa, konsistensi, kondisi *co-missing*, serta ketersediaan gap aktual.

Rekomendasi akhir bersifat eksploratif dan digunakan sebagai dasar empiris dalam menentukan strategi penanganan *missing values* pada Beijing Air Quality Dataset.

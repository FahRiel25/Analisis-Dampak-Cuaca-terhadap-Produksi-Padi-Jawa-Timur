# DATA-ENGINEERING  
# Proyek : Pengaruh Cuaca terhadap Produktifitas Padi di Jawa Timur

---

## Kontributor

| Nama Lengkap                       | NIM         | Peran                |
|------------------------------------|-------------|----------------------|
| Jimli Dwi Assiddiqi                   | 244311015   | Data Analyst        |
| Muhsyam Fahriel S.                     | 244311021   | Data Engineer         |
| Satriyo Wicaksono Y.M.                     | 244311027   | Project Manager      |

---

## Deskripsi Proyek  
Project ini dikembangkan untuk menganalisis hubungan antara curah hujan dan suhu rata-rata dengan perubahan produksi padi di berbagai kabupaten/kota di Jawa Timur. Dengan tujuan memahami dampak cuaca dan memberikan wawasan yang membantu pengambilan keputusan dalam menghadapi risiko pertanian akibat perubahan iklim. Selain itu, project ini menggabungkan data statistik historis dengan data cuaca publik melalui alur pengolahan data untuk melihat tren dari Januari 2018 hingga Desember 2024 di 38 Kabupaten/Kota.
---

## Manfaat Data / Use Case  
- **Tujuan Proyek:** Menyediakan dataset terintegrasi yang menggabungkan parameter cuaca (suhu dan presipitasi) dengan hasil produksi padi per wilayah di Jawa Timur.
- **Manfaat:**  
  - Menyediakan clean dataset yang siap digunakan untuk analisis korelasi statistik lanjutan. 
  - Mendukung pembuatan dashboard visualisasi tren produksi versus cuaca agar lebih mudah dipahami oleh masyarakat umum dan kelompok tani.
  - Menghasilkan laporan analisis mengenai kondisi atau bulan apa yang paling signifikan memengaruhi naik turunnya hasil panen.

---

## Serving Analisis  
Data hasil proses ETL akan disimpan sebagai dataset terintegrasi yang terstruktur. Dataset ini dapat diakses untuk eksplorasi lebih lanjut atau dihubungkan ke platform visualisasi guna membuat grafik tren produksi dan cuaca. Penyimpanan ini memungkinkan pengguna untuk melihat korelasi langsung secara interaktif.

## Serving Machine Learning  
Dataset bersih yang dihasilkan digunakan untuk membangun model klasifikasi produktivitas padi menggunakan Random Forest Classifier. Model memanfaatkan variabel cuaca seperti suhu, curah hujan, dan kelembapan untuk mengelompokkan tingkat produktivitas padi ke dalam beberapa kategori. Hasil model dievaluasi menggunakan Accuracy Score, Classification Report, dan Confusion Matrix, serta didukung analisis Feature Importance untuk mengetahui faktor cuaca yang paling berpengaruh terhadap produktivitas padi.

---

# Pipeline
## Extract ( Pengambilan Data ) 
- **Sumber Data:**  
  - Produksi padi Menurut Kabupaten atau Kota – BPS Jawa Timur
(https://jatim.bps.go.id/id/statistics-table/2/NTc5IzI=/produksi-padi-menurut-kabupaten-kota--ton-.html)
  - Data Cuaca Historis (Suhu & Presipitasi) – Open Meteo API
(https://open-meteo.com/en/docs/historical-weather-api)


- **Metode Pengambilan:**  
  **Data File (BPS Jawa Timur):**  
    - Mengunduh data berformat CSV yang berisi volume produksi padi dari rentang waktu Januari 2018 hingga Desember 2024 dengan kualitas akurasi tinggi.

  **Public API (Open Meteo):**  
    - Menggunakan endpoint API Open Meteo untuk menarik data cuaca historis harian (temperature_2m_mean, precipitation_sum) berdasarkan parameter latitude dan longitude wilayah Jawa Timur pada rentang waktu yang sama. 
---

## Transform ( Pembersihan & Transformasi )   
- **Pembersihan:**  
  - Menghapus kolom dan baris kosong (`dropna()`), Serta mengubah tipe data dari beberapa kolom (contoh:  di kolom `time` dari `object/string` → `datetime`).
  - Normalisasi nama kolom agar mudah dipahami (contoh: `precipitation_sum (mm)` → `Total_Curah_Hujan`)

- **Transformasi:**  
  - Menggabungkan dua dataset (`df_padi_long`, `df_luas_long`) berdasarkan kolom `Wilayah`, `Tahun`, dan `Bulan`.
  - Menggabungkan dua dataset ( `df_padi_luas`, `df_cuaca_bulanan`) berdasarkan kolom `Wilayah`, `Tahun`, dan `Bulan`.
  - Menyelaraskan frekuensi waktu data (contoh: Frekuensi waktu pada data cuaca dari `Hari` menjadi `Bulan` dengan mengambil `Bulan` dan `Tahun` dan masing - masing di taruh di kolom `Bulan` dan `Tahun`)

---

## Load ( Pemindahan ke Target ) 
- **Target:**  
  - Sebuah tabel baru di dalam database PostgreSQL pada server Aiven yang digunakan sebagai penyimpanan akhir dataset hasil ETL. Tabel ini menjadi sumber data utama untuk analisis lanjutan, visualisasi dashboard, dan machine learning.

- **Metode:**  
  - Membuat koneksi dengan database menggunakan `SQLAlchemy`
  - Membuat tabel `produksi_padi_cuaca` menggunakan perintah SQL `CREATE TABLE` untuk menyesuaikan struktur data yang akan disimpan.  
  - Memuat dataset hasil transformasi (`df_final`) ke database menggunakan fungsi `to_sql()` dari pandas.
  - Data diverifikasi dengan menjalankan query SQL dan menampilkan beberapa baris data untuk memastikan proses load berhasil.

---

## Arsitektur / Workflow ETL

- **Alur Modular:**
  - Mengambil data produksi padi dan luas panen dari BPS Jawa Timur serta data cuaca historis dari Open Meteo.
  - Melakukan pembersihan data, konversi tipe data, dan agregasi data cuaca harian menjadi bulanan.
  - Menggabungkan data produksi padi, luas panen, dan cuaca berdasarkan kolom `Wilayah`, `Tahun`, dan `Bulan`.
  - Menghitung nilai produktivitas padi dan menghasilkan dataset akhir (`df_final`).
  - Memuat dataset ke database MySQL Aiven untuk digunakan pada analisis dan machine learning.  

- **Tools yang Digunakan:**

  - Python 3.x
  - Library: `pandas`, `numpy`, `glob`, `os`, `sqlalchemy`, `pymysql`, `matplotlib`, `seaborn`, `sklearn`, `scipy`
  - Platform: Google Colab
  - Database: MySQL (Aiven Cloud)
  - Visualisasi: Data Studio

---

## Kode Program  
- **Struktur Kode:**  
  - Terdapat 2 notebook: satu untuk ETL, satu untuk Machine Learning.
  - Nama variabel dan fungsi dibuat deskriptif untuk memudahkan pengelolaan data, seperti `df_padi_long`, `df_luas_long`, `df_cuaca_bulanan`, `df_final`, dan `Produktivitas_Ton_Ha`.
    
- **Machine Learning:**  
  - Model utama: Random Forest Classifier untuk mengklasifikasikan tingkat produktivitas padi berdasarkan faktor cuaca dan hasil pertanian.
  - Feature yang digunakan meliputi `Suhu_4Bln`, `Curah_4Bln`, dan `Kelembapan_4Bln`, dengan target berupa `Kategori_Produktivitas`.
  - Dataset dibagi menjadi data training dan testing dengan rasio 80:20 menggunakan `train_test_split`.
  - Evaluasi model dilakukan menggunakan Accuracy Score, Classification Report, dan Confusion Matrix.
  - Analisis Feature Importance digunakan untuk mengidentifikasi faktor cuaca yang paling berpengaruh terhadap produktivitas padi.

- **Link Projek:** 
  - ETL Pipeline: [https://colab.research.google.com/drive/1CIGswRpvVqIlc8wK1JrlDxMIHt810rE5?usp=sharing](https://colab.research.google.com/drive/1CIGswRpvVqIlc8wK1JrlDxMIHt810rE5?usp=sharing)
  - Machine Learning: [https://colab.research.google.com/drive/1JwzDRRdS1rDK38jUZiwzFvQpwqVucDfx?usp=sharing](https://colab.research.google.com/drive/1JwzDRRdS1rDK38jUZiwzFvQpwqVucDfx?usp=sharing)
  - Data Studio :[(https://datastudio.google.com/reporting/855631cb-b1d9-4013-9674-8f1c2ff0d0cb)](https://datastudio.google.com/reporting/855631cb-b1d9-4013-9674-8f1c2ff0d0cb)

---
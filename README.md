# UMKMentor — Product Success Predictor

Proyek machine learning untuk memprediksi peluang suatu produk menjadi **laku** atau **tidak laku** berdasarkan data marketplace. Model ini dirancang sebagai alat bantu pengambilan keputusan bagi pelaku UMKM dalam menentukan strategi produk, harga, stok, dan promosi secara lebih berbasis data.

## Gambaran Umum

Proyek ini membangun model klasifikasi berbasis machine learning yang memprediksi apakah suatu produk memiliki potensi untuk menjadi produk **Laku** atau **Tidak Laku**.

Prediksi dilakukan menggunakan berbagai atribut marketplace seperti harga, stok, diskon, rating, atribut toko, dan statistik level kategori. Model kemudian digunakan sebagai komponen utama pada **UMKMentor**, sebuah solusi AI untuk Business Intelligence dan Market Insights yang membantu pelaku UMKM mengambil keputusan bisnis secara lebih objektif dan berbasis data.

## Permasalahan

Banyak pelaku UMKM memilih produk berdasarkan tren atau asumsi, bukan berdasarkan data. Hal ini bisa meningkatkan risiko:

- memilih produk dengan permintaan yang lemah,
- menetapkan harga yang kurang kompetitif,
- memberi diskon tanpa strategi yang jelas,
- mengabaikan sinyal kepercayaan seperti status official store dan rating,
- mengambil keputusan produk tanpa memahami pola spesifik tiap kategori.

Model ini dirancang untuk mengurangi risiko tersebut dengan memberikan prediksi potensi produk yang sederhana namun berbasis data.

## Dataset

Proyek ini menggunakan **Tokopedia Product and Review Dataset** sebagai sumber data real-world utama.

### Ringkasan Dataset
- **Sumber:** data publik produk dan review Tokopedia
- **Jumlah produk:** 5.553
- **Jumlah review:** 1M+
- **Jumlah kategori:** 24 kategori produk
- **Bahasa:** Indonesia
- **Periode pengumpulan:** 2025

### File Utama
- `tokopedia_products_with_review.csv`
- `tokopedia_products_with_review.json`

### Isi Dataset
Dataset memuat:
- informasi produk,
- informasi harga,
- informasi penjualan,
- atribut toko,
- teks review dan rating.

### Kolom Produk Penting
- `product_id`
- `category`
- `name`
- `count_sold`
- `discounted_price`
- `preorder`
- `price`
- `stock`
- `gold_merchant`
- `is_official`
- `is_topads`
- `rating_average`
- `shop_location`

### Kolom Review
- `review_id`
- `variant_name`
- `message`
- `review_rating`
- `review_time`
- `review_timestamp`
- `review_response`
- `review_like`
- `bad_rating_reason`

## Ekstensi Dataset Sintetis

Dataset Tokopedia yang digunakan pada proyek ini tidak memiliki representasi yang memadai untuk kategori **Pertukangan**, sementara kategori tersebut diperlukan sebagai salah satu kategori yang didukung oleh UMKMentor.

Untuk mengatasi keterbatasan tersebut, ditambahkan **800 baris data sintetis** kategori Pertukangan yang dirancang menyerupai karakteristik data marketplace nyata dan kemudian digabungkan ke dalam dataset pelatihan model.

### Alasan Penambahan
Data sintetis ini dibuat untuk:
- menyediakan representasi kategori yang belum tersedia pada dataset asli,
- memungkinkan model mempelajari pola untuk produk Pertukangan,
- membuat model tetap bisa digunakan untuk inferensi pada kategori yang sebelumnya tidak punya data memadai,
- mendukung skenario prediksi produk yang lebih luas dalam konteks UMKM.

### Karakteristik Data Sintetis
Data sintetis dibuat agar menyerupai pola data marketplace, termasuk:
- pola harga produk,
- variasi stok,
- perilaku diskon,
- atribut kepercayaan toko,
- pola penjualan pada level kategori.

### Keterbatasan
Karena kategori ini bersifat sintetis dan bukan sepenuhnya berasal dari data Tokopedia asli, prediksi untuk kategori **Pertukangan** perlu ditafsirkan dengan lebih hati-hati dibanding kategori yang seluruhnya didukung data real-world.

## Persiapan Data

Pada notebook, data mentah dibersihkan dan direduksi menjadi tabel final untuk pemodelan.

### Dataset Final untuk Modeling
- **Baris awal yang dimuat:** 6.353
- **Baris final setelah cleaning:** 3.039
- **Jumlah fitur final:** 22
- **Train set:** 2.431 baris
- **Test set:** 608 baris

### Definisi Label
Target label dibentuk sebagai:

`is_laku = 1` jika `count_sold > cat_sold_median`  
`is_laku = 0` selain itu

Artinya model memprediksi apakah sebuah produk memiliki performa penjualan di atas median kategori tersebut.

Penggunaan median per kategori membuat label lebih adil dibanding membandingkan produk lintas kategori yang pola penjualannya sangat berbeda.

## Feature Engineering

Selain membangun fitur baru berbasis statistik kategori, proses feature engineering juga melibatkan audit fitur menggunakan analisis SHAP dan evaluasi bisnis untuk memastikan bahwa fitur yang digunakan benar-benar relevan terhadap kondisi marketplace.

### Ide Fitur Utama
Fitur final mencakup:
- transformasi log pada harga,
- posisi harga relatif terhadap median kategori,
- ranking harga di dalam kategori,
- ranking stok di dalam kategori,
- kecukupan stok,
- keberadaan diskon,
- persentase diskon,
- sinyal kepercayaan toko,
- one-hot encoding kategori,
- statistik referensi level kategori.

### Audit dan Perbaikan Fitur
Selama proses pengembangan model, dilakukan penyesuaian fitur agar lebih konsisten dengan perilaku inferensi di dunia nyata:

- `is_topads` dihapus dari input model karena dapat menimbulkan korelasi yang menyesatkan pada data observasional marketplace.
- `gold_merchant` ditambahkan sebagai fitur kepercayaan yang berdiri sendiri.
- `is_official` tetap dipakai sebagai fitur trust yang langsung.
- pendekatan trust yang terlalu ambigu disederhanakan agar input model lebih selaras dengan logika deployment.

### Jumlah Fitur Final
Total fitur final yang digunakan model: **22**

## Model yang Dicoba

Beberapa algoritma klasifikasi diuji dalam notebook, yaitu:

- Logistic Regression
- Linear SVM
- Random Forest
- Gradient Boosting

Selain itu, model juga dituning dan dikalibrasi agar output probabilitas lebih stabil dan berguna untuk decision support.

## Model Terbaik

Model final yang dipilih adalah:

**Gradient Boosting yang sudah dituning dan dikalibrasi (sigmoid)**

### Performa Final
- **Test AUC:** 0.745
- Output probabilitas dikalibrasi dengan sigmoid agar lebih mudah diinterpretasikan.

## Insight Utama dari Notebook

Sinyal paling kuat yang terlihat pada model antara lain:

- `discount_pct`
- `rating_average`
- `stock_is_enough`
- `cat_pct_official`
- pola kategori seperti `cat_pertukangan`

Hal ini menunjukkan bahwa strategi diskon, rating produk, kecukupan stok, dan konteks kategori merupakan indikator penting untuk memprediksi potensi produk laku.

## Explainability

Notebook ini menggunakan **SHAP** untuk interpretasi global model.

### Fungsi SHAP di Proyek Ini
Analisis interpretabilitas model dilakukan menggunakan SHAP (SHapley Additive Explanations) untuk memahami kontribusi masing-masing fitur terhadap prediksi model.

Hasil analisis SHAP digunakan untuk:
- mengidentifikasi fitur yang paling berpengaruh terhadap prediksi,
- melakukan audit dan evaluasi fitur,
- mendukung proses perbaikan feature engineering,
- membantu interpretasi perilaku model setelah proses training.

### Batasan SHAP pada Implementasi Saat Ini
Pada notebook final, teks rekomendasi untuk user di fungsi `predict_product()` masih berasal dari **logika aturan bisnis (rule-based)**.

Jadi:
- SHAP dipakai untuk **analisis interpretabilitas model**,
- sedangkan saran yang tampil di inferensi masih dihasilkan oleh aturan bisnis yang ditulis di fungsi prediksi.

## Kategori yang Didukung

Fungsi inferensi mendukung kategori berikut:

- elektronik
- hiburan
- olahraga
- kecantikan
- makanan_minuman
- fashion
- pertukangan

## Inferensi

Notebook menyediakan fungsi `predict_product()` yang menerima input:

- kategori produk
- harga jual
- status official store
- status Gold Merchant
- stok
- rating rata-rata
- harga setelah diskon

Fungsi ini menghasilkan:
- label prediksi,
- skor probabilitas,
- tingkat risiko,
- saran bisnis sederhana.

### Catatan Penting Input
Input diskon pada fungsi ini menggunakan **harga setelah diskon** (`discounted_price`), bukan persen diskon.

### Alur Inferensi
Alur inferensi menggunakan statistik kategori dan logika fitur yang sama dengan proses training, sehingga output dapat dipakai sebagai bantuan keputusan di website.

## Catatan Integrasi untuk Full Stack

Agar website dan backend selaras dengan notebook final, beberapa hal perlu diperhatikan:

- `discounted_price` harus dihitung sebelum dikirim ke model.
- `TopAds` tidak lagi digunakan sebagai input model.
- `gold_merchant` perlu ditambahkan ke UI / API input.
- struktur input model harus sama dengan `FEATURE_COLS` final yang dipakai saat training.

### Contoh Konversi Diskon
Jika frontend menerima:
- `harga_jual = 48000`
- `diskon_pct = 20`

maka backend harus mengubahnya menjadi:

`discounted_price = 48000 * (1 - 20 / 100) = 38400`

lalu nilai tersebut dikirim ke model.

## Artefak yang Diekspor

Notebook mengekspor file berikut:

- `tokped_classifier.pkl`
- `feature_cols.json`
- `category_levels.json`
- `category_stats.json`
- `category_aliases.json`
- `cat_rating_median.json`
- `cat_discount_rate.json`
- `model_metadata.json`

## Tech Stack

- Python
- Pandas
- NumPy
- Scikit-learn
- Gradient Boosting
- Matplotlib
- Seaborn
- SHAP
- Joblib

## Manfaat Bisnis

Proyek ini membantu pelaku UMKM untuk:
- mengurangi risiko memilih produk,
- membandingkan potensi produk secara lebih objektif,
- memperbaiki keputusan harga,
- memahami perilaku kategori,
- mengambil keputusan dengan lebih cepat dan berbasis data.

## Keterbatasan

Beberapa keterbatasan yang masih ada pada versi ini:

- sinyal model bisa lebih lemah pada kategori tertentu,
- `is_topads` dapat menghasilkan pola korelasi yang menyesatkan jika diperlakukan sebagai sinyal kausal,
- data sintetis dipakai untuk kategori `pertukangan`,
- model masih lebih tepat dianggap sebagai MVP daripada sistem forecasting yang siap produksi penuh,
- teks rekomendasi pada inferensi belum sepenuhnya SHAP-based untuk level produk individu.

## Cara Menggunakan

1. Muat file model dan metadata yang diekspor.
2. Siapkan input data dengan logika fitur yang sama seperti notebook.
3. Konversi input diskon menjadi `discounted_price` sebelum inferensi.
4. Panggil logika `predict_product()` atau model secara langsung.
5. Gunakan skor probabilitas sebagai sinyal bantu pengambilan keputusan, bukan sebagai kebenaran mutlak.

## Konteks Proyek

Notebook ini merupakan bagian dari proyek **UMKMentor** untuk business intelligence dan market insights berbasis AI, dengan fokus membantu pelaku UMKM membuat keputusan yang lebih baik berdasarkan data produk, review, dan perilaku marketplace.

# REAL-TIME DATA ENGINEERING AND PREDICTIVE MODELING FOR TOP US STOCKS USING PYSPARK [MATA KULIAH BIG DATA (A)]
## Kelompok 4 Kelas Big Data A

Proyek ini bertujuan untuk membangun pipeline Big Data Realtime dan Offline untuk melakukan analisis dan prediksi harga saham dari 50 perusahaan teratas berdasarkan data yang diperoleh dari Yahoo Finance API. Sistem ini dikembangkan menggunakan kombinasi Kafka, PostgreSQL, dan Metabase sebagai alat visualisasi, serta Python sebagai bahasa pemrograman utama. Model prediktif dibangun menggunakan pendekatan ARIMA (Autoregressive Integrated Moving Average) dari pustaka statsmodels.

**1. Fitur Utama**

a. Streaming Data Saham Real-time dari Yahoo Finance

   Data saham dari 50 saham teratas diambil secara real-time menggunakan modul yfinance, kemudian dikirimkan ke topik Kafka (stock_prices) melalui Kafka Producer dalam format JSON. Selain streaming, data historis juga disimpan dalam file data/historical_stock_prices.csv, yang berisi lebih dari 94.000 baris data mencakup kolom Symbol, Datetime, Close, dan Volume.

b. Pengolahan dan Pembersihan Data menggunakan Python

   Proses ini dilakukan dengan Python menggunakan pustaka pandas. Tahapan yang dilakukan mencakup:
- Parsing format Datetime dan konversi ke format waktu standar.
- Konversi tipe data harga dan volume menjadi numerik.
- Penghapusan data yang mengandung nilai hilang.
- Perhitungan fitur turunan seperti persentase perubahan harga harian (close_ptc) dan rata-rata volume.

c. Penyimpanan Data ke dalam PostgreSQL

   Data yang telah dibersihkan kemudian disimpan ke dalam basis data PostgreSQL bernama stockdb, dengan tabel utama bernama stock_prices_cleaned. Penyimpanan dilakukan melalui koneksi SQLAlchemy dan psycopg2-binary.
   
d. Visualisasi Interaktif menggunakan Metabase

   Metabase digunakan untuk menampilkan visualisasi interaktif berdasarkan data dalam PostgreSQL. Dashboard yang dihasilkan mencakup informasi sebagai berikut.
- Top 5 Predicted Price Gainers
- Top 5 Predicted Price Losers
- ML Decision Summary
- Current vs Prediction Price Gainers
- Most Volatile Stock
- Price and Volume Movement for ORCL Stock 

e. Prediksi Harga Saham Menggunakan Model ARIMA
   
   Pemodelan dilakukan dengan menggunakan algoritma ARIMA (Autoregressive Integrated Moving Average), yang diimplementasikan melalui pustaka statsmodels. Model ARIMA dipilih karena kemampuannya menangani data runtun waktu (time series) yang mengandung pola musiman, tren, dan fluktuasi harga. Model ini dilatih pada data historis masing-masing saham, kemudian digunakan untuk menghasilkan prediksi harga saham di waktu mendatang. Hasil prediksi disimpan dalam tabel prediction_result_arima di PostgreSQL dan divisualisasikan di Metabase.

**2. Setup dan Instalasi** 

 1) Prasyarat Tools
    - Docker
      
      Merupakan platform yang digunakan untuk menjalankan seluruh layanan proyek (Kafka, Zookeeper, PostgreSQL, Metabase, dan Kafka UI) secara otomatis dan terisolasi dalam lingkungan kontainer.
      
    - Apache Kafka

       Digunakan sebagai sistem message broker untuk mengelola aliran data secara real-time. Kafka menerima data dari producer (skrip Python) dan menyediakannya untuk consumer (juga skrip Python) yang kemudian akan menyimpan data ke PostgreSQL.

    - Apache Zookeeper

    Berfungsi sebagai layanan koordinasi yang dibutuhkan oleh Kafka untuk pengelolaan metadata, sinkronisasi antar broker, dan pemeliharaan cluster Kafka agar tetap stabil.
    
    - Python 3.5+
      
    Merupakan bahasa pemrograman utama yang digunakan untuk mengambil data saham dengan yfinance, mengirim data ke Kafka dengan, memproses data, dan menyimpan data ke PostgreSQL.

    - PostgreSQL (penyimpanan database)
      
      Digunakan sebagai sistem manajemen basis data relasional untuk menyimpan data saham, baik historis maupun data hasil streaming.
      
    - Metabase (visualisasi)

      Platform visualisasi data berbasis web yang digunakan untuk menampilkan data dari PostgreSQL secara interaktif dan informatif.
      
 2) Instalasi Library Python

    Untuk menunjang kebutuhan pemrograman, seluruh pustaka Python yang dibutuhkan telah didefinisikan di dalam file requirements.txt. File ini memuat daftar pustaka eksternal yang diperlukan dalam pipeline pengolahan dan analisis data saham, yaitu:
- yfinance: Mengambil data historis saham dari Yahoo Finance.
- kafka-python: Mengirim dan menerima data melalui layanan Kafka.
- pandas: Mengelola dan memanipulasi data dalam format DataFrame.
- sqlalchemy: Menyediakan antarmuka Object Relational Mapping (ORM) untuk koneksi ke PostgreSQL.
- psycopg2-binary: Digunakan sebagai driver PostgreSQL untuk SQLAlchemy.
- scikit-learn: Digunakan untuk proses pemodelan prediktif dan evaluasi model.
- statsmodels: Mendukung analisis statistik lanjutan, termasuk regresi dan peramalan (forecasting). Instalasi seluruh pustaka dapat dilakukan dengan perintah berikut:

    ```
    pip install -r requirements.txt
    ```

  Perintah tersebut akan membaca daftar pustaka dari file requirements.txt dan menginstalnya secara otomatis, sehingga mempermudah replikasi lingkungan pengembangan oleh pengguna lain atau saat deployment.
  
 3) Menjalankan Kafka & PostgreSQL via Container di Docker
    
    Langkah utama dalam proyek ini adalah menjalankan file konfigurasi docker-compose.yaml untuk menginisialisasi dan mengaktifkan seluruh layanan yang dibutuhkan di dalam kontainer. Dengan menjalankan perintah 

    ```
    docker-compose up -d
    ```

    Semua layanan seperti PostgreSQL, Kafka, Kafka UI, Zookeeper, dan Metabase akan berjalan secara otomatis dalam mode detached (latar belakang). Selama proses inisialisasi ini, dibuat juga sebuah database dengan nama stockdb, lengkap dengan pengguna dan kata sandi yang sesuai dengan pengaturan pada file docker-compose.

 4) Menjalankan Kafka Producer

    Menjalankan skrip file Kafka Producer untuk melakukan scrapping harga saham, kemudian menyimpannya denga nama stock_data ke dalam file lokal. File: data_historical.py
    ```
    python data_historical.py
    ```

    Kafka producer bertugas Mengambil data harga saham historis (real-time 1-menit dan selama 7 hari terakhir) dari 50 perusahaan top, lalu menyimpannya ke file CSV. Tahapannya adalah sebagai berikut:

      a. Import Library

      Menggunakan library dari python yaitu yfinance untuk mengambil data saham dari Yahoo Finance, pandas untuk menyimpan dan mengolah data tabular, datetime untuk atur rentang waktu pengambilan data (7 hari ke belakang)

      b. Inisialisasi dan Setup

      Dilakukan dengan membuat folder data untuk menyimpan csv, dan inisialisasi list results untuk menampung semua data saham

      c. Daftar Saham yang Diambil

      Yaitu 50 simbol saham perusahaan besar di AS, seperti AAPL, Microsoft (MSFT), dll

      d. Set Rentang Waktu selama 7 Hari Terakhir

      Dengan cara membuat variabel end= datetime.now() dan start nya days=7

      e. Loop Ambil Data Per Saham

      Dengan cara mengambil data harga saham per 1 menit selama 7 hari terakhir untuk setiap simbol

      f. Simpan Data yang Diperlukan

      Dengan mengambil 3 informasi penting dari tiap baris yaitu symbol= kode saham, price= haga penutupan, volume= jumlah transaksi, dan timestamp= waktu pengambilan data.

      g. Simpan ke CSV

      Semua data disimpan jadi CSV di: data/historical_stock_prices.csv
      Outputnya kurang lebih sebagai berikut:

       ```
        symbol,price,volume,timestamp
      AAPL,204.06500244140625,2312674,2025-06-09 13:30:00
      AAPL,204.21499633789062,313667,2025-06-09 13:31:00
      AAPL,204.42999267578125,164566,2025-06-09 13:32:00
      AAPL,204.63499450683594,126566,2025-06-09 13:33:00
      …
      ```

 5) Menjalankan Structured Streaming ETL
    Pada file historical_to_postgre.py digunakan untuk menjalankan proses ETL (Extract, Transform, Load) secara batch dari data hasil scraping harga saham yang telah disimpan sebelumnya ke dalam database PostgreSQL. File ini dijalankan pada tahap ke-5 dalam pipeline.

a. Extract (Ekstraksi)

Membaca file hasil scraping harga saham yang telah disimpan secara lokal (misalnya stock_data). File tersebut berisi data historis yang diperoleh dari proses sebelumnya menggunakan Kafka Producer.

b. Transform (Transformasi)

Setelah data dibaca, dilakukan proses transformasi/pembersihan data, meliputi proses memastikan tipe data telah sesuai, menghapus data yang tidak lengkap atau duplikat, serta menyesuaikan nama kolom agar konsisten dengan skema database. Langkah ini bertujuan untuk memastikan kualitas data sebelum dimasukkan ke database

c. Load (Memuat data ke PostgreSQL)

Setelah dibersihkan, data dimasukkan ke tabel stock_prices_cleaned di dalam database postgreSQL bernama stockdb. Proses ini dilakukan menggunakan koneksi database melalui pustaka seperti psycopg2. 

Dengan kata lain, script pada proses ini berperan penting sebagai penghubung antara data hasil scraping dan penyimpanan ke database, agar data siap digunakan untuk proses analisis lanjutan atau pelatihan model machine learning.

    ```
    python historical_to_postgre.py
    ```
    
  7) Menjalankan Kafka Producer untuk Data Real-Time
     Setelah data historis 50 perusahaan saham disimpan ke dalam database PostgreSQL melalui skrip sebelumnya. Pipeline selanjutnya adalah dengan melakukan streaming data real-time menggunakan Kafka. Tahap ini dijalankan dengan file skrip: kafka_producer_stock.py

     ```
     python kafka_producer_stock.py
     ```

     Dalam skrip ini, kode di dalamnya bertugas untuk mengambil data harga saham secara real-time dari 50 perusahaan teratas dibantu dengan modul yfinance, lalu mengirimkannya secara berturut-turut selama 30 menit ke Kafka Topic bernama stock_prices. Penjelasan komponen kode sebagai berikut:

a. Inisialisasi Kafka Producer

Kafka Producer dikonfigurasi untuk mengirim data ke broker lokal di localhost:9092, dan akan mengirimkan data dalam format JSON (melalui value_serializer).

b. Membuat list simbol saham

Sebanyak 50 simbol saham dari perusahaan besar dalam web Yahoo Finance didefinisikan dalam list symbols, mewakili sektor teknologi, kesehatan, energi, dan konsumer.

c. Menjalankan loop streaming dengan waktu 30 menit

Program melakukan loop selama 30 menit (for minute in range(30)) untuk mendapatkan data saham secara berkala, dengan catatan pada setiap menit, program mengambil data harga dan volume interval per menit (1m) selama 1 hari (1d). Data yang diambil adalah baris terakhir dari histori (tail(1)), mewakili harga terkini.

d. Membuat dan memvalidasi payload

Untuk setiap simbol, dibuat dictionary payload berisi: symbol, price (dari kolom Close), volume, dan timestamp. Dan hanya data yang valid (tidak null) yang akan dikirim ke Kafka.

e. Pengiriman ke kafka

Payload yang valid dikirim ke Kafka topic stock_prices menggunakan producer.send(). Kafka kemudian akan menyimpan dan menunggu konsumsi oleh konsumer (streaming engine).

f. Pengendalian laju dan logging

Dalam hal ini, time.sleep(0.3) digunakan agar pengambilan data tidak terlalu cepat, lalu time.sleep(30) menunggu 30 detik sebelum iterasi menit berikutnya. Dan semua aktivitas dicatat di konsol (termasuk warning, error, dan pesan terkirim).

g. Handling untuk error

Kesalahan apa pun dalam pengambilan data saham ditangani dengan try-except agar proses tidak terhenti.
Berikut ini adalah sedikit gambaran output pada skrip ini:

```
~~Starting kafka_producer_stock.py for 30-minute streaming~~

[INFO] Minute 1/30 - Fetching real-time prices at 19:34:22
[SENT] {'symbol': 'AAPL', 'price': 196.4499969482422, 'volume': 1003296, 'timestamp': '2025-06-13 15:59:00'}
[SENT] {'symbol': 'MSFT', 'price': 474.9599914550781, 'volume': 473054, 'timestamp': '2025-06-13 15:59:00'}
[SENT] {'symbol': 'GOOGL', 'price': 174.6999969482422, 'volume': 626116, 'timestamp': '2025-06-13 15:59:00'}
[SENT] {'symbol': 'AMZN', 'price': 212.10000610351562, 'volume': 601959, 'timestamp': '2025-06-13 15:59:00'}
…
```

  Data real-time yang telah dikirim ke Kafka Topic stock_prices kemudian diambil oleh konsumer melalui pipeline streaming, yaitu file: pyspark_streaming_kafka_to_postgres.py. 

  7) Menjalankan Kafka Consumer untuk Data Real-Time

     Menjalankan skrip file PySpark untuk melakukan pembersihan dan pengolahan data yang telah di-scrapping secara real-time sebelumnya, kemudian mengirimkannya sebagai stock_prices_cleaned ke dalam database stockdb di PostgreSQL. File: pyspark_streaming_kafka_to_postgres.py

     ```
     python kafka_consumer_stock.py
     ```

     Setelah kafka topic stock_prices menerima data realtime, kafka consumer mengambil data real-time tersebut untuk dilakukan pembersihan dan pengolahan data kemudian mengirimkan pada tabel stock_prices_cleaned di database stockdb di PostgreSQL. Tahapannya adalah sebagai berikut:

a. Import Library 

Json gunanya untuk membaca pesan dari Kafka (yang dikirim dalam format JSON), psycopg2 untuk koneksi ke PostgreSQL, pandas untuk bantu parsing waktu, KafkaConsumer Untuk menerima pesan dari Kafka topic, dan datetime Untuk memproses waktu

b. Koneksi ke PostgreSQL

Untuk Menghubungkan ke database PostgreSQL lokal di port 5433 dengan yang sudah ditentukan yaitu host, username, password, nama database, dan port. kemudian Membuat objek cursor untuk menjalankan perintah SQL

c. Membuat Tabel stock_prices_cleaned

Jika belum dibuat, tabel dibuat dengan symbol nya adalah text kode saham, price adalah harga penutupan, volume adalah jumlah transaksi, dan timestamp adalah waktu

d. Inisialisasi Kafka Consumer

Fungsinya untuk mendengarkan topic kafka yaitu stock_prices dengan server kafka ada di localhost:9092. Variabel value_deserializer untuk mengubah pesan byte ke dict JSON, auto_offset_reset='earliest untuk memulai dari pesan awal jika belum ada offset, dan group_id='stock_group' adalah nama grup consumer

e. Konsumsi dan Simpan ke PostgreSQL

Dengan cara memulai loop konsumsi dari data dict JSON seperti:
{
  "symbol": "AAPL",
  "price": 192.4,
  "volume": 32000,
  "timestamp": "2025-06-15 08:40:00"
}

f. Ekstrak dan Simpan Data

Dilakukan dengan cara mengambil masing-masing kolom dari pesan, konversi waktu ke format datetime, dan menjalankan perintah INSERT ke PostgreSQL, dan commit (simpan perubahan)

g. Menangani error 
Dengan cara mencetak pesan error gagal menyimpan data ketika ada error saat parsing atau insert

Jadi selama producer kafka masih mengirim data, kafka consumer akan terus menerima dan menyimpan datanya ke PostgreSQL. 

8) Pelatihan dan Evaluasi Model ML

  Model yang digunakan adalah Linear Regression. Model dievaluasi menggunakan metrik seperti RMSE (Root Mean Square Error) dan MAE (Mean Absolute Error). Akurasi model saat ini adalah sekitar 0.0209 untuk RMSE dan 0.0078 untuk MAE.

   ```
   python ml_training_batch.py
   ```

   Dataset yang digunakan bernama ‘stock_prices_cleaned’ dan diambil dari database PostgreSQL dengan nama ‘stockdb’. Dataset ini berisi data historis saham yang meliputi beberapa informasi penting yaitu : ‘symbol’ (kode saham), ‘timestamp’ (waktu pencatatan data), ‘price’ (harga saham pada saat itu), dan ‘volume’ (jumlah transaksi yang terjadi). Data ini menjadi dasar dalam proses analisis dan prediksi menggunakan model ARIMA. Tahapan yang dilakukan : 

a. Koneksi ke Database. 

Untuk mengakses data saham, digunakan library SQLAIchemy yang menghubungkan script Python dengan database PostgreSQL. Koneksi ini memungkinkan pengambilan data langsung dari tabel ‘stock_prices_cleaned’ yang tersimpan dalam database ‘stockdb’.

b. Pengolahan Data Awal

Setelah data berhasil dimuat, data diurutkan berdasarka kode saham (symbol) dan waktu (timestamp). Kemudian, data dipisahkan berdasarkan masing-masing simbol saham. Dalam tahap ini, juga dihitung perubahan volume transaksi (volume_change) dan persentase perubahan harga (price_change_pct) dari waktu ke waktu.

c. Pembangunan Model ARIMA

Model ini diterapkan pada data harga saham masing-masing simbol. Proses training dilakukan untuk membuat model memahami pola harga historis, lalu menghasilkan prediksi harga untuk periode berikutnya (satu langkah kedepan).

d. Pembuatan Keputusan Investasi

Setelah prediksi dihasilkan, nilainya dibandingkan dengan harga terakhir. Berdasarkan perbandingan tersebut, sistem memberikan rekomendasi berupa : 
‘Buy’, jika harga diprediksi naik
‘Sell’ jika harga diprediksi turun
‘Hold’ jika tidak ada perubahan signifikan.

e. Evaluasi Model

Digunakan dua indikator evaluasi untuk mengukur seberapa akurat prediksi yang dihasilkan dengan 
‘RMSE’ (Root Mean Squared Error) untuk mengukur deviasi rata-rata kuadrat
‘MAE’ (Mean Absolute Error) untuk menghitung rata-rata kesalahan absolut antara nilai aktual dan hasil prediksi

f. Penyimpanan Output 

Seluruh hasil prediksi dan rekomendasi disimpan kembali ke dalam database PostgreSQL, tepatnya ke tabel baru bernama ‘prediction_result_arima’. 

g. Hasil dan Evaluasi

Dari hasil eksekusi program, diketahui bahwa tidak semua saham dapat diproses. Beberapa simbol dilewati karena jumlah data historisnya terlalu sedikir (kurang dari 30 baris), yang tidak cukup untuk membangun model yang stabil.

Data pelatihan berasal dari "stock_prices_cleaned" sementara data uji menggunakan data realtime "stock_prices_cleaned" dengan data hasil output prediksi pemodelan yaitu "predicted_stock".

   
5. Struktur Dataset Akhir

| Dataset               | Deskripsi                                                                 |
| ----------------------| ------------------------------------------------------------------------- |
|`historical_stock_prices.csv`         | Dataset hasil scraping informasi eksternal                      |
|`stock_prices_cleaned` | Dataset hasil ETL streaming, sudah dibersihkan dan disimpan di PostgreSQL | 
| `predicted_prices`    | Dataset hasil prediksi dari model ARIMA                |

5. Visualisasi Dashboard

   ![Screenshot 2025-06-16 083947](https://github.com/user-attachments/assets/9731cdb8-6592-4d4b-b35c-f4c19812c08a)

1) Top 5 Predicted Price Gainers

Grafik ini menunjukkan 5 saham teratas dengan prediksi kenaikan harga tetinggi dalam periode waktu terbaru. Dari grafik Top 5 Price Gainers, dapat dilihat jika harga saham yang paling tinggi didapatkan oleh saham AAPL (Apple Inc.) dengan kenaikan sebesar 0.012, lalu disusul oleh TSLA (Tesla), MU (Micron), XOM (Exxon Mobil), da CSCO (Cisco).

2) Top 5 Predicted Price Losers

Berkebalikan dari grafik pada nomor 1, grafik ini menunjukkan 5 saham dengan penurunana harga tertinggi. Dari grafik Top 5 Price Losers, dapat dilihat jika harga saham yang paling menurun adalah META (Meta/Facebook) mengalami penurunan paling tajam sebesar -0.1, kemudian diikuti oleh NFLX (Netflix) MSFT (Microsoft), AMGN (Amagen) dan LLY (Eli Lilly).

3) ML Decision Summary

Hasil grafik ini dapat dilihat pada donat chart yang menunjukkan ringkasan hasil presentase prediksi machine learning terhadap keputusan berinvestasi. Dari donat chart, dapat dilihat jika 50% saham direkomendasikan untuk dipertahankan (hold). Kemudian sebanyak 30% saham direkomendasikan untuk dijual (sell), sisanya 20% saham direkomendasikan untuk dibeli (buy). 

4) Current vs Prediction Price Gainers

Pada grafik Current vs Prediction Price Gainers di atas digunakan untuk membandingkan harga aktual (last_price) saham dari data pasar saat ini (ungu), dengan hasil prediksi (prediction) dari model machine learning (garis oranye). Grafik ini memungkinkan untuk dapat mengevaluasi seberapa dekat hasil prediksi model dengan nilai aslinya, dapat juga untuk mengidentifikasi potensi anomali yang bisa terjadi. Dari grafik, dapat dilihat jika area oranye dan ungu memiliki pola yang saling mengikuti dan mirip, sehingga ini berarti hasil prediksi saham tidak begitu jauh dan mirip dengan data aslinya.

5) Most Volatile Stock

Pada grafik batang berwarna hijau ini menampilkan 5 saham teratas dengan volatilitas tertinggi. Volatilitas digunakan untuk mengukur seberapa besar fluktuasi harga terjadi dalam periode waktu tertentu. Dari grafik di atas, dapat dilihat jika saham LLY dengan votilitas 16.68 dimana ini menunjukkan pola volatilitas yang tinggi. Disusul oleh ORCL, NFLX, TSLA, dan ADBE, sehingga ini berarti kelima saham ini memiliki harga yang berubah-ubah secara signifikan.

6) Price and Volume Movement for ORCL Stock

Grafik ini adalah gabungan antara harga saham (line chart) dan volume transaksi (area chart) untuk salah satu saham, yaitu ORCL (Oracle). Sumbu kiri (hijau) menunjukkan pergerakan saham dan sumbu kanan (oranye), menunjukkan jumlah volume saham yang diperdagangkan dalam waktu yang sama. Terlihat jika pada saham ORCL, volume saham yang terjual semakin meningkat. Grafik ini sangat berguna jika ingin mengetahui analisis mikro per saham.

   
6. Lampiran
   Tautan File Kode dan README.md Drive Kelompok: [https://drive.google.com/drive/folders/1_APxOUnnbFFMcGjEe_QQrWE1kUYA_e3M?usp=drive_link]


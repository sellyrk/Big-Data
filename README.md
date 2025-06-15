# Big-Data (A)
## Laporan Akhir Projek Final Big Data (A) Kelompok 4

Projek ini bertujuan untuk membangun pipeline **Big Data Realtime dan Offline** untuk melakukan **analisis dan prediksi harga saham dari 50 top perusahaan** berdasarkan data dari **Yahoo Finance API**. Arsitektur ini menggunakan kombinasi **Kafka**, **PySpark Structured Streaming**, **PostgreSQL**, dan **Metabase** untuk visualisasi, serta **PySpark MLlib** untuk pelatihan model prediktif menggunakan algoritma **Linear Regression**.

1. Fitur Utama

- Streaming data saham real-time 50 top saham dari Yahoo Finance menggunakan Kafka

  Dalam hal ini, menggunakan modul yfinance, harga saham dari 50 saham top diambil dari Yahoo Finance menggunakan yfinance dan disimpan dalam file data/historical_stock_prices.csv, yang sebelumnya diolah dalam bentuk real-time stream menggunakan Kafka Producer. Data ini kemudian dikirimkan ke Kafka Topic untuk diolah secara stream. File historical_stock_prices.csv berisi lebih dari 94.000 baris data dengan kolom seperti Symbol, Datetime, Close, dan Volume.
  
- Pengolahan dan pembersihan data menggunakan PySpark

   PySpark digunakan untuk proses batch maupun stream ETL. Langkah-langkah pembersihan meliputi parsing format datetime, konversi tipe data numerik, penghapusan nilai null, serta kalkulasi fitur tambahan seperti persentase perubahan harga (close_ptc) dan volume rata-rata.
  
- Penyimpanan data yang sudah dibersihkan ke dalam PostgreSQL

  Output dari proses batch maupun streaming disimpan ke dalam database stockdb menggunakan konektor **JDBC dari PySpark** atau **Psycopg2**??. Tabel utama yang digunakan dinamai stock_prices_cleaned.
  
- Visualisasi interaktif menggunakan Metabase Dashboard

  Metabase menyajikan dashboard interaktif untuk melihat tren harga saham, volume perdagangan, serta hasil prediksi. Dashboard dirancang untuk memperlihatkan perubahan harga harian, serta akurasi model prediktif secara real-time.
  
- Prediksi harga saham menggunakan model Linear Regression

  Metabase menyajikan dashboard interaktif untuk melihat tren harga saham, volume perdagangan, serta hasil prediksi. Dashboard digaunakan untuk memperlihatkan perubahan harga harian, serta akurasi model prediktif secara real-time.
  
- Hasil akhir dalam bentuk 3 dataset utama, yaitu Dataset 1: Dataset hasil scraping data dengan KafkaProducer (berisi nilai Symbol, Datetime, Close dan Volume harga saham), Dataset 2: Hasil pembersihan dan transformasi dari PySpark (stock_prices_cleaned) dan Dataset 3: Dataset hasil prediksi model ML (predicted_prices)

2. Setup dan Instalasi (*!PERLU DISESUAIKAN LAGI)

 1) Prasyarat Tools
    - Docker (untuk containerisasi Kafka, PostgreSQL, Metabase)
    - Python 3.5+ (untuk containerisasi Kafka, PostgreSQL, Metabase)
    - Java (untuk PySpark)
    - Metabase (visualisasi)
    - PostgreSQL (penyimpanan database)
 2) Instalasi Library Python
    ```
    pip install yfinance kafka-python pyspark psycopg2
    ```
 3) Menjalankan Kafka & PostgreSQL via Container di Docker
    Menjalankan konfigurasi file docker-compose untuk instalasi awal pada kontainer yang dibutuhkan. Pada tahap ini, juga dibuat database dengan nama stock_db, lengkap dengan user dan passwordnya.
    ```
    docker-compose up -d
    ```
 4) Menjalankan Kafka Producer
    Menjalankan skrip file Kafka Producer untuk melakukan scrapping harga saham, kemudian menyimpannya denga nama stock_data ke dalam file lokal. File: data_historical.py
    ```
    python data_historical.py
    ```
    
 5) Menjalankan PySpark Structured Streaming ETL
    Menjalankan skrip file PySpark untuk melakukan pembersihan dan pengolahan data yang telah di-scrapping sebelumnya, kemudian mengirimkannya sebagai stock_prices_cleaned ke dalam database stockdb di PostgreSQL. File: historical_to_postgre.py
    ```
    python historical_to_postgre.py
    ```
  6) Menjalankan Kafka Producer untuk Data Real-Time
     Menjalankan skrip file Kafka Producer yang kedua untuk mengambil melakukan scrapping harga saham secara real-time. File: kafka_producer_stock.py
     ```
     python kafka_producer_stock.py
     ```
     
  7) Menjalankan Kafka Consumer untuk Data Real-Time

     Menjalankan skrip file PySpark untuk melakukan pembersihan dan pengolahan data yang telah di-scrapping secara real-time sebelumnya, kemudian mengirimkannya sebagai stock_prices_cleaned ke dalam database stockdb di PostgreSQL. File: pyspark_streaming_kafka_to_postgres.py
     ```
     python kafka_consumer_stock.py
     ```
     
  9) Menjalankan Metabase
     Mengakses Metabase ke dalam localhost:3000 untuk melakukan pembuatan visualisasi dalam bentuk dahsboard.
     ```
     docker run -d -p 3000:3000 --name metabase metabase/metabase
     ```
     
3. Pelatihan dan Evaluasi Model ML

   Model yang digunakan adalah Linear Regression. Model dievaluasi menggunakan metrik seperti RMSE (Root Mean Square Error), MAE (Mean Absolute Error), dan R² Score. Akurasi model saat ini adalah sekitar [].
   Pada pemodelan ini digunakan Library: PySpark MLlib. Dengan target prediksi adalah harga saham pada waktu mendatang. Input features: Harga sebelumnya (close_ptc), volume, tren, dan variabel hasil scraping (jika ada) (!perlu disesuaikan).

   Pelatihan:
   ```
   python ml_training_batch.py
   ```

   Data pelatihan berasal dari "stock_prices_cleaned" sementara data uji menggunakan data realtime "stock_prices_cleaned" dengan data hasil output prediksi pemodelan yaitu "predicted_stock"
   
5. Struktur Dataset Akhir

| Dataset               | Deskripsi                                                                 |
| ----------------------| ------------------------------------------------------------------------- |
|`historical_stock_prices.csv`         | Dataset hasil scraping informasi eksternal (opsional)                     |
|`stock_prices_cleaned` | Dataset hasil ETL streaming, sudah dibersihkan dan disimpan di PostgreSQL | 
| `predicted_prices`    | Dataset hasil prediksi dari model Random Forest Regression                |

5. Visualisasi Dashboard

    [Gambar Dashboard Menyusul]

   (penjelasan dashboard)
   
6. Kontributor
    (nama kelompok)

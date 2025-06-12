# Big-Data (A)
## Laporan Akhir Projek Final Big Data (A) Kelompok []

Projek ini bertujuan untuk membangun pipeline **Big Data Realtime dan Offline** untuk melakukan **analisis dan prediksi harga saham dari 50 top perusahaan** berdasarkan data dari **Yahoo Finance API**. Arsitektur ini menggunakan kombinasi **Kafka**, **PySpark Structured Streaming**, **PostgreSQL**, dan **Metabase** untuk visualisasi, serta **PySpark MLlib** untuk pelatihan model prediktif menggunakan algoritma **Random Forest Regression**.

1. Fitur Utama

- Streaming data saham real-time 50 top saham dari Yahoo Finance menggunakan Kafka
- Pengolahan dan pembersihan data menggunakan PySpark
- Penyimpanan data yang sudah dibersihkan ke dalam PostgreSQL
- Visualisasi interaktif menggunakan Metabase Dashboard
- Prediksi harga saham menggunakan model Random Forest Regression
- Hasil akhir dalam bentuk 3 dataset utama, yaitu Dataset 1: Dataset hasil scraping data dengan KafkaProducer (berisi nilai Symbol, Datetime, Close dan Volume harga saham), Dataset 2: Hasil pembersihan dan transformasi dari PySpark (stock_prices_cleaned) dan Dataset 3: Dataset hasil prediksi model ML (predicted_prices)

3. Setup dan Instalasi (*!PERLU DISESUAIKAN LAGI)

 1) Prasyarat Tools
    - Docker
    - Python 3.5+
    - Java (untuk PySpark)
    - Metabase (visualisasi)
    - PostgreSQL (penyimpanan database)
 2) Instalasi Library Python
    ```
    pip install yfinance kafka-python pyspark psycopg2
    ```
 3) Menjalankan Kafka & PostgreSQL via Container di Docker
    Menjalankan konfigurasi file docker-compose untuk instalasi awal
    ```
    docker-compose up -d
    ```
 4) Menjalankan Kafka Producer
    Menjalankan skrip file Kafka Producer untuk melakukan scrapping harga saham. File: kafka_producer.py
    ```
    python kafka_producer.py
    ```
 5) Menjalankan PySpark Structured Streaming ETL
    Menjalankan skrip file PySpark untuk melakukan pembersihan dan pengolahan data yang telah di-scrapping sebelumnya. File: streaming_etl.py
    ```
    spark-submit streaming_etl.py
    ```
  6) Menjalankan Metabase
     Mengakses Metabase ke dalam localhost:3000 untuk melakukan pembuatan visualisasi dalam bentuk dahsboard
     ```
     docker run -d -p 3000:3000 --name metabase metabase/metabase
     ```
     
5. Pelatihan dan Evaluasi Model ML

   Model yang digunakan adalah Random Forest Regression dengan evaluasi akurasi sebesar []. Pada pemodelan ini digunakan Library: PySpark MLlib. Dengan target prediksi adalah harga saham pada waktu mendatang. Input features: Harga sebelumnya (close_ptc), volume, tren, dan variabel hasil scraping (jika ada) (!perlu disesuaikan).

   ```
   from pyspark.ml.regression import RandomForestRegressor
   ```
   Data pelatihan berasal dari "stock_prices_cleaned" dengan data hasil output prediksi pemodelan yaitu "predicted_stock"
   
7. Struktur Dataset Akhir

   | Dataset                | Deskripsi                                                                 |
| ---------------------- | ------------------------------------------------------------------------- |
|`scraped_data`         | Dataset hasil scraping informasi eksternal (opsional)                       |
|`stock_prices_cleaned` | Dataset hasil ETL streaming, sudah dibersihkan dan disimpan di PostgreSQL |              |
| `predicted_prices`     | Dataset hasil prediksi dari model Random Forest Regression                |

9. Visualisasi Dashboard

    [Gambar Dashboard Menyusul]

   (penjelasan dashboard)
   
11. Kontributor
    (nama kelompok)

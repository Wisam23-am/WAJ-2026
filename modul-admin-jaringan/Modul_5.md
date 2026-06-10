## **DOCKER-INTRO Logging Service Docker dengan PostgreSQL WORKSHOP ADMIN JARINGAN** 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0064-01.png)


NAMA: AQILA WISAM MADANI MA’MURRIE NRP : 3124600041 KELAS : D4 IT B KELOMPOK : 2 TANGGAL PRAKTIKUM : 6 MEI 2026 DOSEN : Dr Ferry Astika Saputra ST, M.Sc 

## **POLITEKNIK ELEKTRONIKA NEGERI SURABAYA** 

## **2025/2026** 

## **Pre-Lab** 

1. Mengapa centralized logging penting di lingkungan container? 

2. Apa perbedaan antara Docker logging driver json-file dan fluentd? 

3. Jelaskan keuntungan menyimpan log di database (PostgreSQL) vs file text. 

4. Apa itu structured logging dan mengapa lebih baik daripada plain text log? 

5. Mengapa Fluent Bit lebih cocok untuk sidecar/edge collection dibanding Fluentd? 

## **Jawaban** 

1. Centralized logging penting karena container bersifat: 

   - ephemeral 

   - dinamis 

   - mudah dibuat dan dihapus 

   - tersebar di banyak service 

Tanpa centralized logging, log hanya berada di dalam masing-masing container dan bisa hilang saat container restart atau dihapus. 

## 2. 

- a. Json-file 

   - default logging driver Docker 

   - Log disimpan sebagai file JSON lokal di host 

   - Sederhana 

   - Tidak butuh service tambahan 

   - Cocok untuk development kecil 

   - Sulit untuk central aggregation 

## b. fluentd 

   - Docker langsung mengirim log ke Fluentd/Fluent Bit 

   - Mendukung centralized logging 

   - Scalable 

   - Real-time forwarding 

   - Mendukung parsing/filtering 

   - Cocok production dan distributed system 

3. Menyimpan log di database seperti PostgreSQL memiliki keuntungan dalam hal query, filtering, agregasi, pencarian, dan analisis data. Log dapat dicari berdasarkan level, waktu, service, atau field tertentu dengan cepat menggunakan SQL. Database juga mendukung indexing, backup, retention policy, dan integrasi dengan dashboard monitoring. Sebaliknya, file text lebih sederhana tetapi sulit dianalisis dalam skala besar karena pencarian dan pengolahan data harus dilakukan secara manual atau menggunakan tools tambahan. 

4. Structured logging adalah metode pencatatan log dalam format terstruktur sehingga setiap informasi log memiliki field yang jelas dan konsisten. Structured logging lebih baik daripada plain text log karena lebih mudah diproses oleh mesin, lebih mudah dicari, lebih konsisten, dan lebih cocok untuk sistem monitoring modern. Structured logging juga mempermudah filtering, agregasi, observability, dan integrasi dengan sistem analitik maupun distributed tracing. 

5. Fluent Bit lebih cocok untuk sidecar atau edge collection karena memiliki konsumsi resource yang rendah, ukuran kecil, startup cepat, dan throughput tinggi. Fluent Bit dirancang khusus untuk pengumpulan dan forwarding log dengan overhead minimal sehingga sangat efisien digunakan di container, Kubernetes, edge computing, maupun environment dengan resource terbatas. Sementara itu, Fluentd lebih berat karena menyediakan fitur pemrosesan data dan plugin yang lebih kompleks sehingga lebih cocok digunakan sebagai aggregator pusat dibanding collector ringan di edge. 

## **- Langkah langkah Praktikum** 

1. docker compose ps — 5 service running 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0067-02.png)


2. docker compose logs fluent-bit — Fluent Bit menerima log (JSON lines di stdout) 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0067-04.png)


3. SELECT COUNT(*) FROM logs.fluentbit — jumlah total log > 0 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0067-06.png)


## 4. SELECT tag, time, data FROM logs.fluentbit LIMIT 3 — raw 3-kolom data 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0067-08.png)


5. SELECT * FROM logs.recent_logs LIMIT 10 — log terbaru via view 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0067-10.png)


6. SELECT * FROM logs.structured_logs LIMIT 10 — parsed JSON log 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0067-12.png)


7. Query distribusi per tag — output tabe 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0068-00.png)


8. Query distribusi per level — output tabel 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0068-02.png)


9. SELECT * FROM logs.error_summary — summary error 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0068-04.png)


10. Query log rate per menit — output tabel 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0068-06.png)


11. curl /api/logs/stats — response JSON 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0069-00.png)


12. curl /api/logs/stats — response JSON 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0069-02.png)


## **Post-Lab** 

1. Berapa total log yang masuk ke PostgreSQL setelah 5 menit? Tunjukkan distribusi per container dan per level. 

2. Tulis query SQL yang menampilkan log rate per menit selama 10 menit terakhir. 

3. Apa yang terjadi jika container fluent-bit di-stop? Apakah container lain juga stop? Apakah log hilang? 

4. Jelaskan alur sebuah log entry dari log-generator stdout sampai masuk ke tabel container_logs. 

5. Modifikasi LOG_INTERVAL menjadi 0.5 detik. Berapa log rate per menit yang dihasilkan? 

## **Jawaban** 

## 1. Total log setelah 5 menit: 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0070-08.png)


Distribusi log per tag/container: 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0070-10.png)


Distribusi log per level: 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0070-12.png)


2.  Query SQL @' SELECT date_trunc('minute', time) AS minute, COUNT(*) AS logs_per_minute FROM logs.fluentbit WHERE time > NOW() - INTERVAL '10 minutes' GROUP BY minute ORDER BY minute; '@ | docker exec -i postgres-db psql -U labuser -d labdb 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0071-00.png)


3. Ketika container fluent-bit distop, container lain tidak akan hilang 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0071-02.png)


Lalu untuk log tidak menghilang dan bertambah ketika fluent-bit kembali hidup 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0071-04.png)


4. Alur sebuah log entry dari log-generator stdout sampai masuk ke 

   - container_logs 

      - Docker logging driver menangkap output container 

      - Docker mengirim log ke endpoint Fluent Bit pada 24224 

      - Fluent Bit menerima log lewat input forward 

      - parser membaca field JSON seperti timestamp, level, message 

      - filter modify atau parser menyesuaikan field 

      - output pgsql mengirim data ke PostgreSQL 

      - PostgreSQL menyimpan ke tabel logs.container_logs 

5. Produksi log akan lebih cepat dan menghasilkan log yang lebih banyak 3 detik 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0072-01.png)


## 0.5 detik 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0072-03.png)



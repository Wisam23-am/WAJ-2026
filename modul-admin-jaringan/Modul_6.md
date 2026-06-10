## **DOCKER-INTRO Grafana Service Docker untuk Monitoring Resource WORKSHOP ADMIN JARINGAN** 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0073-01.png)


NAMA: AQILA WISAM MADANI MA’MURRIE NRP : 3124600041 KELAS : D4 IT B KELOMPOK : 2 TANGGAL PRAKTIKUM : 25 MEI 2026 DOSEN : Dr Ferry Astika Saputra ST, M.Sc 

## **POLITEKNIK ELEKTRONIKA NEGERI SURABAYA** 

**2025/2026** 

## **Pre-Lab** 

1. Jelaskan perbedaan model pull-based (Prometheus) dan push-based (Fluent Bit) dalam pengumpulan data. 

2. Apa itu PromQL? Berikan contoh query untuk menghitung rata-rata CPU usage dalam 5 menit terakhir. 

3. Mengapa cAdvisor membutuhkan akses ke /var/run/docker.sock dan /sys? 

4. Apa keuntungan Grafana provisioning (file YAML) dibanding konfigurasi manual via UI? 

5. Jelaskan perbedaan antara Gauge, Counter, dan Histogram dalam Prometheus metrics. 

## **Jawaban** 

1. Pada model _**pull-based**_ , server pemantau (Prometheus) bertindak aktif untuk menarik atau mengambil ( _scrape_ ) metrik secara berkala dari setiap target melalui _endpoint_ tertentu, seperti /metrics. Sebaliknya, pada model _**push-based**_ , agen yang berada di titik target (seperti Fluent Bit) yang bertindak aktif untuk mengirimkan atau mendorong data log/metrik ke tujuan atau server pusat. 

2. **PromQL (Prometheus Query Language)** adalah bahasa kueri bawaan Prometheus yang digunakan untuk mengekstrak, memanipulasi, dan memvisualisasikan data _time-series_ dari Prometheus atau di dalam Grafana. Contoh _query_ untuk rata-rata _CPU usage_ dalam 5 menit terakhir adalah: 100 - 

   - (avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100). 

3. cAdvisor bertugas mengekspos metrik secara spesifik per- _container_ Docker. Akses ke /var/run (termasuk _socket_ Docker) diperlukan agar cAdvisor bisa berkomunikasi dengan _daemon_ Docker untuk melacak _container_ apa saja yang sedang berjalan di _host_ . Sementara itu, akses ke direktori /sys diperlukan untuk membaca data statistik fundamental sistem dari dalam _cgroups_ (Control Groups) kernel Linux yang mencatat konsumsi _resource_ aktual (seperti CPU dan memori) dari masing-masing _container_ . 

4. Keuntungan utamanya adalah otomasi; konfigurasi _data source_ maupun _dashboard_ dapat langsung terpasang dan siap digunakan saat Grafana pertama kali dijalankan ( _start_ ) tanpa perlu campur tangan manual via _User Interface_ (UI). Hal ini memastikan konfigurasi yang konsisten, mudah direplikasi, serta meminimalkan risiko kesalahan manusia (sejalan dengan konsep _Infrastructure as Code_ ). 

5. Berdasarkan implementasi metrik pada modul: 

      - **Counter** : Digunakan untuk metrik nilai kumulatif yang ukurannya hanya bisa bertambah naik atau di-reset ke nol (contoh: akumulasi total HTTP _requests_ yang masuk). 

      - **Gauge** : Digunakan untuk metrik nilai yang secara dinamis bisa naik ataupun turun seiring berjalannya waktu (contoh: jumlah sisa memori, total koneksi _database_ aktif saat ini). 

      - **Histogram** : Digunakan untuk mengumpulkan sampel data dari suatu kejadian lalu mengelompokkannya ke dalam _buckets_ atau keranjang-keranjang rentang nilai (contoh: observasi latensi durasi sebuah _request_ ke dalam keranjang 0.01 detik, 0.5 detik, 5.0 detik, dst). 

## **- Langkah langkah Praktikum** 

1. docker compose ps — 9 service running 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0075-02.png)


Screenshot ini membuktikan bahwa seluruh orkestrasi berhasil dan kesembilan _service_ berjalan tanpa _error_ . 

2. Prometheus Targets — semua status UP 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0075-05.png)


Menunjukkan bahwa Prometheus berhasil melakukan _scrape_ atau menarik data dari semua target metrik yang dikonfigurasi tanpa masalah jaringan. 

3. Prometheus query browser — PromQL CPU usage 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0075-08.png)


Memastikan bahwa bisa dilakukan _query_ data mentah secara langsung di antarmuka Prometheus menggunakan sintaks PromQL. 

4. curl localhost:5000/metrics — Flask Prometheus metrics 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0076-02.png)


Memvalidasi bahwa aplikasi _backend_ Flask yang dibuat sukses mengekspos metrik kustomnya agar bisa dibaca oleh Prometheus. 

5. Grafana login — halaman utama setelah login 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0076-05.png)


Membuktikan bahwa _container_ Grafana sudah terekspos ke _host_ dan UI-nya dapat diakses serta digunakan untuk _login_ . 

6. Data sources — Prometheus dan PostgreSQL keduanya OK 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0077-00.png)



![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0077-01.png)


Menandakan integrasi yang sukses antara Grafana dengan dua sumber data utamanya, yaitu Prometheus (untuk metrik) dan PostgreSQL (untuk log). 

7. Dashboard Docker Host Overview — keseluruhan 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0077-04.png)


Memperlihatkan bahwa metrik tingkat _host_ (CPU, memori, disk) berhasil dikumpulkan oleh Node Exporter dan divisualisasikan 

8. Dashboard Docker Host Overview — gauge CPU/Memory saat stress test (lonjakan terlihat) 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0078-00.png)


Menunjukkan bahwa sistem _monitoring_ merespons secara _real-time_ dengan mendeteksi dan menampilkan grafik lonjakan beban secara langsung. 

9. Dashboard Container Metrics — CPU per container 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0078-03.png)


Membuktikan cAdvisor berhasil melacak dan membedakan beban penggunaan CPU untuk masing-masing _container_ Docker secara spesifik. 

10. Dashboard Container Metrics — Memory per container 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0079-00.png)


Sama seperti poin sebelumnya, ini membuktikan cAdvisor sukses memantau alokasi konsumsi memori per _container_ . 

11. Dashboard Log Analytics — log volume time-series 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0079-03.png)


Memvalidasi pembacaan data dari tabel PostgreSQL untuk melihat pergerakan volume log aplikasi dari waktu ke waktu. 

12. Dashboard Log Analytics — pie chart level distribution 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0079-06.png)


Menunjukkan bahwa log tidak hanya disimpan, tapi bisa dikelompokkan dan dianalisis berdasarkan tingkat keparahannya (seperti ERROR, INFO, dsb). 

13. Custom panel yang dibuat — Flask HTTP Requests 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0080-00.png)


Membuktikan bahwa bisa dibangun visualisasi panel Grafana secara spesifik dari nol untuk memantau metrik khusus, bukan sekadar memakai _template_ bawaan. Ini menunjukkan bahwa kita bisa melihat grafik dari HTTP request di endpointnya 

14. Alerting rules — daftar alert yang dikonfigurasi 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0080-03.png)


Memastikan sistem _monitoring_ disetup secara proaktif untuk mengirim peringatan otomatis jika ada anomali atau metrik yang melampaui batas wajar. 

## **Post-Lab** 

1. Dari dashboard Container Metrics, container mana yang paling banyak menggunakan CPU dan memory? Mengapa? 

2. Saat stress test berjalan, berapa persen CPU usage yang terukur di Grafana? Bandingkan dengan output top atau htop di host. 

   - Ini membuktikan bahwa Node Exporter berhasil membaca metrik _host_ secara akurat. 

3. Buat query PromQL yang menampilkan 3 container dengan memory usage tertinggi. Tunjukkan query dan hasilnya. 

4. Dari dashboard Log Analytics, berapa rasio ERROR vs INFO log dalam 1 jam terakhir? Apakah ini normal untuk aplikasi production? 

5. Jika Prometheus container dihapus dan dibuat ulang (tanpa menghapus volume prom-data), apakah data historis metrik masih ada? Buktikan. 

Ya, data metrik historis masih ada dan utuh. Alasannya adalah mekanisme penyimpanan pada Docker. Pada file docker-compose.yml, direktori data Prometheus 

(/prometheus) di- _mount_ ke dalam _named volume_ Docker bernama prom-data. 

Menghapus _container_ (docker rm) hanya menghapus lapisan aplikasi sementara, tidak menghapus persisten _volume_ -nya. 

## **Jawaban** 

1. Grafik Container: 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0081-04.png)


grafana memakan memori cukup tinggi karena bertugas menyimpan data _time-series_ ke _memory_ sebelum di- _flush_ ke _disk_ , Dengan data yang lumayan fluktuatif. sementara cadvisor memakan CPU tinggi saat _stress test_ berlangsung dengan rata-rata sekitar 3.5%. 

2. Outpot top dan htop dibanding curl 

Hasil screenshot htop: 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0081-08.png)


Hasil CPU Usage dari Grafana 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0081-10.png)


3. Rasio PromQL 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0082-00.png)


topk(3, container_memory_usage_bytes{name!=""}) Kueri ini akan mengambil 3 nilai tertinggi dari metrik memori _container_ , dengan mengabaikan _container_ yang tidak bernama (biasanya _pause container_ atau _cgroup_ dasar). Hasilnya terlihat ada 3 container docker compose dengan confignya 

4. Rasio Error dan info: 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0082-03.png)


Tidak, rasio ini tidak normal untuk skenario production yang sehat. Normalnya, log ERROR berada di bawah 1-5%. Alasan mengapa ERROR terlihat tinggi di lab ini adalah karena _script_ generator.py memang sudah dikonfigurasi ( _hardcoded_ ) untuk membuang beban ( _weight_ ) log secara spesifik, yakni INFO sebesar 50% dan ERROR sebesar 10%. 

5. Produksi log akan lebih cepat dan menghasilkan log yang lebih banyak Terminal menghapus docker 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0082-06.png)


Hasil saat di grafana: 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0083-00.png)


Ya, data metrik historis masih ada dan utuh. Alasannya adalah mekanisme penyimpanan pada Docker. Pada file docker-compose.yml, direktori data Prometheus 

(/prometheus) di- _mount_ ke dalam _named volume_ Docker bernama prom-data. Menghapus _container_ (docker rm) hanya menghapus lapisan aplikasi sementara, tidak menghapus persisten _volume_ -nya. 


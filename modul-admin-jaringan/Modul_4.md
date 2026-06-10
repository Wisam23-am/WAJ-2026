## **DOCKER-INTRO Database Service di Docker — PostgreSQL WORKSHOP ADMIN JARINGAN** 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0047-01.png)


NAMA: AQILA WISAM MADANI MA’MURRIE NRP : 3124600041 KELAS : D4 IT B KELOMPOK : 2 TANGGAL PRAKTIKUM : 6 MEI 2026 DOSEN : Dr Ferry Astika Saputra ST, M.Sc 

## **POLITEKNIK ELEKTRONIKA NEGERI SURABAYA** 

**2025/2026** 

## **MODUL 4: Database Service di Docker — PostgreSQL LANGKAH PRAKTIKUM** 

## **Langkah 0: Persiapan Project** 

mkdir -p ~/docker-lab/postgresql/{init,config,backup} 

cd ~/docker-lab/postgresql 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0048-04.png)


## **Langkah 1: Deploy PostgreSQL dengan Docker Compose** 

1.1 Buat init script (dijalankan saat pertama kali) cat > init/01-create-schema.sql << 'EOF' 

- -- ============================================== 

- -- Init Script: Database Schema untuk Lab PENS 

- -- Dijalankan otomatis saat container pertama kali start 

- -- ============================================== 

## -- Buat database tambahan 

CREATE DATABASE inventory_db; 

-- Gunakan database utama (labdb sudah dibuat via env) 

\c labdb 

-- Buat schema 

CREATE SCHEMA IF NOT EXISTS app; 

-- Tabel: Mahasiswa 

CREATE TABLE app.mahasiswa ( id SERIAL PRIMARY KEY, nrp VARCHAR(15) UNIQUE NOT NULL, nama VARCHAR(100) NOT NULL, kelas CHAR(1) CHECK (kelas IN ('A', 'B', 'C', 'D')), kelompok INTEGER CHECK (kelompok BETWEEN 1 AND 10), email VARCHAR(100), created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ); 

-- Tabel: Mata Kuliah 

CREATE TABLE app.matakuliah ( id SERIAL PRIMARY KEY, kode VARCHAR(10) UNIQUE NOT NULL, nama VARCHAR(100) NOT NULL, sks INTEGER CHECK (sks BETWEEN 1 AND 6) 

); 

-- Tabel: Nilai (relasi many-to-many) CREATE TABLE app.nilai ( id SERIAL PRIMARY KEY, mahasiswa_id INTEGER REFERENCES app.mahasiswa(id) ON DELETE CASCADE, 

matakuliah_id INTEGER REFERENCES app.matakuliah(id) ON DELETE CASCADE, nilai_angka NUMERIC(5,2) CHECK (nilai_angka BETWEEN 0 AND 100), grade CHAR(2), 

semester VARCHAR(10), 

UNIQUE(mahasiswa_id, matakuliah_id, semester) 

## ); 

- -- Tabel: Log Aktivitas (untuk Modul 5 logging) 

CREATE TABLE app.activity_log ( 

id BIGSERIAL PRIMARY KEY, 

timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP, level VARCHAR(10) DEFAULT 'INFO', 

source VARCHAR(50), message TEXT, metadata JSONB 

## ); 

- -- Index untuk performa query 

CREATE INDEX idx_mahasiswa_kelas ON app.mahasiswa(kelas); CREATE INDEX idx_mahasiswa_nrp ON app.mahasiswa(nrp); CREATE INDEX idx_nilai_semester ON app.nilai(semester); CREATE INDEX idx_activity_log_timestamp ON app.activity_log(timestamp); CREATE INDEX idx_activity_log_level ON app.activity_log(level); CREATE INDEX idx_activity_log_metadata ON app.activity_log USING GIN(metadata); 

- -- Insert sample data 

INSERT INTO app.matakuliah (kode, nama, sks) VALUES 

('JAR01', 'Administrasi Jaringan', 3), 

('SBD01', 'Sistem Basis Data', 3), 

('SO01',  'Sistem Operasi', 2), 

('WEB01', 'Pemrograman Web', 3); 

INSERT INTO app.mahasiswa (nrp, nama, kelas, kelompok, email) VALUES 

('3122600001', 'Ahmad Fauzi', 'A', 1, 'ahmad@student.pens.ac.id'), 

('3122600002', 'Budi Santoso', 'A', 1, 'budi@student.pens.ac.id'), 

('3122600003', 'Citra Dewi', 'B', 2, 'citra@student.pens.ac.id'), 

('3122600004', 'Dian Pratama', 'B', 2, 'dian@student.pens.ac.id'), 

('3122600005', 'Eka Putra', 'C', 3, 'eka@student.pens.ac.id'); 

INSERT INTO app.nilai (mahasiswa_id, matakuliah_id, nilai_angka, grade, semester) VALUES 

(1, 1, 85.50, 'A', '2025-1'), 

(1, 2, 78.00, 'B+', '2025-1'), 

(2, 1, 92.00, 'A', '2025-1'), 

(3, 1, 70.25, 'B', '2025-1'), (4, 3, 88.75, 'A', '2025-1'); 

-- Buat read-only user untuk aplikasi CREATE USER app_reader WITH PASSWORD 'reader123'; GRANT USAGE ON SCHEMA app TO app_reader; 

GRANT SELECT ON ALL TABLES IN SCHEMA app TO app_reader; ALTER DEFAULT PRIVILEGES IN SCHEMA app GRANT SELECT ON TABLES TO app_reader; 

RAISE NOTICE 'Database initialization completed successfully!'; EOF 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0050-02.png)


1.2 Buat custom PostgreSQL config 

cat > config/custom-postgresql.conf << 'EOF' 

# ============================================== 

# Custom PostgreSQL Configuration untuk Lab 

# ============================================== 

# Connection listen_addresses = '*' max_connections = 50 

# Memory (sesuaikan untuk container dengan RAM terbatas) shared_buffers = 128MB work_mem = 4MB maintenance_work_mem = 64MB effective_cache_size = 256MB 

# WAL & Checkpoint wal_level = replica max_wal_size = 256MB min_wal_size = 64MB 

# Logging logging_collector = on log_directory = '/var/log/postgresql' log_filename = 'postgresql-%Y-%m-%d.log' log_statement = 'mod' log_min_duration_statement = 1000 log_connections = on log_disconnections = on log_line_prefix = '%t [%p] %u@%d ' 

# Locale & Timezone timezone = 'Asia/Jakarta' log_timezone = 'Asia/Jakarta' 

EOF 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0051-01.png)


1.3 Buat Docker Compose cat > docker-compose.yml << 'EOF' services: # --- PostgreSQL 16 --db: image: postgres:16-alpine container_name: postgres-db environment: POSTGRES_DB: labdb POSTGRES_USER: labuser POSTGRES_PASSWORD: labpass123 TZ: Asia/Jakarta ports: - "5432:5432" volumes: - pg-data:/var/lib/postgresql/data - ./init:/docker-entrypoint-initdb.d:ro - ./config/custom-postgresql.conf:/etc/postgresql/custom.conf:ro - ./backup:/backup - pg-logs:/var/log/postgresql command: > postgres -c config_file=/etc/postgresql/custom.conf -c hba_file=/var/lib/postgresql/data/pg_hba.conf networks: - db-net healthcheck: test: ["CMD-SHELL", "pg_isready -U labuser -d labdb"] interval: 10s timeout: 5s retries: 5 restart: unless-stopped # --- pgAdmin 4 (GUI) --pgadmin: image: dpage/pgadmin4:latest container_name: pgadmin4 

environment: PGADMIN_DEFAULT_EMAIL: admin@pens.ac.id PGADMIN_DEFAULT_PASSWORD: admin123 PGADMIN_LISTEN_PORT: 5050 ports: - "5050:5050" volumes: - pgadmin-data:/var/lib/pgadmin networks: - db-net depends_on: db: condition: service_healthy restart: unless-stopped 

volumes: pg-data: pg-logs: pgadmin-data: 

networks: db-net: EOF 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0052-03.png)


1.4 Deploy docker compose up -d 

docker compose ps 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0052-06.png)


# Tunggu hingga healthcheck pass 

docker compose logs db | tail -20 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0053-01.png)


## **Langkah 2: Koneksi dan Verifikasi Database** 

2.1 Koneksi via psql dari host # Install psql client (jika belum) sudo apt install -y postgresql-client 

# Koneksi ke database psql -h localhost -U labuser -d labdb 

# Di dalam psql: 

- \l                          -- list databases \dn                         -- list schemas \dt app.*                   -- list tabel di schema app \d+ app.mahasiswa           -- describe tabel mahasiswa SELECT * FROM app.mahasiswa; SELECT * FROM app.matakuliah; \q                          -- keluar 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0053-07.png)


## 2.2 Koneksi via docker exec 

# Masuk ke psql di dalam container docker exec -it postgres-db psql -U labuser -d labdb 

# Query join: Nilai mahasiswa SELECT m.nrp, m.nama, mk.nama AS matakuliah, n.nilai_angka, n.grade FROM app.nilai n 

JOIN app.mahasiswa m ON n.mahasiswa_id = m.id 

JOIN app.matakuliah mk ON n.matakuliah_id = mk.id ORDER BY m.nrp, mk.nama; 

\q 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0054-02.png)


## 2.3 Koneksi via pgAdmin4 

Buka browser: http://localhost:5050 Login: admin@pens.ac.id / admin123 Add New Server: 

Name: Lab PostgreSQL Host: db (nama service di Docker Compose) Port: 5432 

Database: labdb 

Username: labuser 

Password: labpass123 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0054-09.png)


Navigate: Servers → Lab PostgreSQL → Databases → labdb → Schemas → app → Tables 

Klik kanan tabel mahasiswa → View/Edit Data → All Rows 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0054-12.png)


## **Langkah 3: Operasi CRUD SQL** 

docker exec -it postgres-db psql -U labuser -d labdb << 'SQLEOF' 

## -- === CREATE === 

INSERT INTO app.mahasiswa (nrp, nama, kelas, kelompok, email) VALUES ('3122600010', 'Fajar Rizki', 'D', 5, 'fajar@student.pens.ac.id'); 

## -- === READ === 

-- Semua mahasiswa kelas A 

SELECT * FROM app.mahasiswa WHERE kelas = 'A'; 

-- Rata-rata nilai per matakuliah SELECT mk.nama, AVG(n.nilai_angka)::NUMERIC(5,2) AS rata_rata, COUNT(*) AS jumlah FROM app.nilai n JOIN app.matakuliah mk ON n.matakuliah_id = mk.id GROUP BY mk.nama ORDER BY rata_rata DESC; 

## -- === UPDATE === 

UPDATE app.mahasiswa SET email = 'fajar.rizki@student.pens.ac.id' WHERE nrp = '3122600010'; 

## -- === DELETE === 

DELETE FROM app.mahasiswa WHERE nrp = '3122600010'; 

-- === JSONB query (untuk tabel activity_log) === INSERT INTO app.activity_log (level, source, message, metadata) VALUES ('INFO', 'web-app', 'User login', '{"user": "admin", "ip": "192.168.1.10"}'); 

SELECT * FROM app.activity_log WHERE metadata->>'user' = 'admin'; 

SQLEOF 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0055-15.png)


## **Langkah 4: Backup dan Restore** 

4.1 Backup database (pg_dump) 

# Backup dalam format custom (compressed, restorable) 

docker exec postgres-db pg_dump -U labuser -d labdb -Fc \ 

- -f /backup/labdb_backup.dump 

## # Backup dalam format SQL plain text 

docker exec postgres-db pg_dump -U labuser -d labdb \ 

- -f /backup/labdb_backup.sql 

## # Backup hanya schema app 

docker exec postgres-db pg_dump -U labuser -d labdb -n app -Fc \ 

- -f /backup/labdb_schema_app.dump 

## # Verifikasi file backup di host 

ls -la backup/ 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0056-10.png)


## 4.2 Restore database 

# Buat database baru untuk restore test 

docker exec postgres-db psql -U labuser -d postgres -c "CREATE DATABASE labdb_restore;" 

# Restore dari custom format 

docker exec postgres-db pg_restore -U labuser -d labdb_restore \ /backup/labdb_backup.dump 

## # Verifikasi restore 

docker exec postgres-db psql -U labuser -d labdb_restore -c "SELECT * FROM app.mahasiswa;" 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0056-18.png)


4.3 Backup otomatis dengan cron di container # Buat script backup cat > backup/auto-backup.sh << 'BASH' #!/bin/sh 

TIMESTAMP=$(date +%Y%m%d_%H%M%S) BACKUP_FILE="/backup/labdb_${TIMESTAMP}.dump" pg_dump -U labuser -d labdb -Fc -f "$BACKUP_FILE" echo "[$(date)] Backup created: $BACKUP_FILE" 

## # Hapus backup lebih dari 7 hari 

find /backup -name "labdb_*.dump" -mtime +7 -delete BASH chmod +x backup/auto-backup.sh 

## # Test jalankan manual 

docker exec postgres-db /backup/auto-backup.sh ls -la backup/ 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0057-06.png)


## **Langkah 5: Monitoring PostgreSQL** 

## 5.1 Statistik database 

docker exec -it postgres-db psql -U labuser -d labdb << 'SQLEOF' 

## -- Ukuran database 

SELECT pg_database.datname, pg_size_pretty(pg_database_size(pg_database.datname)) AS size FROM pg_database ORDER BY pg_database_size(pg_database.datname) DESC; 

-- Ukuran per tabel SELECT schemaname || '.' || tablename AS table_full, pg_size_pretty(pg_total_relation_size(schemaname || '.' || tablename)) AS total_size FROM pg_tables WHERE schemaname = 'app' ORDER BY pg_total_relation_size(schemaname || '.' || tablename) DESC; 

## -- Koneksi aktif 

SELECT pid, usename, datname, client_addr, state, query_start, query 

FROM pg_stat_activity WHERE datname = 'labdb'; 

- -- Statistik tabel (hits, reads, cache ratio) 

SELECT relname, 

seq_scan, seq_tup_read, 

idx_scan, idx_tup_fetch, 

n_tup_ins, n_tup_upd, n_tup_del 

FROM pg_stat_user_tables WHERE schemaname = 'app'; 

## SQLEOF 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0058-08.png)


## 5.2 Cek PostgreSQL log 

# Lihat log PostgreSQL 

docker exec postgres-db ls /var/log/postgresql/ 

docker exec postgres-db cat /var/log/postgresql/postgresql-$(date +%Y-%m-%d).log | tail -30 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0058-13.png)


## **PERTANYAAN** 

## **Pre-lab** 

1. Apa fungsi file/folder /docker-entrypoint-initdb.d/ di image PostgreSQL? Folder /docker-entrypoint-initdb.d/ adalah mekanisme inisialisasi otomatis yang disediakan image resmi PostgreSQL. Saat container PostgreSQL pertama kali dijalankan dan direktori data (PGDATA) masih kosong, entrypoint script akan 

memindai folder ini dan mengeksekusi semua file .sql dan .sh yang ditemukan secara berurutan sesuai urutan alfanumerik nama file. Itulah mengapa penamaan file menggunakan prefix angka seperti 01-create-schema.sql, 02-seed-data.sql — untuk memastikan urutan eksekusi yang benar. 

Mekanisme ini hanya berjalan sekali saja — pada saat volume masih kosong dan PostgreSQL melakukan initdb untuk pertama kali. Jika volume sudah berisi data (container pernah dijalankan sebelumnya), folder ini diabaikan sepenuhnya meski isinya berubah. Ini adalah fitur yang sangat berguna untuk otomatisasi pembuatan schema, user, dan data awal tanpa perlu menjalankan script secara manual setelah container berjalan. 

2. Mengapa POSTGRES_PASSWORD wajib diset? Apa risikonya jika tidak ada password? 

   - Image resmi PostgreSQL menolak untuk start jika POSTGRES_PASSWORD tidak diset dan tidak ada konfigurasi alternatif yang diberikan ini adalah keputusan desain yang disengaja oleh maintainer image untuk mencegah deployment database tanpa password secara tidak sengaja 

Risikonya sangat serius jika dibiarkan tanpa password: siapapun yang bisa menjangkau port PostgreSQL (5432) dapat langsung terhubung sebagai superuser tanpa autentikasi apapun dan memiliki akses penuh ke seluruh database termasuk membaca, mengubah, menghapus semua data, bahkan menghapus seluruh database cluster. Di lingkungan cloud atau server yang port-nya terbuka ke internet, ini berarti database bisa dieksploitasi dalam hitungan menit oleh automated scanner. Bahkan di lingkungan development lokal, kebiasaan tidak memakai password menormalisasi praktik buruk yang berbahaya jika terbawa ke production. 

3. Jelaskan perbedaan antara pg_dump format custom (-Fc) dan format SQL plain text. 

   - Format SQL plain text (default tanpa flag format) menghasilkan file berisi perintah SQL murni yang bisa dibaca manusia dan dieksekusi dengan psql. Kelebihannya sangat portabel bisa dibuka di text editor, bisa diedit secara manual, dan bisa di-restore ke PostgreSQL versi berbeda atau bahkan database lain yang kompatibel. Kelemahannya adalah ukuran file besar karena tidak ada kompresi, dan proses restore harus linear dari awal sampai akhir. 

   - Format custom (-Fc) adalah format biner terkompresi yang eksklusif milik PostgreSQL. Kelebihannya: ukuran file jauh lebih kecil karena menggunakan kompresi zlib, proses restore lebih cepat, dan yang terpenting mendukung restore selektif dengan pg_restore bisa memilih hanya tabel atau schema tertentu yang ingin di-restore tanpa harus memproses seluruh file. Format ini juga mendukung restore paralel dengan flag -j untuk mempercepat proses pada database besar. Kelemahannya tidak bisa dibaca manusia dan hanya bisa di-restore menggunakan pg_restore, tidak bisa di-pipe langsung ke psql. 

Untuk production, format custom selalu lebih disarankan karena efisiensi ruang dan fleksibilitas restore-nya. 

4. Apa itu shared_buffers dan mengapa perlu disesuaikan untuk container? shared_buffers adalah parameter PostgreSQL yang menentukan berapa banyak memory yang dialokasikan sebagai cache bersama untuk menyimpan halaman data (data pages) yang sering diakses. Ketika query membutuhkan data, PostgreSQL pertama kali mencari di shared_buffers jika ada (cache hit) langsung dikembalikan tanpa membaca disk, jika tidak ada (cache miss) baru membaca dari disk dan menyimpan salinannya di shared_buffers untuk akses berikutnya. 

Rekomendasi umum untuk server fisik adalah mengatur shared_buffers sebesar 25% dari total RAM. Namun di lingkungan container, situasinya berbeda: container biasanya diberi memory limit yang jauh lebih kecil dari total RAM host, dan banyak container bisa berjalan bersamaan berbagi RAM yang sama. Jika shared_buffers mengikuti default PostgreSQL (128MB) atau diset terlalu besar tanpa mempertimbangkan memory limit container, PostgreSQL bisa melebihi batas memory container dan di-kill oleh kernel (OOM Killer). Sebaliknya jika terlalu kecil, performa query buruk karena sering terjadi cache miss. Nilai yang wajar untuk container lab dengan RAM terbatas adalah 128MB dengan effective_cache_size disesuaikan proporsional. 

5. Mengapa data PostgreSQL harus disimpan di Docker Volume, bukan di container layer? 

Container layer (writable layer) menggunakan copy-on-write filesystem yang dirancang untuk perubahan sementara selama container hidup bukan untuk penyimpanan data jangka panjang. Ada beberapa alasan kritis mengapa PostgreSQL tidak boleh menyimpan data di container layer: 

Pertama, data hilang permanen saat container dihapus (docker rm) ini skenario yang sangat mudah terjadi secara tidak sengaja. Kedua, performa buruk operasi I/O melalui copy-on-write filesystem (seperti overlay2) secara signifikan lebih lambat dibanding akses langsung ke volume, dan PostgreSQL sangat sensitif terhadap performa I/O terutama untuk operasi WAL (Write-Ahead Log). Ketiga, tidak bisa di-share volume bisa di-mount ke container lain untuk backup atau replikasi, sedangkan container layer tidak. Keempat, tidak bisa di-backup dengan mudah volume bisa di-backup dengan one-liner docker run --volumes-from, sedangkan data di container layer memerlukan cara yang lebih rumit. 

## **Post-lab** 

1. Jalankan docker compose down lalu docker compose up -d. Apakah data mahasiswa masih ada? Buktikan. 

Data mahasiswa tetap ada setelah siklus down-up tersebut. Pembuktiannya dengan menjalankan query setelah container kembali running: 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0060-08.png)


Data tetap ada karena docker compose down tanpa flag -v hanya menghapus container dan network, sementara named volume pg-data tetap tersimpan di host di /var/lib/docker/volumes/. Saat container baru dibuat, ia me-mount volume yang sama dan PostgreSQL menemukan data directory yang sudah terisi sehingga langsung melanjutkan tanpa inisialisasi ulang. 

2. Jalankan docker compose down -v lalu docker compose up -d. Apa yang terjadi? Apakah init script dijalankan ulang? 

   - Setelah docker compose down -v, volume pg-data dihapus permanen. Saat docker compose up -d dijalankan kembali, PostgreSQL mendapati PGDATA kosong dan menjalankan proses initdb dari awal. Karena volume kosong, init script di /docker-entrypoint-initdb.d/ dijalankan ulang secara otomatis, sehingga schema app, semua tabel, index, dan data sample disisipkan kembali seperti pertama kali. 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0061-03.png)


3. Bandingkan ukuran file backup format custom vs SQL. Mana yang lebih kecil dan mengapa? 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0061-05.png)


File format custom (.dump) jauh lebih kecil bisa 3–5 kali lebih kecil dari format SQL plain text untuk database yang sama. Alasannya adalah format custom menggunakan kompresi zlib secara internal terhadap data sebelum ditulis ke file. Format SQL plain text menyimpan setiap perintah INSERT, CREATE TABLE, ALTER TABLE, dan komentar sebagai teks biasa tanpa kompresi apapun semakin banyak data, semakin besar perbedaan ukurannya. Untuk database besar di production yang berisi jutaan baris, perbedaan ukuran ini bisa mencapai puluhan GB dan sangat signifikan untuk storage dan transfer biaya. 

4. Buat query yang menampilkan mahasiswa yang belum memiliki nilai di semester apapun. 

   - Query menggunakan LEFT JOIN antara tabel mahasiswa dan nilai, lalu memfilter baris di mana tidak ada pasangan di tabel nilai (hasilnya NULL): 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0062-00.png)



![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0062-01.png)



![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0062-02.png)


Ketiga query menghasilkan output yang sama. Metode LEFT JOIN dan NOT EXISTS lebih direkomendasikan karena performanya lebih baik dan perilakunya lebih prediktif dibanding NOT IN saat ada nilai NULL. Dari data sample yang diinsert di init script, mahasiswa dengan id yang tidak memiliki entri apapun di tabel nilai akan muncul di hasil query ini. 

5. Jelaskan peran user app_reader yang dibuat di init script. Apa bedanya dengan labuser? 

labuser adalah superuser (atau minimal user dengan hak penuh) yang dibuat melalui environment variable POSTGRES_USER. Ia memiliki hak CREATE, INSERT, UPDATE, DELETE, DROP, ALTER, dan TRUNCATE pada seluruh database pada dasarnya bisa melakukan apapun termasuk menghapus tabel atau bahkan seluruh database. User ini digunakan oleh aplikasi backend (Flask) dan untuk administrasi database. 

app_reader adalah read-only user yang dibuat di init script dengan hak yang sangat terbatas secara eksplisit: 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0062-07.png)


app_reader hanya bisa menjalankan SELECT pada tabel-tabel di schema app tidak bisa INSERT, UPDATE, DELETE, tidak bisa membuat tabel baru, tidak bisa mengakses schema lain. Jika credential app_reader bocor atau aplikasi yang menggunakannya dieksploitasi, attacker hanya bisa membaca data, tidak bisa memodifikasi atau menghapusnya. 

Ini menerapkan prinsip least privilege setiap komponen sistem hanya mendapat hak akses minimum yang diperlukan untuk fungsinya. Misalnya, service reporting atau dashboard yang hanya perlu membaca data menggunakan app_reader, sementara hanya service yang benar-benar perlu menulis data yang menggunakan labuser. Dengan cara ini, jika salah satu service dikompromikan, radius kerusakan yang bisa dilakukan attacker menjadi terbatas. 


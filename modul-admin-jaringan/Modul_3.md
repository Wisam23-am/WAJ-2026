## **DOCKER-INTRO Web Service di Docker — Apache & Nginx WORKSHOP ADMIN JARINGAN** 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0030-01.png)


NAMA: AQILA WISAM MADANI MA’MURRIE NRP : 3124600041 KELAS : D4 IT B KELOMPOK : 2 TANGGAL PRAKTIKUM : 3 MEI 2026 DOSEN : Dr Ferry Astika Saputra ST, M.Sc 

## **POLITEKNIK ELEKTRONIKA NEGERI SURABAYA** 

**2025/2026** 

## **MODUL 3 : Web Service di Docker — Apache & Nginx LANGKAH PRAKTIKUM** 

## **Langkah 0: Persiapan Project** 

mkdir -p 

~/docker-lab/web-service/{apache/{sites,html-site1,html-site2},nginx,flask,certs,logs} cd ~/docker-lab/web-service 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0031-04.png)


## **Langkah 1: Konfigurasi Apache2 dengan Virtual Host** 

1.1 Buat halaman web untuk dua virtual host # Site 1: Company Profile cat > apache/html-site1/index.html << 'EOF' 

<!DOCTYPE html> <html lang="id"> <head><meta charset="UTF-8"><title>Site 1 - Company</title> <style>body{font-family:sans-serif;text-align:center;padding:50px;background:#1a237e; color:white;} 

.box{background:rgba(255,255,255,0.1);padding:30px;border-radius:12px;max-width:50 0px;margin:0 auto;}</style> </head> <body><div class="box"> <h1>🏢 Site 1 — Company Profile</h1> <p>Virtual Host: <strong>site1.lab</strong></p> <p>Server: Apache httpd di Docker</p> </div></body></html> EOF 

# Site 2: Blog cat > apache/html-site2/index.html << 'EOF' <!DOCTYPE html> <html lang="id"> <head><meta charset="UTF-8"><title>Site 2 - Blog</title> <style>body{font-family:sans-serif;text-align:center;padding:50px;background:#1b5e20; color:white;} 

.box{background:rgba(255,255,255,0.1);padding:30px;border-radius:12px;max-width:50 0px;margin:0 auto;}</style> </head> <body><div class="box"> <h1>📝 Site 2 — Blog</h1> <p>Virtual Host: <strong>site2.lab</strong></p> <p>Server: Apache httpd di Docker</p> </div></body></html> EOF 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0032-00.png)


## 1.2 Buat konfigurasi Apache Virtual Host 

cat > apache/sites/vhosts.conf << 'EOF' # ============================================ 

# Apache Virtual Host Configuration (Docker) 

# ============================================ 

# --- Site 1: site1.lab --- 

<VirtualHost *:80> ServerName site1.lab ServerAlias www.site1.lab DocumentRoot /usr/local/apache2/htdocs/site1 

<Directory /usr/local/apache2/htdocs/site1> AllowOverride None Require all granted </Directory> 

# Per-site logging 

ErrorLog /var/log/apache2/site1-error.log CustomLog /var/log/apache2/site1-access.log combined </VirtualHost> 

# --- Site 2: site2.lab --- 

<VirtualHost *:80> ServerName site2.lab ServerAlias www.site2.lab 

DocumentRoot /usr/local/apache2/htdocs/site2 

<Directory /usr/local/apache2/htdocs/site2> AllowOverride None Require all granted </Directory> 

ErrorLog /var/log/apache2/site2-error.log CustomLog /var/log/apache2/site2-access.log combined </VirtualHost> 

EOF 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0033-00.png)


## 1.3 Buat Dockerfile Apache 

cat > apache/Dockerfile << 'EOF' FROM httpd:2.4-alpine 

## # Aktifkan module yang dibutuhkan 

RUN sed -i \ 

- -e 's/#LoadModule vhost_alias_module/LoadModule vhost_alias_module/' \ 

- -e 's/#LoadModule rewrite_module/LoadModule rewrite_module/' \ 

/usr/local/apache2/conf/httpd.conf 

## # Include virtual host config 

RUN echo "Include conf/extra/vhosts.conf" >> /usr/local/apache2/conf/httpd.conf 

## # Buat direktori log 

RUN mkdir -p /var/log/apache2 

## # Copy virtual host config 

COPY sites/vhosts.conf /usr/local/apache2/conf/extra/vhosts.conf 

## # Copy site files 

COPY html-site1/ /usr/local/apache2/htdocs/site1/ 

COPY html-site2/ /usr/local/apache2/htdocs/site2/ 

EXPOSE 80 

EOF 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0033-19.png)


## **Langkah 2: Generate Self-Signed SSL Certificate** 

# Generate SSL certificate untuk *.lab (wildcard) openssl req -x509 -nodes -days 365 \ -newkey rsa:2048 \ 

- -keyout certs/server.key \ 

-out certs/server.crt \ 

- -subj "/C=ID/ST=Jawa Timur/L=Surabaya/O=PENS Lab/CN=*.lab" \ -addext "subjectAltName=DNS:*.lab,DNS:site1.lab,DNS:site2.lab,DNS:app.lab" 

## # Verifikasi certificate 

openssl x509 -in certs/server.crt -noout -text | head -20 

ls -la certs/ 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0034-08.png)


## **Langkah 3: Konfigurasi Nginx sebagai Reverse Proxy + SSL Termination** 

cat > nginx/default.conf << 'EOF' 

# ============================================ 

# Nginx Reverse Proxy + SSL Configuration 

# ============================================ 

# --- Upstream definitions --upstream apache_backend { server apache-web:80; } upstream flask_backend { server flask-app:5000; } # --- HTTP: Redirect ke HTTPS --server { listen 80; server_name site1.lab site2.lab app.lab; return 301 https://$host$request_uri; } # --- HTTPS: site1.lab → Apache --server { listen 443 ssl; 

server_name site1.lab; 

ssl_certificate     /etc/nginx/certs/server.crt; ssl_certificate_key /etc/nginx/certs/server.key; ssl_protocols       TLSv1.2 TLSv1.3; 

location / { 

proxy_pass http://apache_backend; proxy_set_header Host $host; proxy_set_header X-Real-IP $remote_addr; proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; proxy_set_header X-Forwarded-Proto $scheme; } 

access_log /var/log/nginx/site1-access.log; error_log  /var/log/nginx/site1-error.log; } 

# --- HTTPS: site2.lab → Apache --server { listen 443 ssl; server_name site2.lab; 

ssl_certificate     /etc/nginx/certs/server.crt; ssl_certificate_key /etc/nginx/certs/server.key; ssl_protocols       TLSv1.2 TLSv1.3; 

location / { proxy_pass http://apache_backend; proxy_set_header Host $host; proxy_set_header X-Real-IP $remote_addr; proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; proxy_set_header X-Forwarded-Proto $scheme; } access_log /var/log/nginx/site2-access.log; error_log  /var/log/nginx/site2-error.log; } 

# --- HTTPS: app.lab → Flask --server { listen 443 ssl; server_name app.lab; ssl_certificate     /etc/nginx/certs/server.crt; ssl_certificate_key /etc/nginx/certs/server.key; ssl_protocols       TLSv1.2 TLSv1.3; 

location / { proxy_pass http://flask_backend; 

proxy_set_header Host $host; proxy_set_header X-Real-IP $remote_addr; proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; proxy_set_header X-Forwarded-Proto $scheme; } 

access_log /var/log/nginx/app-access.log; error_log  /var/log/nginx/app-error.log; } # --- Default: catch-all --server { listen 80 default_server; listen 443 ssl default_server; 

ssl_certificate     /etc/nginx/certs/server.crt; ssl_certificate_key /etc/nginx/certs/server.key; 

return 444; } EOF 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0036-04.png)


## **Langkah 4: Buat Flask Backend App** 

cat > flask/requirements.txt << 'EOF' flask==3.1.* psycopg2-binary==2.9.* EOF 

cat > flask/app.py << 'PYEOF' import os, socket, datetime from flask import Flask, jsonify, request import psycopg2 

app = Flask(__name__) 

def get_db(): return psycopg2.connect( host=os.environ.get("DB_HOST", "db"), dbname=os.environ.get("DB_NAME", "labdb"), user=os.environ.get("DB_USER", "labuser"), password=os.environ.get("DB_PASS", "labpass123")) 

@app.route("/") def index(): return jsonify({ "service": "Flask Backend API", "hostname": socket.gethostname(), "timestamp": datetime.datetime.now().isoformat(), "client_ip": request.headers.get("X-Real-IP", request.remote_addr), "proto": request.headers.get("X-Forwarded-Proto", "http") }) @app.route("/api/health") def health(): result = {"status": "ok", "database": "unknown"} try: conn = get_db() cur = conn.cursor() cur.execute("SELECT version();") result["database"] = cur.fetchone()[0] result["db_status"] = "connected" cur.close(); conn.close() except Exception as e: result["db_status"] = f"error: {e}" return jsonify(result) @app.route("/api/visitors", methods=["POST"]) def add_visitor(): try: conn = get_db(); cur = conn.cursor() cur.execute(""" CREATE TABLE IF NOT EXISTS visitors ( id SERIAL PRIMARY KEY, name VARCHAR(100), visited_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP )""") name = request.json.get("name", "anonymous") cur.execute("INSERT INTO visitors (name) VALUES (%s) RETURNING id, visited_at", (name,)) row = cur.fetchone() conn.commit(); cur.close(); conn.close() return jsonify({"id": row[0], "name": name, "visited_at": str(row[1])}), 201 except Exception as e: return jsonify({"error": str(e)}), 500 @app.route("/api/visitors", methods=["GET"]) def get_visitors(): try: conn = get_db(); cur = conn.cursor() cur.execute("SELECT id, name, visited_at FROM visitors ORDER BY id DESC LIMIT 20") rows = [{"id": r[0], "name": r[1], "visited_at": str(r[2])} for r in cur.fetchall()] 

cur.close(); conn.close() return jsonify(rows) except Exception as e: return jsonify({"error": str(e)}), 500 if __name__ == "__main__": app.run(host="0.0.0.0", port=5000) PYEOF cat > flask/Dockerfile << 'EOF' FROM python:3.11-slim WORKDIR /app COPY requirements.txt . RUN pip install --no-cache-dir -r requirements.txt COPY app.py . EXPOSE 5000 CMD ["python", "app.py"] EOF 

**Langkah 5: Buat Docker Compose** cat > docker-compose.yml << 'EOF' services: # --- Nginx Reverse Proxy + SSL Termination --proxy: image: nginx:alpine container_name: nginx-proxy ports: - "8080:80" - "8443:443" volumes: - ./nginx/default.conf:/etc/nginx/conf.d/default.conf:ro - ./certs:/etc/nginx/certs:ro - nginx-logs:/var/log/nginx networks: - web-net depends_on: - apache-web - flask-app restart: unless-stopped # --- Apache Web Server (Virtual Hosts) --apache-web: build: ./apache container_name: apache-web volumes: - apache-logs:/var/log/apache2 networks: - web-net restart: unless-stopped 

# --- Flask Backend API --flask-app: build: ./flask container_name: flask-app environment: - DB_HOST=db - DB_NAME=labdb - DB_USER=labuser - DB_PASS=labpass123 networks: - web-net - db-net depends_on: db: condition: service_healthy restart: unless-stopped # --- PostgreSQL Database --db: image: postgres:16-alpine container_name: postgres-db environment: POSTGRES_DB: labdb POSTGRES_USER: labuser POSTGRES_PASSWORD: labpass123 volumes: - pg-data:/var/lib/postgresql/data networks: - db-net healthcheck: test: ["CMD-SHELL", "pg_isready -U labuser -d labdb"] interval: 5s timeout: 5s retries: 5 restart: unless-stopped volumes: pg-data: nginx-logs: apache-logs: networks: web-net: db-net: EOF 

**Langkah 6: Deploy dan Testing** 6.1 Tambahkan DNS lokal # Tambahkan entry ke /etc/hosts echo "127.0.0.1  site1.lab site2.lab app.lab" | sudo tee -a /etc/hosts 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0040-00.png)


## 6.2 Build dan jalankan 

docker compose up --build -d docker compose ps 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0040-03.png)


6.3 Test Virtual Host Apache via Nginx Proxy 

# Test HTTP → HTTPS redirect 

curl -I http://site1.lab:8080 

# Test Site 1 (Apache via Nginx proxy, skip SSL verify) curl -k https://site1.lab:8443 

# Harus tampil: "Site 1 — Company Profile" 

# Test Site 2 

curl -k https://site2.lab:8443 

# Harus tampil: "Site 2 — Blog" 

## # Test Flask API 

curl -k https://app.lab:8443 curl -k https://app.lab:8443/api/health | python3 -m json.tool 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0040-14.png)


6.4 Test API CRUD 

## # Tambah visitor 

curl -k -X POST https://app.lab:8443/api/visitors \ 

- -H "Content-Type: application/json" \ 

- -d '{"name": "Mahasiswa PENS"}' 

## # Lihat daftar visitor 

curl -k https://app.lab:8443/api/visitors | python3 -m json.tool 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0041-07.png)


## 6.5 Cek SSL Certificate 

## # Lihat detail certificate 

echo | openssl s_client -connect site1.lab:8443 -servername site1.lab 2>/dev/null | \ openssl x509 -noout -subject -issuer -dates 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0041-11.png)


## 6.6 Analisis Log 

## # Log Nginx 

docker exec nginx-proxy cat /var/log/nginx/site1-access.log docker exec nginx-proxy cat /var/log/nginx/app-access.log 

## # Log Apache 

docker exec apache-web cat /var/log/apache2/site1-access.log 

## # Atau akses via volume 

docker run --rm -v $(docker compose config --volumes | head -1):/logs alpine ls /logs 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0041-19.png)


## **PERTANYAAN** 

## **Pre-Lab** 

1. Apa keuntungan menjalankan web server di container dibandingkan langsung di host? 

Menjalankan web server di container memberikan beberapa keuntungan dibanding instalasi langsung di host. 

   - Pertama, isolasi konfigurasi setiap project memiliki konfigurasi Nginx atau Apache sendiri tanpa saling mengganggu, berbeda dengan instalasi di host yang berbagi satu file konfigurasi global. 

   - Kedua, reproducibility environment yang sama persis bisa dijalankan di laptop developer, staging, maupun production hanya dengan satu perintah docker compose up, menghilangkan masalah "works on my machine". 

   - Ketiga, kemudahan versi bisa menjalankan Apache 2.4 dan Nginx secara bersamaan di host yang sama tanpa konflik dependensi. 

   - Keempat, portabilitas seluruh konfigurasi, virtual host, dan SSL certificate bisa di-commit ke repository dan dibawa ke server manapun. 

   - Kelima, cleanup bersih menghapus container tidak meninggalkan sisa file konfigurasi di sistem host seperti yang sering terjadi saat uninstall paket dari apt. 

2. Jelaskan perbedaan document root Apache (/usr/local/apache2/htdocs/) vs Nginx (/usr/share/nginx/html/). Perbedaannya murni pada konvensi path yang ditetapkan masing-masing software. 

   - Apache menggunakan /usr/local/apache2/htdocs/ sebagai document root default karena image resminya di-build dari source menggunakan prefix /usr/local/apache2/, sehingga seluruh struktur direktori (config, logs, htdocs) berada di bawah path tersebut. 

   - Nginx menggunakan /usr/share/nginx/html/ karena mengikuti konvensi FHS (Filesystem Hierarchy Standard) Linux di mana konten web statis ditempatkan di bawah /usr/share/. 

Secara fungsional keduanya identik keduanya adalah direktori tempat web server mencari file yang akan dikirim ke browser saat ada request. Implikasi praktisnya adalah saat membuat bind mount atau COPY di Dockerfile, path tujuan harus disesuaikan dengan masing-masing software ini. 

3. Apa itu SSL Termination dan mengapa dilakukan di reverse proxy? SSL Termination adalah proses mendekripsi traffic HTTPS yang masuk di satu titik, lalu meneruskan request tersebut ke backend sebagai HTTP biasa (tidak terenkripsi) di dalam network internal. Reverse proxy seperti Nginx yang melakukan SSL Termination berarti ia yang memegang certificate dan private key, menangani proses TLS handshake dengan client, lalu berkomunikasi dengan backend (Apache, Flask) menggunakan HTTP plaintext di dalam Docker network yang sudah terisolasi. 

Alasan dilakukan di reverse proxy dan bukan di masing-masing backend: pertama, sentralisasi certificate hanya perlu satu certificate untuk semua backend, tidak perlu mendistribusikan certificate ke setiap service; kedua, 

performa proses kriptografi TLS dibebankan ke satu titik yang bisa dioptimasi, backend tidak perlu mengalokasikan resource untuk enkripsi; ketiga, simplifikasi backend Flask atau Apache tidak perlu dikonfigurasi untuk SSL sama sekali; keempat, network internal aman traffic antar container di Docker network internal sudah terisolasi, sehingga HTTP plaintext di sana tidak berisiko seperti di jaringan publik. 

4. Apa perbedaan name-based dan IP-based virtual hosting? IP-based virtual hosting menggunakan IP address yang berbeda untuk setiap website server harus memiliki beberapa IP address, dan request ke IP tertentu diarahkan ke virtual host yang sesuai. Pendekatan ini boros IP address dan tidak praktis di era IPv4 yang terbatas. 

Name-based virtual hosting menggunakan satu IP address untuk banyak website, dibedakan berdasarkan nilai header Host yang dikirim browser dalam setiap HTTP request. Ketika browser mengakses site1.lab, ia mengirim Host: site1.lab di header, dan web server menggunakan nilai ini untuk menentukan virtual host mana yang harus merespons. Inilah yang digunakan di praktikum ini satu container Apache dengan satu IP melayani site1.lab dan site2.lab secara bersamaan berdasarkan header ServerName. Name-based virtual hosting adalah standar yang hampir universal digunakan saat ini karena efisiensi penggunaan IP address. 

5. Mengapa self-signed certificate menghasilkan warning di browser? Browser mempercayai sebuah certificate SSL bukan karena isi certificate-nya, melainkan karena certificate tersebut ditandatangani oleh Certificate Authority (CA) yang sudah ada dalam trust store browser. Trust store adalah daftar CA terpercaya (seperti DigiCert, Let's Encrypt, Comodo) yang sudah diverifikasi secara independen dan dimasukkan ke dalam browser atau sistem operasi. 

Self-signed certificate ditandatangani oleh dirinya sendiri  tidak ada CA pihak ketiga yang memverifikasi bahwa pemilik certificate benar-benar pemilik domain tersebut. Browser tidak mengenal "otoritas" yang menandatangani certificate ini, sehingga tidak bisa memvalidasi keasliannya dan menampilkan warning "Your connection is not private". Warning ini bukan berarti enkripsinya lemah data tetap terenkripsi melainkan berarti identitas server tidak dapat diverifikasi oleh pihak ketiga yang terpercaya. Untuk production, solusinya menggunakan certificate dari CA publik seperti Let's Encrypt yang gratis dan diakui semua browser. 

## **Post-lab** 

1. Bandingkan response header dari Apache vs Nginx. Header apa yang menunjukkan software web server? 

   - Header yang paling jelas menunjukkan software web server adalah Server. Caranya dengan membandingkan header response menggunakan curl -I: 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0044-00.png)


Pada response dari Apache, header Server akan menampilkan sesuatu seperti Server: Apache/2.4.62 (Unix). Pada response dari Nginx (baik sebagai server maupun proxy), header Server akan menampilkan Server: nginx/1.27.x. Ketika mengakses via Nginx reverse proxy ke Apache, yang terlihat di response adalah header Nginx karena Nginx yang merespons langsung ke client. Apache bisa menyembunyikan versinya dengan direktif ServerTokens Prod, dan Nginx dengan server_tokens off, yang merupakan praktik keamanan standar agar versi software tidak terekspos ke publik 

2. Jika Nginx proxy down, apakah Apache masih bisa diakses langsung? Bagaimana cara testnya? 

Ya, Apache masih bisa diakses langsung selama container apache-web masih berjalan, karena Apache berjalan sebagai service independen. Nginx hanyalah reverse proxy di depannya ia tidak mempengaruhi apakah Apache bisa menerima koneksi atau tidak. 

Cara testnya: pertama, stop container Nginx dengan docker stop nginx-proxy. Kedua, coba akses via Nginx hasilnya akan gagal karena proxy tidak ada. Ketiga, akses Apache langsung dengan cara menambahkan port mapping sementara atau exec langsung dari dalam network: 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0044-05.png)


Hasilnya menunjukkan Apache tetap merespons secara normal, membuktikan kedua service benar-benar independent. 

3. Tunjukkan bahwa X-Real-IP header diteruskan dengan benar dari Nginx ke Flask. 

Header X-Real-IP diteruskan oleh Nginx melalui direktif proxy_set_header X-Real-IP $remote_addr di konfigurasi. Flask kemudian bisa membaca header ini dan menampilkannya di response. Cara verifikasinya: 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0045-01.png)


Pada response JSON dari Flask, akan ada field yang menampilkan IP pengirim yang diambil dari header X-Real-IP, bukan IP internal container Nginx. Nilai yang muncul seharusnya adalah IP host (127.0.0.1 atau IP lokal) bukan IP internal Docker container Nginx (misalnya 172.x.x.x). Di dalam kode Flask, header dibaca dengan request.headers.get('X-Real-IP') dan ini membuktikan Nginx telah meneruskan IP asli client. Jika mengakses tanpa Nginx (langsung ke Flask port 5000), header X-Real-IP tidak akan ada di request 

4. Jelaskan mengapa Flask app perlu terhubung ke dua network (web-net dan db-net). 

   - Flask perlu ada di web-net dan db-net sekaligus karena ia berperan sebagai jembatan antara dua lapisan aplikasi yang sengaja diisolasi. 

Flask harus berada di web-net agar bisa menerima request yang di-proxy oleh Nginx. Nginx hanya tahu cara menghubungi Flask melalui nama flask-app, dan komunikasi ini hanya mungkin jika keduanya berada di network yang sama. Tanpa web-net, Nginx tidak bisa menjangkau Flask sama sekali. 

Flask harus berada di db-net agar bisa terhubung ke PostgreSQL di db:5432. PostgreSQL sengaja tidak dimasukkan ke web-net sebagai langkah keamanan database tidak boleh bisa dijangkau langsung dari layer web. Hanya Flask yang berhak berkomunikasi dengan database. 

Desain dua network ini menerapkan prinsip least privilege Nginx hanya bisa bicara ke Flask, Flask bisa bicara ke Nginx dan PostgreSQL, tetapi Nginx tidak punya jalur langsung ke PostgreSQL. Jika ada celah keamanan di layer Nginx, attacker tidak bisa langsung menyerang database. 

5. Apa yang terjadi jika file server.key atau server.crt dihapus saat container running? 

   - Nginx membaca file certificate dan private key satu kali saat proses worker pertama kali distart dan memuatnya ke memory. Artinya, jika file server.crt atau server.key dihapus saat container sedang berjalan, koneksi HTTPS yang sudah aktif maupun yang baru masuk tetap bisa diproses karena Nginx menggunakan data yang sudah ada di memory, bukan membaca file setiap ada request. 

Namun dampaknya terasa saat Nginx perlu reload konfigurasi atau restart: jika dijalankan nginx -s reload atau container di-restart, Nginx akan mencoba membaca ulang file certificate dari disk. Karena file sudah tidak ada, proses reload atau start akan gagal dengan error seperti cannot load certificate "/etc/nginx/certs/server.crt": BIO_new_file() failed. Container akan berhenti atau masuk ke state error dan tidak bisa menerima koneksi HTTPS sama sekali. 

Solusinya: pastikan volume ./certs selalu tersedia dan tidak dihapus. Untuk production, simpan certificate di secret manager (Docker Secrets atau Vault) dan pastikan ada monitoring yang memperingatkan jika certificate mendekati expired atau hilang. 


## **DOCKER-INTRO Docker Service, Volume & Mount Point WORKSHOP ADMIN JARINGAN** 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0017-01.png)


NAMA: AQILA WISAM MADANI MA’MURRIE NRP : 3124600041 KELAS : D4 IT B KELOMPOK : 2 TANGGAL PRAKTIKUM : 3 MEI 2026 DOSEN : Dr Ferry Astika Saputra ST, M.Sc 

## **POLITEKNIK ELEKTRONIKA NEGERI SURABAYA** 

**2025/2026** 

## **MODUL 2: Docker Service, Volume & Mount Point LANGKAH PRAKTIKUM** 

## **Langkah 1: Docker Network** 

1.1 Eksplorasi network default 

# List semua network Docker 

docker network ls 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0018-05.png)


# Inspect network bridge default 

docker network inspect bridge 

1.2 Buat user-defined bridge network 

docker network create --driver bridge --subnet 172.20.0.0/16 lab-net 

## # Verifikasi 

docker network ls 

docker network inspect lab-net 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0018-13.png)


## 1.3 Test DNS resolution antar container 

# Jalankan 2 container di network yang sama docker run -d --name server-a --network lab-net nginx:alpine docker run -d --name server-b --network lab-net nginx:alpine 

# Test DNS — container bisa saling resolve by nama docker exec server-a ping -c 3 server-b docker exec server-b ping -c 3 server-a 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0018-17.png)


## # Bandingkan: default bridge TIDAK bisa resolve nama 

docker run -d --name server-c nginx:alpine docker run -d --name server-d nginx:alpine docker exec server-c ping -c 3 server-d   # GAGAL! 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0019-02.png)


## # Cleanup 

docker rm -f server-a server-b server-c server-d 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0019-05.png)


## **Langkah 2: Docker Volume** 

2.1 Buat dan kelola volume 

docker volume create data-vol docker volume ls 

docker volume inspect data-vol 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0019-10.png)


## 2.2 Gunakan volume di container 

# Container penulis — tulis timestamp ke volume setiap 5 detik 

docker run -d --name writer \ 

- -v data-vol:/app/data \ 

alpine:3.20 sh -c "while true; do date >> /app/data/log.txt; sleep 5; done" 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0019-16.png)


# Tunggu 15 detik, lalu baca dari container BERBEDA sleep 15 

docker run --rm -v data-vol:/data alpine:3.20 cat /data/log.txt 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0020-02.png)


## 2.3 Volume persist setelah container dihapus 

docker rm -f writer 

## # Data masih ada! 

docker run --rm -v data-vol:/data alpine:3.20 cat /data/log.txt 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0020-07.png)


## 2.4 Backup dan restore volume 

## # BACKUP 

docker run --rm \ 

- -v data-vol:/source:ro \ 

- -v $(pwd):/backup \ 

alpine:3.20 tar czf /backup/data-vol-backup.tar.gz -C /source . 

## # RESTORE ke volume baru 

docker volume create data-vol-restored 

docker run --rm \ 

- -v data-vol-restored:/target \ 

- -v $(pwd):/backup:ro \ 

alpine:3.20 tar xzf /backup/data-vol-backup.tar.gz -C /target 

## # Verifikasi 

docker run --rm -v data-vol-restored:/data alpine:3.20 cat /data/log.txt 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0020-22.png)


## **Langkah 3: Bind Mount** 

3.1 Buat project 

mkdir -p ~/docker-lab/web-dev/html && cd ~/docker-lab/web-dev 

cat > html/index.html << 'EOF' <html> <body> <h1>Hello dari Bind Mount!</h1> <p>Timestamp: VERSI-1</p> </body> </html> EOF 

3.2 Jalankan container dengan bind mount docker run -d --name dev-server \ -p 8080:80 \ -v $(pwd)/html:/usr/share/nginx/html:ro \ nginx:alpine 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0021-05.png)


3.3 Test live-reload curl http://localhost:8080 

# Edit file di HOST → langsung terlihat di container tanpa restart! sed -i 's/VERSI-1/VERSI-2 (diedit live)/' html/index.html curl http://localhost:8080 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0021-08.png)


docker rm -f dev-server 

## **Langkah 4: tmpfs Mount** 

docker run -d --name tmpfs-demo \ --tmpfs /app/cache:size=64m \ alpine:3.20 sh -c "echo 'secret-data' > /app/cache/token.txt && sleep 3600" 

## # Baca data 

docker exec tmpfs-demo cat /app/cache/token.txt 

# Stop & start → data HILANG 

docker stop tmpfs-demo && docker start tmpfs-demo docker exec tmpfs-demo cat /app/cache/token.txt   # File tidak ada! 

## docker rm -f tmpfs-demo 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0022-02.png)


## **Langkah 5: Aplikasi Multi-Container dengan Docker Compose** 

5.1 Buat project structure 

mkdir -p ~/docker-lab/compose-app/{html,app} && cd ~/docker-lab/compose-app 5.2 Buat halaman Nginx 

cat > html/index.html << 'EOF' 

<!DOCTYPE html> 

<html lang="id"> 

<head> 

<meta charset="UTF-8"><title>Docker Compose Lab</title> 

<style> 

body { font-family: sans-serif; max-width: 700px; margin: 40px auto; padding: 20px; } .card { background: white; border-radius: 10px; padding: 25px; 

box-shadow: 0 2px 10px rgba(0,0,0,0.1); margin-bottom: 20px; } #result { background: #263238; color: #80CBC4; padding: 15px; border-radius: 8px; font-family: monospace; white-space: pre-wrap; } </style> 

</head> <body> <div class="card"> <h1>🐳 Docker Compose Lab</h1> 

<p>Nginx → Flask → PostgreSQL</p> <button onclick="fetchData()">Cek Koneksi Backend</button> <div id="result">Klik tombol untuk test...</div> 

</div> <script> async function fetchData() { try { const r = await fetch('/api/health'); document.getElementById('result').textContent = JSON.stringify(await r.json(), null, 

2); 

} catch(e) { document.getElementById('result').textContent = 'Error: ' + e.message; } } </script> </body></html> EOF 

5.3 Buat Flask app + Dockerfile cat > app/requirements.txt << 'EOF' flask==3.1.* psycopg2-binary==2.9.* EOF cat > app/app.py << 'PYEOF' import os, socket, datetime from flask import Flask, jsonify import psycopg2 app = Flask(__name__) @app.route("/api/health") def health(): result = {"status": "ok", "hostname": socket.gethostname(), "timestamp": datetime.datetime.now().isoformat()} try: conn = psycopg2.connect( host=os.environ.get("DB_HOST", "db"), dbname=os.environ.get("DB_NAME", "labdb"), user=os.environ.get("DB_USER", "labuser"), password=os.environ.get("DB_PASS", "labpass123")) cur = conn.cursor() cur.execute("SELECT version();") result["database"] = cur.fetchone()[0] result["db_status"] = "connected" cur.close(); conn.close() except Exception as e: result["db_status"] = f"error: {e}" return jsonify(result) if __name__ == "__main__": app.run(host="0.0.0.0", port=5000) PYEOF cat > app/Dockerfile << 'EOF' FROM python:3.11-slim WORKDIR /app COPY requirements.txt . RUN pip install --no-cache-dir -r requirements.txt COPY app.py . EXPOSE 5000 CMD ["python", "app.py"] EOF 5.4 Buat Nginx config cat > nginx.conf << 'EOF' server { listen 80; location / { 

root /usr/share/nginx/html; index index.html; } location /api/ { proxy_pass http://app:5000; proxy_set_header Host $host; proxy_set_header X-Real-IP $remote_addr; } } EOF 5.5 Buat docker-compose.yml cat > docker-compose.yml << 'EOF' services: web: image: nginx:alpine container_name: lab-web ports: - "8080:80" volumes: - ./html:/usr/share/nginx/html:ro - ./nginx.conf:/etc/nginx/conf.d/default.conf:ro networks: - frontend depends_on: - app restart: unless-stopped 

app: build: ./app container_name: lab-app environment: - DB_HOST=db - DB_NAME=labdb - DB_USER=labuser - DB_PASS=labpass123 networks: - frontend - backend depends_on: db: condition: service_healthy restart: unless-stopped 

db: image: postgres:16-alpine container_name: lab-db environment: POSTGRES_DB: labdb POSTGRES_USER: labuser POSTGRES_PASSWORD: labpass123 

volumes: - pg-data:/var/lib/postgresql/data networks: - backend healthcheck: test: ["CMD-SHELL", "pg_isready -U labuser -d labdb"] interval: 5s timeout: 5s retries: 5 restart: unless-stopped 

volumes: pg-data: 

networks: frontend: backend: EOF 

5.6 Jalankan dan verifikasi # Build dan start docker compose up --build -d 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0025-04.png)


# Cek status docker compose ps 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0025-06.png)


## # Test 

curl http://localhost:8080 curl http://localhost:8080/api/health | python3 -m json.tool 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0025-09.png)


## # Cek network dan volume 

docker network ls 

docker volume ls 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0026-03.png)


## # Lifecycle 

docker compose logs -f          # follow log 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0026-06.png)



![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0026-07.png)


**----- Start of picture text -----**<br>
docker compose stop             # stop semua<br>docker compose start            # start kembali<br>docker compose down             # stop + hapus container/network (volume tetap)<br>docker compose down -v          # HATI-HATI: hapus termasuk volume!<br>**----- End of picture text -----**<br>



![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0026-08.png)


## **PERTANYAAN** 

## **Pre-Lab** 

1. Apa perbedaan default bridge dan user-defined bridge network? 

      - Default bridge 

         - Dibuat otomatis oleh Docker 

         - Komunikasi antar container pakai IP address 

         - Tidak ada DNS otomatis 

      - User-defined bridge 

         - Dibuat manual (docker network create) 

         - Bisa komunikasi pakai nama container (DNS) 

         - Lebih aman dan terisolasi 

2. Kapan menggunakan Volume vs Bind Mount vs tmpfs? 

      - Gunakan Volume ketika data perlu persisten dan dikelola sepenuhnya oleh Docker — paling cocok untuk database (PostgreSQL, MySQL), file upload aplikasi, dan data yang perlu dibagi antar container. Docker yang mengurus lokasi penyimpanannya di /var/lib/docker/volumes/ sehingga portabel dan aman dari perubahan struktur direktori host. 

      - Gunakan Bind Mount ketika perlu sinkronisasi dua arah antara host dan container secara real-time — paling cocok untuk development (live-reload source code), inject file konfigurasi custom, atau ketika lokasi file di host sudah ditentukan dan harus tetap di sana. 

      - Gunakan tmpfs ketika data bersifat sementara dan sensitif — misalnya token sesi, secret, atau cache yang tidak boleh tersimpan ke disk. tmpfs menyimpan data di RAM sehingga otomatis hilang saat container stop, menjamin tidak ada jejak di storage. 

3. Apa yang terjadi pada named volume saat docker compose down? Bagaimana jika pakai flag -v? 

   - Saat menjalankan docker compose down tanpa flag, named volume tetap utuh dan tidak dihapus. Docker hanya menghentikan dan menghapus container serta network yang dibuat Compose, tetapi volume sengaja dibiarkan agar data persisten tidak hilang secara tidak sengaja. Ini berarti saat docker compose up kembali dijalankan, container baru akan menemukan data yang sama persis seperti sebelumnya. 

Sebaliknya, docker compose down -v akan menghapus named volume secara permanen bersama container dan network. Ini berguna saat ingin reset total environment (misalnya mengulangi dari awal atau membersihkan data test), namun harus digunakan dengan hati-hati di environment production karena data tidak bisa dipulihkan kecuali ada backup. 

4. Apa fungsi depends_on dan healthcheck di docker-compose.yml? depends_on mendefinisikan urutan pembuatan dan start container — Docker Compose akan memastikan service yang menjadi dependency di-start lebih dulu sebelum service yang bergantung padanya. Namun secara default depends_on hanya menunggu container berstatus running, bukan benar-benar siap menerima koneksi. 

Di sinilah healthcheck berperan: ia mendefinisikan perintah yang dijalankan secara berkala untuk mengecek apakah service benar-benar siap (misalnya pg_isready untuk PostgreSQL). Dengan kombinasi depends_on + condition: service_healthy, Compose akan menunggu sampai healthcheck container dependency lulus sebelum memulai container berikutnya. Tanpa kombinasi ini, Flask bisa mencoba konek ke PostgreSQL sebelum database selesai inisialisasi, mengakibatkan error koneksi di awal. 

5. Mengapa user-defined bridge bisa DNS resolve nama container, sedangkan default bridge tidak? 

Docker menyertakan embedded DNS server (berjalan di 127.0.0.11) yang hanya aktif untuk user-defined network. Setiap container yang bergabung ke user-defined bridge secara otomatis mendaftarkan nama container-nya ke DNS server tersebut. Ketika sebuah container melakukan query hostname nama-container, request diteruskan ke 127.0.0.11 dan dijawab dengan IP container yang dimaksud. 

Default bridge tidak menggunakan mekanisme ini karena alasan historis dan backward compatibility — ia menggunakan pendekatan lama yang hanya mengandalkan entri /etc/hosts yang ditulis secara statis saat container dibuat, sehingga tidak mendukung dynamic name resolution. Oleh karena itu, untuk semua aplikasi multi-container modern, selalu gunakan user-defined bridge agar service bisa saling memanggil cukup dengan nama container. 

## **Post-Lab** 

1. Jalankan docker network inspect lab-frontend. Sebutkan container dan IP masing-masing. 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0028-06.png)


2. Hapus container lab-db lalu docker compose up -d lagi. Apakah data PostgreSQL masih ada? Mengapa? 

Data PostgreSQL masih ada setelah container dihapus dan docker compose up -d dijalankan kembali. Alasannya adalah data PostgreSQL disimpan di named volume pg-data yang dipetakan ke /var/lib/postgresql/data di dalam container. Ketika container lab-db dihapus dengan docker rm atau docker compose down (tanpa -v), volume pg-data tidak ikut dihapus — ia tetap berada di host di bawah /var/lib/docker/volumes/. Saat container baru dibuat ulang, ia me-mount volume yang sama dan menemukan data yang sudah ada, sehingga PostgreSQL tidak menjalankan inisialisasi ulang dan semua data tabel tetap utuh. 

3. Tunjukkan perbedaan output docker inspect untuk mount type volume vs bind. Pada named volume, bagian "Mounts" di output docker inspect akan terlihat seperti: 

json { "Type": "volume", "Name": "pg-data", "Source": "/var/lib/docker/volumes/compose-app_pg-data/_data", "Destination": "/var/lib/postgresql/data", "Driver": "local", "Mode": "z", "RW": true } 

Pada bind mount, outputnya berbeda — field Type bernilai "bind", Name kosong, dan Source menunjuk langsung ke path absolut di filesystem host: json 

{ "Type": "bind", "Source": "/home/user/docker-lab/compose-app/html", "Destination": "/usr/share/nginx/html", "Mode": "ro", "RW": false } 

Perbedaan kunci: volume dikelola Docker (Source di bawah /var/lib/docker/), sedangkan bind mount Source-nya adalah path yang ditentukan pengguna di host. Flag RW: false pada bind mount menunjukkan mount bersifat read-only karena menggunakan opsi :ro. 

4. Jelaskan alur request dari browser → Nginx → Flask → PostgreSQL. Alur sederhana: 

      1. Browser → Nginx 

      2. Nginx → Flask 

      3. Flask → PostgreSQL 

      4. PostgreSQL → Flask 

      5. Flask → Nginx → Browser 

5. Bandingkan ukuran image yang digunakan stack ini. Mana terbesar dan mengapa? 

      - PostgreSQL → paling besar 

      - Flask → sedang 

   - Nginx → paling kecil 

   - Alasan: 

      - PostgreSQL banyak fitur & library 

      - Flask butuh runtime Python 

      - Nginx hanya web server ringan 


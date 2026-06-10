## **DOCKER-INTRO Docker dan Instalasi WORKSHOP ADMIN JARINGAN** 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0001-01.png)


NAMA: AQILA WISAM MADANI MA’MURRIE NRP : 3124600041 

KELAS : D4 IT B KELOMPOK : 2 TANGGAL PRAKTIKUM : 2 MEI 2026 DOSEN : Dr Ferry Astika Saputra ST, M.Sc 

## **POLITEKNIK ELEKTRONIKA NEGERI SURABAYA** 

**2025/2026** 

## **MODUL 1: Docker dan Instalasi LANGKAH PRAKTIKUM** 

## **Langkah 0: Persiapan Environment** 

Pastikan VM/Host sudah terkoneksi internet untuk download image dari Docker Hub. 

## # Cek koneksi internet 

ping -c 3 google.com 

## # Update package list 

sudo apt update && sudo apt upgrade -y Langkah 1: Instalasi Docker Engine di Ubuntu 22.04 1.1 Hapus versi lama (jika ada) # Hapus package Docker versi lama yang mungkin terinstal sudo apt remove -y docker docker-engine docker.io containerd runc 2>/dev/null 1.2 Instal dependensi sudo apt install -y \ ca-certificates \ curl \ gnupg \ lsb-release 1.3 Tambahkan Docker GPG key dan repository # Buat direktori keyrings sudo install -m 0755 -d /etc/apt/keyrings 

## # Download GPG key Docker 

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \ sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg 

sudo chmod a+r /etc/apt/keyrings/docker.gpg 

## # Tambahkan repository Docker 

echo \ 

"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \ https://download.docker.com/linux/ubuntu \ $(lsb_release -cs) stable" | \ sudo tee /etc/apt/sources.list.d/docker.list > /dev/null 1.4 Instal Docker Engine sudo apt update 

sudo apt install -y \ docker-ce \ docker-ce-cli \ containerd.io \ docker-buildx-plugin \ docker-compose-plugin 1.5 Konfigurasi user non-root # Tambahkan user saat ini ke group docker sudo usermod -aG docker $USER 

## # Aktifkan group baru (atau logout/login) 

newgrp docker 1.6 Verifikasi instalasi # Cek versi Docker 

docker version 

# Cek info Docker Engine docker info 

# Pastikan service berjalan sudo systemctl status docker 

# Test container pertama docker run hello-world Expected output docker run hello-world: 

Hello from Docker! This message shows that your installation appears to be working correctly. 

… 

## **Langkah 2: Instalasi Docker Desktop di Windows 10/11** 

Catatan: Langkah ini dilakukan di laptop/PC Windows sebagai referensi. Praktikum utama menggunakan Ubuntu. 

2.1 Aktifkan WSL2 

Buka PowerShell sebagai Administrator: 

## # Aktifkan WSL 

wsl --install 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0003-12.png)


# Set WSL2 sebagai default 

wsl --set-default-version 2 

## # Verifikasi 

wsl --list --verbose 

Restart komputer jika diminta. 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0003-18.png)


2.2 Download dan Instal Docker Desktop Buka https://www.docker.com/products/docker-desktop/ Download Docker Desktop for Windows Jalankan installer, centang "Use WSL 2 instead of Hyper-V" 

Klik Install → tunggu selesai → Restart jika diminta 

## 2.3 Konfigurasi Docker Desktop 

Buka Docker Desktop dari Start Menu 

Masuk ke Settings → General → pastikan "Use the WSL 2 based engine" aktif Masuk ke Settings → Resources → WSL Integration → aktifkan distro yang diinginkan 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0004-04.png)


## 2.4 Verifikasi dari PowerShell / WSL Terminal 

# Dari PowerShell 

docker version 

docker run hello-world 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0004-09.png)


# Dari WSL terminal (Ubuntu) 

docker version 

docker run hello-world 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0005-03.png)


## **Langkah 3: Operasi Dasar Docker Image** 

3.1 Pull image dari Docker Hub 

# Pull image nginx versi latest docker pull nginx 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0005-07.png)


# Pull image nginx versi spesifik docker pull nginx:1.26 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0005-09.png)


# Pull image Ubuntu 22.04 

docker pull ubuntu:22.04 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0005-12.png)


# Pull image Alpine (sangat ringan, ~7MB) 

docker pull alpine:3.20 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0006-02.png)


## 3.2 Manajemen image 

# List semua image lokal 

docker images 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0006-06.png)


## # List dengan format custom 

docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}" 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0006-09.png)


## # Inspect detail image 

docker image inspect nginx 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0006-12.png)


# Lihat history layer sebuah image 

docker image history nginx 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0006-15.png)


## # Hapus image 

docker rmi alpine:3.20 

## # Hapus semua image yang tidak digunakan 

docker image prune -a 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0007-04.png)


## **Langkah 4: Menjalankan dan Mengelola Container** 

4.1 Menjalankan container dasar 

# Jalankan container nginx (foreground — tekan Ctrl+C untuk stop) docker run nginx 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0007-08.png)


# Jalankan nginx di background (detached mode) 

docker run -d --name web-server nginx 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0007-11.png)


# Jalankan nginx dengan port mapping (host:container) docker run -d --name web-public -p 8080:80 nginx 

Buka browser ke http://localhost:8080 → akan tampil halaman default Nginx. 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0007-14.png)


## 4.2 Container interaktif 

# Jalankan Ubuntu container dengan bash interaktif 

docker run -it --name ubuntu-test ubuntu:22.04 /bin/bash 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0008-03.png)


## # Di dalam container: 

cat /etc/os-release 

apt update && apt install -y curl 

curl http://web-server    # akses container lain (jika di network sama) exit                       # keluar (container akan stop) 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0008-08.png)


## 4.3 Monitoring container 

# List container yang sedang berjalan docker ps 

## # List SEMUA container (termasuk stopped) 

docker ps -a 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0008-13.png)


## # Lihat log container 

docker logs web-server 

docker logs -f web-server    # follow (real-time) 

docker logs --tail 20 web-server  # 20 baris terakhir 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0008-18.png)


## # Lihat resource usage (CPU, Memory, I/O) 

docker stats 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0009-02.png)


# Lihat detail container 

docker inspect web-server 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0009-05.png)


4.4 Interaksi dengan container berjalan 

# Eksekusi perintah di container yang sedang berjalan docker exec web-server cat /etc/nginx/nginx.conf 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0009-08.png)


# Masuk ke shell container yang sedang berjalan docker exec -it web-server /bin/bash 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0009-10.png)


# Copy file dari host ke container docker cp index.html web-server:/usr/share/nginx/html/index.html 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0009-12.png)


# Copy file dari container ke host docker cp web-server:/etc/nginx/nginx.conf ./nginx.conf 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0010-01.png)


4.5 Lifecycle management # Stop container docker stop web-server 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0010-03.png)


# Start container yang sudah di-stop docker start web-server 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0010-05.png)


# Restart container docker restart web-server 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0010-07.png)


# Kill container (force stop — SIGKILL) docker kill web-server 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0010-09.png)


# Hapus container (harus dalam keadaan stopped) docker stop web-server && docker rm web-server 

# Hapus container secara paksa (meskipun masih running) docker rm -f web-server 

# Hapus SEMUA stopped container docker container prune 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0010-13.png)


## **Langkah 5: Membuat Custom Image dengan Dockerfile** 

5.1 Buat project directory mkdir -p ~/docker-lab/custom-web && cd ~/docker-lab/custom-web 5.2 Buat halaman web 

cat > index.html << 'EOF' <!DOCTYPE html> <html lang="id"> <head> <meta charset="UTF-8"> <title>Docker Lab - PENS</title> <style> body { font-family: Arial, sans-serif; text-align: center; padding: 50px; 

background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); color: white; } .container { background: rgba(255,255,255,0.1); border-radius: 15px; padding: 40px; max-width: 600px; margin: 0 auto; } h1 { font-size: 2.5em; } .info { background: rgba(0,0,0,0.2); padding: 15px; border-radius: 8px; margin-top: 20px; text-align: left; } </style> </head> <body> <div class="container"> <h1>🐳 Docker Lab PENS</h1> <p>Container berhasil berjalan!</p> <div class="info"> <p><strong>Hostname:</strong> <span id="host"></span></p> <p><strong>Server:</strong> Nginx on Docker</p> <p><strong>Praktikum:</strong> Modul 1 — Instalasi Docker</p> </div> </div> <script> document.getElementById('host').textContent = location.hostname; </script> </body> </html> EOF 

5.3 Buat Dockerfile cat > Dockerfile << 'EOF' # Gunakan base image nginx versi stabil FROM nginx:1.26-alpine 

# Metadata LABEL maintainer="admin@pens.ac.id" LABEL description="Custom Nginx untuk praktikum Docker PENS" LABEL version="1.0" 

# Hapus halaman default dan ganti dengan halaman custom RUN rm -rf /usr/share/nginx/html/* COPY index.html /usr/share/nginx/html/index.html 

# Expose port 80 EXPOSE 80 

# Command default (inherited dari base image, tapi kita tulis eksplisit) CMD ["nginx", "-g", "daemon off;"] EOF 

5.4 Build dan jalankan # Build image (titik di akhir = build context = direktori saat ini) docker build -t pens-web:1.0 . 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0012-00.png)


## # Verifikasi image berhasil dibuat 

docker images | grep pens-web 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0012-03.png)


# Jalankan container dari image custom 

docker run -d --name pens-app -p 9090:80 pens-web:1.0 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0012-06.png)


# Test 

curl http://localhost:9090 Buka browser ke http://localhost:9090. 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0012-09.png)


## 5.5 Lihat layer image 

# Bandingkan layer antara base image dan custom image docker image history nginx:1.26-alpine docker image history pens-web:1.0 

**Langkah 6: Docker System Cleanup** 

# Lihat disk usage Docker docker system df 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0012-14.png)


# Lihat detail disk usage 

docker system df -v 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0013-02.png)


# Hapus semua resource yang tidak digunakan (container, image, network, cache) docker system prune -a 


![](WAJ/gambar/3124600041_AQILA_WISAM_MADANI_MA'MURRI_WAJ_DOCKER_MODUL_1-6_-1-.pdf-0013-04.png)


# Konfirmasi dengan -f (force, tanpa prompt) docker system prune -a -f 

## **PERTANYAAN** 

## **Pre-Lab (jawab sebelum praktikum)** 

1. Sebutkan minimal 3 perbedaan antara Virtual Machine dan Container. 

   - a. Arsitektur 

   - Virtual Machine (VM): menjalankan OS lengkap (guest OS) di atas hypervisor 

   - Container: berbagi kernel OS host (tidak punya OS sendiri) 

   - b. Resource & ukuran 

   - VM: berat (GB), butuh RAM & storage besar 

   - Container: ringan (MB), efisien resource 

   - c. Waktu startup 

   - VM: lama (detik–menit) 

   - Container: sangat cepat (milidetik–detik) 

   - d. Isolasi 

   - VM: isolasi penuh (lebih secure) 

   - Container: isolasi level proses (lebih ringan tapi sedikit kurang ketat) 

2. Apa fungsi dari containerd dan runc dalam arsitektur Docker? containerd 

   - Mengelola lifecycle container (create, start, stop) 

   - Mengatur image, storage, dan networking 

   - Bertindak sebagai “middle layer” antara Docker dan runtime 

runc 

- Runtime low-level yang benar-benar menjalankan container 

- Menggunakan fitur Linux seperti: 

   - namespaces (isolasi) 

   - cgroups (limit resource) 

3. Mengapa Docker membutuhkan kernel Linux? Bagaimana Docker Desktop di Windows mengatasi hal ini? 

Karena Docker menggunakan fitur Linux: 

- Namespaces → isolasi proses 

- cgroups → manajemen resource 

- Union filesystem → layered image 

Docker Desktop mengatasinya dengan: 

   - Menjalankan Linux kernel di dalam VM ringan (WSL2) ● Jadi:Windows → WSL2 → Linux kernel → Docker 

4. Apa keuntungan layered filesystem pada Docker Image? 

   - Efisiensi storage → layer yang sama tidak diduplikasi 

   - Caching build cepat → tidak build ulang dari awal 

   - Reusability → bisa pakai base image yang sama 

   - Distribusi cepat → hanya layer baru yang dikirim 

5. Jelaskan perbedaan antara docker run dan docker exec. docker run 

   - Membuat + menjalankan container baru 

   - Digunakan pertama kali 

   - Contoh: docker run -d nginx 

docker exec 

- Menjalankan perintah di container yang sudah berjalan 

- Tidak membuat container baru 

- Contoh: docker exec -it web-server bash 

## **Post-Lab (jawab setelah praktikum)** 

1. Bandingkan output docker image history nginx dengan docker image history pens-web:1.0. Layer mana saja yang di-share? 

Saat menjalankan docker image history, image pens-web:1.0 memiliki layer tambahan di bagian atas, sedangkan layer di bawahnya sama dengan nginx:1.26-alpine sebagai base image. 

Layer yang di-share adalah seluruh layer dari base image, seperti: 

- OS Alpine 

- Instalasi Nginx 

- Konfigurasi default 

Sedangkan layer yang berbeda hanya: 

- RUN rm -rf /usr/share/nginx/html/* 

- COPY index.html ... 

Karena layer bersifat read-only dan dapat digunakan bersama, Docker tidak 

menduplikasi data. Akibatnya, ukuran pens-web:1.0 hanya sedikit lebih besar dari nginx. 

2. Apa yang terjadi pada data di dalam container setelah container dihapus dengan docker rm? Bagaimana solusinya? 

Data yang disimpan di dalam container akan hilang permanen setelah container dihapus dengan docker rm. Hal ini karena data berada di writable layer yang bersifat sementara (ephemeral). 

Solusi agar data tetap tersimpan: 

   - Docker Volume = data disimpan terpisah dari container 

   - Bind Mount = data disimpan langsung di filesystem host 

3. Jelaskan perbedaan antara EXPOSE di Dockerfile dan flag -p pada docker run. Apakah EXPOSE cukup untuk membuat port dapat diakses dari host? 

   - EXPOSE (di Dockerfile) 

Hanya sebagai informasi bahwa container menggunakan port tertentu. Tidak membuka akses ke luar. 

- -p (saat docker run) Digunakan untuk membuka port ke host, sehingga bisa diakses dari luar (browser, dll). 

## Kesimpulan: 

EXPOSE saja tidak cukup, harus menggunakan -p agar port bisa diakses. 

4. Mengapa menggunakan tag spesifik (misal nginx:1.26) lebih baik daripada nginx:latest untuk production? 

   - **l** atest 

Selalu mengambil versi terbaru → berisiko berubah tanpa disadari 

- Tag spesifik (contoh: nginx:1.26) 

Versi tetap = lebih stabil dan konsisten 

Keuntungan tag spesifik: 

   - Reproducible (hasil selalu sama) 

   - Mudah rollback 

   - Lebih aman untuk production 

5. Berapa ukuran image alpine:3.20 dibanding ubuntu:22.04? Apa trade-off menggunakan Alpine? 

   - **Alpine** : ± 7–8 MB 

   - **Ubuntu** : ± 70–80 MB 

Keuntungan Alpine: 

- Ukuran kecil 

- Lebih ringan dan cepat 

Kekurangan Alpine: 

- Menggunakan _musl libc_ (bukan glibc) → beberapa aplikasi tidak kompatibel 

- Tools terbatas 

- Lebih sulit untuk debugging 

Kesimpulan: 

- Alpine cocok untuk production (ringan & efisien) 

- Ubuntu lebih cocok untuk development (lebih lengkap & mudah digunakan) 


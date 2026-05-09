OSI -> konsep dari network stack
TPC/IP -> implementasi

|Client:|-|Server:|
|apps|<->||
|transport|<TCP,UDP>||
|internet|<IP>||
|net acces|<ARP>||

Net Accees:
-datalink -> framing, physical addres (MAC)
-Physical -> media:
                    cable - tegangan
                    RF- radio frekuensi
                    optic- fiber optik

Routing = mencari rute terrbaik
Logical addressing = IP address

Metode pengiriman data   - Reliable TCP
                         - unreliable UDP

Broadcast = mengirim informasi ke seluruh device

|net id|-|host id|
|8|-|24|
|16|-|16|
|24|-|8|

X1 - X2 | X3 - X4
1st octect - 2nd octate

internet range
A 0-127
B 128-191
C 192-223
D -> multicast
E -> reserved

Netmask = net id yang dimask "1"

jumlah host = 2^hostbit - 2


# RESUME TCP 

---

# 1. Pengertian TCP

TCP (Transmission Control Protocol) adalah protokol pada **Transport Layer** dalam model TCP/IP yang bersifat:

- Reliable (andal)
- Connection-oriented
- Ordered delivery
- Error detection
- Flow control
- Congestion control

TCP digunakan pada layanan seperti:
HTTP, HTTPS, FTP, SMTP, SSH, dll.

---

# 2. Socket

Socket adalah kombinasi:

IP Address : Port

Contoh:
192.168.1.10:80

Artinya:
- IP → alamat perangkat
- Port → layanan/aplikasi pada perangkat

Satu koneksi TCP unik ditentukan oleh:

Source IP + Source Port  
Destination IP + Destination Port  

Inilah yang disebut sebagai **endpoint komunikasi**.

---

# 3. Port Number

Port dibagi menjadi 3 kategori:

| Jenis Port | Range | Keterangan |
|------------|--------|------------|
| Well-known | 1 – 1023 | HTTP (80), HTTPS (443), FTP (21) |
| Registered | 1024 – 49151 | Aplikasi tertentu |
| Dynamic / Ephemeral | 49152 – 65535 | Digunakan client sementara |

---

# 4. TCP Connection Establishment  
(3-Way Handshake)

Sebelum mengirim data, TCP harus membuat koneksi.

Tahapan:
1. SYN
2. SYN + ACK
3. ACK

---

## Visualisasi 3-Way Handshake

Client                              Server  
  |                                    |  
  | ----------- SYN ----------------->  |  
  |                                    |  
  | <------ SYN + ACK ----------------  |  
  |                                    |  
  | ----------- ACK ----------------->  |  
  |                                    |  
  |        Connection Established       |  

Penjelasan:
- SYN = meminta koneksi
- SYN+ACK = server menyetujui
- ACK = konfirmasi akhir
- Setelah ini koneksi aktif

---

# 5. Proses Transfer Data

Setelah koneksi terbentuk, data dikirim menggunakan:

- PUSH → mengirim data
- ACK → konfirmasi penerimaan

Jika tidak ada ACK, maka data dikirim ulang (retransmission).

---

## Visualisasi Data Transfer

Client                              Server  
  |                                    |  
  | ----------- PUSH ---------------->  |  
  |                                    |  
  | <------------- ACK ---------------  |  
  |                                    |  
  | <----------- PUSH ----------------- |  
  |                                    |  
  | ------------ ACK ---------------->  |  
  |                                    |  

Penjelasan:
- TCP menjamin data sampai
- Data diterima sesuai urutan
- Jika paket hilang → dikirim ulang

---

# 6. TCP Connection Termination  
(4-Way Handshake)

Untuk menutup koneksi digunakan proses:

1. FIN
2. ACK
3. FIN
4. ACK

---

## Visualisasi Penutupan Koneksi

Client                              Server  
  |                                    |  
  | ----------- FIN ----------------->  |  
  |                                    |  
  | <------------- ACK ---------------  |  
  |                                    |  
  | <------------- FIN ---------------  |  
  |                                    |  
  | ----------- ACK ----------------->  |  
  |                                    |  
  |           Connection Closed         |  

Keterangan:
- FIN = selesai
- ACK = konfirmasi
- Setelah tahap terakhir → koneksi tertutup

---

# 7. Alur Lengkap TCP

1️⃣ Establishment  
SYN → SYN+ACK → ACK  

2️⃣ Data Transfer  
PUSH → ACK  

3️⃣ Termination  
FIN → ACK → FIN → ACK  

---

# 8. Karakteristik Penting TCP

✔ Reliable (ada ACK & retransmission)  
✔ Connection-oriented  
✔ Urutan data terjaga  
✔ Flow control (Sliding Window)  
✔ Congestion control  

---

# 9. Kesimpulan

- TCP menggunakan 3-way handshake untuk membangun koneksi.
- Data dikirim menggunakan mekanisme PUSH dan dikonfirmasi dengan ACK.
- Koneksi ditutup dengan 4-way handshake.
- TCP cocok untuk komunikasi yang membutuhkan keakuratan dan keandalan tinggi.

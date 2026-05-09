# Resume Jaringan Komputer

## 1. Model OSI dan TCP/IP

### OSI (Open Systems Interconnection)
OSI adalah **model konseptual** yang menjelaskan bagaimana komunikasi jaringan dibagi menjadi beberapa layer (network stack).  
Model ini digunakan sebagai referensi teoritis dalam memahami komunikasi jaringan.

Terdiri dari 7 layer:
1. Application
2. Presentation
3. Session
4. Transport
5. Network
6. Data Link
7. Physical

---

### TCP/IP
TCP/IP adalah **implementasi nyata** dari konsep network stack yang digunakan di internet.

Model TCP/IP terdiri dari 4 layer:

| Client | Protokol | Server |
|--------|----------|--------|
| Application | HTTP, FTP, SMTP, dll | Application |
| Transport | TCP, UDP | Transport |
| Internet | IP | Internet |
| Network Access | ARP, Ethernet | Network Access |

---

## 2. Network Access Layer

Layer ini terdiri dari:

### a. Data Link
- Framing
- Physical Addressing (MAC Address)
- Error detection
- Flow control

### b. Physical
Media transmisi yang digunakan:
- **Cable** → sinyal listrik (tegangan)
- **RF (Radio Frequency)** → gelombang radio (WiFi)
- **Optic** → fiber optik (cahaya)

---

## 3. Konsep Penting dalam Jaringan

### Routing
Proses mencari **rute terbaik** untuk mengirimkan paket dari sumber ke tujuan.

### Logical Addressing
Menggunakan **IP Address** sebagai alamat logis dalam jaringan.

---

## 4. Metode Pengiriman Data

### a. Reliable
- Menggunakan **TCP**
- Ada proses handshake
- Ada konfirmasi (ACK)
- Data terjamin sampai

### b. Unreliable
- Menggunakan **UDP**
- Tidak ada konfirmasi
- Lebih cepat
- Tidak menjamin data sampai

---

## 5. Broadcast

Broadcast adalah metode pengiriman data ke **seluruh device dalam satu jaringan**.

---

## 6. Struktur IP Address

IP Address terdiri dari:

| Net ID | Host ID |
|--------|---------|
| 8 bit  | 24 bit  |
| 16 bit | 16 bit  |
| 24 bit | 8 bit   |

Penjelasan:
- **Net ID** → Identitas jaringan
- **Host ID** → Identitas perangkat dalam jaringan
 
Contoh pembagian: X1 - X2 | X3 - X4

- 1st octet
- 2nd octet
- 3rd octet
- 4th octet

---

## 7. Kelas IP Address (Classful Addressing)

| Kelas | Range | Keterangan |
|-------|-------|------------|
| A | 0 - 127 | Jaringan besar |
| B | 128 - 191 | Jaringan menengah |
| C | 192 - 223 | Jaringan kecil |
| D | 224 - 239 | Multicast |
| E | 240 - 255 | Reserved |

---

## 8. Netmask

Netmask adalah representasi biner untuk menentukan bagian:
- Net ID (bit 1)
- Host ID (bit 0)

Contoh: 255.255.255.0

Artinya 24 bit pertama adalah Net ID.

---

## 9. Perhitungan Jumlah Host

Rumus: Jumlah Host = 2^n - 2

Keterangan:
- n = jumlah bit host
- Dikurangi 2 karena:
  - 1 untuk Network Address
  - 1 untuk Broadcast Address

Contoh:
Jika host bit = 8

2^8 - 2 = 254 host

---

# Kesimpulan

- OSI adalah model konseptual.
- TCP/IP adalah implementasi nyata.
- TCP bersifat reliable, UDP tidak.
- IP Address terdiri dari Net ID dan Host ID.
- Netmask menentukan pembagian jaringan.
- Jumlah host dihitung dengan rumus 2^hostbit - 2.
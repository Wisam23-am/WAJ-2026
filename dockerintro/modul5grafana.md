# MODUL 5: Centralized Logging Docker dengan Fluent Bit, Loki, dan Grafana

**Topik:** Centralized Logging Modern menggunakan Fluent Bit, Loki, dan Grafana
**Durasi:** 120 menit
**Prasyarat:** Modul Docker Compose dan Container dasar selesai

---

# 1. TUJUAN PEMBELAJARAN

Setelah praktikum ini, mahasiswa mampu:

1. Memahami konsep centralized logging pada container
2. Menggunakan Docker logging driver `fluentd`
3. Mengkonfigurasi Fluent Bit sebagai log collector
4. Menggunakan Loki sebagai centralized log storage
5. Menggunakan Grafana untuk visualisasi log
6. Menghasilkan structured logging menggunakan JSON
7. Melakukan observability dan monitoring log secara real-time
8. Mengorkestrasi seluruh stack logging menggunakan Docker Compose

---

# 2. DASAR TEORI

## 2.1 Centralized Logging

Pada lingkungan container dan microservices, setiap container menghasilkan log masing-masing. Jika log hanya disimpan di dalam container:

* sulit melakukan debugging
* sulit melakukan monitoring
* log hilang ketika container dihapus
* sulit melakukan analisis sistem secara keseluruhan

Karena itu digunakan **Centralized Logging**, yaitu seluruh log dikumpulkan ke satu sistem terpusat.

---

## 2.2 Arsitektur Logging Modern

```text
┌───────────────┐
│ Docker Apps   │
│ nginx/flask   │
└───────┬───────┘
        │
        │ fluentd logging driver
        ▼
┌───────────────┐
│ Fluent Bit    │
│ Log Collector │
└───────┬───────┘
        │
        ▼
┌───────────────┐
│ Loki          │
│ Log Storage   │
└───────┬───────┘
        ▼
┌───────────────┐
│ Grafana       │
│ Visualization │
└───────────────┘
```

---

## 2.3 Komponen Sistem

| Komponen              | Fungsi                        |
| --------------------- | ----------------------------- |
| Docker Logging Driver | Mengirim log container        |
| Fluent Bit            | Mengumpulkan dan forward log  |
| Loki                  | Menyimpan log terpusat        |
| Grafana               | Visualisasi dan pencarian log |
| Flask App             | Simulasi aplikasi             |
| Log Generator         | Simulasi log production       |

---

## 2.4 Mengapa Menggunakan Loki?

Loki dirancang khusus untuk logging modern:

* ringan
* cepat
* terintegrasi langsung dengan Grafana
* schema-less
* cocok untuk container dan Kubernetes
* lebih stabil dibanding menyimpan log langsung ke relational database

---

# 3. TOPOLOGI LAB

```text
+-------------------+
| nginx-web         |
| flask-app         |
| log-generator     |
+---------+---------+
          |
          | fluentd logging driver
          v
+-------------------+
| Fluent Bit        |
| Port 24224        |
+---------+---------+
          |
          v
+-------------------+
| Loki              |
| Port 3100         |
+---------+---------+
          |
          v
+-------------------+
| Grafana           |
| Port 3000         |
+-------------------+
```

---

# 4. LANGKAH PRAKTIKUM

## Langkah 0 — Persiapan Project

```bash
mkdir -p ~/docker-lab/logging/{fluent-bit,loki,app,generator}
cd ~/docker-lab/logging
```

---

# Langkah 1 — Konfigurasi Fluent Bit

## 1.1 Buat file konfigurasi Fluent Bit

```bash
nano fluent-bit/fluent-bit.conf
```

Isi:

```ini
[SERVICE]
    Flush        2
    Daemon       Off
    Log_Level    info

[INPUT]
    Name         forward
    Listen       0.0.0.0
    Port         24224

[OUTPUT]
    Name         loki
    Match        *
    Host         loki
    Port         3100
    Labels       job=fluentbit
    Line_Format  json

[OUTPUT]
    Name         stdout
    Match        *
```

Simpan:

* CTRL + O
* Enter
* CTRL + X

---

# Langkah 2 — Konfigurasi Loki

## 2.1 Buat file konfigurasi Loki

```bash
nano loki/loki-config.yml
```

Isi:

```yaml
auth_enabled: false

server:
  http_listen_port: 3100

common:
  path_prefix: /loki
  storage:
    filesystem:
      chunks_directory: /loki/chunks
      rules_directory: /loki/rules
  replication_factor: 1
  ring:
    kvstore:
      store: inmemory

schema_config:
  configs:
    - from: 2025-01-01
      store: tsdb
      object_store: filesystem
      schema: v13
      index:
        prefix: index_
        period: 24h

ruler:
  alertmanager_url: http://localhost:9093
```

Simpan file.

---

# Langkah 3 — Membuat Flask App

## 3.1 Buat requirements

```bash
nano app/requirements.txt
```

Isi:

```text
flask==3.1.*
```

---

## 3.2 Buat aplikasi Flask

```bash
nano app/app.py
```

Isi:

```python
import json
import socket
import datetime
import logging
import sys
from flask import Flask, jsonify, request

app = Flask(__name__)

class JSONFormatter(logging.Formatter):
    def format(self, record):
        return json.dumps({
            "timestamp": datetime.datetime.now().isoformat(),
            "level": record.levelname,
            "hostname": socket.gethostname(),
            "service": "flask-app",
            "message": record.getMessage()
        })

handler = logging.StreamHandler(sys.stdout)
handler.setFormatter(JSONFormatter())
app.logger.handlers = [handler]
app.logger.setLevel(logging.INFO)

@app.route("/")
def index():
    app.logger.info(f"Request from {request.remote_addr}")
    return jsonify({"status": "running"})

@app.route("/health")
def health():
    app.logger.info("Health check accessed")
    return jsonify({"health": "ok"})

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

---

## 3.3 Buat Dockerfile Flask

```bash
nano app/Dockerfile
```

Isi:

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

EXPOSE 5000

CMD ["python", "-u", "app.py"]
```

---

# Langkah 4 — Membuat Log Generator

## 4.1 Buat generator.py

```bash
nano generator/generator.py
```

Isi:

```python
import json
import time
import random
import datetime

levels = ["INFO", "WARN", "ERROR", "CRITICAL"]
messages = [
    "User login successful",
    "Database timeout",
    "Disk usage warning",
    "API request completed",
    "Memory usage critical",
    "Health check passed"
]

while True:
    log = {
        "timestamp": datetime.datetime.now().isoformat(),
        "level": random.choice(levels),
        "service": "log-generator",
        "message": random.choice(messages)
    }

    print(json.dumps(log), flush=True)
    time.sleep(2)
```

---

## 4.2 Buat Dockerfile Generator

```bash
nano generator/Dockerfile
```

Isi:

```dockerfile
FROM python:3.11-alpine

WORKDIR /app

COPY generator.py .

CMD ["python", "-u", "generator.py"]
```

---

# Langkah 5 — Docker Compose

## 5.1 Buat docker-compose.yml

```bash
nano docker-compose.yml
```

Isi:

```yaml
services:

  fluent-bit:
    image: fluent/fluent-bit:latest
    container_name: fluent-bit
    ports:
      - "24224:24224"
    volumes:
      - ./fluent-bit/fluent-bit.conf:/fluent-bit/etc/fluent-bit.conf:ro
    depends_on:
      - loki
    networks:
      - logging-net
    restart: unless-stopped

  loki:
    image: grafana/loki:latest
    container_name: loki
    ports:
      - "3100:3100"
    command: -config.file=/etc/loki/local-config.yaml
    volumes:
      - ./loki/loki-config.yml:/etc/loki/local-config.yaml
    networks:
      - logging-net
    restart: unless-stopped

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=admin123
    depends_on:
      - loki
    networks:
      - logging-net
    restart: unless-stopped

  nginx-web:
    image: nginx:alpine
    container_name: nginx-web
    ports:
      - "8080:80"
    logging:
      driver: fluentd
      options:
        fluentd-address: "localhost:24224"
        tag: "docker.nginx"
    networks:
      - logging-net
    depends_on:
      - fluent-bit
    restart: unless-stopped

  flask-app:
    build: ./app
    container_name: flask-app
    ports:
      - "5000:5000"
    logging:
      driver: fluentd
      options:
        fluentd-address: "localhost:24224"
        tag: "docker.flask"
    networks:
      - logging-net
    depends_on:
      - fluent-bit
    restart: unless-stopped

  log-generator:
    build: ./generator
    container_name: log-generator
    logging:
      driver: fluentd
      options:
        fluentd-address: "localhost:24224"
        tag: "docker.generator"
    networks:
      - logging-net
    depends_on:
      - fluent-bit
    restart: unless-stopped

networks:
  logging-net:
```

---

# Langkah 6 — Deploy Stack

## 6.1 Jalankan Docker Compose

```bash
docker compose up --build -d
```

---

## 6.2 Cek Status Container

```bash
docker compose ps
```

Pastikan semua container status:

```text
Up
```

---

# Langkah 7 — Generate Traffic

## 7.1 Generate log dari nginx

```bash
for i in $(seq 1 20); do
  curl -s http://localhost:8080 > /dev/null
done
```

---

## 7.2 Generate log dari Flask

```bash
for i in $(seq 1 10); do
  curl -s http://localhost:5000/ > /dev/null
done
```

---

# Langkah 8 — Verifikasi Fluent Bit

## 8.1 Cek log Fluent Bit

```bash
docker compose logs fluent-bit --tail=50
```

Pastikan terlihat JSON log masuk.

---

# Langkah 9 — Akses Grafana

Buka browser:

```text
http://localhost:3000
```

Login:

| Username | Password |
| -------- | -------- |
| admin    | admin123 |

---

# Langkah 10 — Tambahkan Loki Data Source

## 10.1 Masuk menu:

```text
Connections → Data Sources
```

---

## 10.2 Tambahkan Loki

Klik:

```text
Add data source
```

Pilih:

```text
Loki
```

---

## 10.3 Isi URL

```text
http://loki:3100
```

Klik:

```text
Save & Test
```

Jika berhasil muncul:

```text
Data source connected successfully
```

---

# Langkah 11 — Explore Log di Grafana

Masuk menu:

```text
Explore
```

---

## Query Log

```logql
{job="fluentbit"}
```

Maka log dari:

* nginx
* flask-app
* log-generator

akan tampil secara real-time.

---

# 5. PERTANYAAN PRE-LAB

1. Apa itu centralized logging?
2. Mengapa logging penting pada container?
3. Apa fungsi Fluent Bit?
4. Apa fungsi Loki?
5. Mengapa Grafana digunakan?

---

# 6. PERTANYAAN POST-LAB

1. Jelaskan alur log dari container hingga Grafana.
2. Apa fungsi Docker logging driver `fluentd`?
3. Apa keuntungan structured logging JSON?
4. Apa yang terjadi jika Fluent Bit dimatikan?
5. Mengapa Loki lebih cocok untuk logging dibanding relational database?

---

# 7. CHECKLIST PRAKTIKUM

* [ ] fluent-bit running
* [ ] loki running
* [ ] grafana running
* [ ] flask-app running
* [ ] nginx-web running
* [ ] log-generator running
* [ ] Fluent Bit menerima log
* [ ] Loki connected ke Grafana
* [ ] Log tampil di Explore Grafana
* [ ] Structured JSON log berhasil tampil

---

# 8. TROUBLESHOOTING

| Masalah                         | Penyebab             | Solusi                                       |
| ------------------------------- | -------------------- | -------------------------------------------- |
| Fluent Bit tidak start          | Config salah         | Cek fluent-bit.conf                          |
| Grafana tidak bisa connect Loki | URL salah            | Gunakan [http://loki:3100](http://loki:3100) |
| Tidak ada log di Grafana        | Logging driver salah | Pastikan driver fluentd                      |
| Container restart terus         | Port bentrok         | Cek port dengan docker ps                    |
| Loki error                      | Config Loki salah    | Cek loki-config.yml                          |

---

# 9. FORMAT LAPORAN

## Screenshot Wajib

1. docker compose ps
2. docker compose logs fluent-bit
3. Browser Grafana login
4. Loki Data Source connected
5. Explore Grafana menampilkan log
6. Log nginx tampil
7. Log Flask tampil
8. Log generator tampil
9. Query LogQL berhasil
10. Semua container running

---

# 10. KESIMPULAN

Pada praktikum ini mahasiswa berhasil:

* membangun centralized logging modern
* menggunakan Fluent Bit sebagai collector
* menggunakan Loki sebagai storage
* menggunakan Grafana untuk visualisasi
* melakukan observability log container secara real-time

---

# 11. REFERENSI

1. [https://docs.fluentbit.io/](https://docs.fluentbit.io/)
2. [https://grafana.com/oss/loki/](https://grafana.com/oss/loki/)
3. [https://grafana.com/grafana/](https://grafana.com/grafana/)
4. [https://docs.docker.com/config/containers/logging/](https://docs.docker.com/config/containers/logging/)
5. [https://grafana.com/docs/loki/latest/](https://grafana.com/docs/loki/latest/)

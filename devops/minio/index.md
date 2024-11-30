---
title: 'Belajar Cara Install MinIO'
date: '2023-08-05 15:00:00'
---

Kali ini kita akan mempelajari cara install `MinIO` menggunakan `docker`, MinIo ini berperan mirip seperti `AWS S3`.

## Syarat
- sudah terinstall docker dan docker-compose. [referensi cara install](https://docs.docker.com/engine/install/)

## Mulai
1. pertama kita buat file `docker-compose.yml` masukan code seperti dibawah ini:

```YAML
version: "3.6"

services:
  ### MINIO Server #########################################
  minio:
    container_name: minio
    restart: unless-stopped
    image: minio/minio:RELEASE.2021-01-08T21-18-21Z
    volumes:
      - ./data:/data
    expose:
      - "9000"
    ports:
      - "9000:9000"
    environment:
      MINIO_REGION_NAME: us-east-1
      MINIO_ROOT_USER: admin
      MINIO_ROOT_PASSWORD: secret
    command: server /data
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]
      interval: 30s
      timeout: 20s
      retries: 3
```

2. jalankan perintah `docker compose up -d` atau `docker-compose up -d`
3. lalu minio sudah bisa diakses pada url: http://localhost:9000
4. username: admin, password: secret
8. selesai
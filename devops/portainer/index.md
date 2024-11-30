---
title: 'Belajar Portainer'
date: '2023-08-05 16:30:00'
---

Kali ini kita akan menjalankan `portainer` menggunakan `docker`.

## Syarat
- sudah terinstall docker dan docker-compose. [referensi cara install](https://docs.docker.com/engine/install/)

## Mulai
1. pertama kita buat file `docker-compose.yml` masukan code seperti dibawah ini:

```YAML
version: "3"
services:
  portainer:
    image: portainer/portainer-ce:latest
    container_name: portainer
    restart: unless-stopped
    volumes:
      - ./portainer-data:/data
      - /var/run/docker.sock:/var/run/docker.sock
    ports:
      - 3900:9000
      - 3910:8000
    extra_hosts:
      - "raw.githubusercontent.com:185.199.111.133"
```

2. jalankan perintah `docker compose up -d` atau `docker-compose up -d`
3. lalu nginx proxy manager sudah bisa diakses pada url: http://localhost:3900
4. selesai

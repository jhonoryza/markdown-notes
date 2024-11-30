---
title: 'Belajar Nginx Proxy Manager'
date: '2023-08-05 13:00:00'
---

Kali ini kita akan menjalankan `nginx proxy manager` menggunakan `docker`.

## Syarat
- sudah terinstall docker dan docker-compose. [referensi cara install](https://docs.docker.com/engine/install/)

## Mulai
1. pertama kita buat file `docker-compose.yml` masukan code seperti dibawah ini:

```YAML
version: '3'

services:
  nginx:
    image: 'jc21/nginx-proxy-manager:latest'
    container_name: nginx
    restart: unless-stopped
    ports:
      # These ports are in format <host-port>:<container-port>
      - '80:80' # Public HTTP Port
      - '443:443' # Public HTTPS Port
      - '81:81' # Admin Web Port
    volumes:
      - ./nginx-data:/data
      - ./letsencrypt:/etc/letsencrypt
    networks:
      - nginx-network

networks:
  nginx-network:
```

2. jalankan perintah `docker compose up -d` atau `docker-compose up -d`
3. lalu nginx proxy manager sudah bisa diakses pada url: http://localhost:81
4. selesai


Default Admin User:

```YAML
Email:    admin@example.com
Password: changeme
```

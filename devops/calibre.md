---
title: 'Ebook Library menggunakan Calibre'
date: '2023-08-10 14:58:00'
---

Kali ini kita akan setup ebook library server menggunakan calibre.

# Syarat
- sudah install traefik di tutorial [berikut](./traefik)

# Mulai
- buat file `docker-compose.yaml` seperti ini

```YAML
version: "3"
services:
  calibre:
    image: lscr.io/linuxserver/calibre-web:latest
    container_name: calibre
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Jakarta
      - DOCKER_MODS=linuxserver/mods:universal-calibre #optional
      - OAUTHLIB_RELAX_TOKEN_SCOPE=1 #optional
    volumes:
      - ./config:/config
      - ./library:/books
    ports:
      - 3300:8083
    restart: unless-stopped
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.calibre.entrypoints=web"
      - "traefik.http.routers.calibre.rule=Host(`ebook.domain.com`)"
      - "traefik.http.services.calibre.loadbalancer.server.port=8083"
    networks:
      - traefik

networks:
  traefik:
    external: true
```

- ubah domain `ebook.domain.com` dengan domain yang dimiliki
- jalankan `docker-compose up -d` 
- calibre dapat di akses di domain tersebut
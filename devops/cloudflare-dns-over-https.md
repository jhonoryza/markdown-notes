---
title: 'DNS over https Cloudflare'
date: '2023-08-09 06:00:00'
---

Kali ini kita akan setup dns over https dari cloudflare.

## Mulai
- buat file `docker-compose.yml`

```YAML
version: "3"

services:
  cloudflared:
    image: visibilityspots/cloudflared
    container_name: cloudflared
    ports:
      - "5054:5054/tcp"
      - "5054:5054/udp"
    environment:
        TZ: Asia/Jakarta
        PORT: 5054
        ADDRESS: 0.0.0.0
    restart: always
    networks:
      cloudflared_net:
        ipv4_address: 10.0.0.2

networks:
  cloudflared_net:
    name: cloudflare_net
    driver: bridge
    ipam:
     config:
       - subnet: 10.0.0.0/29
```

- jalankan `docker compose up -d` atau `docker-compose up -d`

- test dengan perintah ini

```bash
dig @127.0.0.1 -p 5054 google.com
```

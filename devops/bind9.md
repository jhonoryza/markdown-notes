---
title: 'Bind9 DNS Server'
date: '2023-08-10 14:58:00'
---

Kali ini kita akan setup dns server menggunakan bind9.

## Syarat
- sudah install [cloudflare dns over https](./cloudflare-dns-over-https)

## Mulai
- buat file `docker-compose.yml`

```YAML
version: "3"

services:
  bind9:
    container_name: bind9
    image: ubuntu/bind9:latest
    environment:
      - BIND9_USER=root
      - TZ=Asia/Jakarta
    ports:
      - "53:53/tcp"
      - "53:53/udp"
    volumes:
      - ./config:/etc/bind
      - ./cache:/var/cache/bind
      - ./records:/var/lib/bind
    restart: unless-stopped
    networks:
      - cloudflare_net

networks:
  cloudflare_net:
    external: true
```

kali ini kita akan menggunakan network yg sudah dipakai oleh container cloudflared

- buat folder config
- buat file baru config/named.conf

```bash
acl internal {
    192.168.18.0/24;
};

options {
    forwarders {
        10.0.0.2 port 5054;
    };
    allow-query { internal; };
};

zone "local.domainku.dev" IN {
    type master;
    file "/etc/bind/local.zone";
};
```

- buat file baru config/local.zone

```bash
$TTL 2d

$ORIGIN local.domainku.dev.

@               IN      SOA     ns.local.domainku.dev. info.domainku.dev. (
                                2022122800      ; serial
                                12h             ; refresh
                                15m             ; retry
                                3w              ; expire
                                2h              ; minimum ttl
                                )

                IN      NS      ns.local.domainku.dev.

ns              IN      A       192.168.18.106

; -- add dns records below

portainer      IN      A       192.168.18.106
traefik        IN      A       192.168.18.106
```

- ubah ip address `192.168.18.106` dengan ip address dimana bind9 ini akan di jalankan
- ubah `domainku.dev` dengan nama domain kesukaan kalian
- jalankan `docker-compose up -d` atau `docker compose up -d`

- matikan systemd-resolved.service dengan cara run `sudo systemctl stop systemd-resolved.service` dan disable `sudo systemctl disable systemd-resolved.service`

- arahkan device yg akan diubah dns nya ke alamat ip dimana service tersebut dijalankan, contohnya service diatas saya running di laptop/pc dengan ip: 192.168.18.106 maka ubah pengaturan DNS menggunakan ip 192.168.18.106

## testing dns

ada dua cara untuk melakukan testing DNS

1. menggunakan dig, contoh

```bash
dig @192.168.18.106 -p 53 google.com
dig @192.168.18.106 -p 53 portainer.local.domainku.dev
dig @192.168.18.106 -p 53 traefik.local.domainku.dev
```

2. menggunakan nslookup

```bash
nslookup -port=53 google.com 192.168.18.106
nslookup -port=53 portainer.local.domainku.dev 192.168.18.106
nslookup -port=53 traefik.local.domainku.dev 192.168.18.106
```

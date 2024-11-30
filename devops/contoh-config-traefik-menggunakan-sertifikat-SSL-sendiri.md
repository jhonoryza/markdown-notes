Berikut adalah contoh konfigurasi docker-compose.yml untuk Traefik yang
menggunakan sertifikat SSL yang Anda miliki. Dalam contoh ini, kita akan
menggunakan file sertifikat dan kunci privat yang sudah ada.

buat file `docker-compose.yml`

```yaml
version: "3.8"

services:
    traefik:
        image: traefik:v2.9
        container_name: traefik
        restart: unless-stopped
        ports:
            - "80:80"
            - "443:443"
        volumes:
            - ./certs:/certs
            - /var/run/docker.sock:/var/run/docker.sock
        command:
            - --entrypoints.web.address=:80
            - --entrypoints.websecure.address=:443
            - --providers.docker=true
            - --providers.docker.network=web
            - --api.dashboard=true
            - --log.level=INFO
            - --certificatesresolvers.myresolver.acme.tlschallenge=true
            - --certificatesresolvers.myresolver.acme.email=your-email@example.com
            - --certificatesresolvers.myresolver.acme.storage=/acme.json
            - --tls.certificates.0.certfile=/certs/your-certificate.crt
            - --tls.certificates.0.keyfile=/certs/your-private-key.key
        labels:
            - "traefik.enable=true"
            - "traefik.http.routers.api.rule=Host(`traefik.yourdomain.com`)"
            - "traefik.http.routers.api.service=api@internal"
            - "traefik.http.routers.api.tls=true"
            - "traefik.http.routers.api.tls.certresolver=myresolver"
        networks:
            - web

networks:
    web:
        external: false
```

Penjelasan:

1. command:

- --entrypoints.web.address=:80: Mendefinisikan entrypoint untuk HTTP pada
  port 80.
- --entrypoints.websecure.address=:443: Mendefinisikan entrypoint untuk HTTPS
  pada port 443.
- --providers.docker=true: Mengaktifkan Docker sebagai penyedia layanan.
- --providers.docker.network=web: Menggunakan jaringan web untuk layanan Docker.
- --api.dashboard=true: Mengaktifkan dashboard Traefik.
- --log.level=INFO: Mengatur level logging ke INFO.
- --certificatesresolvers.myresolver.acme.tlschallenge=true: Mengaktifkan ACME
  dengan TLS-ALPN-01 challenge (opsional, bisa dikomentari jika tidak
  digunakan).
- --certificatesresolvers.myresolver.acme.email=your-email@example.com: Email
  untuk ACME (opsional, bisa dikomentari jika tidak digunakan).
- --certificatesresolvers.myresolver.acme.storage=/acme.json: Lokasi penyimpanan
  ACME (opsional, bisa dikomentari jika tidak digunakan).
- --tls.certificates.0.certfile=/certs/your-certificate.crt: Path ke file
  sertifikat Anda.
- --tls.certificates.0.keyfile=/certs/your-private-key.key: Path ke file kunci
  privat Anda.

2. labels:

- traefik.enable=true: Mengaktifkan Traefik untuk layanan ini.
- traefik.http.routers.api.rule=Host(traefik.yourdomain.com): Mendefinisikan
  aturan untuk router API.
- traefik.http.routers.api.service=api@internal: Mengarahkan router API ke
  layanan internal Traefik.
- traefik.http.routers.api.tls=true: Mengaktifkan TLS untuk router API.
- traefik.http.routers.api.tls.certresolver=myresolver: Menggunakan resolver
  sertifikat yang telah didefinisikan.

Pastikan struktur folder Anda seperti ini:

```
.
├── docker-compose.yml
└── certs
    ├── your-certificate.crt
    └── your-private-key.key
```

Setelah Anda memiliki semua file yang diperlukan, jalankan Traefik dengan
perintah berikut:

```
docker-compose up -d
```

Traefik sekarang akan berjalan dan menggunakan sertifikat SSL yang Anda miliki
dengan konfigurasi yang diberikan melalui command dan labels.

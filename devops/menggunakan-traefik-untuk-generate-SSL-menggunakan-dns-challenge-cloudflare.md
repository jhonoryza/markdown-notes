## Contoh konfigurasi docker compose

```
traefik:
  image: "traefik:2.3.7"
  container_name: "traefik"
  restart: unless-stopped
  ports:
    - "80:80"
    - "443:443"
    #- "8080:8080"
  environment:
    - CF_DNS_API_TOKEN=${CF_DNS_API_TOKEN}
  volumes:
    - ./traefik/config:/etc/traefik
    - traefik-ssl-certs:/ssl-certs
    - /etc/localtime:/etc/localtime
    - "/var/run/docker.sock:/var/run/docker.sock:ro"
  depends_on:
    - app
  labels:
    - "traefik.enable=true"
    - "traefik.http.routers.traefik.entrypoints=websecure"
    - "traefik.http.routers.traefik.tls.certresolver=myresolver"
    - "traefik.http.routers.traefik.rule=Host(`traefik.domain.id`)"
    - "traefik.http.routers.traefik.service=api@internal"
```

ubah `traefik.domain.id` dengan domain yang anda gunakan

buat file `.env` tambahkan

```
CF_DNS_API_TOKEN=
```

yang di dapat dr cloudflare

### Berikut adalah langkah-langkah untuk membuat API token di Cloudflare dengan izin yang diperlukan untuk mengelola DNS untuk domain Anda:

#### Langkah 1: Masuk ke Akun Cloudflare

1. Buka Cloudflare dan masuk ke akun Anda dengan kredensial yang sesuai.

#### Langkah 2: Akses Pengaturan API Tokens

1. Setelah masuk, klik pada avatar atau nama akun Anda di sudut kanan atas.
2. Pilih My Profile dari menu dropdown.
3. Pada halaman profil, klik tab API Tokens.

#### Langkah 3: Membuat API Token Baru

1. Klik tombol Create Token.
2. Pilih template Edit zone DNS atau klik Create Custom Token jika ingin
   menyesuaikan izin.

#### Langkah 4: Mengatur Izin API Token

Jika memilih Create Custom Token:

1. Token Name: Beri nama token Anda, misalnya Traefik DNS Challenge.

2. Permissions:

   - Klik Add permissions.
   - Pilih Zone sebagai Layanan.
   - Pilih DNS sebagai Sumber.
   - Pilih Edit sebagai Tindakan.

3. Zone Resources:

   - Klik Add Zone Resources.
   - Pilih Include.
   - Pilih All Zones atau Specific Zone jika Anda ingin membatasi akses ke zona
     tertentu saja. Jika memilih Specific Zone, tentukan domain yang ingin
     dikelola, misalnya example.com.

4. Client IP Address Filtering (opsional):

   - Jika ingin membatasi token ini hanya untuk digunakan dari alamat IP
     tertentu, tambahkan alamat IP yang sesuai.

5. TTL (opsional):

   - Tentukan waktu hidup token jika ingin token ini kadaluwarsa setelah periode
     tertentu.

#### Langkah 5: Membuat dan Menyimpan API Token

1. Klik Continue to summary.
2. Tinjau pengaturan token Anda dan pastikan semuanya benar.
3. Klik Create Token.

#### Langkah 6: Menyimpan API Token

1. Setelah token dibuat, salin token tersebut dan simpan di tempat yang aman.
   Anda hanya akan melihat token ini sekali, jadi pastikan untuk mencatatnya
   dengan baik.

buat file `traefik/config/traefik.yaml`

```
global:
  checkNewVersion: false
  sendAnonymousUsage: false

api:
  dashboard: true
  debug: true

accessLog: { }

entryPoints:
  web:
    address: :80

  websecure:
    address: :443

serversTransport:
  insecureSkipVerify: true

http:
  middlewares:
    redirect-to-https:
      redirectScheme:
        scheme: https
        permanent: true

certificatesResolvers:
  myresolver:
    acme:
      email: xxx@gmail.com
      storage: /ssl-certs/acme.json
      dnsChallenge:
        provider: cloudflare
        resolvers:
          - "1.1.1.1:53"
          - "8.8.8.8:53"
        delayBeforeCheck: 5

providers:
  docker:
    endpoint: "unix:///var/run/docker.sock"
    exposedByDefault: false
  file:
    directory: /etc/traefik
    watch: true
```

ganti `xxx@gmail.com` dengan email yang anda gunakan di cloudflare

## Jalankan traefik

ketikan perintah `docker compose up -d`

lalu akses http://traefik.domain.id atau domain yg sudah anda set tadi di awal

## debug proses dns challenge

cek file dengan perintah `cat /ssl-certs/acme.json` tp anda perlu masuk k dalam
container traefik terlebih dahulu dengan cara : `docker exec -it traefik sh`

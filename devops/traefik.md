---
title: 'Traefik'
date: '2023-08-10 22:38:00'
---

Kali ini kita akan setup traefik sebagai reverse proxy menggunakan docker.

## Required

- pertama buat file `docker-compose.yml` isinya seperti ini:

```YAML
version: "3.3"

services:

  traefik:
    image: "traefik:2.3.7"
    container_name: "traefik"
    # command:
      #- "--log.level=DEBUG"
      # - "--api.insecure=true"
      # - "--providers.docker=true"
      # - "--providers.docker.exposedbydefault=false"
      # - "--entrypoints.web.address=:80"
    restart: unless-stopped
    environment:
      - TZ=Asia/Jakarta
    #   - CLOUDFLARE_EMAIL=email@gmail.com #perlu di sesuaikan dengan email di cloudflare
    #   - CLOUDFLARE_DNS_API_TOKEN=apitoken #perlu di sesuaikan dengan api token di cloudflare
    ports:
      - "80:80"
      - "443:443"
      - "8080:8080"
    volumes:
      - ./config:/etc/traefik
      - ./ssl-certs:/ssl-certs
      - /etc/localtime:/etc/localtime
      - "/var/run/docker.sock:/var/run/docker.sock:ro"
    networks:
      - traefik
      - cloudflare_net
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.traefik.entrypoints=web"
      - "traefik.http.routers.traefik.rule=Host(`traefik.local.domain.site`)"
      # - "traefik.http.middlewares.https-redirect.redirectscheme.scheme=https"
      # - "traefik.http.middlewares.ssl-header.headers.customrequestheaders.X-Forwarded-Proto=https"
      # - "traefik.http.routers.traefik.middlewares=https-redirect"
      # - "traefik.http.routers.traefik-secure.entrypoints=websecure"
      # - "traefik.http.routers.traefik-secure.rule=Host(`traefik.local.domain.site`)"
      # - "traefik.http.routers.traefik-secure.tls=true"
      # - "traefik.http.routers.traefik-secure.tls.certresolver=cloudflare"
      # - "traefik.http.routers.traefik-secure.tls.domains[0].main=local.domain.site"
      # - "traefik.http.routers.traefik-secure.tls.domains[0].sans=*.local.domain.site"
      - "traefik.http.routers.traefik.service=api@internal"

networks:
  traefik:
    name: traefik
    driver: bridge
  cloudflare_net:
    external: true
```

- domain yang digunakan pada konfigurasi ini adalah `domain.site` silahkan ganti dengan domain yang dimiliki oleh teman2

- buat file config/traefik.yaml

```YAML
global:
  checkNewVersion: true
  sendAnonymousUsage: false

api:
  dashboard: true
  debug: true

entryPoints:
  web:
    address: :80

  websecure:
    address: :443

serversTransport:
  insecureSkipVerify: true

# certificatesResolvers:
#   cloudflare:
#     acme:
#       email: email@gmail.com #perlu di sesuaikan dengan email di cloudflare
#       storage: /ssl-certs/acme.json
#       caServer: "https://acme-v02.api.letsencrypt.org/directory"
#       dnsChallenge:
#         provider: cloudflare
#         disablePropagationCheck: false
#         delayBeforeCheck: 120
#         resolvers:
#           - "10.0.0.2:5054"

providers:
  docker:
    endpoint: "unix:///var/run/docker.sock"
    exposedByDefault: false
  file:
    directory: /etc/traefik
    watch: true
```

- jalankan `docker-compose up -d` maka traefik bisa dibuka pada url `http://traefik.local.domain.site`
- tentunya dengan catatan domain tersebut sudah di daftarkan di dns server

## Optional

jika kita membutuhkan reverse proxy ke service lain, misal service lain tsb berada di server yang berbeda, maka kita bisa menambahkan konfigurasi static di traefik, contohnya disini saya akan membuat file `config/config.yaml` dimana di sini kita akan mengarahkan ke server yang berbeda untuk aplikasi proxmox

```YAML
http:
  routers:
    proxmox-http:
      entrypoints:
        - web
      rule: "Host(`proxmox.local.domain.site`)"
      service: proxmox-service
      middlewares:
        - https-redirect
    proxmox-https:
      entrypoints:
        - websecure
      rule: "Host(`proxmox.local.domain.site`)"
      service: proxmox-service
      tls: {}
      middlewares:
        - default-headers

  services:
    proxmox-service:
      loadBalancer:
        servers:
          - url: "https://192.168.18.107:8006/"
        passHostHeader: true

  middlewares:
    https-redirect:
      redirectScheme:
        scheme: https
    default-headers:
      headers:
        frameDeny: true
        sslRedirect: true
        browserXssFilter: true
        contentTypeNosniff: true
        forceSTSHeader: true
        stsIncludeSubdomains: true
        stsPreload: true
        stsSeconds: 15552000
        customFrameOptionsValue: SAMEORIGIN
        customRequestHeaders:
          X-Forwarder-Proto: https
```
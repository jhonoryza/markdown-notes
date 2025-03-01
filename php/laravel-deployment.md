# Laravel Deployment

- create `Dockerfile` sebagai base image, disini saya menggunakan database
  postgres

```Dockerfile
FROM dunglas/frankenphp:latest-php8.2

# Install dependencies untuk Composer dan ekstensi PHP
RUN apt-get update && apt-get install -y \
    curl \
    unzip \
    libpq-dev \
    libexif-dev \
    libsodium-dev \
    libmagickwand-dev \
    postgresql-client

# Install Composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

RUN install-php-extensions \
    pgsql \
    pdo_pgsql \
    gd \
    intl \
    zip \
    exif \
    sodium \
    pcntl \
    imagick

# Install Imagick
# RUN pecl install imagick && docker-php-ext-enable imagick
```

- buat file `production.Dockerfile` sebagai image aplikasi
- disini aplikasi nya itu menggunakan stack inertia SSR
- sehingga perlu ada nya step untuk build dan running SSR menggunakan
  supervisord

```Dockerfile
FROM jhonoryza/frankenphp-pgsql:8.2 AS build

WORKDIR /app

COPY . ./

# Install dependencies menggunakan Composer
RUN composer install --no-dev --optimize-autoloader --no-interaction --no-plugins --no-scripts --prefer-dist

# Install Node.js and npm
RUN curl -fsSL https://nodejs.org/dist/v20.14.0/node-v20.14.0-linux-x64.tar.xz | tar -xJ -C /usr/local --strip-components=1

# Install npm dependencies and build assets
RUN npm install
RUN npm run build

# Final stage
FROM jhonoryza/frankenphp-pgsql:8.2

WORKDIR /app

COPY . ./
COPY --from=build /app/public /app/public
COPY --from=build /app/bootstrap/ssr /app/bootstrap/ssr
COPY --from=build /usr/local/bin/node /usr/local/bin/node
COPY --from=build /usr/local/lib/node_modules /usr/local/lib/node_modules
COPY --from=build /app/node_modules /app/node_modules

# Install dependencies menggunakan Composer
RUN composer install --no-dev --optimize-autoloader --no-interaction --no-plugins --no-scripts --prefer-dist

RUN rm -rf /root/.composer
RUN rm -rf ./git

# Install supervisord
RUN apt-get update && apt-get install -y supervisor

# Copy supervisord configuration
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf

# Jalankan supervisord dengan konfigurasi yang ditentukan
CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]
```

- buat file `supervisord.conf`
- jika tidak menggunakan SSR section `inertia-ssr` bisa dihapus

```
[supervisord]
nodaemon=true

[program:frankenphp]
command=php artisan octane:frankenphp --host=0.0.0.0 --port=80 --admin-port=2019
autostart=true
autorestart=true
stderr_logfile=/var/log/frankenphp.err.log
stdout_logfile=/var/log/frankenphp.out.log
user=root

[program:inertia-ssr]
command=php artisan inertia:start-ssr
autostart=true
autorestart=true
stderr_logfile=/var/log/inertia-ssr.err.log
stdout_logfile=/var/log/inertia-ssr.out.log
startretries=10
user=root
```

- kita akan build image aplikasi tsb di github action
- buat file `.github/workflows/main.yml`
- setiap ada push ke branch main akan mentrigger untuk build dan deploy
- persiapkan beberapa variable secrets direpo github pada menu setting
- digunakan untuk push image ke `hub.docker.com`:
  - `DOCKER_PASSWORD`
  - `DOCKER_USERNAME`
- digunakan untuk deploy ke home server menggunakan cloudflare zero trust :
  - `SSH_PRIVATE_KEY`: private key yg digunakan untuk ssh ke server tanpa
    password
  - `CLIENT_ID`: client id zero trust tunnel
  - `CLIENT_SECRET` : client secret zero trust tunnel

```yml
name: deploy

on:
  push:
    branches: [main]

jobs:
  build-docker-image:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout
      uses: actions/checkout@v1

    - name: Dockerhub login
      env:
          DOCKER_PASSWORD: ${{ secrets.DOCKER_PASSWORD }}
          DOCKER_USERNAME: ${{ secrets.DOCKER_USERNAME }}
      run: |
          echo "${DOCKER_PASSWORD}" | docker login --username ${DOCKER_USERNAME} --password-stdin

    - name: Set up Docker Buildx
      id: buildx
      uses: crazy-max/ghaction-docker-buildx@v1
      with:
          buildx-version: latest

    - name: Build base image with push
      run: |
         docker buildx build \
         --platform=linux/amd64 \
         --output "type=image,push=true" \
         --file ./Dockerfile . \
         --tag jhonoryza/frankenphp-pgsql:8.2

    - name: Build dockerfile (with push)
      run: |
          docker buildx build \
          --platform=linux/amd64 \
          --output "type=image,push=true" \
          --file ./production.Dockerfile . \
          --tag jhonoryza/filament-blog:latest

 deploy:
   needs: build-docker-image
   runs-on: ubuntu-latest

   steps:
   - name: Checkout repository
     uses: actions/checkout@v4

   - name: Connect and run command on remote server
     uses: and-fm/cloudflared-ssh-action@v3
     with:
       host: dell.labkita.my.id
       username: root
       private_key_filename: id_rsa
       private_key_value: ${{ secrets.SSH_PRIVATE_KEY }}
       port: 22
       service_token_id: ${{ secrets.CLIENT_ID }}
       service_token_secret: ${{ secrets.CLIENT_SECRET }}
       commands: /media/server_data/docker/laravel-projects/deploy-filament-blog.sh
```

- untuk
  `commands: /media/server_data/docker/laravel-projects/deploy-filament-blog.sh`
  disesuaikan saja sesuai kebutuhan
- buat file `deploy-filament-blog.sh` untuk running proses deployment di server
- karena aplikasi nya menggunakan filament ada beberapa perintah untuk optimasi filament

```bash
#! /bin/bash
docker pull jhonoryza/filament-blog:latest
cd /media/server_data/docker/laravel-projects/laravel-filament-blog/src
git pull origin main
docker compose -f production.docker-compose.yaml up -d
docker image prune -f
docker builder prune --force
docker exec src-frankenphp-filament-blog-1 php artisan filament:cache-components
docker exec src-frankenphp-filament-blog-1 php artisan icons:cache
docker exec src-frankenphp-filament-blog-1 php artisan sitemap:generate
```

- siapkan file `production.docker-compose.yml` yg akan digunakan nantinya untuk
  menjalankan container aplikasi kita
- untuk networks dan nama image bisa disesuaikan sesuai kebutuhan

```yml
services:
    frankenphp-filament-blog:
        image: "jhonoryza/filament-blog:latest"
        restart: unless-stopped
        volumes:
            - ./env:/app/.env
        networks:
            - tunnel_default
            - postgres
            - redis_default

networks:
    tunnel_default:
        external: true
    postgres:
        external: true
    redis_default:
        external: true
```

- untuk menjalankan run `docker compose up -d`
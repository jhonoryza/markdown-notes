---
title: 'Konfigurasi Github Action CI/CD untuk Laravel'
date: '2023-08-05 18:20:00'
---

Kali ini kita akan membuat konfigurasi `ci/cd pada github.com` untuk aplikasi laravel

## Petunjuk jika menggunakan tools laravel envoy
- sudah melakukan setup server seperti yg dijelaskan [article ini](./setup-ubuntu-20-04-ansible)
- sudah melakukan konfigurasi laravel envoy seperti yg dijelaskan [article ini](./deploy-laravel-with-envoy)
- buka repository menu `settings -> secrets -> menu`
- tambahkan secret baru `SSH_PRIVATE_KEY` value nya ambdil dari `private key`
- `private key` dilocal bisa lihat file ini `cat ~/.ssh/id_rsa`
- tambahkan secret baru `SSH_HOST` value nya isi alamat ip server
- tambahkan deploy keys dengan menggunakan `public key` (optional)
- `public key` dilocal bisa lihat file ini `cat ~/.ssh/id_rsa.pub`
- `private key / public key` jika belum ada, bisa generate secara manual dengan perintah `ssh-keygen` lalu enter sampai selesai
- buat folder baru `.github/workflows` di laravel app root dir
- buat file baru `deploy.yml` di `workflows` folder
- edit file `deploy.yml` tambahkan ini:

```bash
name: deploy

on: 
  push:
    branches: [test]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
            php-version: 8.0
            tools: composer:v2
            coverage: none
      - name: Install Composer dependencies
        run: composer update
      - name: Setup SSH
        uses: kielabokkie/ssh-key-and-known-hosts-action@v1.2.0
        with:
          ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}
          ssh-host: ${{ secrets.SSH_HOST }}
      - name: Deploy Environment
        run: ./vendor/bin/envoy run deploy
```

Setial kali ada push code push ke branch master maka proses ci/cd akan berjalan

## Petunjuk jika menggunakan tools Deployer
- sudah melakukan setup server seperti yg dijelaskan [article ini](./setup-ubuntu-20-04-ansible)
- sudah melakukan konfigurasi laravel menggunakan deployer seperti yg dijelaskan [article ini](./deploy-laravel-with-deployer)
- buka repository menu `settings -> secrets -> menu`
- tambahkan secret baru `SSH_PRIVATE_KEY` value nya ambdil dari `private key`
- `private key` dilocal bisa lihat file ini `cat ~/.ssh/id_rsa`
- tambahkan secret baru `SSH_HOST` value nya isi alamat ip server
- tambahkan deploy keys dengan menggunakan `public key` (optional)
- `public key` dilocal bisa lihat file ini `cat ~/.ssh/id_rsa.pub`
- `private key / public key` jika belum ada, bisa generate secara manual dengan perintah `ssh-keygen` lalu enter sampai selesai
- buat folder baru `.github/workflows` di laravel app root dir
- buat file baru `deploy.yml` di `workflows` folder
- edit file `deploy.yml` tambahkan ini:

```
name: deploy

on: 
  push:
    branches: [master]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
            php-version: 8.0
            tools: composer:v2
            coverage: none
      - name: Install Composer dependencies
        run: composer update
      - name: Deploy
        uses: deployphp/action@v1
        with:
          private-key: ${{ secrets.SSH_PRIVATE_KEY }}
          dep: deploy production -v
```

Setial kali ada push code push ke branch master maka proses ci/cd akan berjalan


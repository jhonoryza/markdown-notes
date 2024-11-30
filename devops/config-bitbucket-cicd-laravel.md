---
title: 'Konfigurasi Bitbucket CI/CD untuk Laravel'
date: '2023-08-05 18:10:00'
---

Kali ini kita akan membuat konfigurasi `ci/cd pada bitbucket.com` untuk aplikasi laravel

## Petunjuk jika menggunakan tools laravel envoy
1. sudah melakukan setup server seperti yg dijelaskan [article ini](./setup-ubuntu-20-04-ansible)
2. sudah melakukan konfigurasi laravel envoy seperti yg dijelaskan [article ini](./deploy-laravel-with-envoy)
3. buka menu repository `setting -> pipelines -> ssh keys`
4. masukan private key, dilocal bisa lihat file ini `cat ~/.ssh/id_rsa`
5. masukan public key, dilocal bisa lihat file ini `cat ~/.ssh/id_rsa.pub`
6. jika kedua file diatas belum ada bisa generate secara manual dengan perintah `ssh-keygen` lalu enter sampai selesai
7. fetch known host sampai berhasil
8. buat file `bitbucket-pipelines.yml` di root app directory laravel dengan konfigurasi ini:

```php
image: composer:2.0

pipelines:
  default:
    - parallel:
        - step:
            name: Deploy
            deployment: production
            script:
              - composer update
              - ./vendor/bin/envoy run deploy
```

Setial kali ada push code push ke branch master maka proses ci/cd akan berjalan

## Petunjuk jika menggunakan tools Deployer
1. sudah melakukan setup server seperti yg dijelaskan [article ini](./setup-ubuntu-20-04-ansible)
2. sudah melakukan konfigurasi laravel menggunakan deployer seperti yg dijelaskan [article ini](./deploy-laravel-with-deployer)
3. buka menu repository `setting -> pipelines -> ssh keys`
4. masukan private key, dilocal bisa lihat file ini `cat ~/.ssh/id_rsa`
5. masukan public key, dilocal bisa lihat file ini `cat ~/.ssh/id_rsa.pub`
6. jika kedua file diatas belum ada bisa generate secara manual dengan perintah `ssh-keygen` lalu enter sampai selesai
7. fetch known host sampai berhasil
8. buat file `bitbucket-pipelines.yml` di root app directory laravel dengan konfigurasi ini:

```bash
image: composer:2.0

pipelines:
  default:
    - parallel:
        - step:
            name: Deploy
            deployment: production
            script:
              - composer update
              - ./vendor/bin/dep deploy production -v
```

Setial kali ada push code push ke branch master maka proses ci/cd akan berjalan


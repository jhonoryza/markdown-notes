---
title: 'Konfigurasi Gitlab CI/CD untuk Laravel'
date: '2023-08-05 18:00:00'
---

Kali ini kita akan membuat konfigurasi `ci/cd pada gitlab.com` untuk aplikasi laravel

## Petunjuk jika menggunakan tools laravel envoy
1. sudah melakukan setup server seperti yg dijelaskan [article ini](./setup-ubuntu-20-04-ansible)
2. sudah melakukan konfigurasi laravel envoy seperti yg dijelaskan [article ini](./deploy-laravel-with-envoy)
3. tambahkan `SSH_PRIVATE_KEY` pada repository yang digunakan di menu `settings -> ci/cd -> variables`
4. dilocal bisa lihat file ini `cat ~/.ssh/id_rsa`
5. buat file baru `.gitlab-ci.yml` di laravel app root directory dan tambahkan konfigurasi dibawah:

```bash
stages:
  - Deploy

production-deployment:
  image: lorisleiva/laravel-docker:8.0
  stage: Deploy
  script:
    - 'which ssh-agent || ( apt-get update -y && apt-get install openssh-client -y )'
    - eval $(ssh-agent -s)
    - ssh-add <(echo "$SSH_PRIVATE_KEY")
    - mkdir -p ~/.ssh
    - '[[ -f /.dockerenv ]] && echo -e "Host *\n\tStrictHostKeyChecking no\n\n" > ~/.ssh/config'
    - composer update
    - vendor/bin/envoy run deploy
  only:
  - master
```

Setial kali ada push code push ke branch master maka proses ci/cd akan berjalan

## Petunjuk jika menggunakan tools Deployer
1. sudah melakukan setup server seperti yg dijelaskan [article ini](./setup-ubuntu-20-04-ansible)
2. sudah melakukan konfigurasi laravel menggunakan deployer seperti yg dijelaskan [article ini](./deploy-laravel-with-deployer)
3. tambahkan `SSH_PRIVATE_KEY` pada repository yang digunakan di menu `settings -> ci/cd -> variables`
4. dilocal bisa lihat file ini `cat ~/.ssh/id_rsa`
5. buat file baru `.gitlab-ci.yml` di laravel app root directory dan tambahkan konfigurasi dibawah:

```
stages:
  - Deploy

production-deployment:
  image: lorisleiva/laravel-docker:8.0
  stage: Deploy
  script:
    - 'which ssh-agent || ( apt-get update -y && apt-get install openssh-client -y )'
    - eval $(ssh-agent -s)
    - ssh-add <(echo "$SSH_PRIVATE_KEY")
    - mkdir -p ~/.ssh
    - '[[ -f /.dockerenv ]] && echo -e "Host *\n\tStrictHostKeyChecking no\n\n" > ~/.ssh/config'
    - composer update
    - vendor/bin/dep deploy production -vv
  only:
  - master
```

Setial kali ada push code push ke branch master maka proses ci/cd akan berjalan


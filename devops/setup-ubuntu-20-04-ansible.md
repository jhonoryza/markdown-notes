---
title: 'Setup Linux Server Ubuntu 20.04 LTS untuk aplikasi Laravel menggunakan Ansible'
date: '2023-08-05 17:10:00'
---

Kali ini kita akan menjelaskan bagaimana cara setup ubuntu server 20.04 secara automatis menggunakan `Ansible` agar dapat menjalankan aplikasi laravel

# Ansible Ubuntu Focal Fossa (20.04) LTS #

## Syarat

1. Install ansible di local pc. [cara install](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html).
2. clone repository ini

```bash
    git clone git@github.com:jhonoryza/ansible-ubuntu-focal.git
```

3. cd `ansible-ubuntu-focal`
2. Update host file, dimana variable ``hosts`` ini untuk konfigurasi ip address server yang akan digunakan dan user yg digunakan
3. Tambahkan local's public key ``~/.ssh/id_rsa.pub`` ke dalam remote server's di folder: ``~/.ssh/authorized_keys``
4. Update variable yang terdapat di folder vars jika diperlukan.

## Cara Pakai

run command di local:
``ansible-playbook -i hosts <role>.yml``

Kumpulan list role yang bisa digunakan :
* common
* nginx
* php
* mysql
* redis

contoh:
1. ``ansible-playbook -i hosts nginx.yml`` untuk install nginx
2. ``ansible-playbook -i hosts lemp.yml`` untuk install semuanya.

## Versi yang Terinstall

| Software   | Version |
| ---------- | ------- |
| nginx      | 1.17.10 |
| MySQL      | 8.0     |
| PHP        | 8.0     |
| redis      | latest  |
| phpmyadmin | 5.2.0   |
| phpmyadmin | latest   |

## akses phpmyadmin
buka browser [http://ip-address/phpmyadmin](http://ip-address/phpmyadmin)

## akses adminer
buka browser [http://ip-address/adminer](http://ip-address/adminer)
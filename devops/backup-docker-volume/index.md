---
title: 'Belajar Cara Backup dan Restore Docker Volume yang simple'
date: '2023-08-05 16:00:00'
---

Kali ini kita akan mempelajari cara `membackup` dan `merestore` `docker volume`, hal ini akan sangat berguna ketika kita memiliki data2 yang sangat penting dan diperlukan backup.

Contoh kali ini kita akan membackup docker volume dari aplikasi vaultwarden lalu kita simpan ke MinIO server

## Syarat
- sudah terinstall docker dan docker-compose. [referensi cara install](https://docs.docker.com/engine/install/)
- sudah membaca tutorial [vaultwarden](./vaultwarden)
- sudah membaca tutorial [minio](./minio)
- sudah terinstall vaultwarden dan minio

## Mulai Backup Docker Volume
1. pertama kita buat file `docker-compose.yml` masukan code seperti dibawah ini:

```YAML
version: "3.7"
services:
 mariadb:
  image: "mariadb"
  container_name: "mariadb"
  hostname: "mariadb"
  restart: unless-stopped
  volumes:
   - "mariadb_vol:/var/lib/mysql"
   - "/etc/localtime:/etc/localtime:ro"
  environment:
   - "MYSQL_ROOT_PASSWORD=secret"
   - "MYSQL_PASSWORD=secret"
   - "MYSQL_DATABASE=vaultwarden"
   - "MYSQL_USER=vaultwarden"

 vaultwarden:
  image: "vaultwarden/server:latest"
  container_name: "vaultwarden"
  hostname: "vaultwarden"
  restart: unless-stopped
  volumes:
   - "vaultwarden_vol:/data/"
  environment:
   - "DATABASE_URL=mysql://vaultwarden:secret@mariadb/vaultwarden"
   - "ADMIN_TOKEN=secret"
   - "RUST_BACKTRACE=1"
  ports:
   - "8200:80"

 backup:
  image: offen/docker-volume-backup:v2
  container_name: "pass-backup"
  restart: unless-stopped
  env_file: ./backup.env
  environment:
    AWS_ENDPOINT: localhost:9000
    AWS_S3_BUCKET_NAME: vaultwarden
    AWS_ACCESS_KEY_ID: admin
    AWS_SECRET_ACCESS_KEY: secret
  volumes:
    - vaultwarden_vol:/backup/vaultwarden-vol-backup:ro
    - mariadb_vol:/backup/mariadb-vol-backup:ro
    - /var/run/docker.sock:/var/run/docker.sock:ro

volumes:
 vaultwarden_vol:
 mariadb_vol:
```

2. buat file `backup.env` isi dengan seperti ini:

```
BACKUP_CRON_EXPRESSION="0 3 * * *"
```

disini kita akan menjalan backup secara periodik perhari setiap jam 3 pagi, untuk periodik nya ini bisa diubah2 menggunakan sintax cron, panduan [https://crontab.guru/](https://crontab.guru/)

## Restore Docker Volume dari Backup

1. download file yang telah terbackup melalui aplikasi minio
2. setelah berhasil didownload, jalankan perintah ini:

```bash
  tar -C /tmp -xvf  backup.tar.gz
```

ubah bagian `backup.tar.gz` dengan file name yg telah terdownload.

3. pada step ini kita akan membuat docker volume baru bernama `new_vaultwarden_vol` dan `new_mariadb_vol` dan container sementara bernama `temp1` dan `temp2`

```bash
  docker run -d --name temp -v new_vaultwarden_vol:/restore alpine
  docker cp /tmp/backup/vaultwarden-vol-backup/. temp1:/restore

  docker run -d --name temp -v new_mariadb_vol:/restore alpine
  docker cp /tmp/backup/mariadb-vol-backup/. temp2:/restore
```

4. docker volume `new_vaultwarden_vol` dan `new_mariadb_vol` sudah siap untuk digunakan
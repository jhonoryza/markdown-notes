---
title: 'Deploy Aplikasi Laravel Menggunakan Tools Laravel Envoy'
date: '2023-08-05 17:30:00'
---

Kali ini kita akan melakukan deployment menggunakan tools `Laravel Envoy` ke server ubuntu

## Syarat
- Server sudah tersetup, jika belum bisa cek tutorial [ini](./setup-ubuntu-20-04-ansible) atau [ini](./setup-ubuntu-20-04-manual)

## Persiapan sebelum deployment
1. buat directory baru di /var/www `-uwww-data mkdir -p storage/framework/sessions`
2. buat directory baru di /var/www `-uwww-data mkdir -p storage/framework/views`
3. buat directory baru di /var/www `-uwww-data mkdir -p storage/framework/cache`
4. buat file .env di /var/www `-uwww-data touch .env`

```bash
APP_NAME=envoyapp
APP_ENV=production
APP_KEY=
APP_DEBUG=false
APP_URL=http://envoy.labkita.my.id

LOG_CHANNEL=stack
LOG_DEPRECATIONS_CHANNEL=null
LOG_LEVEL=debug

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=envoy
DB_USERNAME=ubuntu
DB_PASSWORD=secret

BROADCAST_DRIVER=log
CACHE_DRIVER=file
FILESYSTEM_DISK=local
QUEUE_CONNECTION=sync
SESSION_DRIVER=file
SESSION_LIFETIME=120

MEMCACHED_HOST=127.0.0.1

REDIS_CLIENT=predis
REDIS_HOST=127.0.0.1
REDIS_PASSWORD=null
REDIS_PORT=6379

MAIL_MAILER=smtp
MAIL_HOST=mailhog
MAIL_PORT=1025
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null
MAIL_FROM_ADDRESS="hello@example.com"
MAIL_FROM_NAME="${APP_NAME}"

AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_DEFAULT_REGION=us-east-1
AWS_BUCKET=
AWS_USE_PATH_STYLE_ENDPOINT=false

PUSHER_APP_ID=
PUSHER_APP_KEY=
PUSHER_APP_SECRET=
PUSHER_HOST=
PUSHER_PORT=443
PUSHER_SCHEME=https
PUSHER_APP_CLUSTER=mt1

VITE_PUSHER_APP_KEY="${PUSHER_APP_KEY}"
VITE_PUSHER_HOST="${PUSHER_HOST}"
VITE_PUSHER_PORT="${PUSHER_PORT}"
VITE_PUSHER_SCHEME="${PUSHER_SCHEME}"
VITE_PUSHER_APP_CLUSTER="${PUSHER_APP_CLUSTER}"
```

5. buat file `Envoy.blade.php` di root laravel directory

```php
@servers(['production' => ['root@serve-ip-address']])
 
@setup
    $repo = 'https://github.com/jhonoryza/try-envoy.git';
    $appDir = '/var/www';
    $branch = 'master';

    date_default_timezone_set('Asia/Jakarta');
    $date = date('YmdHis');

    $builds = $appDir . '/sources';
    $deployment = $builds . '/' . $date;

    $serve = $appDir . '/source';
    $env = $appDir . '/.env';
    $storage = $appDir . '/storage';
@endsetup

@story('deploy')
    git
    install
    live
@endstory

@task('git', ['on' => 'production'])
    git clone -b {{ $branch }} "{{ $repo }}" {{ $deployment }}
@endtask

@task('install', ['on' => 'production'])
    cd {{ $deployment }}

    rm -rf {{ $deployment }}/storage
    
    ln -nfs {{ $env }} {{ $deployment }}/.env
    
    ln -nfs {{ $storage }} {{ $deployment }}/storage

    composer install --prefer-dist --no-dev
    
    php ./artisan migrate --force
@endtask

@task('live', ['on' => 'production'])
    cd {{ $deployment }}
    
    ln -nfs {{ $deployment }} {{ $serve }}
    
    chown -R www-data: /var/www

    systemctl restart php8.0-fpm

    systemctl restart nginx
@endtask
```

## Proses Deployment
1. jalankan perintah `./vendor/bin/envoy run deploy`
2. generate app key dengan menjalankan perintah `php artisan key:generate` di folder /var/www/source
3. penyesuaian folder permission dengan perintah `chown -R www-data: /var/www/`

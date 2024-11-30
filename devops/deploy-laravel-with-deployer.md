---
title: 'Deploy Aplikasi Laravel Menggunakan Tools Deployer'
date: '2023-08-05 17:20:00'
---

Kali ini kita akan melakukan deployment menggunakan tools `Deployer` ke server ubuntu

## Syarat
- Server sudah tersetup, jika belum bisa cek tutorial [ini](./setup-ubuntu-20-04-ansible) atau [ini](./setup-ubuntu-20-04-manual)

## deploy preparation
1. buat directory baru di /var/www `sudo -uwww-data mkdir -p shared`
2. buat file .env di /var/www `sudo -uwww-data touch shared/.env`

```bash
APP_NAME=deployerapp
APP_ENV=production
APP_KEY=
APP_DEBUG=false
APP_URL=http://domain

LOG_CHANNEL=stack
LOG_DEPRECATIONS_CHANNEL=null
LOG_LEVEL=debug

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=database_name
DB_USERNAME=ubuntu
DB_PASSWORD=secret

BROADCAST_DRIVER=log
CACHE_DRIVER=redis
FILESYSTEM_DISK=local
QUEUE_CONNECTION=sync
SESSION_DRIVER=redis
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

3. buat konfigurasi file `deploy.php` di laravel root directory 

```php
<?php

namespace Deployer;

require 'recipe/laravel.php';

// Set SSH Multiplexing
set('ssh_multiplexing', true);

// Set default branch
set('branch', 'master');

// Set git_tty
set('git_tty', false);

// Set php binary file path
set('bin/php', '/usr/bin/php8.0');

// Project name
set('application', 'app');

// Project repository
set('repository', 'github repository url');

// Shared files/dirs between deploys
add('shared_files', ['.env']);
add('shared_dirs', ['public/files', 'storage']);
add('shared_dirs', ['public/images', 'storage']);

// Writable dirs by web server
add('writable_dirs', []);

set('keep_releases', 2);

// Hosts
host('production')
    ->setHostname('server-ip-address') //need to be configured
    ->set('forward_agent',false)
    ->set('remote_user', 'root')
    ->set('port', 22)
    ->set('deploy_path', '/var/www/{{application}}')
    ->setLabels([
        'type' => 'app',
        'env' => 'production',
    ]);
    
task('deploy:vendors', function () {
    run('cd {{release_path}} && {{bin/php}} /usr/bin/composer update --verbose --prefer-dist --no-progress --no-interaction --optimize-autoloader');
});

task('artisan:clear-compiled', function () {
    run('{{bin/php}} {{release_path}}/artisan clear-compiled');
});

task('restart:web', function () {
    run('sudo service php8.0-fpm restart');
    run('sudo service nginx restart');
})->select('type=app');

task('restart:workers', function () {
    run('{{bin/php}} {{release_path}}/artisan queue:restart');
})->select('type=app');

task('restart:services', ['restart:web', 'restart:workers']);

// [Optional] if deploy fails automatically unlock.
after('deploy:failed', 'deploy:unlock');

/**
 * Default main deploy task from recipe
 * desc('Deploys your project');
 * task('deploy', [
 *     'deploy:prepare',
 *     'deploy:vendors',
 *     'artisan:storage:link',
 *     'artisan:config:cache',
 *     'artisan:route:cache',
 *     'artisan:view:cache',
 *     'artisan:event:cache',
 *     'artisan:migrate',
 *     'deploy:publish',
 * ]);
 */

before('artisan:config:cache', 'artisan:clear-compiled');
before('deploy:success', 'restart:services');
```

## Proses Deployment
1. jalankan perinta `./vendor/bin/dep deploy production`
2. jalankan perinta `php artisan key:generate` di folder /var/www/app/current
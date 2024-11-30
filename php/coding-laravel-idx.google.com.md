## Cara Akses

Buka [idx.google.com](https://idx.google.com)

## Cara menambahkan mysql service

Edit file .idx/dev.nix tambahkan baris ini

```
services.mysql = {
  enable = true;
  package = pkgs.mariadb;
};
```

dan bagian ini untuk tambahkan redis

```
packages = [
  pkgs.php82
  pkgs.php82Packages.composer
  pkgs.nodejs_20
  pkgs.redis
];
```

## Cara Akses mysql

Buka terminal, ketikan perintah `mysql -uroot`, lalu buat database baru
`create database laravel;` dan `show databases;` untuk melihat list database.

## Cara menambahkan postgresql

Edit file .idx/dev.nix tambahkan baris ini

```
services.postgres = {
  enable = true;
  package = pkgs.postgresql;
};
```

## Adjust .env laravel

Masukan baris ini

```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=root
DB_PASSWORD=
```

## Cara running redis server

buka terminal jalankan `redis-server`

## Beberapa kendala yang sering terjadi

1. terkadang file asset tidak terload secara sempurna kita bisa menambahkan ini
   di env laravel

```
APP_URL=
ASSET_URL=${APP_URL}
```

sesuaikan jg APP_URL dgn url yg di dapat dari browser atau buka terminal
jalankan `bash -c 'env' | grep HOST` copy value `WEB_HOST` dan tambahkan
`https://` sebelumnya lalu isi di APP_URL

2. https issue, kita bisa memaksa agar url asset di load dari https url dengan
   cara menambahkan ini di env laravel

```
FORCE_HTTPS_URL=true
```

tambahkan config tambahan di config/app.php

```php
'force_https_url' => env('FORCE_HTTPS_URL', false),
```

tambahkan code berikut di AppServiceProvider

```php
if (config('app.force_https_url')) {
    URL::forceScheme('https');
    request()->server->set('HTTPS', request()->header('X-Forwarded-Proto', 'https') == 'https' ? 'on' : 'off');
}
```

pastikan middleware TrustProxies

```php
protected $proxies = '*';
```

3. untuk livewire project jgn lupa run `php artisan livewire:publish --assets`
   setiap kali run composer install dan `php artisan storage:link`

## Contoh isi dev.nix file

```
# To learn more about how to use Nix to configure your environment
# see: https://developers.google.com/idx/guides/customize-idx-env
{pkgs}: {
  # Which nixpkgs channel to use.
  channel = "stable-23.11"; # or "unstable"
  # Use https://search.nixos.org/packages to find packages
  packages = [
    pkgs.php82
		pkgs.php82Extensions.redis
    pkgs.php82Packages.composer
    pkgs.nodejs_20
    pkgs.redis
  ];
  # Sets environment variables in the workspace
  env = {};
  services.mysql = {
    enable = true;
    package = pkgs.mariadb;
  };
	services.postgres = {
    enable = true;
    package = pkgs.postgresql;
  };
  idx = {
    # Search for the extensions you want on https://open-vsx.org/ and use "publisher.id"
    extensions = [
      # "vscodevim.vim"
    ];
    # Enable previews and customize configuration
    previews = {
      enable = true;
      previews = {
        web = {
          command = ["php" "artisan" "serve" "--port" "$PORT" "--host" "0.0.0.0"];
          manager = "web";
        };
      };
    };
		# Workspace lifecycle hooks
    workspace = {
      # Runs when a workspace is first created
      onCreate = {
        # Example: install JS dependencies from NPM
        # npm-install = "npm install";
      };
      # Runs when the workspace is (re)started
      onStart = {
        # Example: start a background task to watch and re-build backend code
        # watch-backend = "npm run watch-backend";
				watch-redis = "redis-server";
      };
    };
  };
}
```

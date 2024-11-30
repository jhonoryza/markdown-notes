---
title: 'Setup VPS Linux Server Ubuntu 20.04 LTS untuk aplikasi Laravel secara manual'
date: '2023-08-05 17:00:00'
---

Kali ini kita akan menjelaskan bagaimana cara setup ubuntu server 20.04 secara `manual` agar dapat menjalankan aplikasi laravel

# operating system
`Ubuntu 20.04 LTS`

## server preparation
1. install acl
2. install nginx
3. install php
4. install composer
5. install mysql server
6. install git
7. install redis server

### install acl
`apt install acl`

### install nginx
`apt install nginx`

### install php
1. `apt install software-properties-common`
2. `add-apt-repository ppa:ondrej/php`
3. `apt install php8.0 php8.0-fpm php8.0-mysql php8.0-common php8.0-cli php8.0-cgi php8.0-curl php8.0-gd php8.0-mbstring php8.0-intl php8.0-sqlite3 php8.0-xsl php8.0-xml php8.0-zip php8.0-memcached php8.0-opcache`

### install composer
1. buat shell file `install-composer.sh`
```
#!/bin/sh

EXPECTED_CHECKSUM="$(php -r 'copy("https://composer.github.io/installer.sig", "php://stdout");')"
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
ACTUAL_CHECKSUM="$(php -r "echo hash_file('sha384', 'composer-setup.php');")"

if [ "$EXPECTED_CHECKSUM" != "$ACTUAL_CHECKSUM" ]
then
    >&2 echo 'ERROR: Invalid installer checksum'
    rm composer-setup.php
    exit 1
fi

php composer-setup.php --quiet
RESULT=$?
rm composer-setup.php
exit $RESULT
```
2. chmod +x `install-composer.sh`
3. run `./install-composer.sh`
4. mv composer.phar /usr/bin/composer

### install mysql server
1. `apt install mysql-server`
2. `mysql -uroot -p` if password is asked just enter
3. jalankan script ini untuk membuat user baru dan database baru
```
create user 'ubuntu';
grant all privileges on *.* to 'ubuntu' with grant option;
flush privileges;
alter user ubuntu identified with mysql_native_password by 'secret';
create database envoy;
```

### install git
`apt install git`

### install redis server
`apt install redis server`

### nginx configuration
1. backup /etc/nginx/sites-available/default
2. adjust default config

```
  root /var/www/app2/current/public;
  index index.html index.htm index.nginddx-debian.html index.php;
  location / {
		try_files $uri $uri/ /index.php?$query_string;
	}
  location ~ \.php$ {
		include snippets/fastcgi-php.conf;
		fastcgi_pass unix:/var/run/php/php8.0-fpm.sock;
	}
  location ~ /\.ht {
		deny all;
	}
```

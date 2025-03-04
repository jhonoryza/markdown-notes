# Nginx & PHP timeout

1. Mengatur Timeout di Nginx

Edit file konfigurasi Nginx, biasanya di:

    /etc/nginx/nginx.conf (untuk global)
    /etc/nginx/sites-available/default atau file virtual host lainnya (untuk domain tertentu)

Tambahkan atau ubah parameter berikut di dalam blok server {} atau location {}:

```
server {
    listen 80;
    server_name example.com;

    location / {
        proxy_read_timeout 600;
        proxy_connect_timeout 600;
        proxy_send_timeout 600;
        fastcgi_read_timeout 600;
    }
}
```

📌 Penjelasan:

    proxy_read_timeout: Waktu maksimal Nginx menunggu respons dari backend.
    proxy_connect_timeout: Waktu maksimal Nginx menunggu koneksi ke backend.
    proxy_send_timeout: Waktu maksimal Nginx mengirim data ke backend.
    fastcgi_read_timeout: Jika menggunakan PHP-FPM, ini mengatur batas waktu saat membaca respons dari PHP.

🔄 Restart Nginx setelah mengubah konfigurasi:

```bash
sudo systemctl restart nginx
```

2. Mengatur Timeout di PHP

Edit file konfigurasi PHP:

    Jika menggunakan PHP-FPM: /etc/php/{versi}/fpm/php.ini
    Jika menggunakan CLI/Apache: /etc/php/{versi}/cli/php.ini

Cari dan ubah nilai berikut:

```
max_execution_time = 600
max_input_time = 600
```

📌 Penjelasan:

    max_execution_time: Waktu maksimal script PHP berjalan.
    max_input_time: Waktu maksimal PHP membaca input dari request.

Selain itu, jika menggunakan PHP-FPM, edit file /etc/php/{versi}/fpm/pool.d/www.conf dan ubah:

```
request_terminate_timeout = 600s
```

🔄 Restart PHP-FPM setelah perubahan:

```bash
sudo systemctl restart php{versi}-fpm
```

Setelah semua diubah, coba jalankan perintah berikut untuk memastikan konfigurasi diterapkan dengan benar:

```bash
php -i | grep "max_execution_time"
php -i | grep "max_input_time"
```


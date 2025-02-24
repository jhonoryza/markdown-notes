# SMTP Brevo

- setelah berhasil login / registrasi
- buka menu `Senders, Domains & Dedicated IPs`
- buka tab `domains`
  - klik `add a domain` button
  - pastikan domain tervalidasi
  - integrasi domain lebih mudah jika domain menggunakan cloudflare
- buka tab `senders` klik `add sender` masukan
  - name: noreply
  - email: noreply@our-domain.com
- buka menu `SMTP & API`
- klik button `generate a new smtp key`
- simpan `key` tersebut sebagai password stmp
- simpan informasi `Login` sebagai username smtp
- simpan informasi `SMTP Server` sebagai host smtp
- gunakan port `2525` sebagai port smtp

contoh konfigurasi di `.env` file laravel

```
MAIL_MAILER=smtp
MAIL_SCHEME=null
MAIL_HOST=smtp.brevo.com
MAIL_PORT=2525
MAIL_USERNAME="xxx"
MAIL_PASSWORD="xxx"
MAIL_FROM_ADDRESS="noreply@our-domain.com"
MAIL_FROM_NAME="noreply"
```

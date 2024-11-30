Memeriksa Zona Waktu Saat Ini

timedatectl adalah utilitas baris perintah yang memungkinkan Anda untuk melihat
dan mengubah waktu dan tanggal sistem. Perintah timedatectl\
tersedia di semua sistem Linux berbasis sistem modern.

Untuk melihat zona waktu saat ini, aktifkan perintah timedatectl tanpa opsi atau
argumen apa pun:

`timedatectl`

```
                  Local time: Tue 2019-12-03 16:30:44 UTC
                  Universal time: Tue 2019-12-03 16:30:44 UTC
                        RTC time: Tue 2019-12-03 16:30:44
                       Time zone: Etc/UTC (UTC, +0000)
       System clock synchronized: no
systemd-timesyncd.service active: yes
                 RTC in local TZ: no
```

Seperti yang ditunjukkan oleh output di atas, zona waktu sistem diatur ke UTC:

Zona waktu sistem dikonfigurasikan dengan menghubungkan /etc/localtime ke
pengidentifikasi zona waktu biner di direktori /usr/share/zoneinfo.

Jadi, opsi lain untuk memeriksa zona waktu adalah dengan memeriksa jalur symlink
dengan menggunakan perintah ls:

ls -l /etc/localtimelrwxrwxrwx 1 root root 27 Dec 3 16:29 /etc/localtime ->
/usr/share/zoneinfo/Etc/UTC

## Mengubah Zona Waktu di Linux

Sebelum mengubah zona waktu, Anda harus mencari tahu nama panjang untuk zona
waktu yang ingin Anda gunakan. Konvensi penamaan zona waktu biasanya menggunakan
format “Wilayah/Kota”.

Untuk membuat daftar semua zona waktu yang tersedia, Anda dapat menggunakan
perintah timedatectl atau membuat list file di direktori /usr/share/zoneinfo

```
timedatectl list-timezones...
```

```
Asia/Hong_Kong
Asia/Hovd
Asia/Irkutsk
Asia/Jakarta
Asia/Jayapura
Asia/Jerusalem
Asia/Kabul
Asia/Kamchatka
Asia/Karachi
Asia/Kathmandu
```

Setelah Anda mengidentifikasi zona waktu mana yang akurat untuk lokasi Anda,
jalankan perintah berikut sebagai pengguna sudo:

`sudo timedatectl set-timezone <timezone>`

Misalnya, untuk mengubah zona waktu sistem ke waktu lokal Jakarta:

`sudo timedatectl set-timezone Asia/Jakarta`

Jalankan perintah timedatectl untuk memverifikasi perubahan:

```
timedatectl                      


                  Local time: Tue 2019-12-03 13:55:09 WIB
                  Universal time: Tue 2019-12-03 18:55:09 UTC
                        RTC time: Tue 2019-12-03 18:02:16
                       Time zone: Asia/Jakarta (WIB, +0700)
       System clock synchronized: no
systemd-timesyncd.service active: yes
                 RTC in local TZ: no
```

Dengan begini, Anda telah berhasil mengubah zona waktu sistem Anda.

berikut contoh config yang saya gunakan di local

`dnsmasq.conf file`

```
address=/.test/127.0.0.1
listen-address=127.0.0.1
address=/.local.labkita.my.id/192.168.18.106
server=192.168.18.106
server=1.1.1.1
server=8.8.8.8
server=9.9.9.9
server=192.168.18.1
```

penjelasan :

- pengertian `server` disini adalah dns resolver yang saya gunakan 5 server tsb
  memiliki dns resolver port 53
- semua akses domain `*.local.labkita.my.id` akan di redirect ke ip
  `192.168.18.106`
- dan semua akses domain `*.test` akan di redirect ke ip `127.0.0.1`

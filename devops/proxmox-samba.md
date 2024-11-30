---
title: 'Cara integrasi samba diproxmox'
date: '2023-08-15 14:58:00'
---

Kali ini kita akan setup samba di proxmox menggunakan storage type directory.

- pertama login ke server proxmox kalian, edit file `/etc/fstab`

```bash
//[IP Address Samba Server]/[Samba Share Name] /mnt/pve/share/ cifs credentials=/root/.dellsmb,users,rw,iocharset=utf8
```

- ubah [IP Address Samba Server] termasuk `[]` dengan ip address di mana samba server berjalan
- ubah [Samba Share Name] termasuk `[]` dengan nama samba share yang di inginkan
- pastikan konfigurasi sambah share `Read Only = No`
- ubah `/mnt/pve/share/` dengan folder yang di inginkan jika belum ada bisa create folder nya dahulu
- run `mount -a`
- buka halaman proxmox di browser -> Datacenter -> Storage -> Add Directory
- isikan ID bebas
- isikan Directory dengan `/mnt/pve/share/`
- save
- refresh browser
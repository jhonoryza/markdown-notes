---
title: 'Proxmox cheatsheet'
date: '2023-08-15 01:01:00'
---

Beberapa command yang berguna ketika menggunakan proxmox server.

- import cloud image ke disk, contoh:

```bash
qm disk import 101 /var/lib/vz/template/iso/ubuntu-22.04-server-cloudimg-amd64.img local-lvm
```

- resize disk, contoh:

```bash
qm disk resize 101 scsi0 +15G
```


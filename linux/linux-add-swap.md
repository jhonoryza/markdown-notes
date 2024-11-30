---
title: 'Add swap in linux'
date: '2023-08-22 20:16:00'
---

Kali ini kita akan membuat swap file di linux.

```bash
sudo fallocate -l 8G /swapfile && \
sudo chmod 600 /swapfile && \
sudo mkswap /swapfile && \
sudo swapon /swapfile && \
sudo echo "/swapfile swap swap defaults 0 0" >> /etc/fstab && \
sudo swapon --show && \
sudo free -h && \
sudo sysctl vm.swappiness=1 && \
sudo echo "vm.swappiness=1" >> /etc/sysctl.conf && \
```

untuk mematikan swap

```bash
sudo swapoff -v /swapfile
```

untuk mematikan secara permanent

hapus `/swapfile swap swap defaults 0 0` di /etc/fstab

```bash
sudo rm /swapfile
```

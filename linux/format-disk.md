# Format dan Mount Disk di Linux

Panduan langkah demi langkah untuk memformat dan mount disk baru di Linux.

## 1. Cek Partisi pada Disk

Pastikan disk tidak memiliki partisi yang perlu dihapus atau sudah siap digunakan:

```bash
lsblk
# atau
fdisk -l /dev/sdb
```

## 2. Membuat Partisi Baru

### Opsi 1: Menggunakan fdisk (MBR)

```bash
sudo fdisk /dev/sdb
```

Langkah-langkah:
1. Ketik `n` → (new partition)
2. Pilih `p` → (primary partition)
3. Tekan Enter untuk menerima default (menggunakan seluruh disk)
4. Ketik `w` → (write changes)

> Note: Jika ingin GPT (bukan MBR), gunakan `g` sebelum `n` untuk membuat tabel partisi GPT.

### Opsi 2: Menggunakan parted (GPT)

```bash
sudo parted /dev/sdb
```

Kemudian jalankan:
```bash
mklabel gpt
mkpart primary ext4 0% 100%
quit
```

## 3. Format Partisi

### Format dengan ext4

```bash
sudo mkfs.ext4 /dev/sdb1
```

### Format dengan XFS

```bash
sudo mkfs.xfs /dev/sdb1
```

## 4. Mount Disk

1. Buat folder untuk mount point:
```bash
sudo mkdir -p /mnt/disk
```

2. Mount partisi:
```bash
sudo mount /dev/sdb1 /mnt/disk
```

3. Verifikasi mounting:
```bash
df -hT
```

## 5. Konfigurasi Auto-Mount (Persistent)

1. Dapatkan UUID disk:
```bash
blkid /dev/sdb1
```

Output contoh:
```
/dev/sdb1: UUID="abc123" TYPE="ext4"
```

2. Tambahkan ke /etc/fstab:
```bash
echo 'UUID=abc123 /mnt/disk ext4 defaults 0 2' | sudo tee -a /etc/fstab
```

3. Terapkan konfigurasi:
```bash
sudo mount -a
```

## Selesai! 

Disk Anda sekarang sudah siap digunakan dan akan otomatis ter-mount saat sistem boot. 🚀

## Tips
- Selalu backup data penting sebelum melakukan format disk
- Pastikan menggunakan device path yang benar (/dev/sdb, dll)
- Untuk disk >2TB, gunakan GPT daripada MBR
# Adding Swap in Linux

This guide will show you how to create a swap file in Linux.

```bash
sudo fallocate -l 8G /swapfile && \
sudo chmod 600 /swapfile && \
sudo mkswap /swapfile && \
sudo swapon /swapfile && \
sudo echo "/swapfile swap swap defaults 0 0" >> /etc/fstab && \
sudo swapon --show && \
sudo free -h && \
sudo sysctl vm.swappiness=1 && \
sudo echo "vm.swappiness=1" >> /etc/sysctl.conf
```

Explanation of the commands:

- `sudo fallocate -l 8G /swapfile`: Allocate 8GB for the swap file.
- `sudo chmod 600 /swapfile`: Set the correct permissions for the swap file.
- `sudo mkswap /swapfile`: Set up the swap file.
- `sudo swapon /swapfile`: Enable the swap file.
- `sudo echo "/swapfile swap swap defaults 0 0" >> /etc/fstab`: Make the swap
  file permanent by adding it to `/etc/fstab`.
- `sudo swapon --show`: Verify that the swap file is active.
- `sudo free -h`: Display the current swap usage.
- `sudo sysctl vm.swappiness=1`: Set the swappiness value to 1.
- `sudo echo "vm.swappiness=1" >> /etc/sysctl.conf`: Make the swappiness setting
  permanent.

## Disabling Swap

To disable the swap file, use the following command:

```bash
sudo swapoff -v /swapfile
```

To disable the swap file permanently, remove the line
`/swapfile swap swap defaults 0 0` from `/etc/fstab` and delete the swap file:

```bash
sudo rm /swapfile
```

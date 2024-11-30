this only for ubuntu server we can install firewalld

```bash
sudo apt-get install firewalld
sudo systemctl enable firewalld
sudo systemctl start firewalld
```

You only need to do this one time. It will still be there after a reboot.

Then, to open port 80:

```bash
sudo firewall-cmd --zone=public --add-port=80/tcp --permanent  #  or --add-service=http 
sudo firewall-cmd --reload
```

To verify:

`sudo firewall-cmd --list-all`

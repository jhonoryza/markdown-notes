1. install [orbstack](https://docs.orbstack.dev/)
2. buat virtual machine ubuntu 23.10
3. sebelumnya download versi arm
   [di sini](https://www.oracle.com/database/technologies/instant-client/linux-arm-aarch64-downloads.html)
4. untuk step instalasi

```bash
# jadi super user
sudo su

# buat folder
mkdir /usr/lib/oracle
mkdir /usr/lib/oracle/19.22
mkdir /usr/lib/oracle/19.22/client

# copy basic and sdk file to this path /usr/lib/oracle/19.22/client
cd /usr/lib/oracle/19.22/client

# unzip basic and sdk file
unzip instantclient-basic.zip
unzip instantclient-sdk.zip

# rename folder 
mv instantclient_19_22 lib

# registrasi ldconfig
echo /usr/lib/oracle/19.22/client/lib > /etc/ld.so.conf.d/oracle.conf
ldconfig

# install depedency
apt install install php-dev php-pear build-essential libaio1

# gunakan pecl untuk compile oci8.so
pecl channel-update pecl.php.net
pecl install oci8

# jika ditanya ORACLE_HOME input /usr/lib/oracle/19.22/client

# install extensi ke `php.ini` 
echo extension=oci8.so >> /etc/php/8.2/cli/php.ini
```

6. untuk testing, klo udh keinstall buat `test.php`

```php
<?php
if (extension_loaded('oci8')) {
    echo "OCI8 extension is installed and loaded.";
} else {
    echo "OCI8 extension is not installed or loaded.";
}
```

`php test.php`

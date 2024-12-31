# Checking the Current Time Zone

`timedatectl` is a command-line utility that allows you to view and change the
system's time and date. The `timedatectl` command is available on all modern
Linux-based systems.

To view the current time zone, run the `timedatectl` command without any options
or arguments:

```bash
timedatectl
```

Example output:

```
                  Local time: Tue 2019-12-03 16:30:44 UTC
              Universal time: Tue 2019-12-03 16:30:44 UTC
                    RTC time: Tue 2019-12-03 16:30:44
                   Time zone: Etc/UTC (UTC, +0000)
     System clock synchronized: no
systemd-timesyncd.service active: yes
             RTC in local TZ: no
```

As shown in the output above, the system time zone is set to UTC.

The system time zone is configured by linking `/etc/localtime` to the binary
time zone identifier in the `/usr/share/zoneinfo` directory.

Another way to check the time zone is by inspecting the symlink path using the
`ls` command:

```bash
ls -l /etc/localtime
```

Example output:

```
lrwxrwxrwx 1 root root 27 Dec 3 16:29 /etc/localtime -> /usr/share/zoneinfo/Etc/UTC
```

## Changing the Time Zone in Linux

Before changing the time zone, you need to find out the long name for the time
zone you want to use. The time zone naming convention usually follows the
`Region/City` format.

To list all available time zones, you can use the `timedatectl` command or list
the files in the `/usr/share/zoneinfo` directory:

```bash
timedatectl list-timezones
```

Example output:

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

Once you have identified the correct time zone for your location, run the
following command as a sudo user:

```bash
sudo timedatectl set-timezone <timezone>
```

For example, to change the system time zone to Jakarta local time:

```bash
sudo timedatectl set-timezone Asia/Jakarta
```

Run the `timedatectl` command to verify the change:

```bash
timedatectl
```

Example output:

```
                  Local time: Tue 2019-12-03 13:55:09 WIB
              Universal time: Tue 2019-12-03 18:55:09 UTC
                    RTC time: Tue 2019-12-03 18:02:16
                   Time zone: Asia/Jakarta (WIB, +0700)
     System clock synchronized: no
systemd-timesyncd.service active: yes
             RTC in local TZ: no
```

You have successfully changed your system's time zone.

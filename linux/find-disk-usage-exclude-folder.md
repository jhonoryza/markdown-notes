# Calculating Disk Usage While Excluding Specific Folders

Suppose you have a directory structure like this:

```
/home/user
    ├── folder1
    ├── folder2
    ├── excluded_folder1
    └── excluded_folder2
```

To calculate the disk usage of `/home/user` while excluding `excluded_folder1`
and `excluded_folder2`, you can use the following command:

```bash
du -sh --exclude='/home/user/excluded_folder1' --exclude='/home/user/excluded_folder2' /home/user
```

Alternatively, you can use the `find` command:

```bash
du -sh $(find /home/user -mindepth 1 -maxdepth 1 -not -path '/home/user/excluded_folder1' -not -path '/home/user/excluded_folder2')
```

These commands will give you the total size of the `/home/user` directory
excluding both `excluded_folder1` and `excluded_folder2`.

# Example of Using the zip Command

To create a zip archive of a folder while excluding specific subfolders, use the
following command:

```bash
zip -r archive.zip /path/to/source/folder -x "subfolder1/*" "subfolder2/*"
```

The command above will create an `archive.zip` file containing all files and
folders in `/path/to/source/folder` except `subfolder1` and `subfolder2`. The
original folder and its contents will not be deleted or modified.

If you want to zip only the folders that start with `sub` inside the `folder`
directory, use the following command:

```bash
zip -r archive.zip folder/sub*
```

This command will create a zip file named `archive.zip` containing all
directories inside `folder` that start with "sub".

## Explanation

- `-r` tells `zip` to include files and directories recursively.
- `archive.zip` is the name of the output zip file.
- `/path/to/source/folder` is the path to the source folder you want to zip.
- `-x "subfolder1/*" "subfolder2/*"` excludes the specified subfolders from the
  zip archive.
- `folder/sub*` is the wildcard pattern to match directories that start with
  `sub` inside the `folder` directory.

This way, only the directories that match the wildcard pattern will be included
in the zip file.

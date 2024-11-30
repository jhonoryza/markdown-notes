Example of using the zip command:

```
zip -r archive.zip /path/to/source/folder -x "subfolder1/*" "subfolder2/*"
```

The command above will create an archive.zip file containing all files and
folders in /path/to/source/folder except subfolder1 and subfolder2. The original
folder and its contents will not be deleted or modified

if you want to zip only the folders that start with "sub" inside the "folder"
directory:

```
zip -r archive.zip folder/sub*
```

This command will create a zip file named archive.zip containing all directories
inside folder that start with "sub".

#### Explanation:

- -r tells zip to include files and directories recursively.
- archive.zip is the name of the output zip file.
- folder/sub* is the wildcard pattern to match directories that start with "sub"
  inside the "folder" directory.

This way, only the directories that match the wildcard pattern will be included
in the zip file.

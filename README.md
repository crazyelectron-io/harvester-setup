# harvester-setup

## Effect of Permissions on Files

Permission |	Character |	Meaning on File
-----------|--------------|----------------
Read |	- |	The file is not readable. You cannot view the file contents.
Read|  r |	The file is readable.
Write |	- |	The file cannot be changed or modified.
Write | w |	The file can be changed or modified.
Execute	| - |	The file cannot be executed.
Execute | x |	The file can be executed.
Execute | s |	If found in the user triplet, it sets the setuid bit. If found in the group triplet, it sets the setgid bit. It also means that x flag is set. When the setuid or setgid flags are set on an executable file, the file is executed with the file’s owner and/or group privileges.
Execute | S |	Same as s, but the x flag is not set. This flag is rarely used on files.
Execute | t |	If found in the others triplet, it sets the sticky bit.
It also means that x flag is set. This flag is useless on files.
Execute | T |	Same as, t but the x flag is not set. This flag is useless on files.

## Effect of Permissions on Directories (Folders)

Directories are special types of files that can contain other files and directories.

Permission |	Character |	Meaning on Directory
-----------|--------------|---------------------
Read |	- |	The directory’s contents cannot be shown.
Read | r |	The directory’s contents can be shown.
(e.g., You can list files inside the directory with ls .)
Write |	- |	The directory’s contents cannot be altered.
Write | w |	The directory’s contents can be altered (e.g., You can create new files , delete files ..etc.).
Execute |	- |	The directory cannot be changed to.
Execute | x |	The directory can be navigated using cd .
Execute | s |	If found in the user triplet, it sets the setuid bit. If found in the group triplet it sets the setgid bit. It also means that x flag is set. When the setgid flag is set on a directory, the new files created within it inherits the directory group ID (GID) instead of the primary group ID of the user who created the file. setuid has no effect on directories.
Execute | S | 	Same as s, but the x flag is not set. This flag is useless on directories.
Execute | t |	If found in the others triplet, it sets the sticky bit. It also means that x flag is set. When the sticky bit is set on a directory, only the file’s owner, the directory’s owner, or the administrative user can delete or rename the files within the directory.
Execute | T |	Same as t, but the x flag is not set. This flag is useless on directories.

> Tip: Use the `--preserve-root` flag to prevent chmod from acting recursively on `/`. This can, for example, prevent one from removing the executable bit systemwide and thus breaking the system. To use this flag every time, set it within an **alias**.

## Numeric chmod flags

When the 4 digits number is used, the first digit has the following meaning:

    setuid=4
    setgid=2
    sticky=1
    no changes = 0

> Note: `chown` always clears the setuid and setgid bits.

You can check the file’s permissions in the numeric notation using the stat command:

```bash
stat -c "%a" file_name
```

You can view permissions along a path with:

```bash
namei -l path
```

Generally directories and files should not have the same permissions. If it is necessary to bulk modify a directory tree, use find to selectively modify one or the other.

To chmod only directories to 755:

```bash
find directory -type d -exec chmod 755 {} +
```

To chmod only files to 644:

```bash
find directory -type f -exec chmod 644 {} +
```

## Appying defaultpermissions

Set the setgid bit, so that files/folder under <directory> will be created with the same group as <directory>

chmod g+s <directory>

Set the default ACLs for the group and other

setfacl -d -m g::rwx /<directory>
setfacl -d -m o::rx /<directory>

Next we can verify:

getfacl /<directory>

Example output:

# file: ../<directory>/
# owner: <user>
# group: media
# flags: -s-
user::rwx
group::rwx
other::r-x
default:user::rwx
default:group::rwx
default:other::r-x

Using the default switch (-d) and the modify switch (-m) will only modify the default permissions but leave the existing ones intact:

setfacl -d -m g::rwx /<directory>

If you want to change folder's entire permission structure including the existing ones (you'll have to do an extra line and make it recursive with -R):

setfacl -R -m g::rwx /<directory>

Examples:

# Gives group read,write,exec permissions for currently existing files and
# folders, recursively.
setfacl -R -m g::rwx /home/limited.users/directory 

# Revokes read and write permission for everyone else in existing folder and
# subfolders.
setfacl -R -m o::x /home/limited.users/directory  

# Gives group rwx permissions by default, recursively.
setfacl -R -d -m g::rwx /home/limited.users/directory

# Revokes read, write and execute permissions for everyone else. 
setfacl -R -d -m o::--- /home/limited.users/directory




## Umask

The umask utility is used to control the file-creation mode mask, which determines the initial value of file permission bits for newly created files.
The umask command uses the following syntax:

```bash
umask [-p] [-S] [mask]
```

- `[mask]`: The new permissions mask you are applying. By default, the mask is presented as a numeric (octal) value.
- `[-S]`: Displays the current mask as a symbolic value.
- `[-p]`: Displays the current mask along with the umask command, allowing it to be copied and pasted as a future input.

Linux uses the following default mask and permission values:

- The system default permission values are 777 (rwxrwxrwx) for folders and 666 (rw-rw-rw-) for files.
- The default mask for a non-root user is 002, changing the folder permissions to 775 (rwxrwxr-x), and file permissions to 664 (rw-rw-r--).
- The default mask for a root user us 022, changing the folder permissions to 755 (rwxr-xr-x), and file permissions to 644 (rw-r--r--).

This shows us that the final permission value is the result of subtracting the umask value form the default permission value (777 or 666).
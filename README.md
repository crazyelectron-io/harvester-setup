# harvester-setup

## Effect of permissions on files

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
Execute | t |	If found in the others triplet, it sets the sticky bit. It also means that x flag is set. This flag is useless on files.
Execute | T |	Same as, t but the x flag is not set. This flag is useless on files.

## Effect of permissions on directories

Directories are special types of files that can contain other files and directories.

Permission |	Character |	Meaning on Directory
-----------|--------------|---------------------
Read |	- |	The directory’s contents cannot be shown.
Read | r |	The directory’s contents can be shown (e.g., You can list files inside the directory with ls .).
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

Generally directories and files should not have the same permissions.
If it is necessary to bulk modify a directory tree, use find to selectively modify one or the other.

To `chmod` only directories to 755:

```bash
find <directory> -type d -exec chmod 755 {} +
```

To `chmod` only files to 644:

```bash
find <directory> -type f -exec chmod 644 {} +
```

## Appying default permissions

Set the _setgid_ bit, so that files/folder under 'directory' will be created with the same group as 'directory':

```bash
chmod g+s <directory>
```

## Default ownership and permissions

When you create a new file, the ownership is determined by the user and group of the creating process.
To see your current user and group, use the `id` command. Permissions are determined by your `umask`.
Keep in mind that setgid on a directory will cause new files to be owned by directory’s group.
The permissions on a newly created file depend on your umask. A umask is a 4 digit number that looks like the numeric version of a file’s mode. The `umask` command will tell you what yours is.

## Umask

The `umask` utility is used to control the file-creation mode mask, which determines the initial value of file permission bits for newly created files.
The `umask` command uses the following syntax:

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

## ACLs

Set the default ACLs for the group and other:

```bash
setfacl -d -m g::rwx /<directory>
setfacl -d -m o::rx /<directory>
```

Next we can verify:

```bash
$ getfacl /<directory>
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
```

Using the default switch (-d) and the modify switch (-m) will only modify the default permissions but leave the existing ones intact:

```bash
setfacl -d -m g::rwx /<directory>
```

If you want to change folder's entire permission structure including the existing ones (you'll have to do an extra line and make it recursive with -R):

```bash
setfacl -R -m g::rwx /<directory>
```

Examples:

```bash
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
```

### Identifying files with ACLs

The `-l` option of `ls` will show you if you have an ACL or not.

```bash
$ ls -l
total 12
drwxr-xr-x+ 3 tyler tyler 4096 Feb 19 17:13 default
drwxr-xr-x  2 tyler tyler 4096 Feb 19 19:30 ldap
drwxr--r-x+ 3 tyler tyler 4096 Feb 19 17:16 mask
```

Permission strings with a `+` at the end indicate the file in question has an ACL.
Default ACLs only apply to directories.
These are the ACLs entries that are automatically added to any new file or directory created inside of it.
ACL entries have precedence over the other permission set.

### Adding and modifying ACL entries

To add or change an ACL entry, use `setfacl -m`, followed by the ACL specification, and then a list of files you wish to apply the changes to.

```bash
# setfacl -m u:tyler:rwx myfile
# getfacl myfile
# file: myfile
# owner: root
# group: root
user::rw-
user:tyler:rwx
group::r--
mask::rwx
other::r--
```

The ACL specification consists of 2-4 fields separated by :. Specifications for adding and changing user and group entries start with a single letter denoting whether it is a user or group. The characters u and g correspond to user and group, respectively. The next field is the name of the user or group. The last field is used to specify the permissions that you wish for the entry to have. If an entry for the applicable group or user exists, any permissions granted by the existing entry are replaced by the permissions in the final part of the specification.

### Modifying default ACLs

When working with default ACLs, all of the above applies with a single exception. Just add d: to the beginning of the ACL specification. Only directories may have default ACL entries.

```bash
# setfacl -m d:u:tyler:rwx acl_example
# getfacl acl_example
# file: acl_example
# owner: root
# group: root
user::rwx
group::r-x
other::r-x
default:user::rwx
default:user:tyler:rwx
default:group::r-x
default:mask::rwx
default:other::r-x
```

### Removing ACL entries

Use `setfacl -x` to remove an ACL entry. When removing entries, omit the permission string:

```bash
# setfacl -x g:openldap myfile 
# getfacl myfile 
# file: myfile
# owner: root
# group: root
user::rw-
user:tyler:r--
group::r--
mask::r--
other::---
```

As with regular entries, `setfacl -x` can be used to remove the mask or default entries.

```bash
$ setfacl -x m dir
$ setfacl -x d:m dir
$ setfacl -x d:g:openldap dir
```

Use the `-b` option to remove all ACL entries, masks, and defaults:

```bash
$ getfacl default
# file: default
# owner: tyler
# group: tyler
user::rwx
group::r-x
other::r-x
default:user::rwx
default:user:openldap:r--
default:group::r-x
default:mask::r-x
default:other::r-x

$ setfacl -b default
$ getfacl default
# file: default
# owner: tyler
# group: tyler
user::rwx
group::r-x
other::r-x
```

### Common setfacl Options

OPTION |	DESCRIPTION
---|---
-b |	Removes all ACL entries from a file.
-n |	Prevents the mask from being modified.
-R |	Recursively applies changes to a directory and everything in it.
-m |	Modify or add an ACL entry.
-x |	Removes an ACL entry.

### ACLs on ZFS pools and datasets

sudo zfs set aclinherit=passthrough hddpool/media
sudo zfs set acltype=posixacl hddpool/media
sudo zfs set xattr=sa hddpool/media



### How default ACLs work

Setting default ACL entries on directories causes new files and directories created within it to have the entries listed as default. Newly created directories inherit default entries as well.

I have used default ACLs on Samba and NFS shares where users vary in the level access they should have.

```bash
# setfacl -m d:u:tyler:rwx acl_example
# getfacl acl_example
# file: acl_example
# owner: root
# group: root
user::rwx
group::r-x
other::r-x
default:user::rwx
default:user:tyler:rwx
default:group::r-x
default:mask::rwx
default:other::r-x

# cd acl_example
# touch file
# getfacl file
# file: file
# owner: root
# group: root
user::rw-
user:tyler:rwx			#effective:rw-
group::r-x	 		    #effective:r--
mask::rw-
other::r--

# mkdir new_dir
# getfacl new_dir
# file: new_dir
# owner: root
# group: root
user::rwx
user:tyler:rwx
group::r-x
mask::rwx
other::r-x
default:user::rwx
default:user:tyler:rwx
default:group::r-x
default:mask::rwx
default:other::r-x
```

As you can see the file file inherited the ACL entry for the user tyler from acl_example. The directory new_dir inherited both the default and regular entry for the tyler user.

Directories can have default masks as well:

```bash
$ setfacl -m d:m:r-x default
$ getfacl default
# file: default
# owner: tyler
# group: tyler
user::rwx
group::r-x
other::r-x
default:user::rwx
default:group::r-x
default:mask::r-x
default:other::r-x
```

### ACL examples

COMMAND |	DESCRIPTION
---|---
setfacl -m u:tyler:rwx file	| Adds the user tyler with read, write, and execute
setfacl -b file	| Removes all ACL entries, defaults, and masks from file
setfacl -x g:openldap file	| Removes the openldap group ACL entry
setfacl -m d:u:tyler:r-x directory	| Adds a default ACL entry allowing the user tyler to read and execute files created within the directory
setfacl -R m:r-x directory	| Sets the mask of the directory and everything in it to read and execute
getfacl template \| setfacl --set-file=- hello.c	| Copies the ACL entries from the file template to the file hello.c
setfacl -m d:m:r-- directory	| Sets a default mask of read only

## Setup SABnzbd directory permissions

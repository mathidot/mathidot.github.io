---
layout : post
titile : Understanding Linux File Permission
tags : Missing Semester
---

# Understanding Linux File Permission

In Linux, file permissions, attributes, and ownership control the access level that the system processes and users have to files. This ensures that only authorized users and process can access specific files and directories.

## Linux File Permissions

The basic Linux permissions models works by associating each system file with an owner and a group and assigning permissions access rights for three different classes of users:

* The file owner
* The group members
* Others(everybody else)

File ownership can be changed using the [`chown`](https://linuxize.com/post/linux-chown-command/) and [`chgrp`](https://linuxize.com/post/chgrp-command-in-linux/) commands.

Three files permissions types apply to  each class of users:

* The read permission
* The write permission
* The execute permission

This concept allows u to control which users can read the file, write to the file, or execute the file.

To view the file permissions, use the [`ls`](https://linuxize.com/post/how-to-list-files-in-linux-using-the-ls-command/) command:

```shell
ls -l file_name
```

```shell
-rw-r--r-- 12 linuxize users 12.0K Apr  28 10:10 file_name
|[-][-][-]-   [------] [---]
| |  |  | |      |       |
| |  |  | |      |       +-----------> 7. Group
| |  |  | |      +-------------------> 6. Owner
| |  |  | +--------------------------> 5. Alternate Access Method
| |  |  +----------------------------> 4. Others Permissions
| |  +-------------------------------> 3. Group Permissions
| +----------------------------------> 2. Owner Permissions
+------------------------------------> 1. File Type

```

The first character indicates the file type. It can be a regular file (-), directory (d), a symbolic link (l), or other special types of files. The following nine characters represent the file permissions, three triplets of three characters each. The first triplet shows the owner permissions, the second one group permissions, and the last triplet shows everybody else permissions.

In the example above (`rw-r--r--`) means that the file owner has read and write permissions (`rw-`), the group and others have only read permissions (`r--`).

File permissions have a different meaning depending on the file type.

Each of the three permission triplets can be constructed of the following characters and have different effects, depending on whether they are set to a file or to a directory:

**Effect of Permissions on Files**

| Permission  | Character | Meaning on File                                              |
| :---------: | :-------: | :----------------------------------------------------------- |
|  **Read**   |     -     | The file is not readable.                                    |
|             |     r     | The file is readable                                         |
|  **Write**  |     -     | The file cannot be changed or modified.                      |
|             |     w     | The file can be changed or modified.                         |
| **Execute** |     -     | The file cannot be executed.                                 |
|             |     x     | The file can be executed.                                    |
|             |     s     | If found in the `user` triplet, it sets the `setuid` bit. If found in the `group` triplet, it sets the `setgid` bit. It also means that `x` flag is set. When the `setuid` or `setgid` flags are set on an executable file, the file is executed with the file’s owner and/or group privileges. |
|             |     S     | Same as `s`, but the `x` flag is not set. This flag is rarely used on files. |
|             |     t     | If found in the `others` triplet, it sets the `sticky` bit. It also means that `x` flag is set. This flag is useless on files. |
|             |     T     | Same as, `t` but the `x` flag is not set. This flag is useless on files. |

## Changing File permissions

The File permissions can be changed using the `chmod` command. Only root, the file owner, or user with sudo privileges can change the permissions of a file. Be extra careful when using `chmod`, especially when recursively changing the permissions. The command can accept one or more files and/or directories separated by space as arguments.






































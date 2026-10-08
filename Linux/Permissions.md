## 1. File & Directory Permissions

Each file or directory has specific permissions for three categories of users: **Owner (`u`)**, **Group (`g`)**, and **Others (`o`)**.

| Permission | Symbol | Octal value | Description |
| :--- | :---: | :---: | :--- |
| **Read** | `r` | `4` | Read file contents and list files inside a directory |
| **Write** | `w` | `2` | Modify or overwrite file and create, delete, or rename files in directory |
| **Execute** | `x` | `1` | Run an executable program and traverse directory path |

## Octal Notation
* **`7`** (`4+2+1`) = `rwx` -> Read, Write, Execute
* **`6`** (`4+2+0`) = `rw-` -> Read and Write
* **`5`** (`4+0+1`) = `r-x` -> Read and Execute
* **`4`** (`4+0+0`) = `r--` -> Read only
* **`0`** (`0+0+0`) = `---` -> No permissions

*Example:* `chmod 754 <file>` ---> User: `7` (`rwx`), Group: `5` (`r-x`), Others: `4` (`r--`).
### Common Permission Combinations
755 (rwxr-xr-x) - Executable files and directories:
- Owner can do anything
- Group and others can read and execute
- Common for scripts and programs

644 (rw-r--r--) - Regular files:
- Owner can read and write
- Group and others can only read
- Common for documents and configuration files

600 (rw-------) - Private files:
- Only owner can read and write
- Nobody else has any access
- Common for SSH keys and sensitive data

777 (rwxrwxrwx) - Full access for everyone:
- Everyone can do anything
- Generally considered insecure
- Avoid unless absolutely necessary
## Changing Permissions (chmod)
```bash
chmod 755 <file>      # Set rwxr-xr-x
chmod 600 <file>         # Set rw-------
chmod u+x,g-w <file>   # Add execute for Owner, remove write for Group
```

## User & Group Management
| Command | Description |
| :--- | :--- |
| **`sudo`** | Execute a command as another user |
| **`su`** | Run a command with substitute user and group ID |
| **`useradd`** | Create a new user or update default new user information |
| **`userdel`** | Delete a user account and related files |
| **`usermod`** | Modify a user account |
| **`addgroup`** | Add a group to the system |
| **`delgroup`** | Remove a group from the system |
| **`passwd`** | Change user password |

## Changing Ownership (chown and chgrp)

```bash
sudo chown alice <file>         # Change Owner to 'alice'
sudo chown alice:devs <file>     # Change Owner to 'alice' and Group to 'devs'
sudo chown -R alice:devs /var/www/ # Recursive ownership change
```

## Special Permissions (SUID(s), SGID(s), Sticky Bit(t))
SetUID (SUID):
- When set on an executable file, the program runs with the permissions of the file's owner, not the user who executed it.

```bash
# Set SUID 
chmod u+s /usr/bin/passwd

# Or using octal 
chmod 4755 /usr/bin/passwd
```

SetGID (SGID):
- For files: The file runs with the permissions of the file's group.
- For directories: Files created within the directory inherit the directory's group, not the creator's primary group.
  
```bash
# Set SGID
chmod g+s <directory>  # or chmod g+s <file>

# Or using octal
chmod 2775 <directory> # or chmod 2755 <file>
```

Sticky Bit:
- When set on a directory, only the file's owner, the directory's owner, or root can delete or rename files within that directory.

```bash
# Set sticky bit
chmod +t /tmp
# Or using octal
chmod 1777 /tmp
```
## Default Permissions and umask
When you create a new file or directory, Linux applies default permissions based on the umask value.

The umask is a mask that subtracts permissions from the base defaults:
- Base default for files: 666 (rw-rw-rw-)
- Base default for directories: 777 (rwxrwxrwx)
- The umask value is subtracted from these base permissions.

```bash
# View current umask
umask

# View in symbolic notation
umask -S

# Set umask
umask 0022
```
Common umask values:

 0022 - Default on many systems:

 - Files created with 644 (rw-r--r--): 666 - 022 = 644
 - Directories created with 755 (rwxr-xr-x): 777 - 022 = 755

 0002 - Common for shared environments:
 - Files created with 664 (rw-rw-r--): 666 - 002 = 664
 - Directories created with 775 (rwxrwxr-x): 777 - 002 = 775

 0077 - Restrictive (private files)_
 - Files created with 600 (rw-------): 666 - 077 = 600
 - Directories created with 700 (rwx------): 777 - 077 = 700
To make the umask permanent, add it to your shell's configuration file (~/.bashrc or ~/.zshrc).







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



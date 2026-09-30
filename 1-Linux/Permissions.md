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

##  User & Group Management
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

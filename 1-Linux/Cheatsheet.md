# Cheatsheet

| Command | Usage  | Description |
| :--- | :--- | :--- |
| `whoami` | `whoami` | Print effective username. |
| `id` | `id` / `id <user>` | Print user and group IDs. |
| `hostname` | `hostname` | Show or set system hostname. |
| `uname` | `uname -a` | Print system and kernel information. |
| `pwd` | `pwd` | Print current working directory. |
| `ifconfig` | `ifconfig` / `ifconfig <iface>` | Display or configure network interfaces . |
| `ip` | `ip a` | Show or manage network devices and IP addresses. |
| `netstat` | `netstat -tunlp` | Display active connections and listening ports. |
| `ss` | `ss -tunlp` | Dump socket statistics (modern `netstat` replacement). |
| `ps` | `ps aux` | Display snapshot of active processes. |
| `who` | `who` | Show logged-in users. |
| `env` | `env` | Print environment variables or run a command in a modified environment. |
| `lsblk` | `lsblk` | List block devices and partitions. |
| `lsusb` | `lsusb` | List connected USB devices. |
| `lsof` | `lsof -i` | List open files and network sockets. |
| `lspci` | `lspci` | List PCI devices. |
| `cd` | `cd <dir>` | Change working directory. |
| `touch` | `touch <file>` | Create empty file or update timestamps. |
| `mkdir` | `mkdir <dir>` | Create new directory. |
| `mv` | `mv <src> <dst>` | Move or rename files and directories. |
| `cp` | `cp <src> <dst>` | Copy files and directories. |
| `which` | `which <cmd>` | Print full path of executable binary. |
| `find` | `find / -name <file> 2>/dev/null` | Search filesystem for files (suppress permission errors). |
| `locate` | `locate <file>` | Find files by name using indexed database. |
| `more` | `more <file>` | View file page-by-page (forward navigation only). |
| `less` | `less <file>` | View file page-by-page (bidirectional scroll). |
| `head` | `head <file>` | Output first 10 lines of file. |
| `tail` | `tail <file>` | Output last 10 lines of file. |
| `sort` | `sort <file>` | Sort lines of text files. |
| `grep` | `grep "pattern" <file>` | Filter lines matching pattern (use -v to invert match). |
| `tr` | `tr ":" " "` | Translate, squeeze, and/or delete characters from standard input, writing to standard output. |
| `column` | `column -t` | Format input into aligned columns. |
| `awk` | `awk '{print $1}' <file>` | Extract and process specific column fields. |
| `ping` | `ping <ip>` | Test network reachability using ICMP ECHO. |
| `traceroute` | `traceroute <ip>` | Trace packet route path to target host. |

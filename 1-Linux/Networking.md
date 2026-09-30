# Linux Network Configuration & Security

## Network Configuration 

Modern Linux distributions use the `iproute2` suite (`ip`) to replace legacy networking utilities (`ifconfig`, `route`).

| Action | Legacy Command (`ifconfig` / `route`) | Modern Command (`ip`) |
| :--- | :--- | :--- |
| **Display IP & Interfaces** | `ifconfig` | `ip addr` |
| **Enable Interface** | `sudo ifconfig eth0 up` | `sudo ip link set eth0 up` |
| **Assign IP & Subnet** | `sudo ifconfig eth0 192.168.1.2 netmask 255.255.255.0` | `sudo ip addr add 192.168.1.2/24 dev eth0` |
| **Set Default Gateway** | `sudo route add default gw 192.168.1.1 eth0` | `sudo ip route add default via 192.168.1.1` |

## DNS Configuration (`/etc/resolv.conf`)

DNS servers translate domain names to IP addresses and are set in `/etc/resolv.conf`:

```bash
nameserver 8.8.8.8
nameserver 8.8.4.4
```
> Note:
Direct edits to /etc/resolv.conf are non-persistent after a reboot because the file can be overwritten automatically by services like NetworkManager or systemd-resolved. To make changes persistent, edit /etc/network/interfaces and restart the networking service:
>

| Protocol / Service |  Port | Description |
| :--- | :---: | :--- |
| **RDP** | `TCP 3389` | Remote Desktop Protocol (Windows GUI) |
| **VNC** | `TCP 5900+` | Virtual Network Computing (Cross-platform GUI) |
| **X11 / XServer** | `TCP 6000-6010` | Native Linux/Unix graphical display system |
| **XDMCP** | `UDP 177` | Remote GUI management, unencrypted |

## Linux Security and Hardening

System and packets updates:

```bash
  sudo apt update && sudo apt dist-upgrade -y
  ```
SSH configuration(/etc/ssh/sshd_config):

```bash
  PermitRootLogin no          # Disables direct root login via SSH
  PasswordAuthentication no   # Disables password authentication (forces SSH keys)
  ```

Brute-Force Protection (`fail2ban`):
- Monitors failed login attempts and temporarily bans IP addresses that exceed the maximum threshold.

TCP Wrappers (`tcpd`):
- Software security system for Linux and Unix acting as a filter to control access to network services.

* **`/etc/hosts.allow` (Authorized network services):**

  ```bash
  sshd : 10.129.14.0/24   # Authorizes SSH access for the subnet
  ftpd : 10.129.14.10     # Authorizes FTP access for an IP
  ```
* **`/etc/hosts.deny` (Blocked network services):**

 ```bash
  ALL  : .example.com     # Block ALL network services from this domain
  sshd : 10.129.22.22     # Block SSH service for this specific IP
  ```

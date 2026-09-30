# Linux Network Configuration & Security

## 1. Network Configuration 

Modern Linux distributions use the `iproute2` suite (`ip`) to replace legacy networking utilities (`ifconfig`, `route`).

| Action | Legacy Command (`ifconfig` / `route`) | Modern Command (`ip`) |
| :--- | :--- | :--- |
| **Display IP & Interfaces** | `ifconfig` | `ip addr` |
| **Enable Interface** | `sudo ifconfig eth0 up` | `sudo ip link set eth0 up` |
| **Assign IP & Subnet** | `sudo ifconfig eth0 192.168.1.2 netmask 255.255.255.0` | `sudo ip addr add 192.168.1.2/24 dev eth0` |
| **Set Default Gateway** | `sudo route add default gw 192.168.1.1 eth0` | `sudo ip route add default via 192.168.1.1` |

## 2. DNS Configuration (`/etc/resolv.conf`)

DNS servers translate domain names to IP addresses and are set in `/etc/resolv.conf`:

```bash
nameserver 8.8.8.8
nameserver 8.8.4.4
```
> Note:
Direct edits to /etc/resolv.conf are non-persistent after a reboot because the file can be overwritten automatically by services like NetworkManager or systemd-resolved. To make changes persistent, edit /etc/network/interfaces and restart the networking service:
> 

# Nmap

> Official Manual: `man nmap` | [nmap.org](https://nmap.org)

## Common use cases for Nmap

Nmap ("Network Mapper") is a free and open source utility for network discovery and security auditing. Many systems and network administrators also find it useful for tasks such as network inventory, managing service upgrade schedules, and monitoring host or service uptime. Nmap uses raw IP packets in novel ways to determine what hosts are available on the network, what services (application name and version) those hosts are offering, what operating systems (and OS versions) they are running. It was designed to rapidly scan large networks, but works fine against single hosts.


## Host Discovery

### Host Discovery Defaults (No options specified)
- **Privileged (sudo/root):** Sends ICMP Echo, TCP SYN (443), TCP ACK (80), and ICMP Timestamp (`-PE -PS443 -PA80 -PP`).
- **Unprivileged (Non-root):** Sends TCP SYN to ports 80 and 443 using `connect()`.
- **Local LAN:** Automatically uses **ARP Ping** (IPv4) or **ND Ping** (IPv6).
  
| Flag | Function |
| :--- | :--- |
| `-sL` | List Scan - simply list targets to scan |
| `-sn` | Ping Scan - disable port scan |
| `-Pn` | Treat all hosts as online -- skip host discovery |
| `-PS/PA/PU/PY[portlist]` | TCP SYN, TCP ACK, UDP or SCTP discovery to given ports |
| `-PE/PP/PM` | ICMP echo, timestamp, and netmask request discovery probes |
| `-PO[protocol list]` | IP Protocol Ping |
| `-n`/`-R` | Never do DNS resolution/Always resolve |
| `--disable-arp-ping` | No ARP or ND Ping |
| `--traceroute` | Trace hop path to each host |

```bash
# Scan targets within a subnet
sudo nmap -sn 10.129.2.0/24 

# Scan targets from a file
sudo nmap -sn -iL targets.txt 

# Scan IP ranges or multiple specific IPs
sudo nmap -sn 10.129.2.14-19
sudo nmap -sn 10.129.2.18 10.129.2.19 10.129.2.21

# Force ICMP ping on local LAN (bypassing default ARP ping)
sudo nmap -sn -PE --disable-arp-ping 192.168.1.50 


# Skip host discovery (treat all host as online)
sudo nmap -Pn 10.129.2.18


# List Scan (list targets without sending packets)
nmap -sL 10.129.2.0/24
```



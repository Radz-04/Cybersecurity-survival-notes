# Nmap

> Official Manual: `man nmap` | [nmap.org](https://nmap.org)

## Common use cases for Nmap

Nmap ("Network Mapper") is a free and open source utility for network discovery and security auditing.

It uses **raw IP packets** to determine:
* What hosts are available on the network (**Host Discovery**).
* What services, application names, and versions hosts are offering (**Service & Version Detection**).
* What operating systems and OS versions they are running (**OS Detection**).
* What type of packet filters, firewalls, or Intrusion Detection Systems (IDS) are in use.

## Cheatsheet

```bash
nmap <scan-types> <options> <target>
```

```bash
sudo nmap -sn 10.129.2.0/24 

# Scan targets from a file
sudo nmap -sn -iL targets.txt 

# Scan IP ranges or multiple specific IPs
sudo nmap -sn 10.129.2.18-20
sudo nmap -sn 10.129.2.18 10.129.2.19 10.129.2.20

# Force ICMP ping on local LAN (bypassing default ARP ping)
sudo nmap -sn -PE --disable-arp-ping 192.168.1.50 


# Skip host discovery (treat all host as online)
sudo nmap -Pn 10.129.2.18


# List Scan (list targets without sending packets)
nmap -sL 10.129.2.0/24
```


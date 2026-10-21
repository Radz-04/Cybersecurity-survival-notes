# Nmap

> Official Manual: `man nmap` | [nmap.org](https://nmap.org)


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


## Port Scanning

> While Nmap has grown in functionality over the years, it began as an efficient port scanner, and that remains its core function. The simple command `nmap target` scans 1,000 TCP ports on the host target. While many port scanners have traditionally lumped all ports into the open or closed states, Nmap is much more granular. It divides ports into six states: open, closed, filtered, unfiltered, open|filtered, or closed|filtered.

### Port States

| State | Official Definition |
| :--- | :--- |
| `open` | An application is actively accepting TCP connections, UDP datagrams or SCTP associations on this port. |
| `closed` | A closed port is accessible (it receives and responds to Nmap probe packets), but there is no application listening on it. |
| `filtered` | Nmap cannot determine whether the port is open because packet filtering prevents its probes from reaching the port. |
| `unfiltered` | The unfiltered state means that a port is accessible, but Nmap is unable to determine whether it is open or closed. |
| `open\|filtered` | Nmap places ports in this state when it is unable to determine whether a port is open or filtered. |
| `closed\|filtered` | This state is used when Nmap is unable to determine whether a port is closed or filtered. |

### Scan Techniques

| Flag | Function |
| :--- | :--- |
| `-sS` | TCP SYN scan |
| `-sT` | TCP connect scan |
| `-sU` | UDP scan |
| `-sY` | SCTP INIT scan |
| `-sN` | TCP Null scan |
| `-sF` | TCP FIN scan |
| `-sX` | TCP Xmas scan |
| `-sA` | TCP ACK scan |
| `-sW` | TCP Window scan |
| `-sM` | TCP Maimon scan |
| `--scanflags` | Custom TCP scan |
| `-sZ` | SCTP COOKIE ECHO scan |
| `-sI <zombie host>` | Idle scan |
| `-sO` | IP protocol scan |
| `-b <FTP relay host>` | FTP bounce scan |

### Port Specification and Scan Order

| Flag | Function |
| :--- | :--- |
| `-p <port ranges>` | Only scan specified ports |
| `--exclude-ports <port ranges>` | Exclude the specified ports from scanning |
| `-F` | Fast (limited port) scan |
| `-r` | Don't randomize ports |
| `--top-ports <n>` | Scan the `<n>` highest-ratio ports |
| `--port-ratio <ratio>` | Scan all ports in nmap-services file with ratio greater than specified |

```bash
# Default SYN scan on top 10 ports (Requires root/sudo)
sudo nmap 10.129.2.28 --top-ports=10

# TCP Connect scan on port range 1-1024 without host discovery
nmap -sT -p 1-1024 -Pn -n --disable-arp-ping 10.129.2.28

# Fast UDP scan on top 100 ports
sudo nmap -sU -F 10.129.2.28

# Explicit TCP and UDP port selection in a single command
sudo nmap -sS -sU -p U:53,161,T:22,80,443 10.129.2.28

# Sequential scan of all 65535 ports (disables port randomization)
sudo nmap -p- -r 10.129.2.28

# Exclude specific ports from scan
sudo nmap -p 1-1024 --exclude-ports 80,443 10.129.2.28

# Map firewall statefulness using ACK scan
sudo nmap -sA -p 22,80,443,3389 10.129.2.28

# Null, FIN, and Xmas scans
sudo nmap -sN -p 22,80 10.129.2.28
sudo nmap -sF -p 22,80 10.129.2.28
sudo nmap -sX -p 22,80 10.129.2.28

# Trace raw packet exchange on port 21
sudo nmap -p 21 --packet-trace -Pn -n --disable-arp-ping 10.129.2.28
```

## Saving the results-different formats

- Normal output (-oN) with the .nmap file extension
- Grepable output (-oG) with the .gnmap file extension
- XML output (-oX) with the .xml file extension
- -oA Save the results in all formats

## Service and Version Detection

| Flag | Function |
| :--- | :--- |
| `-sV` | Version detection |
| `--allports` | Don't exclude any ports from version detection |
| `--version-intensity <level>` | Set version scan intensity (0 to 9, default 7) |
| `--version-light` | Enable light mode (alias for `--version-intensity 2`) |
| `--version-all` | Try every single probe (alias for `--version-intensity 9`) |
| `--version-trace` | Trace version scan activity |

```bash
# Standard service version scan
sudo nmap -sV 10.129.2.28

# Fast version scan on specific ports
sudo nmap -sV --version-light -p 22,80,445 10.129.2.28

# High-intensity version detection on all ports without exclusion
sudo nmap -sV --version-all --allports -p- 10.129.2.28
```

## OS Detection

| Flag | Function |
| :--- | :--- |
| `-O` | Enable OS detection |
| `--osscan-limit` | Limit OS detection to promising targets |
| `--osscan-guess` / `--fuzzy` | Guess OS detection results |
| `--max-os-tries <number>` | Set the maximum number of OS detection tries against a target |

```bash
# Standard OS detection
sudo nmap -O 10.129.2.28

# Limit OS detection to targets with at least one open and one closed TCP port
sudo nmap -O --osscan-limit 10.129.2.28

# Force aggressive OS guessing when match is not perfect
sudo nmap -O --osscan-guess 10.129.2.28
```

## Nmap Scripting Engine (NSE)

> The Nmap Scripting Engine (NSE) is one of Nmap's most powerful and flexible features. It allows users to write (and share) simple scripts (using the Lua programming language) to automate a wide variety of networking tasks.

| Flag | Function |
| :--- | :--- |
| `-sC` | Performs a script scan using the default set of scripts. It is equivalent to `--script=default`. |
| `--script <filename>\|<category>\|<directory>\|<expression>[,...]` | Runs a script scan using the comma-separated list of filenames, script categories, and directories. |
| `--script-args <n1>=<v1>,<n2>={<n3>=<v3>},,n4={v4,v5}` | Lets you provide arguments to NSE scripts. |
| `--script-args-file <filename>` | Lets you load arguments to NSE scripts from a file. |
| `--script-help <filename>|<category>|<directory>\|<expression>|all[,...]` | Shows help about scripts. |
| `--script-trace` | If this option is specified all incoming and outgoing communication performed by a script is printed. |
| `--script-updatedb` | This option updates the script database found in `scripts/script.db`. |

```bash
# Loads all scripts whose name starts with http-
nmap --script "http-*"

# Loads every script except for those in the intrusive category.
nmap --script "not intrusive"

# Loads all scripts that are in the default category or the safe category or both.
nmap --script "default or safe"

# Loads those scripts that are in both the default and safe categories.
nmap --script "default and safe"

# Loads scripts in the default, safe, or intrusive categories, except for those whose names start with http-.
nmap --script "(default or safe or intrusive) and not http-*"

# A complex example of script arguments is
nmap --script-args 'user=foo,pass=",{}=bar",whois={whodb=nofollow+ripe},xmpp-info.server_name=localhost'

# For example if you want help about the ftp-anon script, you would run
nmap --script-help ftp-anon

# Preview of what scripts will be run for a specification, for example with
nmap --script-help default

# This option is generally used by itself:
nmap --script-updatedb
```

# Enumeration

Active service interrogation, protocol-specific enumeration, Nmap Scripting Engine (NSE), and auxiliary scanning techniques.

**Author:** Radzivon Ferrigni Dioguardi

## Contents

1. [Enumeration scope and methodology](#1-enumeration-scope-and-methodology)
2. [Staged scanning and workspace preservation](#2-staged-scanning-and-workspace-preservation)
3. [Nmap Scripting Engine (NSE)](#3-nmap-scripting-engine-nse)
4. [Network scanning with auxiliary modules](#4-network-scanning-with-auxiliary-modules)
5. [FTP enumeration (TCP/21)](#5-ftp-enumeration-tcp21)
6. [SSH enumeration (TCP/22)](#6-ssh-enumeration-tcp22)
7. [SMTP enumeration (TCP/25, 465, 587)](#7-smtp-enumeration-tcp25-465-587)
8. [DNS service enumeration (TCP/UDP 53)](#8-dns-service-enumeration-tcpudp-53)
9. [HTTP and HTTPS service enumeration (TCP/80, 443, 8080)](#9-http-and-https-service-enumeration-tcp80-443-8080)
10. [SMB and NetBIOS enumeration (TCP/139, 445)](#10-smb-and-netbios-enumeration-tcp139-445)
11. [SNMP enumeration (UDP/161)](#11-snmp-enumeration-udp161)
12. [MySQL enumeration (TCP/3306)](#12-mysql-enumeration-tcp3306)
13. [RDP and WinRM enumeration (TCP/3389, 5985, 5986)](#13-rdp-and-winrm-enumeration-tcp3389-5985-5986)
14. [Additional network services quick reference](#14-additional-network-services-quick-reference)
15. [Technical references](#15-technical-references)

## Command conventions

Commands use Linux shell syntax unless a Windows command is explicitly labeled. Replace these documentation placeholders with targets **inside the authorized scope**. The examples show command usage and do not claim that any scan or enumeration activity has been executed.

```bash
TARGET='192.0.2.10'
NETWORK='192.0.2.0/24'
DOMAIN='example.com'
URL="http://$TARGET"
USERS='/usr/share/wordlists/usernames.txt'
PASSWORDS='/usr/share/wordlists/passwords.txt'
mkdir -p enum
```

Run tools with `--version` or `--help` to verify installed paths and flag compatibility, as option syntax and wordlist locations vary across Linux distributions. IPv4 blocks reserved for documentation (`192.0.2.0/24`, RFC 5737) and reserved domain names (`example.com`, RFC 2606) are used strictly as placeholders and must be replaced with authorized target assets.

## 1. Enumeration scope and methodology

Enumeration follows host discovery and port scanning. While port scanning identifies open transport endpoints and basic service banners, enumeration establishes active, protocol-aware connections to extract structured configuration data, identities, resources, and attack surfaces from exposed services.

An open port or version string does not by itself prove the existence of an exploitable vulnerability. Reliable enumeration preserves both the exact probes executed and the raw responses returned by the service.

| Enumeration domain | Primary interrogation method | Examples of collectible evidence | Operational limitation |
| --- | --- | --- | --- |
| Identity and accounts | Protocol queries (`VRFY`, `RCPT`, `RID` cycling, LDAP/SMB). | Local or domain usernames, email addresses, group memberships, password lockout policies. | Account lockout thresholds may be triggered by repeated verification or authentication probes. |
| Shares and storage | Unauthenticated or guest protocol sessions (SMB, NFS, FTP, DB). | Network shares, anonymous FTP directories, exported filesystems, database schemas. | Directory listings may hide unlinked paths or enforce per-user access controls. |
| Service configuration | Banner grabbing, handshake inspection, and NSE/RPC probes. | Daemon versions, supported ciphers, authentication mechanisms, RPC endpoints. | Banners can be modified, stripped, or proxied by load balancers and reverse proxies. |
| Application and web endpoints | HTTP header inspection, metadata requests, and controlled fuzzing. | Virtual hosts, hidden routes, API endpoints, TLS certificate SANs, framework fingerprints. | WAFs, CDNs, and rate-limiting mechanisms may throttle probes or return deceptive responses. |
| Naming and trust relationships | Zone transfers (`AXFR`), LDAP base queries, and broadcast inspection. | Internal DNS zones, Active Directory domain/forest details, signing requirements. | Detailed directory objects often require low-privilege domain credentials or local segment access. |
| Network topology | Routing table queries (SNMP), traceroute, and pivot scans. | Routing tables, internal hostnames, adjacent subnets, dual-homed interfaces. | Visibility depends strictly on the scanner's network position and firewall rules. |

### Structured workflow

1. **Confirm open endpoints** across all 65,535 TCP ports and high-value UDP ports before launching protocol-specific probes.
2. **Fingerprint daemon versions** on confirmed open ports only, avoiding unnecessary traffic on closed or filtered endpoints.
3. **Execute safe, non-intrusive scripts** and manual banner grabs to inspect default configurations and unauthenticated access (e.g., anonymous FTP, SMB null sessions, SNMP public communities).
4. **Enumerate resources and identities** (shares, virtual hosts, directories, RPC interfaces, valid accounts) while respecting scope and lockout limits.
5. **Preserve structured outputs** (`-oA`, raw logs) so findings can be queried, correlated, and verified before moving to vulnerability analysis.

## 2. Staged scanning and workspace preservation

Running intrusive scripts across full port ranges on multiple hosts generates excessive network noise and delays results. Separating discovery from enumeration keeps traffic predictable and outputs organized.

```bash
# Stage 1: Fast discovery of all open TCP ports
sudo nmap -sS -p- --min-rate 5000 -T4 "$TARGET" -oN enum/tcp_all_ports.txt

# Stage 2: Version detection and default safe scripts on open ports only
sudo nmap -sC -sV -p 21,22,80,445 "$TARGET" -oA enum/tcp_services

# Stage 3: Targeted UDP sweep on common administrative services
sudo nmap -sU --top-ports 50 -T4 "$TARGET" -oN enum/udp_top50.txt
```

### Importing scan results into a structured database

Saving Nmap results in XML format (`-oX` or `-oA`) enables structured host and service tracking across multi-target engagements inside a local workspace.

```bash
msfconsole -q -x "workspace -a engagement_01; db_import enum/tcp_services.xml; hosts; services; exit"
```

## 3. Nmap Scripting Engine (NSE)

The Nmap Scripting Engine (NSE) uses Lua scripts (typically stored in `/usr/share/nmap/scripts/`) to automate service discovery, advanced version detection, misconfiguration checks, and vulnerability verification.

| NSE category | Execution behavior and operational risk |
| --- | --- |
| `default` | Executed automatically with `-sC` or `-A`; fast, low-noise, and generally safe against standard services. |
| `safe` | Written to avoid crashing services, exhausting resources, or modifying target state. |
| `discovery` | Actively queries services, directories, or registries to map additional network resources. |
| `version` | Extends `-sV` probes to identify specialized protocols and application builds. |
| `vuln` | Tests for specific known vulnerabilities; may trigger IDS/IPS alerts or destabilize legacy daemons. |
| `brute` | Performs automated credential guessing; must only be used when explicitly permitted by scope. |
| `intrusive` | High network noise or significant risk of service disruption; requires explicit authorization. |

```bash
# Locate local NSE scripts matching a protocol or keyword
ls /usr/share/nmap/scripts/ | grep -i smb

# Read script documentation, categories, and supported arguments
nmap --script-help "smb-enum-*"

# Execute targeted scripts with custom script arguments
sudo nmap -p 80 --script http-enum --script-args http-enum.basepath='/admin/' "$TARGET"
```

## 4. Network scanning with auxiliary modules

When direct raw-socket scanning with Nmap is constrained—such as when enumerating an internal subnet routed through an established pivot host or SOCKS proxy—application-layer TCP scanners and protocol-specific auxiliary modules provide controlled discovery.

```text
[Analyst Host] ---> [Pivot / Foothold Host] ---> [Internal Target Subnet]
```

| Objective | Auxiliary module | Technical behavior |
| --- | --- | --- |
| TCP port discovery | `auxiliary/scanner/portscan/tcp` | Establishes full TCP three-way handshakes; works reliably through SOCKS proxies and routed sessions. |
| SYN port discovery | `auxiliary/scanner/portscan/syn` | Sends raw SYN packets; requires direct Layer 2/3 access and raw socket privileges. |
| UDP service discovery | `auxiliary/scanner/discovery/udp_sweep` | Sends protocol-specific UDP payloads (DNS, SNMP, NTP, NetBIOS) to identify active daemons. |
| Service fingerprinting | `auxiliary/scanner/<proto>/<proto>_version` | Connects to open ports across CIDR ranges to extract banners and software versions. |

When scanning across tunneled or high-latency links, keep concurrency (`THREADS`) low (e.g., `1` to `5`) and increase connection timeouts to avoid socket exhaustion and false negatives.

## 5. FTP enumeration (TCP/21)

File Transfer Protocol (FTP) operates over TCP port 21 for control commands and TCP port 20 (active mode) or ephemeral high ports (passive mode) for data transfers. Misconfigured servers frequently permit unauthenticated access or share directories with web servers.

### Verification checklist

* Identify the exact FTP server software and version (`vsftpd`, `ProFTPD`, `Pure-FTPd`, `Microsoft IIS FTP`).
* Test for **anonymous login** (`anonymous` / `anonymous` or `ftp` with an empty password).
* Check read and write permissions (`LIST`, `RETR`, `PUT`) inside accessible directories.
* Verify whether files uploaded via FTP are directly reachable over HTTP/HTTPS.

```bash
# Manual banner grab
echo "" | nc -nv "$TARGET" 21

# Automated NSE enumeration for anonymous access, system type, and bounce support
sudo nmap -p 21 -sV --script "ftp-anon,ftp-syst,ftp-bounce" "$TARGET"

# Interactive client connection
ftp "$TARGET"
```

## 6. SSH enumeration (TCP/22)

Secure Shell (SSH) provides encrypted remote administration over TCP port 22. Enumeration focuses on identifying daemon versions, supported authentication methods, host keys, and legacy cryptographic algorithms.

### Verification checklist

* Extract the SSH banner (`SSH-2.0-OpenSSH_...`), which often indicates the exact Linux or BSD distribution release.
* Enumerate supported authentication methods (`publickey`, `password`, `keyboard-interactive`) to verify whether password authentication is enabled.
* Inspect supported key exchange (`kex`), encryption ciphers, and MAC algorithms for deprecated configurations.

```bash
# Manual banner grab
nc -nv "$TARGET" 22

# Enumerate cryptographic algorithms, authentication methods, and host keys
sudo nmap -p 22 --script "ssh2-enum-algos,ssh-auth-methods,ssh-hostkey" "$TARGET"

# Dedicated SSH configuration and cryptography audit (when installed)
ssh-audit "$TARGET"
```

## 7. SMTP enumeration (TCP/25, 465, 587)

Simple Mail Transfer Protocol (SMTP) manages email transmission and relay operations on TCP port 25 (cleartext/STARTTLS), 465 (SMTPS), and 587 (submission). Legacy or misconfigured MTAs can disclose valid system usernames or allow unauthenticated mail relaying.

| SMTP command | Protocol function and enumeration value |
| --- | --- |
| `EHLO` / `HELO` | Initiates the SMTP session and returns supported extensions (`VRFY`, `EXPN`, `STARTTLS`, `AUTH`). |
| `VRFY <user>` | Queries the server to verify whether a specific local username or mailbox exists. |
| `EXPN <list>` | Requests expansion of a mailing list or alias into underlying member addresses. |
| `RCPT TO:<user>` | Indicates valid vs. invalid local recipients during the envelope phase of a mail transaction. |

```bash
# Enumerate supported SMTP commands, local users, and open relay behavior
sudo nmap -p 25,465,587 --script "smtp-commands,smtp-enum-users,smtp-open-relay" "$TARGET"

# Dictionary-based user enumeration using VRFY
smtp-user-enum -M VRFY -U "$USERS" -t "$TARGET"

# Manual SMTP session check
nc -nv "$TARGET" 25
```

## 8. DNS service enumeration (TCP/UDP 53)

When a target exposes port 53 directly, it may serve internal DNS zones, allow unrestricted recursion, or permit unauthorized zone transfers (`AXFR`) over TCP.

```bash
# Query nameserver version string via CHAOS class record
dig CH TXT version.bind @"$TARGET"

# Attempt full zone transfer over TCP/53
dig axfr "$DOMAIN" @"$TARGET"

# Run Nmap DNS service scripts
sudo nmap -p 53 --script "dns-nsid,dns-zone-transfer,dns-recursion" --script-args dns-zone-transfer.domain="$DOMAIN" "$TARGET"
```

## 9. HTTP and HTTPS service enumeration (TCP/80, 443, 8080)

Web applications frequently represent the primary exposed surface on a host. Enumeration maps server headers, TLS certificates, virtual hosts, unlinked directories, and administrative interfaces.

### Verification checklist

* Inspect HTTP response headers (`Server`, `X-Powered-By`, `Set-Cookie`, `WWW-Authenticate`, CORS policies).
* Request standard metadata and disclosure files: `/robots.txt`, `/sitemap.xml`, `/.well-known/security.txt`.
* Identify technologies, frameworks, and CMS versions from headers, cookies, and HTML comments.
* Enumerate hidden paths, backup files (`.bak`, `.old`, `.zip`), and virtual hosts using controlled wordlists.

```bash
# Inspect HTTP response headers only
curl -sI "$URL"

# Run Nmap HTTP enumeration scripts
sudo nmap -p 80,443 -sV --script "http-title,http-headers,http-methods,http-robots.txt" "$TARGET"

# Fingerprint web technologies and server components
whatweb "$URL"

# Directory and file enumeration
gobuster dir -u "$URL" -w /usr/share/wordlists/dirb/common.txt -x php,txt,bak,html -o enum/gobuster_dir.txt

# Virtual host enumeration via Host header fuzzing
ffuf -u "$URL" -H "Host: FUZZ.$DOMAIN" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -fs 0
```

## 10. SMB and NetBIOS enumeration (TCP/139, 445)

Server Message Block (SMB) and Samba provide file, printer, and IPC sharing across Windows and Unix-like systems. SMB runs directly over TCP/445 and historically over NetBIOS session service on TCP/139 (with NetBIOS name/datagram services on UDP/137-138).

### Verification checklist

* Identify the OS version, domain/workgroup name, SMB dialect (`SMBv1`, `SMBv2`, `SMBv3`), and message signing status.
* Test for **null sessions** (unauthenticated IPC$ connections with empty credentials) and guest access.
* List shared folders and verify read/write permissions on non-default shares.
* Enumerate domain/local users, groups, RIDs, and password lockout policies before testing credentials.

```bash
# Enumerate OS metadata, shares, users, and SMB signing configuration
sudo nmap -p 139,445 --script "smb-os-discovery,smb-enum-shares,smb-enum-users,smb-security-mode" "$TARGET"

# List available SMB shares using a null session
smbclient -L "//$TARGET/" -N

# Connect interactively to an accessible share
smbclient "//$TARGET/share_name" -N

# Comprehensive SMB and RPC enumeration
enum4linux-ng -A "$TARGET" -oA enum/enum4linux
nxc smb "$TARGET" -u '' -p '' --shares --users --pass-pol
```

## 11. SNMP enumeration (UDP/161)

Simple Network Management Protocol (SNMP) exposes system metrics, network interfaces, routing tables, running processes, and installed software through Management Information Bases (MIBs). SNMPv1 and SNMPv2c transmit community strings in cleartext and frequently retain default read-only (`public`) or read-write (`private`) strings.

| Common OID subtree | Exposed system information |
| --- | --- |
| `1.3.6.1.2.1.1` | System description, hostname, uptime, contact, and location (`system`). |
| `1.3.6.1.2.1.4.20` | IPv4 addresses and subnet masks assigned to local interfaces. |
| `1.3.6.1.2.1.25.1.6` | Count of running system processes. |
| `1.3.6.1.2.1.25.4.2.1.2` | Names and paths of running processes (frequently exposes CLI arguments). |
| `1.3.6.1.2.1.25.6.3.1.2` | Installed software packages and versions. |

```bash
# Identify valid community strings
onesixtyone -c /usr/share/seclists/Discovery/SNMP/common-snmp-community-strings.txt "$TARGET"

# Query the full accessible MIB tree with SNMPv2c
snmpwalk -v2c -c public "$TARGET" > enum/snmpwalk_public.txt

# Query running processes specifically
snmpwalk -v2c -c public "$TARGET" 1.3.6.1.2.1.25.4.2.1.2
```

## 12. MySQL enumeration (TCP/3306)

MySQL relational database servers listen on TCP port 3306 by default. Exposed instances may reveal exact server versions, authentication plugins, or permit unauthenticated `root` access.

```bash
# Enumerate MySQL protocol version, configuration variables, and empty-password accounts
sudo nmap -p 3306 -sV --script "mysql-info,mysql-empty-password,mysql-variables" "$TARGET"

# Test direct client connection
mysql -h "$TARGET" -u root --ssl-mode=DISABLED
```

## 13. RDP and WinRM enumeration (TCP/3389, 5985, 5986)

Remote Desktop Protocol (RDP, TCP/3389) and Windows Remote Management (WinRM, TCP/5985 HTTP / TCP/5986 HTTPS) provide remote administrative interfaces on Windows systems. During NTLM negotiation, these services frequently disclose NetBIOS domain names, DNS domain names, hostname strings, and exact OS build numbers.

```bash
# Extract NTLM domain/host metadata and verify RDP encryption/NLA settings
sudo nmap -p 3389 --script "rdp-ntlm-info,rdp-enum-encryption" "$TARGET"

# Inspect WinRM HTTP/HTTPS endpoints and supported authentication schemes
sudo nmap -p 5985,5986 -sV --script "http-title,http-auth" "$TARGET"
```

## 14. Additional network services quick reference

| Port / Protocol | Service | Primary enumeration commands |
| --- | --- | --- |
| `23/tcp` | Telnet | `nc -nv "$TARGET" 23`, `nmap -p 23 --script telnet-ntlm-info "$TARGET"` |
| `69/udp` | TFTP | `nmap -sU -p 69 --script tftp-enum "$TARGET"` |
| `111/tcp,udp` | RPCbind | `rpcinfo -p "$TARGET"`, `nmap -p 111 --script rpcinfo "$TARGET"` |
| `123/udp` | NTP | `ntpq -c readlist "$TARGET"`, `nmap -sU -p 123 --script ntp-info "$TARGET"` |
| `389,636/tcp` | LDAP / LDAPS | `ldapsearch -x -H "ldap://$TARGET" -s base namingcontexts` |
| `2049/tcp,udp` | NFS | `showmount -e "$TARGET"`, `nmap -p 2049 --script nfs-ls,nfs-showmount "$TARGET"` |
| `5432/tcp` | PostgreSQL | `psql -h "$TARGET" -U postgres` |
| `6379/tcp` | Redis | `redis-cli -h "$TARGET" INFO` |
| `27017/tcp` | MongoDB | `nmap -p 27017 --script mongodb-info,mongodb-databases "$TARGET"` |

## 15. Technical references

* [Nmap Scripting Engine (NSE) Documentation](https://nmap.org/book/man-nse.html) — Official reference for NSE script categories, execution phases, and arguments.
* [Nmap NSEDoc Portal](https://nmap.org/nsedoc/) — Complete index of built-in Nmap scripts and protocol-specific probes.
* [OWASP Web Security Testing Guide (WSTG)](https://owasp.org/www-project-web-security-testing-guide/) — Open methodology for HTTP/HTTPS reconnaissance, metadata inspection, and application enumeration.
* [RFC 5321 — Simple Mail Transfer Protocol](https://www.rfc-editor.org/rfc/rfc5321) — Protocol specification defining SMTP commands (`EHLO`, `VRFY`, `EXPN`, `RCPT`).
* [RFC 1035 — Domain Names: Implementation and Specification](https://www.rfc-editor.org/rfc/rfc1035) — Specification for DNS queries and zone transfer (`AXFR`) mechanics.
* [RFC 3416 — Version 2 of the Protocol Operations for SNMP](https://www.rfc-editor.org/rfc/rfc3416) — Specification for SNMPv2c operations and MIB object retrieval.
* [RFC 5737 — IPv4 Address Blocks Reserved for Documentation](https://www.rfc-editor.org/rfc/rfc5737) — Standard defining `TEST-NET-1`, `TEST-NET-2`, and `TEST-NET-3` documentation ranges.
* [RFC 2606 — Reserved Top Level DNS Names](https://www.rfc-editor.org/rfc/rfc2606) — Standard reserving `example.com` for technical documentation.
* [Samba smbclient Manual](https://www.samba.org/samba/docs/current/man-html/smbclient.1.html) — Official documentation for querying SMB/CIFS shares and null sessions.
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
Run tools with --version or --help to verify installed paths and flag compatibility, as option syntax and wordlist locations vary across Linux distributions. IPs reserved for documentation (192.0.2.0/24, RFC 5737) are used strictly as placeholders and must be replaced with authorized target addresses.

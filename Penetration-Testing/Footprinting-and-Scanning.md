# Footprinting and Scanning

Active reconnaissance, network mapping, port scanning, host fingerprinting, and evidence analysis.

**Author:** Radzivon Ferrigni Dioguardi

## Contents

1. [Reconnaissance scope and objectives](#1-reconnaissance-scope-and-objectives)
2. [Network mapping: goals and methodology](#2-network-mapping-goals-and-methodology)
3. [Network models and encapsulation](#3-network-models-and-encapsulation)
4. [IP addressing, subnetting, and routing](#4-ip-addressing-subnetting-and-routing)
5. [IPv4 and IPv6 packet structure](#5-ipv4-and-ipv6-packet-structure)
6. [ICMP and network diagnostics](#6-icmp-and-network-diagnostics)
7. [TCP connections, ports, and flags](#7-tcp-connections-ports-and-flags)
8. [UDP behavior and probe limitations](#8-udp-behavior-and-probe-limitations)
9. [Scan origin and network visibility](#9-scan-origin-and-network-visibility)
10. [Host discovery techniques](#10-host-discovery-techniques)
11. [Nmap host discovery and ping sweeps](#11-nmap-host-discovery-and-ping-sweeps)
12. [Target selection, ports, and exclusions](#12-target-selection-ports-and-exclusions)
13. [Understanding Nmap port states](#13-understanding-nmap-port-states)
14. [TCP port scanning: SYN, Connect, ACK, and other methods](#14-tcp-port-scanning-syn-connect-ack-and-other-methods)
15. [UDP port scanning](#15-udp-port-scanning)
16. [Service and version detection](#16-service-and-version-detection)
17. [Operating system fingerprinting](#17-operating-system-fingerprinting)
18. [Nmap Scripting Engine (NSE)](#18-nmap-scripting-engine-nse)
19. [Firewalls, NAT, and IDS/IPS](#19-firewalls-nat-and-idsips)
20. [Advanced scanning and filter analysis](#20-advanced-scanning-and-filter-analysis)
21. [Scan timing and performance](#21-scan-timing-and-performance)
22. [Output formats and result preservation](#22-output-formats-and-result-preservation)
23. [Packet analysis with Wireshark and tcpdump](#23-packet-analysis-with-wireshark-and-tcpdump)
24. [Verifying services on Linux and Windows](#24-verifying-services-on-linux-and-windows)
25. [Practical scanning methodology](#25-practical-scanning-methodology)
26. [Reading and interpreting scan results](#26-reading-and-interpreting-scan-results)
27. [Troubleshooting scans](#27-troubleshooting-scans)
28. [Nmap command reference](#28-nmap-command-reference)
29. [Technical references](#29-technical-references)

## Command conventions

Commands use Linux shell syntax unless a PowerShell block is explicitly labeled. Replace the documentation placeholders with targets **inside the authorized scope**. The examples show command usage and do not claim that any scan has been executed.

```bash
TARGET='ENTER_AUTHORIZED_IP_OR_HOST'
NETWORK='ENTER_AUTHORIZED_CIDR'
mkdir -p scans
```

Run `nmap --version` and check `nmap --help` or the installed manual when options or supported scan types may differ between platforms and versions. IPs reserved for documentation are not real test results. Local lab IPs and interfaces in examples must be replaced with those of the actual authorized environment.

## 1. Reconnaissance scope and objectives

Footprinting is the process of reconstructing the characteristics of an environment from available evidence: IP addresses, hosts, reachable networks, ports, services, and the relationships between them. Scanning is part of active reconnaissance, not a substitute for a methodology. A useful sequence is **scope → network mapping → host discovery → port scanning → service identification → verification → reporting**.

A list of addresses is not yet a network map. An open port does not prove that an application is vulnerable, and a version string alone does not establish the presence of a vulnerability. A credible assessment preserves both its conclusions and the evidence behind them.

| Item | What must be established before scanning |
| --- | --- |
| In-scope assets | Authorized IPs, CIDR ranges, FQDNs, applications, interfaces, or environments. |
| Exclusions | Sensitive hosts, CDNs, third-party tenants, and connected networks outside the scope. |
| Permitted activities | Host discovery, TCP/UDP scanning, fingerprinting, scripts, and application requests. |
| Operational limits | Testing windows, scan rate, number of attempts, and acceptable service impact. |
| Evidence | Scan origin, date/time, command, tool version, output, and interpretation. |

If an authorized hostname resolves to a CDN address, that **does not** authorize scanning the entire provider network. DNS resolution can change over time, especially for distributed services. Likewise, an address appearing in older documentation may no longer belong to the same organization.

### Passive and active reconnaissance

Consulting a public archive is generally passive *with respect to the target*, even though the client communicates with the archive provider. An ICMP Echo Request, a TCP connection, or an HTTP request to the target server is active reconnaissance. The classification depends on **the requests actually sent**, not merely on the name of the tool.

## 2. Network mapping: goals and methodology

Network mapping collects and organizes technical information about assets within the test scope. A map may describe authorized subnets, gateways, identified hosts, exposed endpoints, associated names, and differences between network segments. It must be considered a **partial representation that depends on the observation point**.

A device may respond on a LAN but be unreachable through a VPN. Firewalls can change port visibility; NAT can expose multiple services behind one address; and a multihomed system may have interfaces in different networks. A list of scan responses does not reveal the entire physical topology.

| Information | Useful observation | Limitation |
| --- | --- | --- |
| IP address | Host or interface responding to probes. | NAT, proxies, and virtual addresses may intervene. |
| Hostname | Name returned by DNS or a service. | The name may be outdated or unverified. |
| MAC address | Link-layer address visible on the local segment. | MAC addresses do not normally cross routers. |
| Port | Transport endpoint and its state from the scanner's perspective. | A port number does not identify the application. |
| Service/version | Match inferred from an application response. | Results may be incomplete, concealed, or customized. |
| Operating system | Hypothesis based on TCP/IP behavior or services. | Identification may be ambiguous. |

An asset inventory should record at least the target, observation point, technique, findings, and limitations.

### Logical topology and traceroute

A port scan does not show how traffic reaches a target. **Traceroute** can reveal responding intermediate routers by using the IPv4 TTL or IPv6 Hop Limit: when the value reaches zero, a router may generate a time-exceeded message. Nmap also supports `--traceroute` alongside scans of authorized hosts.

```bash
traceroute -n "$TARGET"   # IPv4: responding intermediate hops
sudo nmap -sS -p 443 --traceroute -n "$TARGET"
```

Routes may be asymmetric. Some devices ignore probes or do not expose addresses that directly identify their functions. The result describes **the observed path**, not every physical connection. Intermediate hops may belong to external operators; review the route and rules of engagement before probing paths that traverse their systems.

## 3. Network models and encapsulation

A protocol defines communication rules understood by the devices implementing it. The network stack combines link-layer, IP, transport, and application protocols rather than relying on a single protocol. The seven-layer OSI model helps analyze behavior; the Internet protocol suite is commonly represented using four layers. This mapping is a conceptual aid, not a rigid description of every implementation.

| OSI layers | Internet model | Function and examples |
| --- | --- | --- |
| 7 Application, 6 Presentation, 5 Session | Application | HTTP, DNS, SSH, SMTP; application data and sessions. |
| 4 Transport | Transport | TCP and UDP; endpoints and port numbers. QUIC runs over UDP. |
| 3 Network | Internet | IPv4, IPv6, ICMP, and packet forwarding. |
| 2 Data Link, 1 Physical | Network access | Ethernet, Wi-Fi, frames, and physical transmission. |

**Encapsulation.** For an HTTP request transported over TCP, the application data is carried by a TCP segment inside an IP packet, which in turn is carried by an Ethernet frame on an Ethernet link. Each layer adds its own fields; the receiving system reverses the process. Frames may also contain trailers, and control protocols may have different structures. The statement that every packet always has precisely a header and a payload is a simplification.

```text
Application        application request / response
Transport          [TCP or UDP | application data]
Network            [IPv4/IPv6 | transport unit]
Data link          [Ethernet | IP packet | FCS]
Medium             electrical, optical, or radio signal
```

A rejected TCP connection and an HTTP 404 response represent different events. In the first case, a service connection was not established; in the second, an HTTP server received a request and returned an application-level response.

## 4. IP addressing, subnetting, and routing

IP addresses are network-layer identifiers assigned to interfaces. Ports identify transport endpoints, while MAC addresses are used for communication on local links. To reach a remote server over Ethernet, the sender can preserve the remote destination IP inside the packet while sending the **frame** to the MAC address of the local gateway. The router builds a new frame for the next link.

On IPv4 Ethernet, ARP resolves the next-hop IP to the MAC address used on that segment. IPv6 instead uses Neighbor Discovery, which relies on ICMPv6. An ARP scan generally cannot discover the MAC address of a server beyond a router.

```bash
ip -br address            # Local interfaces and addresses on Linux
ip route                  # IPv4 routing table
ip -6 route               # IPv6 routing table
ip neigh show             # IPv4/IPv6 neighbor cache
ip route get 192.0.2.10   # Route choice, not proof of connectivity
```

An IPv4 `/24` prefix reserves 24 network bits and leaves eight bits for addresses within the block. The private network `192.168.56.0/24` has 256 address values; in a conventional broadcast subnet, `.0` is the network address and `.255` is the directed broadcast address. The rule “usable addresses = 2^n - 2” does not apply to every prefix: point-to-point `/31` networks are a notable exception.

The private IPv4 ranges are `10.0.0.0/8`, `172.16.0.0/12`, and `192.168.0.0/16`. They are different from address blocks reserved exclusively for documentation examples.

## 5. IPv4 and IPv6 packet structure

IPv4 uses 32-bit addresses; IPv6 uses 128-bit addresses. The two protocols differ in header structure, neighbor discovery, and fragmentation. Understanding packet fields is important because firewalls and capture tools may classify traffic using addresses, protocol numbers, TTL/Hop Limit, fragmentation data, and other fields.

### Main IPv4 header fields

| Field | Size | Why it matters |
| --- | ---: | --- |
| Version | 4 bits | Identifies IPv4. |
| IHL | 4 bits | Header length in 32-bit words. |
| DSCP / ECN | 8 bits | Traffic treatment and congestion signaling. |
| Total Length | 16 bits | Total IPv4 packet length, including header. |
| Identification | 16 bits | Correlates fragments during reassembly. |
| Flags / Fragment Offset | 3 / 13 bits | Fragmentation control and position. |
| TTL | 8 bits | Decreases at routers and limits packet lifetime. |
| Protocol | 8 bits | Identifies the payload protocol, e.g., ICMP=1, TCP=6, UDP=17. |
| Header Checksum | 16 bits | Checks the IPv4 header, not the whole payload. |
| Source / Destination | 32 bits each | Logical source and destination IPv4 addresses. |
| Options | Variable | Optional fields that may be absent. |

An IPv4 header without options is normally **20 bytes** long. An observed TTL does not equal the hop count: without the initial TTL value, the route cannot be reconstructed exactly, nor can the operating system be identified conclusively. An IPv4 router may fragment packets when conditions require it and the Don't Fragment flag permits it. An IPv6 router **does not** fragment transit packets.

IPv6 uses **Hop Limit** and **Next Header** fields; optional information can be carried in extension headers. When comparing packet captures, identify the actual protocol instead of projecting the IPv4 header structure onto IPv6.

## 6. ICMP and network diagnostics

ICMP supports IP through control messages, error reporting, and diagnostics. On IPv4, `ping` typically uses ICMP Echo Request and Echo Reply. ICMP messages are identified by **type and code**, not TCP or UDP ports. Under RFC 792, an ICMPv4 Echo Request is type 8/code 0, and Echo Reply is type 0/code 0.

| Response | Careful interpretation |
| --- | --- |
| Echo Reply | A device responded to an ICMP probe. |
| Destination Unreachable | A host or router reported a delivery problem; details depend on type/code. |
| Time Exceeded | Packet lifetime expired because of TTL or another defined condition. |
| No response | The probe may be filtered, lost, rate-limited, or the target may be inactive. |

```bash
ping -c 3 "$TARGET"     # Linux: send three Echo Requests
traceroute "$TARGET"   # If available; the path may be incomplete
```

Traceroute sends probes with increasing TTL/Hop Limit values to observe devices along a path. Routers can ignore or rate-limit responses, and return paths may be asymmetric. `* * *` does not prove that a specific firewall or failure exists at that hop.

## 7. TCP connections, ports, and flags

TCP provides an ordered byte stream using acknowledgments, retransmissions, flow control, and congestion control. The conventional connection setup is the **three-way handshake**, as defined by the TCP standard.

```text
Client                              Server
  | ------ SYN, seq=x -------------> |
  | <----- SYN+ACK, seq=y ---------- |
  | ------ ACK, ack=y+1 -----------> |
  |             ESTABLISHED          |
```

The client sends SYN, the server replies with SYN+ACK, and the client acknowledges with ACK. SYN and Connect scanning rely on this behavior. Receiving SYN+ACK suggests that a system accepted an initial connection request; identifying the service on that port requires additional verification.

| TCP flag | Core meaning |
| --- | --- |
| SYN | Synchronization during connection establishment. |
| ACK | Acknowledgment of received sequence numbers. |
| RST | Reset: refusal or termination of a connection. |
| FIN | Orderly shutdown of one direction of the byte stream. |
| PSH | Requests prompt delivery of data to the application. |
| URG | Indicates use of the urgent pointer field. |

TCP and UDP port numbers range from **0 to 65535**, but service assignments are conventions, not guarantees. IANA divides them into System Ports (0–1023), User Ports (1024–49151), and Dynamic/Private Ports (49152–65535). An HTTP server can listen on 80, 8080, or another configured port: identify the application response rather than guessing from the number.

## 8. UDP behavior and probe limitations

UDP exchanges datagrams without a transport-layer handshake. By itself, it does not guarantee delivery, ordering, or retransmission; applications may provide such mechanisms. DNS, NTP, DHCP, various media applications, and QUIC use UDP. It is incorrect to claim that UDP is **always faster** or that a silent port is necessarily open.

| Observation after a UDP probe | Assessment |
| --- | --- |
| Valid UDP application response | Strong evidence that the service is reachable. |
| ICMPv4 Destination Unreachable, port unreachable | Typically indicates `closed`. |
| Certain other ICMP errors | May indicate `filtered`, depending on type/code. |
| No response after retries | May indicate `open\|filtered`; uncertainty remains. |

A service may ignore malformed datagrams, and ICMP responses can be rate-limited. Results depend on the technique and actual replies. For some UDP ports, Nmap sends protocol-specific payloads to improve identification. `-sV` may send additional application probes against relevant results.

## 9. Scan origin and network visibility

Before scanning, establish **which interface, namespace, and network segment** will carry the traffic. A NAT-configured virtual machine has different visibility from a VM on a host-only network. A container may use an isolated bridge, and a VPN may add routes or filtering policies.

```bash
ip -br address
ip route
ip neigh show
nmap --version
```

A routing table shows the route the kernel would select, not whether a remote server will answer. An ARP/ND neighbor cache lists previously observed or resolved neighbors, not a complete inventory. Capture interfaces also matter: monitoring the wrong interface can make transmitted traffic appear absent.

At minimum, record the scanning device, operating system, interface, source address, VPN/NAT context, permitted subnets, and test time. These details help explain differences across scans made from distinct points or at different times.

## 10. Host discovery techniques

Host discovery sends probes to identify addresses that exhibit activity. “Host down” in an output is an **operational conclusion based on the discovery attempted**, not proof that the physical device is powered off. Probe selection depends on the link, privileges, routing, and filtering policies.

| Technique | Mechanism | Consideration |
| --- | --- | --- |
| IPv4 ARP | Resolves link-layer addresses on the local segment. | Often effective on a LAN; does not cross routers. |
| IPv6 ND | Neighbor Solicitation/Advertisement. | Local IPv6 discovery; not ARP. |
| ICMP Echo | Echo request and reply. | May be blocked by policy. |
| TCP SYN ping | SYN probe to selected ports. | May reveal an active host even if ICMP is blocked. |
| TCP ACK ping | ACK probe and response observation. | Depends on firewalls and TCP behavior. |
| UDP ping | UDP datagram to a selected port. | Often depends on application or ICMP replies. |
| IP protocol ping | Probe using a selected IP protocol number. | Results depend on protocol and responses. |

Nmap can combine methods. Host discovery is not always synonymous with ICMP ping. On a local Ethernet network, Nmap may prefer ARP/ND even when ICMP options or `-Pn` are specified, unless the appropriate controls are used.

## 11. Nmap host discovery and ping sweeps

A **ping sweep** applies discovery probes to a set of addresses. The historical term suggests ICMP, but TCP or ARP probes are also used in practice. Nmap's host-discovery-only option is `-sn`: it skips the ordinary port scan but can still send discovery traffic and perform name resolution.

```bash
# Authorized targets only; does not start the usual port scan.
sudo nmap -sn -n "$NETWORK"

# ICMP Echo discovery against a non-local target.
sudo nmap -sn -PE -n "$TARGET"

# TCP discovery probes against selected ports.
sudo nmap -sn -PS22,443 -n "$TARGET"
sudo nmap -sn -PA80,443 -n "$TARGET"

# UDP probes to selected ports.
sudo nmap -sn -PU53,161 -n "$TARGET"

# Explicit ARP discovery on the local IPv4 LAN.
sudo nmap -sn -PR -n "$NETWORK"
```

**`-PE` does not automatically force ICMP-only discovery on a LAN**: ARP-based local discovery is handled separately. `-n` disables DNS name resolution, reducing extra traffic and external dependencies. TCP and UDP probes may elicit responses from a service or a filtering device; interpret these results accordingly.

### Other discovery probes

In addition to ICMP Echo (`-PE`), Nmap supports ICMP Timestamp (`-PP`) and Address Mask (`-PM`). Modern systems do not necessarily respond to these requests. IP protocol ping (`-PO`) probes IP protocol numbers rather than TCP or UDP ports. Availability depends on the address family, privileges, and network: combining every option without a reason is not useful.

```bash
sudo nmap -sn -n -PP "$TARGET"         # ICMP Timestamp, IPv4
sudo nmap -sn -n -PO1,6,17 "$TARGET"   # IP protocol ping
```

To distinguish ICMP responses from ARP responses on a local network, `--disable-arp-ping` can be used **when appropriate for the link and chosen method**. On IPv6 networks, local discovery uses Neighbor Discovery rather than ARP.

### ARP discovery with additional tools

On IPv4 LANs, **arp-scan** and **netdiscover** can complement Nmap by identifying addresses that reply to ARP requests. Unlike routed IP probes, ARP operates on the local link and cannot provide a full view of remote subnets.

```bash
# Authorized lab network connected to eth0; replace as necessary.
sudo arp-scan --interface=eth0 192.168.56.0/24
sudo netdiscover -i eth0 -r 192.168.56.0/24
```

Check the installed tools' help for exact syntax. `192.168.56.0/24` is a private network example, **not automatically an authorized network**; replace `eth0` with the actual interface. An observed MAC address may belong to a virtual machine, proxy ARP implementation, or device responding on behalf of other addresses. Correlate findings with routes and local configuration.

### `-Pn`: scanning without preliminary discovery

```bash
nmap -Pn -sT -p 22,80,443 -n "$TARGET"
```

`-Pn` **skips host discovery** and treats the specified targets as available for the requested scan phases. It does not prove that those hosts are online. Using it indiscriminately on a large subnet causes Nmap to attempt the specified scans against every address, potentially increasing traffic and duration. It can help when discovery probes are filtered, but should be limited to authorized targets. On a LAN, ARP/ND exchanges required for link-layer delivery may still occur with `-Pn`.

### Nmap on Windows and Npcap

Nmap is available on Windows. It normally relies on **Npcap** to send and capture raw packets; the supported methods depend on the driver, network interface, and privileges. From PowerShell, once the permitted subnet is confirmed, Nmap discovery options use the same syntax:

```powershell
$Network = 'ENTER_AUTHORIZED_CIDR'
nmap --version
nmap -sn -n $Network
```

On certain VPN connections and interfaces that do not support raw packet operations, results may differ from those obtained on Ethernet. Without Npcap, a TCP Connect scan with `-sT -Pn` may work for a **single authorized host** through the operating system's sockets API, but it is not equivalent to host discovery. Avoid applying `-Pn` blindly to entire networks.

## 12. Target selection, ports, and exclusions

A CIDR range, a target file, and a port list describe different aspects of a scan. `-sL` **lists the selected targets** before active scanning; unless `-n` is specified, name-resolution queries may still occur. `-iL` reads targets from a file; `--exclude` and `--excludefile` explicitly remove destinations.

```bash
nmap -sL -n "$NETWORK"
nmap -sL -n -iL targets.txt --excludefile excluded.txt

# Only the selected ports.
nmap -sT -p 22,80,443 "$TARGET"

# The 20 most common ports according to Nmap's database.
nmap -sT --top-ports 20 "$TARGET"

# All TCP ports numbered 1-65535.
nmap -sT -p- "$TARGET"
```

With `-p-`, Nmap scans ports **1–65535** for the selected protocol, not every possible transport protocol unless the corresponding scan modes are enabled. Without explicit port selection, a typical Nmap scan checks the **1,000 most common TCP ports** according to `nmap-services`, not simply numeric ports 1–1000. `-F` selects a smaller set; `-r` changes port scan ordering.

Verify target names, resolved addresses, explicit exclusions, and potential third-party hosts before scanning. Discovery does not expand the agreed scope.

## 13. Understanding Nmap port states

Port scanning probes transport endpoints and infers their states from responses, errors, and timeouts. **A state is Nmap's assessment from a particular observation point using a particular technique**, not a permanent property of a port. A server may appear `open` on an internal network and `filtered` from the Internet.

| Nmap state | What it does—and does not—mean |
| --- | --- |
| `open` | An endpoint accepts the probed kind of communication; this does not imply authentication is absent. |
| `closed` | The port is reachable, but no service appears to be listening for that transport. |
| `filtered` | Filtering prevents a confident determination of whether the port is open. |
| `unfiltered` | The port is reachable, but this scan method does not establish whether it is open or closed. |
| `open\|filtered` | Available responses cannot distinguish an open service from filtering. |
| `closed\|filtered` | Certain scan methods cannot distinguish closure from filtering. |

The **REASON** field, enabled with `--reason`, is important: it identifies observations such as `syn-ack`, `reset`, an ICMP response, or no response that led to the classification. Always interpret a state alongside the scan method and network conditions.

```bash
nmap -sT -p 22,80,443 --reason -n "$TARGET"
```

A service might be reachable only over IPv6, through a VPN, from a specific source address, or during a limited window. Do not combine results obtained from different methods or observation points without recording that context.

## 14. TCP port scanning: SYN, Connect, ACK, and other methods

### TCP SYN scan (`-sS`)

A SYN scan sends a crafted TCP SYN probe. SYN+ACK typically indicates `open`; RST indicates `closed`; missing replies or relevant ICMP errors may indicate `filtered`. In a normal SYN scan, Nmap does not complete the three-way handshake as an application would. Linux and Unix systems generally require privileges for raw packet operations.

```bash
sudo nmap -sS -p 22,80,443 --reason -n "$TARGET"
```

### TCP Connect scan (`-sT`)

A Connect scan uses the operating system's `connect()` call to attempt a TCP connection. It works without raw-packet privileges and is often used when SYN scanning is unavailable. Services that accept connections may log it more directly, and its behavior differs from a SYN scan. It is **not passive reconnaissance**.

```bash
nmap -sT -p 22,80,443 --reason -n "$TARGET"
```

### TCP ACK scan (`-sA`)

An ACK scan primarily helps analyze **firewall rules and filtering**, not whether an application is listening. An RST can show that the probe reached an endpoint, producing `unfiltered`; no response or certain errors produce `filtered`. Do not interpret `unfiltered` as `open`.

```bash
sudo nmap -sA -p 22,80,443 --reason -n "$TARGET"
```

### Null, FIN, and Xmas scans

These methods alter TCP flags: no flags (`-sN`), FIN (`-sF`), or FIN/PSH/URG (`-sX`). They rely on particular TCP stack response behaviors. They may report `open|filtered`, be affected by firewalls, or behave differently on systems that do not match the scanner's assumptions, including many Windows systems. **They are not universally superior or invisible techniques.**

```bash
sudo nmap -sN -p 80,443 -n "$TARGET"
sudo nmap -sF -p 80,443 -n "$TARGET"
sudo nmap -sX -p 80,443 -n "$TARGET"
```

Other techniques, such as Window and Maimon scans, depend on specific implementations. IP protocol scan `-sO` tests **IP protocol numbers**, not ports. Idle scanning uses an intermediary host and requires special conditions. These options do not replace SYN, Connect, and application-level checks in ordinary assessments; the official manual documents their prerequisites and limits.

### Interpreting TCP probe responses

| Probe / typical response | Likely result | Limitation |
| --- | --- | --- |
| SYN → SYN+ACK | TCP port `open` | The actual service still needs identification. |
| SYN → RST | TCP port `closed` | An intermediary may alter the response. |
| SYN → no response | Possibly `filtered` | Timeouts and packet loss need consideration. |
| ACK → RST | `unfiltered` in an ACK scan | Does not mean the port is open. |

## 15. UDP port scanning

Discovering UDP services can be slower and more ambiguous than TCP scanning because applications often ignore invalid datagrams. With `-sU`, Nmap sends UDP probes and interprets application replies or ICMP messages. Protocols such as DNS and SNMP have their own request formats, versions, and environmental constraints.

```bash
# A restricted set of authorized UDP ports.
sudo nmap -sU -p 53,123,161 --reason -n "$TARGET"

# Service detection on the selected UDP ports.
sudo nmap -sU -sV -p 53,123,161 -n "$TARGET"

# Different TCP and UDP port sets in the same command.
sudo nmap -sS -sU -p T:22,80,443,U:53,123 -n "$TARGET"
```

`T:` and `U:` within `-p` separate the TCP and UDP port sets, but they are meaningful only when the corresponding TCP and UDP scan methods are selected. The numeric port may match, but `53/tcp` and `53/udp` are different transport endpoints.

Repeated high-speed UDP scans do not guarantee improved accuracy when services remain silent. ICMP rate limits and packet loss can make interpretation harder. Restrict the port set, inspect `--reason`, and, when permitted, send recognizable application-level requests.

## 16. Service and version detection

A name in Nmap's `SERVICE` column may come from the conventional port-to-service mapping. **It is not necessarily proof that the named application is responding.** With `-sV`, Nmap sends application probes and compares replies against `nmap-service-probes`. A port may host HTTP, SSH, or a custom protocol regardless of its usual assignment.

```bash
nmap -sT -sV -p 22,80,443,8080 -n "$TARGET"

# Decrease version detection probe intensity.
nmap -sT -sV --version-light -p 22,80,443 -n "$TARGET"

# Show reasons for port state classifications.
nmap -sT -sV --reason -p 22,80,443 -n "$TARGET"
```

`--version-intensity` accepts values from **0 to 9**; higher values allow a wider set of candidate probes, potentially increasing scan duration, interactions, and load. `--version-light` uses a less intensive configuration. Output version strings may come from modified banners, vendor-maintained backports, or intermediaries: they do not prove a vulnerability by themselves.

### Application protocol verification

After identifying a likely protocol, verify the observed behavior with a suitable request if permitted by the rules of engagement. `curl` is helpful for HTTP and TLS endpoints. A raw TCP connection can expose some text banners but does not replace a protocol-aware client.

```bash
# Only if this port was identified as an HTTP endpoint.
curl -I --max-time 5 "http://${TARGET}:8080/"

# Only if this port was identified as an HTTPS endpoint.
curl -kI --max-time 5 "https://${TARGET}:8443/"
```

In the second command, `-k` disables certificate validation. Use it only for necessary diagnostics, **not as a normal security configuration**. An HTTP `Server:` header may be omitted, altered, or supplied by a reverse proxy.

Be careful with special-purpose devices. Nmap documentation warns that some printers listen on `9100/tcp` and may literally print the payloads they receive. Accordingly, Nmap excludes certain version-detection probes from that port by default. Intrusive verification is not suitable for every environment.

## 17. Operating system fingerprinting

With `-O`, Nmap compares TCP/IP stack responses against fingerprints in its database. It examines characteristics of TCP, UDP, and ICMP packets, among other signals. This does not directly read the remote operating system's name and cannot guarantee an exact match. Conditions improve when at least **one open and one closed TCP port** are available.

```bash
sudo nmap -O -p 22,80,443,65000 -n "$TARGET"
```

Port `65000` is only a **candidate** included in the hope of eliciting a closed-port response; there is no guarantee that it is closed. Restrictive filtering or a limited port selection can weaken fingerprinting. A firewall, load balancer, or proxy can return responses associated with intermediary systems rather than the application server.

It is useful to distinguish three levels of confidence:

| Information | Indicative confidence |
| --- | --- |
| Observed response and packet flags | Direct network evidence, subject to intermediaries. |
| OS fingerprint match | Probabilistic inference. |
| Service banner or self-reported identifier | Declared information that should be corroborated. |

`-A` enables several features together: **OS detection, service/version detection, default script scanning, and traceroute**. It is not simply a harmless alias for `-O`. In a controlled assessment, enable only the features that are required.

## 18. Nmap Scripting Engine (NSE)

The **Nmap Scripting Engine** uses Lua scripts to collect information, examine services, and perform other actions. Categories include `default`, `discovery`, `safe`, `auth`, `brute`, `vuln`, `intrusive`, `exploit`, and `dos`. Categories are organizational labels, **not guarantees of zero impact**. Read a script's documentation, prerequisites, and behavior before running it.

```bash
# View locally available script documentation.
nmap --script-help http-title
nmap --script-help http-headers
nmap --script-help ssl-cert

# Targeted examples for services already verified.
nmap -sT -p 80,8080 --script http-title,http-headers "$TARGET"
nmap -sT -p 443 --script ssl-cert "$TARGET"
nmap -sT -p 22 --script ssh-hostkey "$TARGET"
```

Scripts are documented individually in **NSEDoc**, including categories and parameters. Even a discovery script may generate multiple requests, retrieve certificates, or perform a protocol handshake. Credential attacks, exploitation, denial of service, and manipulation require specific authorization and are not part of a generic footprinting check.

### Main NSE options

| Option | Effect |
| --- | --- |
| `-sC` | Runs the `default` script category. |
| `--script http-title` | Selects a script by name. |
| `--script-help name` | Displays a script's help information. |
| `--script-args` | Passes parameters to selected scripts. |
| `--script-trace` | Displays script-level network interactions for troubleshooting. |

Avoid running `--script all` indiscriminately: it includes scripts with very different purposes and impact. Even `-sC` should be checked against the test's rules of engagement.

## 19. Firewalls, NAT, and IDS/IPS

A firewall may filter by source or destination address, port, protocol, connection state, or application rules. An IPS may react to probe content or rate. NAT changes externally visible addressing; a reverse proxy can respond for multiple backend systems. A scanner therefore observes **the entire network path**, not just the final service.

A **stateless** filter evaluates packets according to rules without necessarily tracking sessions. A **stateful** filter can consider connection context, for example distinguishing an initial SYN from traffic belonging to an established connection. An ACK scan can provide clues about filtering, but it cannot identify every firewall rule or device by itself. Interpret `filtered`, `closed`, and `open|filtered` through actual responses rather than absolute assumptions.

```bash
# Compare techniques while keeping the port set constant.
sudo nmap -sS -p 22,80,443 --reason -n "$TARGET"
sudo nmap -sA -p 22,80,443 --reason -n "$TARGET"
```

If the first scan reports `filtered` and the second `unfiltered`, this does not prove a fault or identify a firewall vendor. Differences may result from filtering, timing, NAT, host firewalls, or path conditions.

To document suspected filtering, record the scan origin, interface, IP version, date/time, protocol, port, actual probes, responses, and Nmap's stated reason.

## 20. Advanced scanning and filter analysis

Nmap documents options that modify probes or their apparent origins, including fragmentation (`-f`, `--mtu`), source port (`-g` or `--source-port`), data length, decoys (`-D`), and source-address selection or spoofing (`-S`). These are technical mechanisms, **not guaranteed ways to bypass a firewall or avoid IDS detection**. They may change measurement conditions and create unexpected traffic.

| Mechanism | What changes | Important limitation |
| --- | --- | --- |
| Fragmentation | Splits some raw IP packets into fragments. | Devices may reassemble or discard fragments. |
| Source port | Sets the source port on supported probes. | Stateful and application-layer filtering can ignore this approach. |
| Decoys | Introduces additional apparent source identities. | May involve third parties and confuse logs; requires explicit scope. |
| Spoofed source | Changes the source IP in packets. | Replies may not return; anti-spoofing controls may apply. |
| Modified payload/MTU | Alters probe content or segmentation. | Can affect how endpoints and intermediaries respond. |

`-f` and `--mtu` cannot be combined. Nmap fragmentation applies only to certain raw-packet scan methods and not to TCP Connect scanning. To examine a suspected filtering rule, design **authorized, repeatable probes**, preferably in a controlled environment. A port changing state does not, by itself, prove that filtering has been bypassed.

The priority is not to evade protections but to establish **which tests are permitted, which probes were blocked, and which conclusions the available evidence supports**.

## 21. Scan timing and performance

Scan duration depends on the number of hosts and ports, chosen methods, round-trip time (RTT), rate limiting, retransmissions, privileges, and enabled scripts. Nmap adjusts its behavior dynamically. A timeout on a congested network should not be treated as conclusive proof of filtering without further checks. Timing settings also affect the operational impact of a test.

| Option | Purpose | Caution |
| --- | --- | --- |
| `-T0` … `-T5` | Timing templates. | `-T4` and `-T5` are not always appropriate for fragile networks. |
| `--max-rate N` | Requests an upper limit on probe sending rate. | Does not guarantee zero impact on a service. |
| `--scan-delay` | Delays probes against a host. | Can significantly increase scan time. |
| `--max-retries N` | Limits retransmissions. | Too few retries can increase false negatives. |
| `--host-timeout` | Limits scan duration per host. | A scan may finish incomplete. |
| `--max-rtt-timeout` | Caps the RTT timeout. | May be too short on high-latency networks. |

```bash
# Narrow port set, conservative rate limit, persistent output.
nmap -sT -p 22,80,443 --max-rate 20 --reason \
     -oA scans/controlled-tcp "$TARGET"
```

The first optimization is usually to **scan only what is needed**: select relevant hosts and ports, avoid scripts and version detection when unnecessary, and reduce redundant scans. Seeking speed through `--min-rate` or `--max-retries 0` may reduce reliability or increase load; it is not automatically an improvement.

## 22. Output formats and result preservation

Nmap provides human-readable output, structured XML, and a legacy grepable format. Retaining raw output matters because a later summary can omit ports, timestamps, hostnames, errors, or the reasons behind state classifications.

```bash
# Three files in one run: .nmap, .xml, .gnmap.
nmap -sT -sV -p 22,80,443 -oA scans/services "$TARGET"

# Normal output only.
nmap -sT -p 22,80 -oN scans/ports.nmap "$TARGET"

# XML for imports and structured parsers.
nmap -sT -p 22,80 -oX scans/ports.xml "$TARGET"
```

| Format | Preferred use |
| --- | --- |
| `.nmap` (`-oN`) | Manual reading, quick review, and text archiving. |
| `.xml` (`-oX`) | Structured processing, imports, and automated comparison. |
| `.gnmap` (`-oG`) | Legacy grepable output retained for compatibility; prefer XML in new tooling. |
| `-oA prefix` | Produces the three main output files in one run. |

### Verbosity, debugging, and packet tracing

```bash
nmap -sT -p 22,80,443 -v --reason "$TARGET"

# Single host and port: display traffic reported by Nmap.
sudo nmap -sS -p 80 --packet-trace -n "$TARGET"
```

`-v` increases verbosity; `-d` enables debugging details; `--reason` shows why Nmap assigned a port state; and `--packet-trace` reports sent and received packets. Tracing an entire network can create enormous output, so restrict it to a small number of hosts and ports.

XML output can include the tool version, command arguments, time, hosts, states, services, and other results. When a structured XML parser is available, do not treat XML as loosely formatted text to be extracted with fragile regular expressions.

### Zenmap and result visualization

**Zenmap** is Nmap's official graphical interface. It can run scans through profiles and display the original command output and results organized by host, port, and service. It does not change the meaning of Nmap states or turn an OS estimate into certainty. When sharing a Zenmap screenshot, preserve the **corresponding Nmap command line** and original output as well: a standalone graphical view may omit essential options and target details.

## 23. Packet analysis with Wireshark and tcpdump

Packet captures help distinguish **what was actually sent** from **what Nmap inferred**. This is particularly valuable when SYN and Connect scans differ, when UDP results are `open|filtered`, or when the same host responds from one network but not another.

```bash
# First confirm the correct interface name.
ip -br link

# Limited capture of TCP and ICMP on an authorized interface.
sudo tcpdump -i eth0 -nn 'tcp or icmp' -w scans/analysis.pcap
```

`eth0` is a placeholder for the real interface. A capture may include credentials, application data, and addresses; retain it only when authorized and apply suitable access controls and retention periods. The filter shown is a tcpdump **capture filter**. Wireshark display filters use different syntax.

| Wireshark display filter | What it shows |
| --- | --- |
| `tcp` | Packets decoded as TCP. |
| `tcp.flags.syn == 1` | Segments with SYN set, including SYN+ACK. |
| `tcp.flags.syn == 1 && tcp.flags.ack == 0` | SYN segments without ACK, typical of connection initiation. |
| `tcp.flags.reset == 1` | Segments with RST set. |
| `icmp` | ICMPv4 messages. |
| `udp.port == 53` | UDP packets with source or destination port 53. |

A display filter showing only SYN packets does not necessarily document the replies. To reconstruct a handshake, examine SYN, SYN+ACK, and ACK with the correct addresses and ports. Correlate sequence numbers, flags, timestamps, and checksums, remembering that network interface offloading can affect local observations.

## 24. Verifying services on Linux and Windows

External scanning and local inspection answer different questions. A local command can show that a process listens on a port, but this does not prove the port is reachable from another network. Conversely, NAT, proxies, and containers may expose an endpoint externally that does not correspond to a socket on the particular system being inspected.

### Linux

```bash
ip -br address
ip route
ss -lntup       # Listening TCP/UDP sockets with processes, when visible
sudo ss -lntup  # More process details when privileges permit
```

`ss` queries the kernel socket tables. `-l` selects listening sockets, `-n` prevents name/port resolution, `-t` and `-u` select TCP and UDP, and `-p` adds process information when authorized.

### Windows / PowerShell

```powershell
Get-NetIPAddress
Get-NetRoute
Get-NetTCPConnection -State Listen |
    Select-Object LocalAddress,LocalPort,OwningProcess
Get-Process -Id 1234
Test-NetConnection -ComputerName 192.0.2.10 -Port 443
```

`1234` is an example PID to replace with an observed value. `Test-NetConnection` checks TCP connectivity to the specified port; it is not a substitute for a full port scan. `Get-NetTCPConnection` reports locally known TCP endpoints and connections. It does not establish the state of remote UDP ports.

Differences between external Nmap output and local sockets can arise from host firewalls, services bound to `127.0.0.1`, interface-specific bindings, NAT rules, load balancing, proxies, and containers. Identifying the cause requires path and configuration checks rather than repeating the same scan without changing the conditions.

## 25. Practical scanning methodology

The following workflow is **a reusable procedure, not a claim that a laboratory assessment was performed**. Use it only on owned or explicitly authorized systems. Each phase establishes a question and a form of evidence to preserve. Select options according to traffic restrictions, testing windows, and asset sensitivity.

### Phase A — Define scope and scan origin

Record the scanning system, connection type (LAN, VPN, virtual network), interface, source IP, authorized ranges, and exclusions. Do not reuse a CIDR from an old document without confirming that it remains in scope. Check that Nmap supports the selected options and that privileges are sufficient for raw-packet methods.

```bash
nmap --version
ip -br address
ip route
nmap -sL -n "$NETWORK"
```

The final command lists the selected addresses without ordinary host discovery or port scanning; `-n` suppresses DNS lookups. It does **not** prove that any listed host is online. Remove excluded assets **before** proceeding.

### Phase B — Discover hosts

```bash
sudo nmap -sn -n -oA scans/discovery "$NETWORK"
```

The actual method depends on the link, privileges, and options. ARP may be used on a LAN; ICMP and TCP probes may be used across routed networks. Record why a host was considered responsive and whether a second type of probe is justified for **individual authorized hosts** that did not respond initially.

### Phase C — Select and scan ports

Do not treat the discovery output as an exhaustive list of active hosts: filtering can hide systems. Subsequent target selection must account for both the scope and observations. Begin with a limited port set when appropriate:

```bash
nmap -sT -p 22,80,443 --reason -n \
     -oA scans/initial-ports "$TARGET"
```

If authorized, a broader scan can check the remaining TCP ports. UDP should normally use a deliberately selected set of ports because its timing and response characteristics differ.

```bash
nmap -sT -p- --reason -n -oA scans/tcp-ports "$TARGET"
sudo nmap -sU -p 53,123,161 --reason -n \
     -oA scans/udp-ports "$TARGET"
```

### Phase D — Identify the actual services

Perform version detection where it is relevant to the ports, observed behavior, and permitted activities. Verified HTTP/HTTPS endpoints may warrant limited application requests; other protocols require suitable clients. Do not report a vulnerability based on a banner alone.

```bash
nmap -sT -sV -p 22,80,443 --reason -n \
     -oA scans/services "$TARGET"
```

Select NSE scripts by name only after consulting their NSEDoc entries. Even basic inspection of an HTTPS endpoint may disclose certificates, names, and service details; handle these as assessment material.

### Phase E — Correlate findings and report

For each service, record **address, transport, port, state, scanning technique, application evidence, source, and confidence**. Include timestamps and scan origins. Separate confirmed observations from questions that still require verification. Do not present false positives as facts or make vulnerability claims from an unverified version string.

| Report field | What to record, without inventing values |
| --- | --- |
| Asset | Actually authorized host/IP. |
| Provenance | Scanning system and interface. |
| Time window | Local timestamp including time zone. |
| Command and version | Full command line and `nmap --version`. |
| Raw evidence | `.nmap` and `.xml` files, plus captures where applicable. |
| Interpretation | Why a port is considered open, filtered, or ambiguous. |
| Corroboration | Confirmation from an application client or authorized local checks. |
| Limitations | Timeouts, filtering, and differences between observation points. |

## 26. Reading and interpreting scan results

Nmap output may contain execution metadata, host status, a port table, and—if requested—service detection, scripts, or OS fingerprinting. These sections answer different questions.

Read the `PORT STATE SERVICE VERSION` table as four separate pieces of information:

| Field | Correct question |
| --- | --- |
| `PORT` | Which port number and transport protocol were examined? |
| `STATE` | What did Nmap conclude with this specific technique? |
| `SERVICE` | Is this a conventional port name or an identified protocol? |
| `VERSION` | What version evidence was obtained, and with what limitations? |

### Example published in the official documentation

The official Nmap tutorial **A Quick Port Scanning Tutorial** provides historical scan outputs published by Nmap's documentation authors and explains the selected methods. It is preferable to fabricating realistic-looking terminal outputs. The original source is linked in the references below.

The tutorial describes **two historical scans** against `scanme.nmap.org`. In the first, using the default 1,000 TCP ports, Nmap reported **994 filtered ports**, three open ports (`22/tcp`, `53/tcp`, `80/tcp`), and three closed ports (`25/tcp`, `70/tcp`, `113/tcp`), taking roughly **5 seconds**. In the second, using `-p0- -v -A -T4`, it examined 65,536 port numbers **from 0 to 65535** and included version detection, OS detection, NSE, and traceroute. The reported duration was about **477 seconds**.

**These are historical results reproduced from the tutorial, not a scan conducted for this document, and not evidence of that server's current configuration.** The historical `-p0-` includes port zero, unlike `-p-` in the earlier operational examples. A broader scan may produce more detail and take far longer without necessarily finding additional open ports. Confirming current exposure requires fresh, authorized observations.

### Comparing different scan results

Differences can be meaningful without indicating that a tool malfunctioned. Compare:

- the IP address and DNS resolution at the time of each test;
- scan origin, VPN, routes, and IPv4/IPv6 address family;
- method (`-sS`, `-sT`, `-sA`, `-sU`), privileges, and selected ports;
- timeouts, retries, rate limits, and execution errors;
- load balancers, proxies, dynamic rules, or services started between runs.

If a port appears `open` in one test and `closed` in another, first check that **the same endpoint was examined under comparable conditions**.

## 27. Troubleshooting scans

| Symptom | Possible explanations | Useful first check |
| --- | --- | --- |
| `Host seems down` | Filtered discovery, missing route, powered-off target, DNS problem. | Address/routing, permitted probes, and justified `-Pn` on one host. |
| All ports `filtered` | ACLs, host/network firewall, timeouts, unreachable target. | `--reason`, scan origin, small packet capture, route inspection. |
| UDP `open\|filtered` | Silent application or filtering; no reply received. | Limited `-sV`, correct protocol request, ICMP rate limiting. |
| No OS match | Too few useful ports, filtering, unknown fingerprint. | Check for an open and a closed TCP port; avoid forced certainty. |
| Unusual service version | Customized banner, reverse proxy, nonstandard port. | Appropriate protocol request and corroboration. |
| Very slow scan | Retries, loss, rate limiting, too many hosts/ports, expensive scripts. | Reduce scope and intensity; compare network conditions. |
| Insufficient privileges | Raw packet method unavailable or driver missing. | Check privileges and environment; use `-sT` when appropriate. |
| Different Linux/Windows outcomes | TCP/IP stack, firewall, interface, Npcap, routing, scan method. | Confirm the actual technique and observation point. |
| No traffic in Wireshark | Wrong interface, capture filter, namespace, offloading. | Route, interface, filter, and capture location. |

### Common interpretation errors

**“Ping did not reply, so the PC is off.”** Incorrect: a firewall may block ICMP while allowing services. **“Port 80 is open, so the server runs Apache.”** Incorrect: port numbers do not identify products. **“`-sA` found an open port.”** Incorrect: ACK scanning assesses filtering, not listening services. **“UDP is silent, so the port is closed.”** Incorrect: silence may produce `open|filtered`. **“`-A` is the most complete mode, so it is always best.”** Incorrect: it enables extra detection and scripts whose use should be justified.

Repeatable analysis preserves **negative and ambiguous findings**, too. Removing inconclusive scans from a report can make the environment appear more certain than the evidence supports.

## 28. Nmap command reference

This table summarizes options covered above. It is **not** a set of commands to run automatically against every network.

| Purpose | Main options |
| --- | --- |
| List selected targets | `-sL -n`, `-iL`, `--exclude`, `--excludefile` |
| Host discovery without an ordinary port scan | `-sn` |
| Skip host discovery | `-Pn` |
| ICMP Echo probe | `-PE` |
| TCP SYN/ACK discovery probes | `-PS`, `-PA` |
| UDP discovery probe | `-PU` |
| Local IPv4 ARP discovery | `-PR` |
| TCP SYN scan | `-sS` |
| TCP Connect scan | `-sT` |
| TCP ACK scan | `-sA` |
| UDP scan | `-sU` |
| Specific ports | `-p 22,80,443` |
| All TCP ports numbered 1–65535 | `-p-` with a TCP scan method |
| Common ports | `--top-ports N` |
| Service/version detection | `-sV`, `--version-light` |
| OS fingerprinting | `-O` |
| Specific NSE script | `--script name` |
| Display classification reasons | `--reason` |
| Limit probe rate | `--max-rate N` |
| Verbosity and trace | `-v`, `-d`, `--packet-trace` |
| Save scan results | `-oN`, `-oX`, `-oA` |
| IPv6 scanning | `-6` with appropriate IPv6 targets |

### Ports commonly associated with services

| Conventional endpoint | Commonly associated protocol or service | Further confirmation |
| --- | --- | --- |
| 22/tcp | SSH | Banner, SSH negotiation, server host-key fingerprint. |
| 53/udp and 53/tcp | DNS | Valid query, recursion/authority within scope. |
| 80/tcp | HTTP | HTTP response, headers, application behavior. |
| 123/udp | NTP | NTP response and service characteristics. |
| 139/tcp, 445/tcp | NetBIOS/SMB | Negotiation and SMB security policy. |
| 161/udp | SNMP | Replies only with authorized access and credentials. |
| 443/tcp | HTTPS or another TLS service | TLS negotiation and application data. |
| 3389/tcp | RDP | Service negotiation and access policy. |
| 8080/tcp | Often an alternative HTTP port | Verify the actual protocol. |

This list is not exhaustive, and assignments are configurable. IANA's registry, rather than a local services file alone, is the reference for officially registered port numbers.

## 29. Technical references

The following sources help verify terminology, behavior, and command options. **Official Nmap documentation** is the primary source for Nmap flags. IETF standards/RFCs describe network protocols, while operating system and Wireshark manuals support local inspection. The official Nmap GitHub repository is an implementation reference; its latest code does **not** mean the same behavior is available in every installed version. Always check the installed program's help before using a command.

**Nmap documentation**

- [Nmap — Reference Guide](https://nmap.org/book/man.html). Overview and option syntax.
- [Host Discovery](https://nmap.org/book/man-host-discovery.html). `-sn`, `-Pn`, ARP/ND, and discovery probes.
- [Host Discovery Techniques](https://nmap.org/book/host-discovery-techniques.html). Probe mechanisms and interpretation.
- [Target Specification](https://nmap.org/book/man-target-specification.html). Target files, exclusions, and list scans.
- [Port Scanning Basics](https://nmap.org/book/man-port-scanning-basics.html). The six Nmap port states.
- [Port Scanning Techniques](https://nmap.org/book/man-port-scanning-techniques.html). SYN, Connect, ACK, and special methods.
- [UDP Scan](https://nmap.org/book/scan-methods-udp-scan.html). UDP replies and ambiguity.
- [Service and Version Detection](https://nmap.org/book/man-version-detection.html). Application probes and intensity.
- [OS Detection](https://nmap.org/book/man-os-detection.html). TCP/IP stack fingerprinting.
- [Nmap Scripting Engine](https://nmap.org/book/man-nse.html). Script selection, arguments, and precautions.
- [NSEDoc — Reference Portal](https://nmap.org/nsedoc/). Documentation for available scripts.
- [Firewall/IDS Evasion and Spoofing](https://nmap.org/book/man-bypass-firewalls-ids.html). Mechanisms, limitations, and behavior.
- [Timing and Performance](https://nmap.org/book/man-performance.html). Timing templates, rates, retries, and timeouts.
- [Output](https://nmap.org/book/man-output.html). Verbosity, reasons, and formats.
- [XML Output](https://nmap.org/book/output-formats-xml-output.html). Structured output and import.

**Registries, standards, and network protocols**

- [IANA — Service Name and Port Number Registry](https://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.xhtml). Service names and port numbers.
- [IANA — IPv4 Special-Purpose Address Registry](https://www.iana.org/assignments/iana-ipv4-special-registry/iana-ipv4-special-registry.xhtml). Special-purpose and documentation address blocks.
- [RFC 791 — Internet Protocol](https://www.rfc-editor.org/rfc/rfc791). IPv4 structure and semantics; see subsequent updates.
- [RFC 9293 — Transmission Control Protocol](https://www.rfc-editor.org/rfc/rfc9293). Consolidated TCP specification.
- [RFC 826 — Address Resolution Protocol](https://www.rfc-editor.org/rfc/rfc826) and [RFC 4861 — Neighbor Discovery for IPv6](https://www.rfc-editor.org/rfc/rfc4861). Local link address resolution and discovery.
- [RFC 768 — User Datagram Protocol](https://www.rfc-editor.org/rfc/rfc768). UDP transport specification.

**Host inspection, packet captures, and original projects**

- [Wireshark — User's Guide](https://www.wireshark.org/docs/wsug_html_chunked/). Packet capture analysis and display filters.
- [Linux `ss(8)`](https://man7.org/linux/man-pages/man8/ss.8.html) and [`ip-route(8)`](https://man7.org/linux/man-pages/man8/ip-route.8.html). Local sockets and routes.
- [Microsoft — Get-NetTCPConnection](https://learn.microsoft.com/en-us/powershell/module/nettcpip/get-nettcpconnection), [Get-NetIPAddress](https://learn.microsoft.com/en-us/powershell/module/nettcpip/get-netipaddress), and [Test-NetConnection](https://learn.microsoft.com/en-us/powershell/module/nettcpip/test-netconnection). PowerShell commands.
- [Nmap — A Quick Port Scanning Tutorial](https://nmap.org/book/port-scanning-tutorial.html). Published examples with original outputs and explanations.
- [GitHub — Official `nmap/nmap` Project](https://github.com/nmap/nmap). Source code and documentation mirror.
- [Zenmap — Official GUI](https://nmap.org/book/zenmap.html). Display and management of Nmap results.
- [Nmap — Host Discovery and Traceroute](https://nmap.org/book/man-host-discovery.html). Observed paths, `--traceroute`, and limitations.
- [arp-scan — Original Repository](https://github.com/royhills/arp-scan). ARP scanner for local IPv4 hosts.
- [Netdiscover — Official Repository](https://github.com/netdiscover-scanner/netdiscover). ARP-based discovery on Ethernet networks.
- [RFC 792 — Internet Control Message Protocol](https://www.rfc-editor.org/rfc/rfc792). ICMPv4 types and codes.
- [RFC 8200 — IPv6 Specification](https://www.rfc-editor.org/rfc/rfc8200). IPv6 headers, Hop Limit, and fragmentation.
- [Nmap — Windows Installation](https://nmap.org/book/inst-windows.html). Npcap support, interface limitations, and TCP Connect scanning.


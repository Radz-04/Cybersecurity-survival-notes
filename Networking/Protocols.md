# Network Protocols

A **protocol** defines rules for exchanging and interpreting data. Several protocols usually take part in the same communication: they handle different jobs, such as resolving a name, carrying application data, or forwarding a packet.

The layer names below refer to the TCP/IP model described in [Networking Basics](./Basics.md).

## Common protocols and ports

| Protocol | TCP/IP layer | Purpose | Common service ports / transport |
| --- | --- | --- | --- |
| HTTP | Application | Web requests and responses. | TCP 80 for conventional unencrypted HTTP |
| HTTPS | Application | HTTP over a secure connection. | TCP 443; UDP 443 for HTTP/3 |
| FTP | Application | File transfer using separate control and data connections. | TCP 21 for control |
| SMTP | Application | Email transfer and message submission. | TCP 25 for mail transfer; TCP 587 for submission |
| IMAP | Application | Access to messages stored on a mail server. | TCP 143; TCP 993 with implicit TLS |
| DNS | Application | Retrieval of records associated with domain names, including IP address records. | UDP/TCP 53 for conventional DNS |
| DHCP | Application | Automatic network configuration. | UDP 67 for servers and UDP 68 for clients in DHCPv4 |
| SSH | Application | Secure remote access and encrypted communication. | TCP 22 |
| NTP | Application | Clock synchronization. | UDP 123 |
| SNMP | Application | Network device monitoring and management. | Commonly UDP 161 for agents and UDP 162 for notification receivers |
| TCP | Transport | A reliable, ordered byte stream. | Uses TCP port numbers |
| UDP | Transport | Datagrams without built-in delivery or ordering guarantees. | Uses UDP port numbers |
| QUIC | Transport | Secure transport with multiple streams, carried over UDP. | Runs over UDP |
| IP | Internet | Logical addressing and packet delivery across networks. | No TCP/UDP ports |
| ICMP | Internet | Network control, diagnostic, and error messages. | No TCP/UDP ports |
| ARP | Link / local network | Resolution of IPv4 addresses to link-layer addresses on supported local networks. | No TCP/UDP ports |

These are selected common service ports. Other configurations are possible, and some protocols use additional connections. FTP, for example, uses separate data connections as well as its control connection.

A familiar port number alone does not establish which application is using it or whether the traffic is legitimate.

## What happens when you open an HTTPS website?

If an address is not already available, DNS can be used to find an IP address associated with the site's domain name. IP provides the addressing and forwarding needed to reach the server.

The browser uses the transport and security mechanisms for the selected HTTP version:

- HTTPS with HTTP/1.1 or HTTP/2 commonly uses TLS over TCP.
- HTTP/3 uses QUIC over UDP. QUIC uses TLS 1.3 for authentication and key negotiation, and provides its own packet protection.

The browser sends an HTTP request, and the server returns an HTTP response through the protected connection.

DNS, IP, the transport protocol, and HTTP all contribute to this exchange. They perform different parts of the job.

## TCP and UDP

TCP provides a reliable, ordered byte stream. Sequence numbers, acknowledgments, and retransmissions allow it to handle loss and deliver data in order.

UDP sends datagrams without providing those delivery or ordering guarantees. Applications and protocols using UDP can add the mechanisms they need.

Choosing UDP does not automatically make an application faster. Performance depends on the application, network conditions, and the protocols involved.

TCP does not provide built-in encryption. Encryption requires a suitable security protocol or mechanism.

## Reading traffic

Protocol choice affects what can be observed in a capture. Traditional FTP sends data without encryption by default; SSH provides encrypted communication. HTTP without TLS and HTTPS also provide different protection for application data.

Wireshark can help follow these exchanges and identify the protocols present in captured traffic.

## References

- [IANA — Service Name and Transport Protocol Port Number Registry](https://www.iana.org/assignments/service-names-port-numbers/)
- [RFC 9293 — TCP](https://www.rfc-editor.org/rfc/rfc9293.html)
- [RFC 768 — UDP](https://www.rfc-editor.org/rfc/rfc768.html)
- [RFC 9000 — QUIC](https://www.rfc-editor.org/rfc/rfc9000.html)
- [RFC 9001 — Using TLS to Secure QUIC](https://www.rfc-editor.org/rfc/rfc9001.html)
- [RFC 9114 — HTTP/3](https://www.rfc-editor.org/rfc/rfc9114.html)
- [RFC 6409 — Message Submission for Mail](https://www.rfc-editor.org/rfc/rfc6409.html)
- [RFC 9051 — IMAP](https://www.rfc-editor.org/rfc/rfc9051.html)
- [RFC 2131 — DHCP](https://www.rfc-editor.org/rfc/rfc2131.html)
- [RFC 826 — Address Resolution Protocol](https://www.rfc-editor.org/rfc/rfc826.html)

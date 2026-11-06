# Network Addressing and Communication

When devices communicate over a network, different types of addresses are used to deliver data to the correct destination.

Three important elements are:

- **MAC addresses:** used for communication at the data-link layer.
- **IP addresses:** used to identify network interfaces and route packets across IP networks.
- **Port numbers:** used by transport protocols such as TCP and UDP to identify communication endpoints.

Each serves a different purpose.

## MAC Addresses

A **Media Access Control (MAC) address** is a link-layer address used by network technologies such as Ethernet.

Ethernet MAC addresses are typically 48 bits long and written as six hexadecimal pairs, for example:

`00:1A:2B:3C:4D:5E`

MAC addresses can be universally or locally administered. A universally administered address is intended to be globally unique, while a locally administered address does not necessarily have global uniqueness.

Ethernet switches use MAC addresses to forward frames within a local network.

### ARP (Address Resolution Protocol)

On an IPv4 Ethernet network, a device may need to discover the MAC address associated with a particular IPv4 address.

**ARP** provides this address resolution.

For example, suppose two computers are connected to the same LAN:

- **PC-A:** `192.168.1.10`
- **PC-B:** `192.168.1.20`

If PC-A wants to communicate with PC-B, it can use ARP to discover PC-B's MAC address, provided the mapping is not already known.

When communicating with a device outside its local network, PC-A normally sends the Ethernet frame to its next-hop router instead. In that case, ARP can be used to discover the router's MAC address.

IPv6 uses **Neighbor Discovery (ND)** for link-layer address resolution rather than ARP.

## IP Addresses

An **Internet Protocol (IP) address** provides logical addressing for communication within and between IP networks.

There are two main versions:

- **IPv4:** uses 32-bit addresses, such as `192.168.1.10`.
- **IPv6:** uses 128-bit addresses, such as `2001:db8::1`.

IP addresses are used by routers to forward packets toward their destinations.

Unlike MAC addresses, which are used for delivery over an individual link, IP addresses allow packets to travel across multiple interconnected networks.

## Port Numbers

An IP address identifies a network destination, but a device may run several network services simultaneously.

TCP and UDP use **port numbers** to distinguish communication endpoints.

For example:

- **HTTP:** commonly uses TCP port `80`.
- **HTTPS:** commonly uses TCP port `443`; HTTP/3 uses QUIC over UDP, typically on port `443`.
- **SSH:** commonly uses TCP port `22`.

A TCP or UDP communication uses both source and destination port numbers.

Port numbers range from `0` to `65535`.

IANA divides them into three categories:

| Category | Range |
|---|---|
| System Ports | 0–1023 |
| User Ports | 1024–49151 |
| Dynamic/Private Ports | 49152–65535 |

These ranges describe port-number assignment categories. They do not guarantee that a particular service is running on its conventional port.

Not every network protocol uses ports. For example, ICMP does not use TCP or UDP port numbers.

## Example: Accessing a Website

Consider a computer accessing a website outside its local network, using HTTPS over TCP.

1. **DNS resolution:** If necessary, the computer resolves the website's domain name to an IP address.
2. **Routing decision:** The operating system determines how to reach the destination, usually through the default gateway.
3. **Address resolution:** On an IPv4 Ethernet LAN, ARP can be used to find the gateway's MAC address if it is not already known.
4. **Local delivery:** The computer sends an Ethernet frame addressed to the gateway's MAC address. Inside the frame, the IP packet is addressed to the remote server.
5. **Packet forwarding:** Routers forward the IP packet toward the destination.
6. **Application communication:** TCP uses source and destination ports to identify the communication endpoints. The browser and web server exchange HTTPS traffic over the established secure connection.

>**Key point:** The destination IP address identifies the remote endpoint, while the destination MAC address is used to reach the next device on the local Ethernet link.

## References

- [IEEE — Guidelines for Use of EUI, OUI, and CID](https://standards.ieee.org/wp-content/uploads/import/documents/tutorials/eui.pdf)
- [RFC 826 — Address Resolution Protocol](https://www.rfc-editor.org/rfc/rfc826.html)
- [RFC 4861 — Neighbor Discovery for IPv6](https://www.rfc-editor.org/rfc/rfc4861.html)
- [RFC 791 — Internet Protocol (IPv4)](https://www.rfc-editor.org/rfc/rfc791.html)
- [RFC 8200 — IPv6 Specification](https://www.rfc-editor.org/rfc/rfc8200.html)
- [RFC 9293 — Transmission Control Protocol](https://www.rfc-editor.org/rfc/rfc9293.html)
- [RFC 768 — User Datagram Protocol](https://www.rfc-editor.org/rfc/rfc768.html)
- [IANA — Service Name and Transport Protocol Port Number Registry](https://www.iana.org/assignments/service-names-port-numbers)

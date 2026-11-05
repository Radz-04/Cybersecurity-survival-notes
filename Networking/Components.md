# Network Components

A network includes devices that send and receive data, devices that forward traffic, and software that provides services or manages access.

## Hosts and network interfaces

An **end device**, or **host**, acts as a source or destination of network communication. Computers, smartphones, printers, and servers are common examples.

A **network interface** connects a system to a network. A Network Interface Card (NIC) is a hardware implementation, such as an Ethernet adapter or Wi-Fi adapter. Interfaces can also be virtual and implemented in software.

On Ethernet networks, a **MAC address** provides link-layer addressing. An **IP address** provides logical addressing for communication within and across IP networks.

MAC addresses are often assigned by the manufacturer, but they can also be changed or locally administered.

## Routers

An IP router forwards packets between networks. It uses the destination IP address and its routing information to choose where to forward a packet.

IP routing is associated with Layer 3 of the OSI model.

On a home network, the router usually acts as the default gateway used to reach destinations outside the local network.

## Switches

A traditional Layer 2 Ethernet switch connects devices in a LAN and forwards frames using MAC addresses.

It learns dynamic entries from the **source MAC addresses** of received frames. Its MAC address table associates addresses with ports and VLANs.

For a typical unicast frame, forwarding uses the **destination MAC address**:

- If the destination is known on another port, the frame is forwarded through that port.
- If the destination is unknown, the frame is normally flooded through the other eligible ports in the same VLAN.
- If the destination is associated with the incoming port, the frame is filtered rather than sent back through that port.

Broadcast frames are normally forwarded through the other eligible ports in the same VLAN.

Some switches also support IP routing. These are commonly called Layer 3 or multilayer switches.

## Wireless access points

A **wireless access point (AP)** provides Wi-Fi connectivity. In a common deployment, it bridges wireless clients to a wired LAN.

A phone can be connected to an AP and communicate with local devices even when the Internet connection is down. Wi-Fi connectivity and Internet access are separate.

## Hubs

A **hub** is a legacy Ethernet device that repeats incoming signals to its other ports. It operates at Layer 1, the Physical layer.

It does not inspect destination MAC addresses or maintain a MAC address table. Devices connected through a traditional Ethernet hub share the same collision domain.

Switches have largely replaced hubs in modern Ethernet networks.

## Modems

A **modem**, short for modulator-demodulator, modulates and demodulates signals to transmit and receive data over a communication link.

For Internet access, a modem handles the signaling required by its access technology. A router handles IP forwarding between networks.

A home networking device may combine these functions in one unit.

## Transmission media

Copper cables, fiber optics, and wireless links carry data between devices. Their characteristics affect distance, bandwidth, interference, and installation requirements.

See [Data Transmission](./Transmission.md) for signal types, media, and transmission modes.

## Network management software

Administrators use management tools to monitor and maintain network infrastructure. Common tasks include:

- Monitoring performance, traffic, and resource usage.
- Managing device configurations and tracking changes.
- Detecting faults and investigating failures.
- Controlling administrative access and monitoring security-related events.

## Firewalls

A **firewall** controls network traffic according to configured rules and policies. It can run on a network appliance or on an individual computer.

A **host-based firewall** controls traffic entering or leaving that system. For example, it can allow incoming SSH connections from an authorized network while blocking connections that do not match the permitted rules.

A firewall's effectiveness depends on its configuration and capabilities. It does not automatically detect every attack.

### Linux example: nftables and iptables

Linux provides packet-filtering facilities through its networking infrastructure. **nftables** supports filtering, packet classification, and Network Address Translation (NAT). Its command-line utility is `nft`.

`sudo nft list ruleset` displays the configured nftables rules without changing them.

**iptables** is an older interface that remains available in some environments. Compatibility tools can use iptables syntax over the nftables infrastructure. Tools such as `firewalld` provide higher-level management.

The available tools depend on the distribution and configuration.

## Servers

A **server** is a program or system that provides a service or resource to clients. Server software can run on physical or virtual systems.

Client and server describe roles. When a laptop requests a web page, the browser acts as a client and the web server processes the request and returns a response.

| Server type | Service |
| --- | --- |
| Web | Handles HTTP requests and serves web resources. |
| File | Provides access to shared files. |
| Mail | Handles email services. |
| Database | Provides access to data managed by a database system. |
| Authentication | Provides authentication-related services. |

See [Network Protocols](./Protocols.md) for the protocols used by these services.

## References

- [Cisco — Switches and routers](https://www.cisco.com/site/us/en/learn/topics/small-business/network-switch-vs-router.html)
- [Cisco — MAC address learning and forwarding](https://www.cisco.com/c/en/us/td/docs/switches/lan/c9000/lyr2-fwd/cdp-lldp-mac-udld/cdp-lldp-mac-udld-configuration-guide/c-configure-mac.html)
- [Cisco — Wireless access points](https://www.cisco.com/site/us/en/learn/topics/small-business/what-is-an-access-point.html)
- [Cisco — Troubleshooting Ethernet](https://www.cisco.com/en/US/docs/internetworking/troubleshooting/guide/tr1904.html)
- [Cisco — Network management](https://www.cisco.com/c/en/us/support/docs/availability/high-availability/15114-NMS-bestpractice.html)
- [IBM — Glossary: modem](https://www.ibm.com/docs/en/i/7.5.0?topic=glossary-m)
- [IBM — Client/server model](https://www.ibm.com/docs/en/cics-ts/5.5.0?topic=programs-clientserver-model)
- [NIST SP 800-41 Rev. 1 — Firewalls and firewall policy](https://csrc.nist.gov/pubs/sp/800/41/r1/final)
- [Netfilter — nftables](https://netfilter.org/projects/nftables/)

[Back to Networking](./README.md)

# Networking Basics

**Networking** is the process of connecting two or more devices so they can communicate and share data.

For devices to communicate over a network, they need:

- At least two nodes, such as computers or other devices.
- Connectivity through a transmission medium, such as Ethernet cables or wireless signals.
- Compatible communication protocols: rules that allow devices to exchange and interpret data.

## Nodes and Links

A **node** is a device connected to a network that can send, receive, or forward data.

A **link** is a physical or logical connection between neighboring network nodes.

## Types of Networks

Computer networks can be classified according to their geographical coverage and the way they interconnect devices and other networks.

Two common categories are **Local Area Networks (LANs)** and **Wide Area Networks (WANs)**.

### Local Area Network (LAN)

A **Local Area Network (LAN)** connects devices within a limited geographical area, such as a house, office, school, or university building.

A LAN allows connected devices to exchange data and access shared resources, such as printers, file servers, and other local services.

Common characteristics:

- **Limited geographical scope:** usually operates within a home, building, or group of nearby buildings.
- **Local administration:** typically managed by one person or organization.
- **Connectivity:** devices may connect through Ethernet or Wi-Fi.
- **Resource sharing:** devices can communicate and share local resources without necessarily requiring an Internet connection.

**Example:**

A computer connected through Ethernet and a laptop connected through Wi-Fi can belong to the same home LAN.

If properly configured, they can communicate with each other even when the Internet connection is unavailable.

### Wide Area Network (WAN)

A **Wide Area Network (WAN)** provides connectivity across larger geographical areas and can interconnect networks located in different cities, regions, or countries.

WAN connections may use infrastructure provided by telecommunications companies or Internet Service Providers (ISPs).

Common characteristics:

- **Large geographical coverage:** connects distant locations and networks.
- **Interconnection:** enables communication between separate LANs and other networks.
- **Infrastructure:** can use fiber-optic connections, leased lines, cellular networks, and satellite links.
- **Administration:** may involve multiple organizations and service providers.

**Example:**

A company with offices in Rome and Milan could use a WAN connection to allow both locations to access shared business applications and services.

The Internet is a global interconnection of networks and is a familiar example of wide-area connectivity.

### How LANs and WANs Work Together

A LAN provides connectivity within a local environment, while a WAN allows communication with networks beyond that environment.

Consider a home network:

1. A laptop connects to the home LAN using Wi-Fi.
2. For a destination reached through its default route, the laptop sends packets to its default gateway, typically the home router.
3. The router forwards the packets toward the ISP.
4. The packets travel across other networks until they reach their destination.

The router connects the local network to external networks and forwards IP packets between them.

**Important:** A WAN is not automatically slower than a LAN. Actual performance depends on factors such as bandwidth, latency, network equipment, and the technologies used.

## Network Types vs. Network Topologies

Network type and network topology describe different characteristics.

- **Network type:** describes characteristics such as the geographical scope of a network, as with LAN and WAN.
- **Network topology:** describes how network nodes and connections are arranged, physically or logically.

Examples of network topologies include star, bus, ring, mesh, and tree.

A LAN or WAN can use different topologies depending on its design and requirements.

## Sources

- [IBM — What Is Computer Networking?](https://www.ibm.com/think/topics/networking)
- [Cisco — What Is a LAN?](https://www.cisco.com/site/us/en/learn/topics/networking/what-is-a-lan-local-area-network.html)
- [Cisco — What Is a WAN?](https://www.cisco.com/site/us/en/learn/topics/networking/what-is-a-wan-wide-area-network.html)

# DHCP (Dynamic Host Configuration Protocol)

DHCP (Dynamic Host Configuration Protocol) is a network protocol that allows devices to automatically obtain an IP address and other network configuration settings.

Without DHCP, a network administrator would have to manually configure the IP address, subnet mask, default gateway, and DNS servers on each device.

This is manageable in a small network, but it becomes time-consuming when hundreds of devices need to be configured.

DHCP makes this process easier by automatically assigning network settings to devices when they connect to a network.

## DHCP Client and Server

DHCP works using a client-server model.

| Component | Description |
| --- | --- |
| DHCP Client | A device that requests an IP address and network configuration. |
| DHCP Server | A device or service responsible for assigning IP addresses and network settings. |
| DHCP Pool | A range of IP addresses available for assignment. |
| Lease | The period during which a device is allowed to use an assigned IP address. |

A DHCP server can provide:

- IP address
- Subnet mask
- Default gateway
- DNS server addresses
- Lease duration

The DHCP server is often integrated into a home router, but it can also run on a dedicated server.

## How DHCP Works

When a device connects to a network and needs an IPv4 address, it usually follows a four-step process called DORA.

DORA stands for:

- Discover
- Offer
- Request
- Acknowledge

### 1. Discover

The DHCP client sends a DHCPDISCOVER message to find available DHCP servers.

Since the client may not have an IP address yet, the message is normally sent as a broadcast.

This allows DHCP servers on the local network to receive the request.

### 2. Offer

A DHCP server receives the Discover message and responds with a DHCPOFFER.

The server proposes an available IP address and other configuration settings.

If multiple DHCP servers respond, the client can choose one of the offers.

### 3. Request

The client sends a DHCPREQUEST message to indicate which offer it wants to accept.

During the initial address assignment, this message is normally broadcast so that the other DHCP servers know which offer was selected.

### 4. Acknowledge

The selected server sends a DHCPACK message confirming the assignment.

The client can then configure its network interface using the information provided by the server.

## Example: Connecting a Laptop to a Network

Imagine a laptop connecting to an office network.

The network has the following configuration:

| Setting | Value |
| --- | --- |
| Network | 192.168.10.0/24 |
| Default Gateway | 192.168.10.1 |
| DHCP Pool | 192.168.10.100 - 192.168.10.199 |
| DNS Server | 192.168.10.1 |
| Lease Time | 24 hours |

The process works as follows:

1. The laptop connects to the network and sends a DHCPDISCOVER.
2. The server offers the address 192.168.10.105.
3. The laptop sends a DHCPREQUEST to accept the address.
4. The server sends a DHCPACK confirming the assignment.

The laptop can now use the assigned network configuration.

These addresses are examples, not results from a real network capture.

## DHCP Lease

IP addresses assigned through DHCP are usually temporary.

The period during which a client can use an assigned address is called the lease time.

For example, a DHCP server might assign an address for 24 hours.

The client does not normally wait until the lease expires to renew it.

The renewal process typically involves two timers:

| Timer | Description |
| --- | --- |
| T1 | Usually 50% of the lease duration. The client attempts to renew directly with the original DHCP server. |
| T2 | Usually 87.5% of the lease duration. If renewal fails, the client attempts to contact other available DHCP servers. |

For a 24-hour lease:

- T1 occurs after approximately 12 hours.
- T2 occurs after approximately 21 hours.

These values can be configured by the DHCP server.

If the lease expires without a successful renewal, the client must stop using that address.

A successful renewal does not necessarily change the client's IP address.

## DHCP Ports

DHCPv4 uses UDP.

| Device | UDP Port |
| --- | --- |
| DHCP Server | 67 |
| DHCP Client | 68 |

During the initial configuration, the client commonly uses broadcasts because it does not yet know the DHCP server's address.

DHCP messages can also use unicast in other situations, such as lease renewal.

## DHCP Relay

DHCP broadcasts normally remain within the local network.

If the DHCP server is located on another subnet, a DHCP relay agent can forward DHCP messages between the client and the server.

This allows a central DHCP server to provide addresses to devices located on different networks or VLANs.

## DHCP and IP Conflicts

DHCP reduces the risk of IP address conflicts by keeping track of addresses assigned from its pool.

However, DHCP does not completely prevent conflicts.

For example, a computer configured with a static IP address could use an address already assigned by the DHCP server.

Network administrators must therefore plan DHCP pools and static address assignments carefully.

## Analyzing DHCP Traffic

Wireshark can be used to observe DHCP messages.

A useful display filter for DHCPv4 is:

    udp.port == 67 || udp.port == 68

During an initial assignment, look for:

1. DHCP Discover
2. DHCP Offer
3. DHCP Request
4. DHCP ACK

Inspecting these packets helps explain how a device receives its network configuration.

## References

- [RFC 2131 — Dynamic Host Configuration Protocol](https://www.rfc-editor.org/rfc/rfc2131.html)
- [RFC 2132 — DHCP Options](https://www.rfc-editor.org/rfc/rfc2132.html)
- [Cisco — Dynamic Host Configuration Protocol](https://www.cisco.com/c/en/us/td/docs/switches/lan/c9000/lyr3-fwd/dhcp/dhcp-configuration-guide/dynamic-host-configuration-protocol.html)
- [Microsoft Learn — DHCP Basics](https://learn.microsoft.com/en-us/windows-server/troubleshoot/dynamic-host-configuration-protocol-basics)

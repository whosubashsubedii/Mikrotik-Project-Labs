# IPv6-Only WireGuard VPN with NAT66 Using MikroTik RouterOS and GNS3

A practical networking project that demonstrates how **IPv6, WireGuard, ULA addressing, NAT66, and MikroTik RouterOS** can be combined to build a secure and cost-effective remote-access VPN, especially in environments where IPv4 connectivity is limited by **Carrier-Grade NAT (CGNAT)**.

> **Lab status:** Successfully implemented and tested using **MikroTik CHR in GNS3**.  
> **Physical deployment:** Not performed as part of this project.

---

## Project Overview

Many ISPs place customers behind **IPv4 CGNAT**, which means the customer does not receive a directly reachable public IPv4 address. This can make inbound services such as self-hosted VPN servers difficult to deploy.

This project explores an alternative approach by using a **globally routable IPv6 address** as the public WireGuard endpoint.

Inside the VPN tunnel:

- The MikroTik WireGuard interface uses a **ULA IPv6 /64**
- Each remote client receives its own **/128 ULA address**
- **NAT66** translates client ULA traffic to the MikroTik WAN IPv6 address
- IPv6 firewall rules control WireGuard and forwarded VPN traffic

The objective is to demonstrate a low-cost remote-access design that could be useful for **small and medium-sized businesses (SMEs)** with IPv6-capable Internet connections.

---

## Problem Being Solved

### IPv4 with CGNAT

```text
Remote User
     |
     | Internet
     v
Shared Public IPv4
     |
  ISP CGNAT
     |
     v
Customer Private / Shared IPv4
     |
     v
Office Router
```

Because the ISP controls the public IPv4 translation, directly accepting inbound VPN connections can be difficult.

### IPv6-Based Approach

```text
Remote Client
      |
      | IPv6 Internet
      v
Global IPv6 on MikroTik
      |
      | WireGuard UDP/51820
      v
WireGuard Tunnel
      |
      v
ULA VPN Network
      |
      | NAT66
      v
IPv6 Internet / Office Resources
```

---

## Technologies Used

- **MikroTik RouterOS**
- **MikroTik CHR**
- **GNS3**
- **IPv6**
- **WireGuard**
- **ULA (Unique Local Addressing)**
- **NAT66**
- **IPv6 Firewall**
- **ICMPv6**
- **Torch / Packet Sniffer**
- **Wireshark** for packet-level analysis

---

## IPv6 Addressing Plan

| Device / Role | Interface | IPv6 Address | Purpose |
|---|---|---|---|
| MikroTik CHR | `ether1` | Global IPv6 from ISP/upstream | Public WireGuard endpoint |
| MikroTik CHR | `WG-VPN-IPv6` | `fd7a:115c:a1e0:92::1/64` | VPN gateway |
| Remote Client | WireGuard | `fd7a:115c:a1e0:92::2/128` | VPN client |

WireGuard ULA prefix:

```text
fd7a:115c:a1e0:92::/64
```

---

## Implementation Overview

1. Configure IPv6 WAN connectivity
2. Verify the global IPv6 address
3. Verify the IPv6 default route
4. Test native IPv6 Internet connectivity
5. Create the WireGuard interface
6. Assign the ULA IPv6 gateway
7. Create the WireGuard client peer
8. Generate the client configuration / QR code
9. Configure IPv6 firewall rules
10. Configure NAT66
11. Verify the WireGuard handshake
12. Test IPv6 connectivity through the tunnel
13. Verify NAT66 counters
14. Analyze traffic using RouterOS tools and Wireshark

---

# Configuration

## 1. Configure IPv6 WAN

The WAN interface requests an IPv6 address from the upstream network using DHCPv6 and automatically installs the IPv6 default route.

```routeros
/ipv6 dhcp-client
add interface=ether1 \
    request=address \
    add-default-route=yes \
    use-peer-dns=yes \
    use-interface-duid=yes \
    comment="DHCPv6 Client on WAN"
```

Verify:

```routeros
/ipv6 dhcp-client print detail
/ipv6 address print detail
/ipv6 route print detail
```

Test IPv6 connectivity:

```routeros
/ping 2606:4700:4700::1111 count=5
/ping 2001:4860:4860::8888 count=5
```

---

## 2. Create WireGuard Interface

```routeros
/interface wireguard
add name=WG-VPN-IPv6 \
    listen-port=51820 \
    mtu=1420 \
    comment="IPv6 WireGuard VPN"
```

Verify:

```routeros
/interface wireguard print detail
```

---

## 3. Assign ULA IPv6 Address

```routeros
/ipv6 address
add address=fd7a:115c:a1e0:92::1/64 \
    interface=WG-VPN-IPv6 \
    advertise=no \
    comment="WireGuard IPv6 Gateway"
```

Verify:

```routeros
/ipv6 address print detail where interface=WG-VPN-IPv6
```

---

## 4. Create WireGuard Peer

> Replace `<GLOBAL-IPv6-ADDRESS>` with the globally reachable IPv6 address available on the MikroTik WAN interface.

```routeros
/interface wireguard peers
add interface=WG-VPN-IPv6 \
    name=Remote-Client-01 \
    private-key=auto \
    allowed-address=fd7a:115c:a1e0:92::2/128 \
    client-address=fd7a:115c:a1e0:92::2/128 \
    client-allowed-address=::/0 \
    client-dns=2606:4700:4700::1111 \
    client-endpoint="[<GLOBAL-IPv6-ADDRESS>]" \
    client-keepalive=25 \
    responder=yes
```

Verify:

```routeros
/interface wireguard peers print detail
```

---

## 5. Generate Client Configuration / QR Code

```routeros
/interface wireguard peers
show-client-config Remote-Client-01
```

The generated client configuration should follow this format:

```ini
[Interface]
PrivateKey = <CLIENT-PRIVATE-KEY>
Address = fd7a:115c:a1e0:92::2/128
DNS = 2606:4700:4700::1111

[Peer]
PublicKey = <MIKROTIK-PUBLIC-KEY>
AllowedIPs = ::/0
Endpoint = [<GLOBAL-IPv6-ADDRESS>]:51820
PersistentKeepalive = 25
```

### Important IPv6 Endpoint Format

Correct:

```text
[2400:xxxx:xxxx::5]:51820
```

Incorrect:

```text
2400:xxxx:xxxx::5:51820
```

Incorrect:

```text
[2400:xxxx:xxxx::5]:51820:51820
```

---

## 6. IPv6 Firewall Rules

### Allow WireGuard on the WAN

```routeros
/ipv6 firewall filter
add chain=input \
    action=accept \
    protocol=udp \
    dst-port=51820 \
    in-interface=ether1 \
    comment="Allow WireGuard IPv6"
```

### Allow WireGuard Clients to Reach the Router

```routeros
/ipv6 firewall filter
add chain=input \
    action=accept \
    src-address=fd7a:115c:a1e0:92::/64 \
    in-interface=WG-VPN-IPv6 \
    comment="Allow WG clients to router"
```

### Allow IPv6 Forwarding from WireGuard

```routeros
/ipv6 firewall filter
add chain=forward \
    action=accept \
    src-address=fd7a:115c:a1e0:92::/64 \
    in-interface=WG-VPN-IPv6 \
    comment="Allow WG IPv6 forwarding"
```

> These are the minimum rules used for the lab. A production deployment should also include proper stateful firewall rules, essential ICMPv6 handling, management restrictions, and a default-deny policy where appropriate.

---

## 7. Configure NAT66

The WireGuard clients use ULA IPv6 addresses, which are not globally routable. NAT66 translates the client source address to the MikroTik WAN IPv6 address.

```routeros
/ipv6 firewall nat
add chain=srcnat \
    src-address=fd7a:115c:a1e0:92::/64 \
    out-interface=ether1 \
    action=masquerade \
    comment="NAT66 WireGuard to WAN"
```

Verify:

```routeros
/ipv6 firewall nat print detail
/ipv6 firewall nat print stats
```

---

# Verification

## Check WireGuard Handshake

```routeros
/interface wireguard peers print detail
```

Check:

```text
last-handshake
current-endpoint-address
current-endpoint-port
rx
tx
```

A recent handshake and increasing RX/TX counters indicate that the tunnel is active.

---

## Test VPN Connectivity

From MikroTik:

```routeros
/ping fd7a:115c:a1e0:92::2 count=5
```

From the remote client:

```bash
ping fd7a:115c:a1e0:92::1
```

---

## Test IPv6 Internet Through the VPN

From the client:

```bash
ping 2606:4700:4700::1111
```

or:

```bash
ping 2001:4860:4860::8888
```

If the client can reach these addresses and the NAT66 counters increase, IPv6 Internet traffic is successfully passing through the WireGuard tunnel.

---

## Verify NAT66 Counters

```routeros
/ipv6 firewall nat print stats
```

The `BYTES` and `PACKETS` values should increase when the VPN client generates IPv6 traffic.

---

## Traffic Analysis

### Torch

```routeros
/tool torch WG-VPN-IPv6
/tool torch ether1
```

### Packet Sniffer

```routeros
/tool sniffer quick interface=WG-VPN-IPv6
/tool sniffer quick interface=ether1
```

These tools help compare traffic before and after NAT66.

---

# Troubleshooting

## WireGuard Has No Handshake

Check:

```routeros
/interface wireguard print detail
/interface wireguard peers print detail
/ipv6 firewall filter print detail
```

Verify:

- Correct global IPv6 endpoint
- UDP port `51820`
- Correct peer keys
- Correct `allowed-address`
- Remote client has working IPv6 connectivity
- IPv6 firewall permits WireGuard traffic

---

## QR Code Cannot Be Imported

A common error is:

```text
Cannot parse endpoint
```

The endpoint must use IPv6 bracket notation:

```text
[GLOBAL-IPv6-ADDRESS]:51820
```

Regenerate the client configuration after correcting the peer:

```routeros
/interface wireguard peers show-client-config Remote-Client-01
```

---

## Tunnel Works but Internet Does Not

Check the routing table:

```routeros
/ipv6 route print detail
```

The router should have:

```text
::/0
```

and a connected route for:

```text
fd7a:115c:a1e0:92::/64
```

Also test the MikroTik itself:

```routeros
/ping 2606:4700:4700::1111 count=5
```

---

## NAT66 Counters Stay at Zero

Check:

```routeros
/ipv6 firewall nat print detail
/ipv6 firewall nat print stats
```

Verify:

- Source prefix is `fd7a:115c:a1e0:92::/64`
- WAN interface is `ether1`
- Client address is correct
- IPv6 forwarding is allowed
- Client uses `AllowedIPs = ::/0`
- IPv6 default route is active

---

# Test Results

| Test | Result |
|---|---|
| IPv6 WAN Connectivity | Successful |
| Global IPv6 Address | Successfully Assigned |
| IPv6 Default Route | Active |
| WireGuard Interface | Running |
| WireGuard Handshake | Successful |
| VPN IPv6 Communication | Successful |
| IPv6 Internet Access | Successful |
| NAT66 Translation | Successful |
| NAT66 Counters | Increasing |
| IPv6 Firewall Rules | Working |

---

# Real-World Use for SMEs

This project demonstrates that an organization with:

- Existing IPv6-capable broadband
- A compatible MikroTik router
- Basic networking knowledge

may be able to create secure remote access without purchasing a public IPv4 service only for VPN connectivity.

Potential use cases include:

- Router and network management
- Internal web applications
- Monitoring systems
- File services
- Server administration
- Remote access to authorized office resources

This does **not** replace dedicated business Internet in every situation. Organizations requiring guaranteed bandwidth, static addressing, SLA support, high availability, or enterprise redundancy may still require dedicated connectivity.

---

# Limitations

- The project was implemented and tested in **GNS3 using MikroTik CHR**
- A physical MikroTik deployment was **not** performed
- The remote client must have working IPv6 connectivity
- The ISP must provide a globally reachable IPv6 address and permit inbound WireGuard traffic
- NAT66 is used because the VPN clients use ULA addressing
- If the ISP provides a usable delegated global IPv6 prefix, native IPv6 routing may be preferred over NAT66
- Dynamic WAN IPv6 addressing may require an additional endpoint update or DDNS strategy

---

# Security Notes

- Never publish WireGuard **private keys**
- Never publish a client QR code containing VPN credentials
- Use a separate WireGuard peer/key for each user or device
- Restrict management access in production
- Keep RouterOS updated
- Maintain secure configuration backups
- Monitor VPN sessions and firewall activity

---

# Conclusion

The project successfully demonstrates an **IPv6-only WireGuard remote-access VPN with NAT66** using MikroTik RouterOS in GNS3.

The implementation confirms that:

```text
IPv6
   +
WireGuard
   +
ULA Addressing
   +
NAT66
   +
MikroTik RouterOS
```

can be combined to provide secure remote connectivity without relying on a directly reachable public IPv4 address.

The project also provided practical experience with:

- IPv6 WAN configuration
- IPv6 routing
- WireGuard peer management
- ULA addressing
- NAT66
- IPv6 firewall configuration
- VPN troubleshooting
- Traffic analysis
- GNS3-based network validation

---

## Author

**Subash Subedi**  
Networking & IT Security

GitHub: [whosubashsubedii](https://github.com/whosubashsubedii)

---

> **Disclaimer:** This repository documents a lab implementation for learning, testing, and technical demonstration. Review and harden the configuration before using it in a production environment.

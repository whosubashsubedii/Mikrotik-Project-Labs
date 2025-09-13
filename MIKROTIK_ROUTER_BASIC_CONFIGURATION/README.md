

# Basic MikroTik Router Configuration

This project documents the fundamental configuration required to get a MikroTik router operational with LAN and internet access. It includes setting up a bridge, assigning IP addresses, configuring DNS and DHCP, setting up routing, and enabling NAT for internet connectivity.

## Prerequisites

- MikroTik Router with RouterOS
- Access via Winbox or CLI
- Basic understanding of IP addressing and networking

## Configuration Overview

1. **Create a Bridge and Add Ports**  
   Create a bridge interface and assign multiple Ethernet ports to it for LAN aggregation.

2. **Assign IP Addresses**  
   Assign a public IP to the WAN interface and a private IP to the bridge for LAN access.

3. **Configure DNS Servers**  
   Set DNS servers provided by your ISP or use public resolvers like Google DNS.

4. **Set Up DHCP Server**  
   Configure an IP pool and assign a DHCP server to the bridge interface to automatically distribute IPs to LAN devices.

5. **Add Default Route (Gateway)**  
   Set a default route using the public gateway IP to enable internet access.

6. **Enable NAT (Masquerade)**  
   Apply source NAT to allow devices in the LAN to access the internet using the public IP.

## Example Commands

```bash
# Create bridge and add Ethernet ports
/interface bridge add name=bridge1
/interface bridge port add bridge=bridge1 interface=ether2
/interface bridge port add bridge=bridge1 interface=ether3
/interface bridge port add bridge=bridge1 interface=ether4
/interface bridge port add bridge=bridge1 interface=ether5

# Assign IP addresses
/ip address add address=10.10.69.50/24 interface=ether1  # Public IP
/ip address add address=192.168.1.1/24 interface=bridge1  # LAN IP

# Configure DNS
/ip dns set servers=8.8.8.8

# Create DHCP pool and server
/ip pool add name=dhcp_pool ranges=192.168.1.2-192.168.1.254
/ip dhcp-server add name=dhcp1 interface=bridge1 address-pool=dhcp_pool disabled=no
/ip dhcp-server network add address=192.168.1.0/24 gateway=192.168.1.1 dns-server=8.8.8.8,8.8.4.4
/ip dhcp-server enable dhcp1

# Add default route
/ip route add dst-address=0.0.0.0/0 gateway=10.10.69.250

# Enable NAT
/ip firewall nat add chain=srcnat action=masquerade
````

## Notes

* Replace IP addresses with those assigned by your ISP or as per your network design.
* This configuration assumes `ether1` is your WAN interface and remaining ports are for LAN.
* NAT is configured using masquerade to allow LAN access to the internet via the WAN IP.

---

**Author:**
Subash Subedi
Mikrotik Project Labs, 2025

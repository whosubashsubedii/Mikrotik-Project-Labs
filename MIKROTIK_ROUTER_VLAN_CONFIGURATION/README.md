# VLAN Configuration on MikroTik Router

This guide provides a comprehensive step-by-step process to set up VLANs on a MikroTik router, including bridging, VLAN filtering, IP assignment, DHCP configuration, routing, NAT, and firewall rules.

## Prerequisites

- MikroTik Router with RouterOS
- Access to MikroTik RouterOS via Winbox or CLI
- Basic networking knowledge (VLANs, IP addressing)

## Steps Overview

1. **Create a Bridge and Add Ports**  
   Create a bridge and add all Ethernet ports with VLAN filtering enabled.

2. **Configure VLAN Interfaces**  
   Define VLAN interfaces on the bridge with unique VLAN IDs.

3. **Assign IP Addresses**  
   Assign IP addresses to each VLAN interface for network segmentation.

4. **Enable VLAN Filtering on Bridge**  
   Configure bridge VLAN settings to tag/untag traffic correctly.

5. **Set Port VLAN IDs (PVIDs)**  
   Assign PVIDs to bridge ports to manage untagged traffic.

6. **Setup DHCP Server for VLANs**  
   Configure DHCP pools and servers for each VLAN to provide IPs automatically.

7. **Configure DNS and Routing**  
   Set DNS servers and define default routes for internet access.

8. **Enable NAT (Masquerade)**  
   Set up NAT to allow VLAN clients to access the internet.

9. **Firewall Rules to Block VLAN-to-VLAN Traffic**  
   Optionally block communication between VLANs for security.

## Example Commands

```bash
/interface bridge add name=bridge1 vlan-filtering=yes

/interface bridge port
add bridge=bridge1 interface=ether2
add bridge=bridge1 interface=ether3

/interface vlan
add interface=bridge1 name=vlan2 vlan-id=2

/ip address add address=192.168.2.1/24 interface=vlan2

/ip pool add name=pool_vlan2 ranges=192.168.2.2-192.168.2.200

/ip dhcp-server add interface=vlan2 address-pool=pool_vlan2 name=dhcp_vlan2 disabled=no

/ip firewall nat add chain=srcnat out-interface=ether1 action=masquerade


---

**Author:**
Subash Subedi
Mikrotik Project Labs, 2025

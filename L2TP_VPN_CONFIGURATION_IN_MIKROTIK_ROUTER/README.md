

# L2TP VPN Configuration on MikroTik Router

This guide provides a step-by-step process to configure an L2TP VPN server on a MikroTik router, including server setup, IP pool creation, PPP profile, user credentials, firewall configuration, and NAT for internet access.

## Prerequisites

- MikroTik Router with RouterOS  
- Access to MikroTik RouterOS via Winbox or CLI  
- Basic networking knowledge (VPN, IP addressing, firewall)

## Steps Overview

1. **Configure L2TP Server**  
   Enable L2TP server with IPsec and set a pre-shared key.

2. **Create IP Pool for VPN Clients**  
   Define the IP address range assigned to VPN clients.

3. **Configure PPP Profile**  
   Create a profile that uses the IP pool and enables encryption.

4. **Add VPN User Credentials**  
   Add VPN users with usernames and passwords.

5. **Configure Firewall Rules**  
   Allow necessary UDP ports and IPsec traffic for L2TP VPN.

6. **Set Up NAT (Masquerade)**  
   Enable NAT for VPN clients to access the internet.

## Example Commands

```bash
# 1. Enable L2TP Server with IPsec
/interface l2tp-server server
set enabled=yes default-profile=default-encryption use-ipsec=required ipsec-secret=YourStrongPreSharedKey

# 2. Create IP Pool for VPN clients
/ip pool
add name=L2TP-VPN ranges=192.168.1.200-192.168.1.250 comment="VPN Client IP Pool"

# 3. Create PPP profile using the IP pool
/ppp profile
add name=L2TP-VPN local-address=192.168.1.1 remote-address=L2TP-VPN change-tcp-mss=yes use-upnp=default use-encryption=yes

# 4. Add VPN user credentials
/ppp secret
add name=vpnuser password=yourpassword service=l2tp comment=customer1

# 5. Firewall rules to allow L2TP/IPsec traffic
/ip firewall filter
add chain=input protocol=udp port=500,1701,4500 action=accept comment="Allow L2TP/IPsec"
add chain=input protocol=ipsec-esp action=accept comment="Allow ESP"

# 6. NAT masquerade for VPN clients
/ip firewall nat
add chain=srcnat src-address=192.168.1.0/24 out-interface=bridge1 action=masquerade
````

---

### **Author**

**Subash Subedi**
**MikroTik VPN Setup Guide — 2025**
GitHub: [whosubashsubedii](https://github.com/whosubashsubedii)

---
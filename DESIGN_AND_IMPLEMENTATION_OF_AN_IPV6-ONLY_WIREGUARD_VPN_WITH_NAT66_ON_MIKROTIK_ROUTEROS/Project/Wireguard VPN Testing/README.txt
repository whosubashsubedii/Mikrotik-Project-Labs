Project: 'Worldlink Wireguard VPN Testing' created on 2026-08-09
Author: SUBASH SUBEDI <project@subashsubedi0.com.np>

This project focuses on designing and testing a low-cost remote access VPN solution using IPv6, WireGuard, NAT66, MikroTik RouterOS, and GNS3. The main purpose of this project is to explore how IPv6 can be used as an alternative when users do not have a directly reachable public IPv4 address because of Carrier-Grade NAT (CGNAT). 

The whole setup is first created and tested in GNS3 using MikroTik CHR. WireGuard is used to create the VPN tunnel, while Unique Local IPv6 Addresses are used for VPN clients. NAT66 is then used so that the VPN clients can access the IPv6 Internet through the global IPv6 address available on the MikroTik router. 

This project mainly looks at whether this type of setup can be useful for small and medium-sized businesses that need secure remote access but do not want to pay extra for a dedicated Internet connection or public IPv4 address.
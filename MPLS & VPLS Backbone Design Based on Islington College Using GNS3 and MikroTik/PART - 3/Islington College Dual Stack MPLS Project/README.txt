Project title
Islington College Dual Stack MPLS Project 

Author: SUBASH SUBEDI <project@subashsubedi0.com.np>

This project is about...
Islington College operates a multi-block campus network consisting of 9 academic and administrative blocks. The college owns a public IPv4 address range (160.30.132.0/24) and a public IPv6 address range (2001:df4:2b40::/48). The Private IPv4 range is used for router loopback interfaces within the MPLS backbone. The Private IPv6 range is used for router loopbacks, point-to-point core links, and VLAN services in the upgraded dual-stack design. 
  
We are required to design, implement, and verify an MPLS-based core network using MikroTik routers in GNS3. The network must provide VPLS-based Layer 2 connectivity across all blocks while using a centralized DHCP server for end-device addressing. IPv6 will be deployed in dual-stack mode to support modern routing and services. 
  
The LONDON_BLOCK is the main block, connected to the Internet (ISP). All other blocks connect to it directly and also through additional mesh links for redundancy and load balancing. 
  
Network Architecture and IP Addressing Plan 
  
Core Network Architecture 
· All block routers act as MPLS core routers 
· IS-IS is the IGP across all routers for IPv4 and IPv6 
· MPLS with LDP must be enabled on all core and mesh links 
· ECMP / load balancing must be implemented 
· Each router must maintain 3 - 4 routed link connections with other core routers to ensure redundancy, optimal traffic distribution, and backbone load balancing 
  
Routing & Connectivity 
· iBGP used within the network for internal route distribution across core routers 
· eBGP used for ISP connectivity simulation and external route exchange 
· IPv6 dual-stack addressing and routing 
· Public IPv6 reserved for ISP edge simulation and selected external services 
· Private IP addressing used for backbone, loopbacks, point-to-point links, and internal VLAN services 
  
Layer 2 Services (VPLS) 
· Multiple VPLS domains: Student, Staff, CCTV, Voice, Management, Server, and Guest Wi-Fi 
· VPLS must be used to extend VLANs across all blocks using separate service domains 
  
VLAN Design & Traffic Optimization 
· Voice VLAN, QoS planning, and Management/Server VLAN segmentation for VoIP prioritization and service separation 
· Jumbo frames enabled for MPLS and service transport efficiency 
  
Security & Access Control 
· Inter-network security and traffic isolation 
· RADIUS and Jump Server integration for centralized AAA authentication 
  IP Services 
· Centralized DHCP server only; no DHCP services on routers
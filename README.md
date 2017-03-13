# APIC-EM/iWAN-app based deployment in Cisco VIRL environment 

This project contains a complete VIRL-based topology demonstrating APIC-EM/iWAN-app automated Hub / Branch deployment in an enterprise

Scope of APIC-EM automated iWAN deployment in this project:
- Active-Active Data Centers
	- Dual transport per DC (MPLS + INET)
	- Dedicated Master Controller per DC
	- APIC-EM NAT to outside through internet firewall
- Two branches
	- Single Router, dual-transport (title: R41)
	- Dual Router, dual-transport (title: R51 branch)

## Prerequisites

What things you need to install the software and how to install them

```
Software:

1. Cisco IOS XE Software, Version 03.16.05.S - Extended Support Release, in qcow2 format
2. Cisco Smart License AX evaluation licenses for CSR1000v router. These are required to enable 1Gbps throughput on this platform as non-licensed default installation gives you only 100kbps
3. APIC-EM Version 1.4.0.1959
4. ViRL 1.2.83 - this is the simulation platform
5. Ostinato-drone + Ostinato-client(mac) - used for synthetic traffic generation
6. Prime Infrastructure 3.x - visibility, monitoring and troubleshooting
7. LiveAction LiveNX - flow visibility
8. VMWare ESXi 5.5

Hardware:
1. 2x UCSC-C240-M4SX / 2x Sockets / 8x Cores 2.4 GHz E5 
2. Cisco Catalyst 3650 L2 switch

```


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

## Project Topology

[Project Topology](virl_iwan_topo.png)

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

```
Setup nuances:
- CSR1000v router requires AX license to enable >100kbps throughput. In this setup I am utilizing Cisco Smart Licensing. Requires a Smart Virtual Account, and 20x CSR1000v eval licenses. 
- Smart Licensing Configlet required to auto-provision required licensing (configlet is pushed to devices as part of PnP provisioning process):

!-----------------
! UPDATE <TOKENID> with Smart Account generated tokenid
! UPDATE <PROXYIP> with your Internet Proxy address
!
service call-home
!
call-home
 ! If contact email address in call-home is configured as sch-smart-licensing@cisco.com
 ! the email address configured in Cisco Smart License Portal will be used as contact email address to send SCH notifications.
 contact-email-addr sch-smart-licensing@cisco.com
 http-proxy "$PROXYIP" port 80
 profile "CiscoTAC-1"
  active
  destination transport-method http
  no destination transport-method email
!
license accept end user agreement
license smart enable
platform hardware throughput level MB 1000
!
event manager applet POST_PNP
event timer countdown time 30
action 1.0 cli command "enable"
action 1.1 cli command "license smart register idtoken $TOKENID"
action 1.8 cli command "config t"
action 2.3 cli command "no event manager applet POST_PNP"
action 2.8 cli command "end"
action 2.9 cli command "wr mem"
action 3.0 cli command "end"
action 4.0 cli command "exit"
!----------------- 

- APIC-EM controller is located behind the 'flat' network connection in the 176.16.1.x/24 address space
- When provisioning Hub and Transit Hub through iWAN app ensure that in this topology the NAT address is specified (the Internet router emulates Internet Firewall). The Controller NAT address in this setup is 100.64.31.100
- Once the Hub and Transit Hub in DC1/DC2 are deployed, routing to/from 172.16.1.x/24 address space needs to be enabled otherwise branch provisioning will fail

!-----------------
router eigrp IWAN-EIGRP
address-family ipv4 unicast autonomous-system 400
topology base
redistribute ospf 1 metric 100000 1 255 1 1500
!-----------------

- Integration with Prime: iWAN wizard should reference 172.16.1.101 address, port 9991
- Integration with LA: Each device interface's Netflow will reference LA Server 172.16.1.99
```

```
PnP (Plug-and-Play) process:
- All the devices initial configuration contains PnP profile

!-----------------
! My APIC-EM address is 172.16.100 on MPLS link
! and 100.64.31.100 (NAT'ed on Internet Router) on INET
! It is also recommended to be explicit about the Source Interface where PnP req's
! will be initiated, as that will be elected as the Management IP address in APIC-EM for the device
! transport http ipv4 172.16.1.100 port 80 source InterfaceId
! Once PnP registered, a certificate is downloaded to bootflash and PnP profile is updated with
! HTTPS transport configuration automatically. You dont want to 'extract' your ViRL config with that
! already in place, as the next time PnP process will fail due to previous trustpoint configuration
pnp profile apicem-pnp
 transport http ipv4 172.16.1.100 port 80
!-----------------

- Previous PnP configuration reset on a device:

!-----------------
configure terminal
crypto key zeroize
no crypto pki certificate pool
no pnp profile pnp-zero-touch
end
delete nvram:*.cer
delete stby-nvram:*.cer (if the device has stack members)
write erase
reload
!-----------------

```

```
iWAN App provisioning process:

- Initial configuration of the devices assumes no config other than
-- IP Address assignment on the interface that will be used for Device Discovery
-- Static/DHCP default route
-- SSH service
-- SNMPv2 / SNMPv3 configuration

For Branch deployments, it is highly recommended to implement SNMPv3 AuthPriv configuration (Authentication and Encryption), as SNMP protocol will be used across Internet for Initial Device Discovery from APIC-EM.

In any practical deployment scenario for provisioning iWAN to Branches across public Internet, APIC-EM controller should be placed in DMZ zone behind a Firewall facing the Internet.

!-----------------
! Please NOTE TCP80 from Internet to DMZ requirement. 
! TCP80 is not the best port choice as it may be confused with cleartext HTTP. 
! TCP80 in this scenario is used to tunnel TLS
! to APIC-EM controller
!
Internet Branch to APIC-EM:
-PKI—TCP 80
-PNP—TCP 80, 443
-NTP—UDP 123

APIC-EM to Internet Branch:
-SNMPv3—TCP and UDP ports: 161, 162
-SSH—TCP 22

Internet Branch to Hub Routers (DMVPN):
-GRE and IPsec—UDP 500, 4500, IP—50
!-----------------

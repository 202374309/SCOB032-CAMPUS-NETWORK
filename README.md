 SCOB032 Campus Network

Project Overview

This repository contains the design, implementation and testing of a secure multi-site Smart Campus Network implemented using Cisco Packet Tracer.

The network connects the Main Campus and Satellite Campus and supports users, servers, Wi-Fi, VoIP, IoT and CCTV devices.

 Key Features

- VLANs and Inter-VLAN Routing
- VLSM Addressing
- OSPF Routing
- HSRP Redundancy
- Zero-Trust ACLs
- Cisco ASA Firewall and NAT
- Site-to-Site IPSec VPN
- VoIP and QoS
- Wireless Networking
- Port Security
- Syslog, SNMP and NTP

Network Design

The network follows a hierarchical architecture:

Core Layer → Distribution Layer → Access Layer → End Devices

The Main Campus and Satellite Campus are securely connected through an IPSec VPN.

Repository Structure

configs/        - Network device configurations
diagrams/       - Network topology diagram
packet-tracer/  - Cisco Packet Tracer project
tests/          - Testing and verification results

Testing

Testing was performed for connectivity, Zero-Trust ACLs, IPSec VPN, VoIP, QoS, HSRP redundancy, firewall/NAT and network monitoring.

Detailed results are available in the tests folder.

Conclusion

The project demonstrates a secure, scalable and redundant Smart Campus Network with multi-site connectivity, network segmentation, security controls, VoIP, wireless services and centralized monitoring.

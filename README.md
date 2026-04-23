# RiverBank Network Architecture Report

This document outlines the comprehensive network design and topology for RiverBank (formerly BB Bank), encompassing the Headquarters in Ho Chi Minh City and two regional branches in Da Nang and Ha Noi.

## 🏢 Executive Summary

RiverBank requires a highly secure, available, and scalable network infrastructure to support daily banking operations, customer transactions, and administrative tasks. The system utilizes modern technologies seamlessly integrating Headquarters with Branches using reliable WAN and robust internal segmentation.

## 🌐 Site Specifications

### Headquarters (Ho Chi Minh City)

*   **Building details:** 7 floors with an IT room and Cabling Central Local on the 1st floor.
*   **Scale:** 120 workstations, 5 servers, 12+ networking devices (including security apparatus).
*   **Infrastructure:** Fiber cabling (GPON), Gigabit/10-Gigabit Ethernet, and comprehensive WLAN/LAN setup.
*   **Logical structure:** VLAN-based segregation for different departments with a DMZ for exposed servers.
*   **Internet connection:** Acts as the primary gateway for all Internet traffic. Dual xDSL lines with a load-balancing mechanism to enhance availability.

### Regional Branches (Da Nang & Ha Noi)

*   **Building details:** 2 floors each, equipped with 1 IT room and Cabling Central Local.
*   **Scale:** Small-scale configuration (30 workstations, 3 servers, 5+ networking devices).
*   **Integration:** Relies on the Headquarters for Internet routing.

## 🔒 Security & Connectivity

*   **WAN Connectivity:** SD-WAN or MPLS securely connects HQ with the two branches, ensuring optimal path selection and prioritized banking traffic.
*   **Security measures:** Enterprise-grade Firewalls, IPS/IDS integration, phishing detection, and strict ACL implementations.
*   **Remote Access:** Site-to-Site VPNs for branch interconnectivity and Client-to-Site VPNs for remote teleworker access to the internal network.
*   **Physical Security:** Subnet segregation for an integrated surveillance camera system.

## 📈 Capacity & Workload Estimation

*   **Servers:** Web, database, and updates processing approx. 1000 MB/day (Download) and 2000 MB/day (Upload).
*   **Workstations:** Customer transactions, browsing (500 MB/day Download, 100 MB/day Upload per workstation).
*   **Guest WiFi:** Allocated ~500 MB/day for customer connectivity.
*   **Growth projection:** Architecture accommodates a 20% growth rate in structural nodes and traffic scaling over the next 5 years. Peak hours operate mostly between 09:00-11:00 and 15:00-16:00.

## 🗺️ Topology Diagram

```text
                                   [ Internet ]
                                        |
                              (2x xDSL, Load Balanced)
                                        |
                              [ HQ Perimeter Firewall ]
                                        | (IPS/IDS)
                                [ HQ Core Switch ] === [ DMZ / Server Farm ]
                                 /      |        \         (5 Servers)
                                /       |         \
 [ HQ VLANs (Workstations) ]---+    [ HQ WiFi ]    +---[ HQ Surveillance ]
       (Reduced: 2 PCs)                 |                 (IP Cameras)
                                        |
                            (WAN: SD-WAN / MPLS / VPN)
                                        |
              +-------------------------+-------------------------+
              |                                                   |
   [ Da Nang Branch Router ]                           [ Ha Noi Branch Router ]
              |                                                   |
   [ Branch Core Switch ]                              [ Branch Core Switch ]
     /                \                                  /                \
[ Branch VLANs ]  [ Server Farm ]                   [ Branch VLANs ]  [ Server Farm ]
   (2 PCs)          (3 Servers)                        (2 PCs)          (3 Servers)
```

## 🛠️ Validation & Testing Protocol

*   Internal routing verification between Inter-VLANs.
*   WAN routing verifications between branches and HQ.
*   Ping & Traceroute testing towards Internet Web Servers and isolated Servers inside the DMZ.
*   Strict policy verification to deny Guest WiFi / Customer devices access to Internal LANs.

# RiverBank Network: Enterprise Cisco Packet Tracer Setup Guide

This document provides a comprehensive walkthrough to build, configure, and verify the RiverBank network topology in Cisco Packet Tracer 9.0.0. This guide strictly implements the `equipment-inventory.md` requirements (with workstation/endpoint counts reduced by 50% as requested for simulation performance).

## 🗺️ Visual Network Topology Diagram

Before building the topology, review this high-level structure to understand how the massive 350+ node scale is logically grouped into Core, Distribution, and Access layers:

```mermaid
graph TD
    %% Internet & External
    ISP((Internet / ISP)) -->|Outside 203.0.113.2| FW1[HQ-FW1 ASA 5506-X]
    ISP -->|Outside Backup| FW2[HQ-FW2 ASA 5506-X]

    %% HQ Firewalls to Routers
    FW1 -->|Inside 192.168.1.1| HQR1{HQ-R1 Router}
    FW2 -->|Inside| HQR2{HQ-R2 Router}
    
    %% HQ Core Routing to Core Switching
    HQR1 -->|HSRP 10.0.x.1| HQCORE1[HQ-CORE1 L3 Switch]
    HQR2 -->|HSRP 10.0.x.1| HQCORE2[HQ-CORE2 L3 Switch]
    HQCORE1 <-->|Trunk| HQCORE2

    %% HQ Core to Access Layer
    HQCORE1 -->|Trunks| HQACC[HQ-ACC1 to ACC6\n6x 2960 Switches]
    HQCORE2 -->|Trunks| HQACC

    %% HQ Endpoints & DMZ
    HQCORE1 -.->|VLAN 20| HQDMZ[HQ DMZ & IT\n6x Servers, 2x NVRs]
    HQCORE1 -.->|Trunk| HQWLC[HQ-WLC-3504]
    
    HQACC -.->|VLAN 10| HQPC[72x Wired PCs]
    HQACC -.->|VLAN 11/30| HQLAP[14x Lightweight APs\n108 Staff, 108 Guest]
    HQACC -.->|VLAN 40| HQCCTV[21x IP Cameras]

    %% WAN Links (OSPF Area 0)
    HQR1 <-->|Se0/1/0 P2P\n10.255.255.0/30| DNR1{DN-R1 Router}
    HQR1 <-->|Se0/1/1 P2P\n10.255.255.4/30| HNR1{HN-R1 Router}

    %% Da Nang Branch
    DNR1 --> DNCORE[DN-CORE1 L3 Switch]
    DNCORE --> DNACC[DN-ACC1 & ACC2]
    DNACC -.-> DNSRV[4x Servers, 1x NVR]
    DNACC -.-> DNPC[18x PCs, 6x Cameras]
    DNACC -.-> DNAP[4x Autonomous APs\n27 Staff, 27 Guest]

    %% Ha Noi Branch
    HNR1 --> HNCORE[HN-CORE1 L3 Switch]
    HNCORE --> HNACC[HN-ACC1 & HN-ACC2]
    HNACC -.-> HNSRV[4x Servers, 1x NVR]
    HNACC -.-> HNPC[18x PCs, 6x Cameras]
    HNACC -.-> HNAP[4x Autonomous APs\n27 Staff, 27 Guest]
    
    classDef hq fill:#004085,stroke:#001A33,stroke-width:2px,color:#ffffff;
    classDef branch fill:#155724,stroke:#0B2E13,stroke-width:2px,color:#ffffff;
    class HQR1,HQR2,FW1,FW2,HQCORE1,HQCORE2,HQACC,HQDMZ,HQWLC hq;
    class DNR1,DNCORE,DNACC,HNR1,HNCORE,HNACC branch;
```

## Phase 1: Physical Enterprise Topology Deployment

### 1. Hardware Deployment List

Drag and drop the following devices onto your workspace. Group them logically by site.

**Headquarters (HQ) - Ho Chi Minh City:**

*   **Edge & Security:**
    *   2x Cisco 4331 Routers (`HQ-R1`, `HQ-R2`). *Turn off, add `NIM-2T` modules, turn on.*
    *   2x Cisco ASA 5506-X Firewalls (`HQ-FW1`, `HQ-FW2`)
*   **Core & Distribution Switching:**
    *   2x Layer 3 Switches - Cisco 3650 (`HQ-CORE1`, `HQ-CORE2`)
*   **Access Switching:**
    *   6x Layer 2 Switches - Cisco 2960 (`HQ-ACC1` through `HQ-ACC6`)
*   **Wireless Infrastructure:**
    *   1x WLC-3504 Wireless LAN Controller (`HQ-WLC`)
    *   14x Lightweight APs - LAP-PT (`HQ-LAP1` to `HQ-LAP14`)
*   **Servers & Surveillance (DMZ & IT Room):**
    *   6x Server-PT (Web, DNS, Email, File/FTP, Syslog, Backup)
    *   2x Server-PT or IoT-Monitor (acting as NVRs)
*   **Endpoints (Reduced by 50% for PT performance):**
    *   72x PCs (Wired)
    *   108x Laptops / Smartphones (Staff WiFi)
    *   108x Smartphones (Guest WiFi)
    *   21x IP Cameras (PoE)

**Regional Branches (Da Nang & Ha Noi) - *Quantities per branch*:**

*   **Edge & Security:**
    *   1x Cisco 4331 Router (`DN-R1` / `HN-R1`). *Turn off, add `NIM-2T`, turn on.*
*   **Core & Access Switching:**
    *   1x Layer 3 Switch - Cisco 3650 (`DN-CORE1` / `HN-CORE1`)
    *   2x Layer 2 Switches - Cisco 2960 (`DN-ACC1` to `DN-ACC2` / `HN-ACC1` to `HN-ACC2`)
*   **Wireless Infrastructure:**
    *   4x AccessPoint-PT (`DN-AP1` to `DN-AP4` / `HN-AP1` to `HN-AP4`)
*   **Servers & Surveillance:**
    *   4x Server-PT (File/FTP, Local DNS, Print, DHCP backup)
    *   1x Server-PT (acting as NVR)
*   **Endpoints (Reduced by 50% for PT performance):**
    *   18x PCs (Wired)
    *   27x Laptops / Smartphones (Staff WiFi)
    *   27x Smartphones (Guest WiFi)
    *   6x IP Cameras (PoE)

### 2. Physical Cabling & Connections

Given the massive number of endpoints, use the following structured layout:

**HQ Connectivity:**

*   **Internet:** ISP Router <--> `HQ-FW1` and `HQ-FW2` (Outside interfaces).
*   **Firewalls to Routers:** `HQ-FW1/FW2` (Inside interfaces) <--> `HQ-R1` and `HQ-R2`.
*   **HA Routers to Core:** `HQ-R1` <--> `HQ-CORE1` and `HQ-R2` <--> `HQ-CORE2`.
*   **Core to Access:** Connect `HQ-CORE1` and `HQ-CORE2` using cross-over or fiber. Trunk from both Cores down to `HQ-ACC1` through `HQ-ACC6`.
*   **WLC:** Connect `HQ-WLC` to `HQ-CORE1` on a trunk port.
*   **DMZ Servers:** Connect all 6 Servers to `HQ-CORE1` or a dedicated DMZ Access switch assigned to VLAN 20.
*   **Endpoints:** Distribute the 72 PCs, 21 IP Cameras, and 14 LAPs evenly across the 6 Access Switches (`HQ-ACC1` through `HQ-ACC6`). Configure switch ports to VLAN 10 (PCs), VLAN 40 (Cameras), and VLAN 11/30 (LAPs trunk/access).

**Branches (Da Nang & Ha Noi) Connectivity:**

*   **WAN:** `HQ-R1` (Se0/1/0) <--> `DN-R1` (Se0/1/0). `HQ-R1` (Se0/1/1) <--> `HN-R1` (Se0/1/0).
*   **Router to Core:** Branch Router <--> Branch Core Switch.
*   **Core to Access:** Branch Core Switch <--> Branch Access Switches (x2).
*   **Endpoints:** Distribute the 18 PCs, 6 Cameras, 4 Servers, and 4 APs evenly across the 2 Access Switches.

---

## Phase 2: Core Device Configurations (CLI)

*To configure these devices efficiently, use the modular scripts provided in `config/cisco-config.md`. Copy and paste the blocks into the CLI tab of the respective devices.*

### 1. High Availability HQ Core Setup

1.  **Firewalls (`HQ-FW1`, `HQ-FW2`):** Configure the Active/Standby ASA block. Set outside IP to `203.0.113.2` and inside to `192.168.1.1` (See Section 10.2).
2.  **Routers (`HQ-R1`, `HQ-R2`):** Apply the base WAN, OSPF, NAT, DHCP, and ACL configurations (See Section 1). Then apply the **HSRP** configuration (See Section 10.1) to ensure `.1` is the virtual gateway while `.2` and `.3` act as physical interface IPs.
3.  **Switches (`HQ-CORE1/2`, `HQ-ACC1-6`):**
    *   Create VLANs 10, 11, 20, 30, 40, 99 on all switches.
    *   Configure Gigabit uplinks as `mode trunk`.
    *   Assign FastEthernet ports to respective Access VLANs based on the endpoint plugged into it (See Section 8 and 10.3).

### 2. Branch Setups

1.  **Routers (`DN-R1`, `HN-R1`):** Apply standard Branch router config (OSPF Area 0, DHCP Pools, NAT Overload, WAN IPs) from Sections 4 and 6.
2.  **Switches (`DN-CORE`, `DN-ACC1-2`):** Create VLANs and configure trunks and access ports appropriately.

---

## Phase 3: Server & Wireless Configuration (GUI)

### 1. DMZ Servers (HQ) & Local Servers (Branches)

1.  Click **Server** -> **Desktop** -> **IP Configuration**.
2.  Assign static IPs from the `.2` to `.10` range of their respective Server/DMZ subnets (e.g., VLAN 20).
3.  *Web Server (HQ):* IP `10.0.20.5`. Turn HTTP service ON.
4.  *DNS Server (HQ):* IP `10.0.20.6`. Turn DNS service ON. Add A-Record for `www.riverbank.com` pointing to `10.0.20.5`.
5.  *(Optional)* Enable Syslog and Email services on the other dedicated servers to simulate active loads.

### 2. HQ Wireless LAN Controller (WLC-3504)

1.  Connect a configuration PC to the WLC management port and access `http://<WLC-IP>`.
2.  Navigate to **CONTROLLER** -> **Interfaces**. Create `vlan11_staff` and `vlan30_guest`.
3.  Navigate to **WLANs**. Create `RiverBank_Staff` (WPA2-PSK: `RiverBank2024`) and `RiverBank_Guest` (`WelcomeGuest`).
4.  Ensure the HQ DHCP scope provides Option 43 pointing to the WLC IP. The 14 LAPs will automatically join and broadcast the networks.

### 3. Branch Wireless (Autonomous APs)

1.  Click each AP -> **Config** -> **Port 1**.
2.  Set SSID to `RiverBank_Staff` or `RiverBank_Guest` with corresponding WPA2-PSK keys.

---

## Phase 4: Final Validation Testing

Due to the massive size of this Packet Tracer simulation (over 400 total nodes), test connectivity segment-by-segment:

1.  **DHCP Scalability Check:** Verify that the massive swarms of Wireless devices and PCs are successfully pulling IP addresses in their respective VLANs. Packet Tracer may take a few minutes to converge spanning-tree and DHCP leases.
2.  **Inter-Branch Connectivity:** Ping from an HQ PC (`10.0.10.x`) to a Da Nang PC (`10.1.10.x`) to ensure OSPF Area 0 is routing correctly across the WAN links.
3.  **High Availability (HA) Failover:**
    *   Start a continuous ping to the internet (`ping 8.8.8.8 -t`) from an HQ PC.
    *   Power off `HQ-R1`.
    *   HSRP should elect `HQ-R2` as Active, and traffic should resume after minor packet loss.
4.  **Security Segmentation:** Use a device connected to the Guest WiFi and try to ping the DMZ Web Server (`10.0.20.5`). The ASA/Router ACLs MUST return *Destination host unreachable*.
5.  **CCTV Isolation:** Try to ping a PC from an IP Camera. It should fail due to VLAN 40 isolation policies.
6.  **OSPF MD5 Authentication:** Run `show ip ospf neighbor` on the edge routers to verify that the routing adjacencies successfully establish over the WAN links using the encrypted MD5 keys.

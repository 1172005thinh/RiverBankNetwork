# RiverBank Network: Cisco Packet Tracer 9.0.0 Setup Guide

This document provides a comprehensive walkthrough to build, configure, and verify the RiverBank network topology in Cisco Packet Tracer 9.0.0. It incorporates both the baseline logic (for simplified testing) and the **Enterprise High Availability (HA) expansions** to fully comply with the `equipment-inventory.md`.

## Phase 1: Standard Physical Topology Setup

### 1. Hardware Deployment

Drag and drop the following devices onto your workspace:

**Headquarters (HQ):**

*   **Routers:** 1x Cisco 4331 (`HQ-R1`). *Turn off, add `NIM-2T` module, turn on.*
*   **Switches:** 1x Cisco 3650 (`HQ-SW1`)
*   **Servers:** 1x Server-PT (`DMZ Server`)
*   **Wireless:** 2x AccessPoint-PT (`Staff AP`, `Guest AP`)
*   **Endpoints:** 1x PC, 1x Laptop, 1x Smartphone

**Branches (Da Nang & Ha Noi):**

*   **Routers:** 1x Cisco 4331 each (`DN-R1`, `HN-R1`). *Add `NIM-2T` module.*
*   **Switches:** 1x Cisco 2960 each (`DN-SW1`, `HN-SW1`)
*   **Endpoints:** 1x PC each

### 2. Standard Cabling

*   **WAN (Serial):** `HQ-R1` (Se0/1/0) <--> `DN-R1` (Se0/1/0). `HQ-R1` (Se0/1/1) <--> `HN-R1` (Se0/1/0).
*   **HQ LAN:** `HQ-R1` (Gig0/0/1) <--> `HQ-SW1` (Gig0/1).
*   **Access Ports:** Connect Server to `F0/1`, PCs to `F0/6`, APs to `F0/11` and `F0/16`.

---

## Phase 2: Enterprise Scaling & High Availability Setup (Optional but Recommended)

*To fully comply with `equipment-inventory.md`, add these devices to HQ.*

1. **HA Routing:** Add a second Router (`HQ-R2`) next to `HQ-R1`. Connect it to the core switch. Both will run HSRP for gateway redundancy.
2. **Perimeter Firewalls:** Add a Cisco ASA 5506-X (`HQ-FW1`) between the ISP Router and the `HQ-R1`/`R2` routers.
3. **Core/Access Switching:** Change `HQ-SW1` to act purely as a distribution core. Add two Cisco 2960 switches (`HQ-ACC1`, `HQ-ACC2`) as Access layers. Move all PCs and APs to these Access switches.
4. **WLC (Wireless LAN Controller):** Replace the standard AccessPoint-PTs with **LAP-PTs** (Lightweight APs) and add a **WLC-3504** connected to the Core Switch.

---

## Phase 3: Core CLI Configurations

Double-click each device, go to **CLI** (type `no` for initial dialog), and paste the following. *(Refer to `config/cisco-config.md` Sections 1 to 7 for the full standard scripts).*

### 1. Standard Router & Switch Configs (Summary)

*   **HQ-R1, DN-R1, HN-R1:** Configure sub-interfaces (`Gig0/0/1.10 - .99`), Serial IP addresses (`10.255.255.x/30`), OSPF Area 0, DHCP Pools (excluding `.1` to `.10`), NAT Overload, and ACLs blocking Guest/DMZ traffic.
*   **HQ-SW1, DN-SW1, HN-SW1:** Configure VLANs (10, 11, 20, 30, 40, 99), Trunk ports (`Gig0/1`), Access ports, and Management IP (`VLAN 99`).

### 2. Enterprise HA Configurations (HQ Only)

**HQ-R1 & HQ-R2 (HSRP Gateway Redundancy):**
Change HQ-R1 physical sub-interfaces to `.2`, and set HSRP virtual IP to `.1`.

```text
interface GigabitEthernet0/0/1.10
 ip address 10.0.10.2 255.255.255.0
 standby 10 ip 10.0.10.1
 standby 10 priority 110
 standby 10 preempt
```

On HQ-R2, configure sub-interfaces as `.3`:

```text
interface GigabitEthernet0/0/1.10
 ip address 10.0.10.3 255.255.255.0
 standby 10 ip 10.0.10.1
 standby 10 priority 90
 standby 10 preempt
```

**HQ-FW1 (Cisco ASA):**

```text
interface GigabitEthernet1/1
 nameif outside
 security-level 0
 ip address 203.0.113.2 255.255.255.252
 no shutdown
exit
interface GigabitEthernet1/2
 nameif inside
 security-level 100
 ip address 192.168.1.1 255.255.255.0
 no shutdown
```

---

## Phase 4: GUI Application Config (Servers & WLC)

### 1. DMZ Web & DNS Server

1. Click **Server** -> **Desktop** -> **IP Configuration**. Set Static IP: `10.0.20.5`, Mask: `255.255.255.0`, Gateway: `10.0.20.1`, DNS: `10.0.20.5`.
2. Go to **Services** -> **HTTP**. Turn ON.
3. Go to **Services** -> **DNS**. Turn ON. Add A-Record for `www.riverbank.com` pointing to `10.0.20.5`.

### 2. Wireless Setup (Standard vs WLC)

**If using standard APs:** Set Port 1 SSID to `RiverBank_Staff` (WPA2-PSK: `RiverBank2024`) and `RiverBank_Guest` (`WelcomeGuest`).

**If using WLC (Enterprise):**

1. Access WLC GUI via web browser (`http://<WLC-IP>`).
2. Navigate to **CONTROLLER** -> **Interfaces** and create Dynamic interfaces for VLAN 11 and VLAN 30.
3. Navigate to **WLANs** -> Create new WLANs tied to the VLAN interfaces with WPA2-PSK security.
4. LAPs will auto-join the WLC via DHCP Option 43.

---

## Phase 5: Validation Testing

1. **DHCP Check:** Verify laptops/PCs automatically pull an IP in their respective `.10`, `.11`, or `.30` subnets.
2. **OSPF & WAN Check:** Ping from HQ PC (`10.0.10.11`) to Da Nang PC (`10.1.10.11`).
3. **ACL Security Check:** From the Guest WiFi Smartphone, ping the DMZ Server (`10.0.20.5`). Packet Tracer MUST return *Destination host unreachable*.
4. **HA Failover (Enterprise Only):** Run a continuous ping (`ping 8.8.8.8 -t`) from an HQ PC. Power off `HQ-R1`. HSRP will automatically failover to `HQ-R2` keeping the connection alive.

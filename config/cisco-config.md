# Cisco Configurations & Implementation Guides

This file acts as the single source of truth for all fully generated, validated Cisco IOS configurations and Packet Tracer physical implementation protocols for the RiverBank network.

---

## 1. Headquarters (HQ) Core Router (HQ-R1)

```text
enable
configure terminal
hostname HQ-R1

! --- Basic Security ---
enable secret class
service password-encryption
line console 0
 password cisco
login
exit
line vty 0 4
 password cisco
 login
 transport input ssh
exit

! --- WAN Connections (P2P Serial Links) ---
interface Serial0/1/0
 description P2P to Da Nang
 ip address 10.255.255.1 255.255.255.252
 ip ospf message-digest-key 1 md5 RiverBankSecureOSPF
 no shutdown
exit

interface Serial0/1/1
 description P2P to Ha Noi
 ip address 10.255.255.5 255.255.255.252
 ip ospf message-digest-key 1 md5 RiverBankSecureOSPF
 no shutdown
exit

! --- Internet Connection ---
interface GigabitEthernet0/0/0
 description Uplink to ISP (Simulated Public IP)
 ip address 203.0.113.2 255.255.255.252
 no shutdown
exit

! --- Internal LAN Trunk (Router-on-a-Stick) ---
interface GigabitEthernet0/0/1
 description Trunk to HQ Core Switch
 no ip address
 no shutdown
exit

interface GigabitEthernet0/0/1.10
 description VLAN 10 (HQ Wired PCs)
 encapsulation dot1Q 10
 ip address 10.0.10.1 255.255.255.0
exit

interface GigabitEthernet0/0/1.11
 description VLAN 11 (HQ Staff WiFi)
 encapsulation dot1Q 11
 ip address 10.0.11.1 255.255.255.0
exit

interface GigabitEthernet0/0/1.20
 description VLAN 20 (DMZ / Server Farm)
 encapsulation dot1Q 20
 ip address 10.0.20.1 255.255.255.0
exit

interface GigabitEthernet0/0/1.30
 description VLAN 30 (HQ Guest WiFi)
 encapsulation dot1Q 30
 ip address 10.0.30.1 255.255.255.0
exit

interface GigabitEthernet0/0/1.40
 description VLAN 40 (HQ Surveillance)
 encapsulation dot1Q 40
 ip address 10.0.40.1 255.255.255.0
exit

interface GigabitEthernet0/0/1.99
 description VLAN 99 (IT Management)
 encapsulation dot1Q 99 native
 ip address 10.0.99.1 255.255.255.0
exit

! --- Routing Protocol (OSPF Area 0) ---
router ospf 1
 router-id 1.1.1.1
 area 0 authentication message-digest
 ! Advertise internal HQ networks
 network 10.0.0.0 0.0.255.255 area 0
 ! Advertise P2P links
 network 10.255.255.0 0.0.0.3 area 0
 network 10.255.255.4 0.0.0.3 area 0
 ! Advertise default route to internet
 default-information originate
exit

ip route 0.0.0.0 0.0.0.0 203.0.113.1

! --- Access Control Lists (ACLs) ---
! Guest WiFi Block
ip access-list extended HQ_GUEST_WIFI_ACL
 remark Block HQ Guest WiFi from All Internal RiverBank LANs & DMZ
 deny ip 10.0.30.0 0.0.0.255 10.0.0.0 0.255.255.255
 remark Allow HQ Guest WiFi to Internet
 permit ip 10.0.30.0 0.0.0.255 any
exit

! DMZ Containment Block
ip access-list extended HQ_DMZ_CONTAINMENT
 remark Prevent DMZ Servers from initiating to Internal LANs
 deny ip 10.0.20.0 0.0.0.255 10.0.10.0 0.0.0.255
 deny ip 10.0.20.0 0.0.0.255 10.0.11.0 0.0.0.255
 remark Allow DMZ Servers to reply/route to Internet
 permit ip 10.0.20.0 0.0.0.255 any
 permit ip any any
exit

! Apply ACLs to Sub-Interfaces
interface GigabitEthernet0/0/1.30
 ip access-group HQ_GUEST_WIFI_ACL in
exit

interface GigabitEthernet0/0/1.20
 ip access-group HQ_DMZ_CONTAINMENT in
exit

end
write memory
```

---

## 2. Headquarters (HQ) Core Switch (HQ-SW1)

```text
enable
configure terminal
hostname HQ-SW1

! --- Create VLANs ---
vlan 10
 name HQ_Wired_PCs
vlan 11
 name HQ_Staff_WiFi
vlan 20
 name DMZ_Server_Farm
vlan 30
 name HQ_Guest_WiFi
vlan 40
 name HQ_Surveillance
vlan 99
 name IT_Management
exit

! --- Uplink Trunk to HQ Router ---
interface GigabitEthernet1/1/1
 description Trunk to HQ-R1
 switchport mode trunk
 switchport trunk native vlan 99
 no shutdown
exit

! --- Management Interface ---
interface vlan 99
 ip address 10.0.99.2 255.255.255.0
 no shutdown
exit
ip default-gateway 10.0.99.1

end
write memory
```

---

## 3. Detailed Implementation Guide (Packet Tracer Setup)

Follow these exact steps in Packet Tracer to ensure the CLI configurations apply without error:

1. **Deploy the Hardware:**
   - Drag a **Cisco 4331 Router** (or 2911/1941) onto the canvas for the HQ-R1. *Note: If using PT, ensure you power off the router and add the HWIC-2T (or NIM-2T) serial module to support the Serial0/1/0 and Serial0/1/1 WAN connections, then power it back on.*
   - Drag a **Cisco 2960 (or 3560/3650) Switch** onto the canvas for HQ-SW1.
2. **Physical Wiring (Critical):**
   - Use a **Copper Straight-Through** cable.
   - Connect **GigabitEthernet0/0/1** on the Router to **GigabitEthernet0/1** on the Switch. *(If you use a different port, the router-on-a-stick config will fail).*
   - Connect **GigabitEthernet0/0/0** on the Router to your simulated ISP/Cloud (optional for now).
3. **Apply Configurations:**
   - Double-click the **Router (HQ-R1)**, go to the **CLI** tab. Type `no` if asked for the initial configuration dialog.
   - Paste the HQ-R1 configuration script entirely.
   - Double-click the **Switch (HQ-SW1)**, go to the **CLI** tab, and paste the Switch configuration script.
4. **Validation Test:**
   - Connect a PC to any FastEthernet port on the switch, assign it to VLAN 99, and configure it statically as 10.0.99.10. You should successfully be able to ping 10.0.99.1 (the Router).

---

## 4. Da Nang Branch Router (DN-R1)

```text
enable
configure terminal
hostname DN-R1

! --- Basic Security ---
enable secret class
service password-encryption
line console 0
 password cisco
login
exit
line vty 0 4
 password cisco
 login
 transport input ssh
exit

! --- WAN Connections (P2P Serial Links) ---
interface Serial0/1/0
 description P2P to HQ Int Se0/1/0
 ip address 10.255.255.2 255.255.255.252
 ip ospf message-digest-key 1 md5 RiverBankSecureOSPF
 no shutdown
exit

! --- Internal LAN Trunk (Router-on-a-Stick) ---
interface GigabitEthernet0/0/1
 description Trunk to Da Nang Switch
 no ip address
 no shutdown
exit

interface GigabitEthernet0/0/1.10
 description VLAN 10 (DN Wired PCs)
 encapsulation dot1Q 10
 ip address 10.1.10.1 255.255.255.0
exit

interface GigabitEthernet0/0/1.11
 description VLAN 11 (DN Staff WiFi)
 encapsulation dot1Q 11
 ip address 10.1.11.1 255.255.255.0
exit

interface GigabitEthernet0/0/1.20
 description VLAN 20 (Local Server Farm)
 encapsulation dot1Q 20
 ip address 10.1.20.1 255.255.255.0
exit

interface GigabitEthernet0/0/1.30
 description VLAN 30 (DN Guest WiFi)
 encapsulation dot1Q 30
 ip address 10.1.30.1 255.255.255.0
exit

interface GigabitEthernet0/0/1.40
 description VLAN 40 (DN Surveillance)
 encapsulation dot1Q 40
 ip address 10.1.40.1 255.255.255.0
exit

interface GigabitEthernet0/0/1.99
 description VLAN 99 (IT Management)
 encapsulation dot1Q 99 native
 ip address 10.1.99.1 255.255.255.0
exit

! --- Routing Protocol (OSPF Area 0) ---
router ospf 1
 router-id 2.2.2.2
 area 0 authentication message-digest
 network 10.1.0.0 0.0.255.255 area 0
 network 10.255.255.0 0.0.0.3 area 0
exit

! --- Access Control Lists (ACLs) ---
! Guest WiFi Block
ip access-list extended DN_GUEST_WIFI_ACL
 remark Block DN Guest WiFi from All Internal RiverBank LANs & DMZ
 deny ip 10.1.30.0 0.0.0.255 10.0.0.0 0.255.255.255
 remark Allow DN Guest WiFi to Internet
 permit ip 10.1.30.0 0.0.0.255 any
exit

! DMZ Containment Block
ip access-list extended DN_DMZ_CONTAINMENT
 remark Prevent DMZ Servers from initiating to Internal LANs
 deny ip 10.1.20.0 0.0.0.255 10.1.10.0 0.0.0.255
 deny ip 10.1.20.0 0.0.0.255 10.1.11.0 0.0.0.255
 remark Allow DMZ Servers to reply/route to Internet/HQ
 permit ip 10.1.20.0 0.0.0.255 any
 permit ip any any
exit

! Apply ACLs to Sub-Interfaces
interface GigabitEthernet0/0/1.30
 ip access-group DN_GUEST_WIFI_ACL in
exit

interface GigabitEthernet0/0/1.20
 ip access-group DN_DMZ_CONTAINMENT in
exit

end
write memory
```

---

## 5. Da Nang Branch Switch (DN-SW1)

```text
enable
configure terminal
hostname DN-SW1

! --- Create VLANs ---
vlan 10
 name DN_Wired_PCs
vlan 11
 name DN_Staff_WiFi
vlan 20
 name Local_Server_Farm
vlan 30
 name DN_Guest_WiFi
vlan 40
 name DN_Surveillance
vlan 99
 name IT_Management
exit

! --- Uplink Trunk to DN Router ---
interface GigabitEthernet1/1/1
 description Trunk to DN-R1
 switchport mode trunk
 switchport trunk native vlan 99
 no shutdown
exit

! --- Management Interface ---
interface vlan 99
 ip address 10.1.99.2 255.255.255.0
 no shutdown
exit
ip default-gateway 10.1.99.1

end
write memory
```

---

## 6. Ha Noi Branch Router (HN-R1)

```text
enable
configure terminal
hostname HN-R1

! --- Basic Security ---
enable secret class
service password-encryption
line console 0
 password cisco
login
exit
line vty 0 4
 password cisco
 login
 transport input ssh
exit

! --- WAN Connections (P2P Serial Links) ---
interface Serial0/1/0
 description P2P to HQ Int Se0/1/1
 ip address 10.255.255.6 255.255.255.252
 ip ospf message-digest-key 1 md5 RiverBankSecureOSPF
 no shutdown
exit

! --- Internal LAN Trunk (Router-on-a-Stick) ---
interface GigabitEthernet0/0/1
 description Trunk to Ha Noi Switch
 no ip address
 no shutdown
exit

interface GigabitEthernet0/0/1.10
 description VLAN 10 (HN Wired PCs)
 encapsulation dot1Q 10
 ip address 10.2.10.1 255.255.255.0
exit

interface GigabitEthernet0/0/1.11
 description VLAN 11 (HN Staff WiFi)
 encapsulation dot1Q 11
 ip address 10.2.11.1 255.255.255.0
exit

interface GigabitEthernet0/0/1.20
 description VLAN 20 (Local Server Farm)
 encapsulation dot1Q 20
 ip address 10.2.20.1 255.255.255.0
exit

interface GigabitEthernet0/0/1.30
 description VLAN 30 (HN Guest WiFi)
 encapsulation dot1Q 30
 ip address 10.2.30.1 255.255.255.0
exit

interface GigabitEthernet0/0/1.40
 description VLAN 40 (HN Surveillance)
 encapsulation dot1Q 40
 ip address 10.2.40.1 255.255.255.0
exit

interface GigabitEthernet0/0/1.99
 description VLAN 99 (IT Management)
 encapsulation dot1Q 99 native
 ip address 10.2.99.1 255.255.255.0
exit

! --- Routing Protocol (OSPF Area 0) ---
router ospf 1
 router-id 3.3.3.3
 area 0 authentication message-digest
 network 10.2.0.0 0.0.255.255 area 0
 network 10.255.255.4 0.0.0.3 area 0
exit

! --- Access Control Lists (ACLs) ---
! Guest WiFi Block
ip access-list extended HN_GUEST_WIFI_ACL
 remark Block HN Guest WiFi from All Internal RiverBank LANs & DMZ
 deny ip 10.2.30.0 0.0.0.255 10.0.0.0 0.255.255.255
 remark Allow HN Guest WiFi to Internet
 permit ip 10.2.30.0 0.0.0.255 any
exit

! DMZ Containment Block
ip access-list extended HN_DMZ_CONTAINMENT
 remark Prevent DMZ Servers from initiating to Internal LANs
 deny ip 10.2.20.0 0.0.0.255 10.2.10.0 0.0.0.255
 deny ip 10.2.20.0 0.0.0.255 10.2.11.0 0.0.0.255
 remark Allow DMZ Servers to reply/route to Internet/HQ
 permit ip 10.2.20.0 0.0.0.255 any
 permit ip any any
exit

! Apply ACLs to Sub-Interfaces
interface GigabitEthernet0/0/1.30
 ip access-group HN_GUEST_WIFI_ACL in
exit

interface GigabitEthernet0/0/1.20
 ip access-group HN_DMZ_CONTAINMENT in
exit

end
write memory
```

---

## 7. Ha Noi Branch Switch (HN-SW1)

```text
enable
configure terminal
hostname HN-SW1

! --- Create VLANs ---
vlan 10
 name HN_Wired_PCs
vlan 11
 name HN_Staff_WiFi
vlan 20
 name Local_Server_Farm
vlan 30
 name HN_Guest_WiFi
vlan 40
 name HN_Surveillance
vlan 99
 name IT_Management
exit

! --- Uplink Trunk to HN Router ---
interface GigabitEthernet1/1/1
 description Trunk to HN-R1
 switchport mode trunk
 switchport trunk native vlan 99
 no shutdown
exit

! --- Management Interface ---
interface vlan 99
 ip address 10.2.99.2 255.255.255.0
 no shutdown
exit
ip default-gateway 10.2.99.1

end
write memory
```

---

## 8. INFRASTRUCTURE POLISH: NAT, DHCP & Access Ports

Apply these modular commands to fully finalize the hardware implementation, resolving NAT translation, DHCP endpoint addressing, and Switch access port mappings.

### HQ Router (HQ-R1) - NAT, PAT, DHCP, and Clock Rates

```text
enable
configure terminal

! --- DCE Clock Rates for Packet Tracer ---
interface Serial0/1/0
 clock rate 64000
exit
interface Serial0/1/1
 clock rate 64000
exit

! --- DHCP Exclusions (Reserve .1 to .10) ---
ip dhcp excluded-address 10.0.10.1 10.0.10.10
ip dhcp excluded-address 10.0.11.1 10.0.11.10
ip dhcp excluded-address 10.0.30.1 10.0.30.10
ip dhcp excluded-address 10.0.40.1 10.0.40.10

! --- DHCP Pools ---
ip dhcp pool HQ_Wired_PCs
 network 10.0.10.0 255.255.255.0
 default-router 10.0.10.1
 dns-server 10.0.20.5
exit
ip dhcp pool HQ_Staff_WiFi
 network 10.0.11.0 255.255.255.0
 default-router 10.0.11.1
 dns-server 10.0.20.5
exit
ip dhcp pool HQ_Guest_WiFi
 network 10.0.30.0 255.255.255.0
 default-router 10.0.30.1
 dns-server 8.8.8.8
exit
ip dhcp pool HQ_Surveillance
 network 10.0.40.0 255.255.255.0
 default-router 10.0.40.1
exit

! --- NAT Configuration (based on hint.txt) ---
! 1. Define NAT Outside
interface GigabitEthernet0/0/0
 ip nat outside
exit
! 2. Define NAT Inside
interface Serial0/1/0
 ip nat inside
exit
interface Serial0/1/1
 ip nat inside
exit
interface GigabitEthernet0/0/1.10
 ip nat inside
exit
interface GigabitEthernet0/0/1.11
 ip nat inside
exit
interface GigabitEthernet0/0/1.20
 ip nat inside
exit
interface GigabitEthernet0/0/1.30
 ip nat inside
exit
interface GigabitEthernet0/0/1.40
 ip nat inside
exit
interface GigabitEthernet0/0/1.99
 ip nat inside
exit

! 3. NAT Overload (PAT) ACL
access-list 1 permit 10.0.0.0 0.255.255.255
access-list 1 permit 10.1.0.0 0.255.255.255
access-list 1 permit 10.2.0.0 0.255.255.255

ip nat inside source list 1 interface GigabitEthernet0/0/0 overload

! 4. Static NAT (Port Forwarding for DMZ Web Server)
! Assumes DMZ Web Server is assigned IP 10.0.20.5
ip nat inside source static tcp 10.0.20.5 80 203.0.113.2 80
ip nat inside source static tcp 10.0.20.5 443 203.0.113.2 443

end
write memory
```

### Da Nang Router (DN-R1) - DHCP

```text
enable
configure terminal

! --- DHCP Exclusions (Reserve .1 to .10) ---
ip dhcp excluded-address 10.1.10.1 10.1.10.10
ip dhcp excluded-address 10.1.11.1 10.1.11.10
ip dhcp excluded-address 10.1.30.1 10.1.30.10
ip dhcp excluded-address 10.1.40.1 10.1.40.10

! --- DHCP Pools ---
ip dhcp pool DN_Wired_PCs
 network 10.1.10.0 255.255.255.0
 default-router 10.1.10.1
 dns-server 10.0.20.5
exit
ip dhcp pool DN_Staff_WiFi
 network 10.1.11.0 255.255.255.0
 default-router 10.1.11.1
 dns-server 10.0.20.5
exit
ip dhcp pool DN_Guest_WiFi
 network 10.1.30.0 255.255.255.0
 default-router 10.1.30.1
 dns-server 8.8.8.8
exit
ip dhcp pool DN_Surveillance
 network 10.1.40.0 255.255.255.0
 default-router 10.1.40.1
exit

end
write memory
```

### Ha Noi Router (HN-R1) - DHCP

```text
enable
configure terminal

! --- DHCP Exclusions (Reserve .1 to .10) ---
ip dhcp excluded-address 10.2.10.1 10.2.10.10
ip dhcp excluded-address 10.2.11.1 10.2.11.10
ip dhcp excluded-address 10.2.30.1 10.2.30.10
ip dhcp excluded-address 10.2.40.1 10.2.40.10

! --- DHCP Pools ---
ip dhcp pool HN_Wired_PCs
 network 10.2.10.0 255.255.255.0
 default-router 10.2.10.1
 dns-server 10.0.20.5
exit
ip dhcp pool HN_Staff_WiFi
 network 10.2.11.0 255.255.255.0
 default-router 10.2.11.1
 dns-server 10.0.20.5
exit
ip dhcp pool HN_Guest_WiFi
 network 10.2.30.0 255.255.255.0
 default-router 10.2.30.1
 dns-server 8.8.8.8
exit
ip dhcp pool HN_Surveillance
 network 10.2.40.0 255.255.255.0
 default-router 10.2.40.1
exit

end
write memory
```

### Switch Access Port Mappings (Run on Cisco 2960 Access Switches)

Run this identically on all Access Layer switches (HQ-ACC1-6, DN-ACC1-2, HN-ACC1-2) to standardize endpoint connections:

```text
enable
configure terminal

interface range FastEthernet0/1 - 4
 description VLAN 20 (Local / DMZ Servers)
 switchport mode access
 switchport access vlan 20
 spanning-tree portfast
exit

interface range FastEthernet0/5 - 16
 description VLAN 10 (Wired PCs)
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
exit

interface range FastEthernet0/17 - 18
 description VLAN 11 (Staff WiFi APs)
 switchport mode access
 switchport access vlan 11
 spanning-tree portfast
exit

interface range FastEthernet0/19 - 20
 description VLAN 30 (Guest WiFi APs)
 switchport mode access
 switchport access vlan 30
 spanning-tree portfast
exit

interface range FastEthernet0/21 - 24
 description VLAN 40 (Surveillance / IT Mgmt)
 switchport mode access
 switchport access vlan 40
 spanning-tree portfast
exit

end
write memory
```

---


### Core Switch Server & WLC Mappings (Run on HQ-SW1, DN-SW1, HN-SW1)

For the Cisco 3650 Core switches, endpoints like DMZ Servers and the WLC connect to the Gigabit ports.

`	ext
enable
configure terminal

interface range GigabitEthernet1/0/1 - 4
 description Trunk to Access Switches
 switchport mode trunk
 switchport trunk native vlan 99
 no shutdown
exit

interface range GigabitEthernet1/0/10 - 15
 description VLAN 20 (Local / DMZ Servers)
 switchport mode access
 switchport access vlan 20
 spanning-tree portfast
exit

interface GigabitEthernet1/0/24
 description Trunk to WLC (HQ Only)
 switchport mode trunk
 switchport trunk native vlan 99
 no shutdown
exit

end
write memory
`

---

## 9. PACKET TRACER GUI SETUP GUIDE (Servers & Wireless)

These configurations cannot be done via the Cisco IOS CLI. You must manually configure these components using the Packet Tracer visual GUI.

### 1. Server Configuration (DMZ Web & DNS Server)

*Drag a Server-PT into the HQ workspace and connect it to HQ-SW1 FastEthernet0/1 (VLAN 20).*

1. Click the **Server** -> **Desktop** tab -> **IP Configuration**.
2. Select **Static** and configure the variables:
   - **IP Address:** 10.0.20.5
   - **Subnet Mask:** 255.255.255.0
   - **Default Gateway:** 10.0.20.1
   - **DNS Server:** 10.0.20.5 (Points to itself)
3. Go to the **Services** tab:
   - **HTTP / HTTPS:** Ensure both are turned **ON**. (Optional: Edit index.html to say "Welcome to RiverBank Secure Portal" for visual verification).
   - **DNS:** Turn Service **ON**. Add an A Record:
     - **Name:** `www.riverbank.com`
     - **Address:** 10.0.20.5
     - Click **Add**.

### 2. Wireless Access Points (Staff & Guest)

*Drag two AccessPoint-PT devices into the workspace. Connect the Staff AP to 'FastEthernet0/11' (VLAN 11) and the Guest AP to 'FastEthernet0/16' (VLAN 30).*

1. Click the **Staff Access Point** -> **Config** tab -> **Port 1** (under Interface).
   - **SSID:** RiverBank_Staff
   - **Authentication:** WPA2-PSK
   - **PSK Passphrase:** RiverBank2024 (or any secure password).
2. Click the **Guest Access Point** -> **Config** tab -> **Port 1**.
   - **SSID:** RiverBank_Guest
   - **Authentication:** Open (or WPA2-PSK with WelcomeGuest).

### 3. End Devices (Laptops, PCs, Smartphones)

1. **Wired PCs:** Connect a PC to Switch ports FastEthernet0/6 - 0/10 (VLAN 10).
   - Click **PC** -> **Desktop** tab -> **IP Configuration**. Select **DHCP**. It should automatically ping the server and pull an IP (e.g., 10.0.10.11).
2. **Wireless Devices:** Click **Laptop/Smartphone** -> **Config** tab -> **Wireless0** interface.
   - Enter the corresponding SSID (RiverBank_Staff or RiverBank_Guest) and password.
   - Go to **Desktop** -> **IP Configuration**. Verify it pulls the correct DHCP address (e.g., 10.0.11.11 for Staff).

### 4. Final Testing Checklist

- **DNS & Web Check:** Open a Web Browser from a Staff PC in Packet Tracer and navigate to `www.riverbank.com`. It should load the Bank Portal.
- **NAT / Port Forwarding Check:** Open a Web Browser on a PC connected to the *outside* ISP router and navigate to 203.0.113.2. It should trigger your NAT rules and load the internal Bank Portal.
- **Security Check (ACL):** From a device on the Guest WiFi, attempt to Ping a Staff PC or the Web Server (10.0.20.5). Packet Tracer should say Destination host unreachable confirming the Guest isolation ACL is working perfectly.

---

## 10. ENTERPRISE SCALING & HIGH AVAILABILITY (Equipment Inventory Match)

To fully comply with the `equipment-inventory.md` requirements, the following advanced configurations should be added to expand the logical base into a full enterprise-scale topology.

### 10.1 High Availability HQ Router (HQ-R2) with HSRP

To provide gateway redundancy, add a second Cisco 4331 Router (`HQ-R2`). Both HQ-R1 and HQ-R2 will share the `10.0.X.1` default gateway IP using HSRP.

*(Note: In a true HA setup, HQ-R1 physical IPs must be changed to `.2`, HQ-R2 uses `.3`, and the HSRP virtual IP is `.1`. You must update HQ-R1's sub-interfaces to match this concept before applying HQ-R2.)*

```text
enable
configure terminal
hostname HQ-R2

! --- Basic Security ---
enable secret class
service password-encryption
line console 0
 password cisco
 login
exit
line vty 0 4
 password cisco
 login
 transport input ssh
exit

! --- Internet Connection (Backup ISP Link) ---
interface GigabitEthernet0/0/0
 description Uplink to ISP (Backup)
 ip address 203.0.113.6 255.255.255.252
 ip nat outside
 no shutdown
exit

! --- Internal LAN Trunk to HQ-SW2 ---
interface GigabitEthernet0/0/1
 description Trunk to HQ-SW2
 no ip address
 no shutdown
exit

! --- Sub-Interfaces with HSRP ---
interface GigabitEthernet0/0/1.10
 description VLAN 10 (HQ Wired PCs)
 encapsulation dot1Q 10
 ip address 10.0.10.3 255.255.255.0
 ip nat inside
 standby 10 ip 10.0.10.1
 standby 10 priority 90
 standby 10 preempt
exit

interface GigabitEthernet0/0/1.11
 description VLAN 11 (HQ Staff WiFi)
 encapsulation dot1Q 11
 ip address 10.0.11.3 255.255.255.0
 ip nat inside
 standby 11 ip 10.0.11.1
 standby 11 priority 90
 standby 11 preempt
exit

interface GigabitEthernet0/0/1.20
 description VLAN 20 (DMZ / Server Farm)
 encapsulation dot1Q 20
 ip address 10.0.20.3 255.255.255.0
 ip nat inside
 ip access-group HQ_DMZ_CONTAINMENT in
 standby 20 ip 10.0.20.1
 standby 20 priority 90
 standby 20 preempt
exit

interface GigabitEthernet0/0/1.30
 description VLAN 30 (HQ Guest WiFi)
 encapsulation dot1Q 30
 ip address 10.0.30.3 255.255.255.0
 ip nat inside
 ip access-group HQ_GUEST_WIFI_ACL in
 standby 30 ip 10.0.30.1
 standby 30 priority 90
 standby 30 preempt
exit

interface GigabitEthernet0/0/1.40
 description VLAN 40 (HQ Surveillance)
 encapsulation dot1Q 40
 ip address 10.0.40.3 255.255.255.0
 ip nat inside
 standby 40 ip 10.0.40.1
 standby 40 priority 90
 standby 40 preempt
exit

interface GigabitEthernet0/0/1.99
 description VLAN 99 (IT Management)
 encapsulation dot1Q 99 native
 ip address 10.0.99.3 255.255.255.0
 ip nat inside
 standby 99 ip 10.0.99.1
 standby 99 priority 90
 standby 99 preempt
exit

! --- Routing Protocol (OSPF Area 0) ---
router ospf 1
 router-id 4.4.4.4
 network 10.0.0.0 0.0.255.255 area 0
 default-information originate
exit

ip route 0.0.0.0 0.0.0.0 203.0.113.5

! --- Access Control Lists (ACLs) ---
! Guest WiFi Block
ip access-list extended HQ_GUEST_WIFI_ACL
 remark Prevent HQ Guest WiFi from accessing Internal LANs & DMZ
 deny ip 10.0.30.0 0.0.0.255 10.0.0.0 0.255.255.255
 remark Allow HQ Guest WiFi to Internet
 permit ip 10.0.30.0 0.0.0.255 any
exit

! DMZ Containment Block
ip access-list extended HQ_DMZ_CONTAINMENT
 remark Prevent DMZ Servers from initiating to Internal LANs
 deny ip 10.0.20.0 0.0.0.255 10.0.10.0 0.0.0.255
 deny ip 10.0.20.0 0.0.0.255 10.0.11.0 0.0.0.255
 remark Allow DMZ Servers to reply/route to Internet
 permit ip 10.0.20.0 0.0.0.255 any
 permit ip any any
exit

! --- NAT Configuration ---
access-list 1 permit 10.0.0.0 0.255.255.255
access-list 1 permit 10.1.0.0 0.255.255.255
access-list 1 permit 10.2.0.0 0.255.255.255

ip nat inside source list 1 interface GigabitEthernet0/0/0 overload
! Static NAT for DMZ Web Server via Backup ISP Link
ip nat inside source static tcp 10.0.20.5 80 203.0.113.6 80
ip nat inside source static tcp 10.0.20.5 443 203.0.113.6 443

end
write memory
```

### 10.2 High Availability HQ Switch (HQ-SW2)

The `HQ-SW2` acts as a redundant core switch providing a trunk to `HQ-R2`, a cross-over inter-switch link to `HQ-SW1`, and trunks down to the access layer switches.

```text
enable
configure terminal
hostname HQ-SW2

! --- Create VLANs ---
vlan 10
 name HQ_Wired_PCs
vlan 11
 name HQ_Staff_WiFi
vlan 20
 name DMZ_Server_Farm
vlan 30
 name HQ_Guest_WiFi
vlan 40
 name HQ_Surveillance
vlan 99
 name IT_Management
exit

! --- Uplink Trunk to HQ Router (HQ-R2) ---
interface GigabitEthernet1/1/1
 description Trunk to HQ-R2
 switchport mode trunk
 switchport trunk native vlan 99
 no shutdown
exit

! --- Inter-Switch Trunk to HQ-SW1 ---
interface GigabitEthernet1/1/2
 description Trunk to HQ-SW1
 switchport mode trunk
 switchport trunk native vlan 99
 no shutdown
exit

! --- Trunks to Access Switches ---
interface range GigabitEthernet1/0/1 - 4
 description Trunk to Access Switches
 switchport mode trunk
 switchport trunk native vlan 99
 no shutdown
exit

! --- Management Interface ---
interface vlan 99
 ip address 10.0.99.3 255.255.255.0
 no shutdown
exit
ip default-gateway 10.0.99.1

end
write memory
```

*(You must also update HQ-R1 to have physical IP `10.0.x.2` and `standby 10 priority 110` so it remains the active router).*

### 10.2 Cisco ASA Perimeter Firewall (HQ-FW1)

Place the ASA between the ISP and the Core Routers.

```text
enable
configure terminal
hostname HQ-FW1

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
exit

! Route traffic inside to HQ-R1/R2 and outside to ISP
route outside 0.0.0.0 0.0.0.0 203.0.113.1 1
route inside 10.0.0.0 255.0.0.0 192.168.1.2 1
```

### 10.3 Core vs Access Layer Switching (HQ-ACC1)

Deploy a dedicated Access Switch (`HQ-ACC1`) to offload end-user ports from the Core Switch (`HQ-SW1`).

```text
enable
configure terminal
hostname HQ-ACC1

! Uplink to Core Switch
interface GigabitEthernet0/1
 switchport mode trunk
 no shutdown
exit

! Access Ports for PCs and APs
interface range FastEthernet0/1 - 10
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
exit

end
write memory
```

### 10.4 Wireless LAN Controller (WLC)

Instead of autonomous APs, use a **WLC-3504** and **LAP-PT** (Lightweight Access Points) in Packet Tracer.

1. Connect WLC to `HQ-SW1` on a trunk port or management VLAN 99.
2. Go to WLC GUI via Web Browser.
3. Configure Dynamic Interfaces for VLAN 11 (Staff) and VLAN 30 (Guest).
4. Create WLANs tied to these interfaces with WPA2-PSK.
5. LAPs will automatically pull IP via DHCP (Option 43 pointing to WLC IP) and download the SSIDs.

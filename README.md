# Multisite Enterprise Network Architecture: Static WAN & Centralized DHCP Relay

## 📌 Project Overview
This baseline enterprise infrastructure lab demonstrates the implementation, convergence, and operational verification of a production-ready two-site corporate topology inside Cisco Packet Tracer. The topology establishes secure communication lanes for a Headquarters (HQ) hub site linked over a point-to-point Wide Area Network (WAN) link to a remote Branch field office.

The technical core of this deployment highlights logical broadcast domain isolation via **IEEE 802.1Q Virtual Local Area Networks (VLANs)**, **Inter-VLAN routing** utilizing a Router-on-a-Stick (ROAS) design, and a complex multi-hop **DHCP Relay (IP Helper-Address)** pipeline crossing Layer 3 boundaries.

---

## 🗺️ Logical Network Topology
```text
          [ HQ-Staff-PC ] (VLAN 11)
                 │ (Fa0/1)
                 ▼
          [ HQ-Core-SW ] (VLAN 10,11) ◄── (Fa0/24) ── [ Central DHCP Server ] (VLAN 10)
                 │ (Gig0/1 - Trunk)
                 ▼
           [ HQ-Router ] 
                 │ (Se0/1/0) [192.168.12.1/30]
                 ▼
             WAN LINK  (Serial Point-to-Point)
                 ▲
                 │ (Se0/1/0) [192.168.12.2/30]
          [ Branch-Router ]
                 │ (Gig0/0/0 - Trunk)
                 ▼
        [ Branch-Access-SW ] (VLAN 21)
                 │ (Fa0/1)
                 ▼
         [ Branch-Staff-PC ] (VLAN 21)

# Multisite Enterprise Network Baseline: Static WAN & Centralized DHCP Relay

## 📊 Structural Network Addressing Matrix

| Functional Location | Logical Layer / Interface | VLAN Assignment | IP Subnet Profile | Subnet Mask | Default Gateway |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Inter-Site WAN Link** | HQ-Router Se0/1/0 | N/A (Point-to-Point) | 192.168.12.0/30 | 255.255.255.252 | N/A |
| **Inter-Site WAN Link** | Branch-Router Se0/1/0 | N/A (Point-to-Point) | 192.168.12.0/30 | 255.255.255.252 | N/A |
| **Headquarters LAN** | HQ-Router Gig0/0/0.10 | VLAN 10 (Servers) | 10.1.10.0/24 | 255.255.255.0 | 10.1.10.1 |
| **Headquarters LAN** | HQ-Router Gig0/0/0.11 | VLAN 11 (HQ-Staff) | 10.1.11.0/24 | 255.255.255.0 | 10.1.11.1 |
| **Branch Office LAN** | Branch-Router Gig0/0/0.21 | VLAN 21 (Br-Staff) | 10.2.21.0/24 | 255.255.255.0 | 10.2.21.1 |

---

## 🛠️ Physical Device Interconnections (Port Maps)

### 1. Headquarters (HQ Site)
* **HQ-Router [Gig0/0/0]** $\rightarrow$ **HQ-Core-SW [Gig0/1]** *(802.1Q Native Trunk Protocol)*
* **HQ-Core-SW [Fast0/1]** $\rightarrow$ **HQ-Staff-PC [Fast0/0]** *(Access Domain: VLAN 11)*
* **HQ-Core-SW [Fast0/24]** $\rightarrow$ **DHCP-Server [Fast0/0]** *(Access Domain: VLAN 10)*

### 2. Branch Office Site
* **Branch-Router [Gig0/0/0]** $\rightarrow$ **Branch-Access-SW [Gig0/1]** *(802.1Q Native Trunk Protocol)*
* **Branch-Access-SW [Fast0/1]** $\rightarrow$ **Branch-Staff-PC [Fast0/0]** *(Access Domain: VLAN 21)*

---

## ⚙️ Engineering Deep-Dive: Centralized DHCP Relay

By default, Layer 3 interfaces act as structural firewalls against broadcast frames; they do not forward DHCP discovery broadcasts beyond their localized broadcast domain. To solve this restriction across the topology, `ip helper-address 10.1.10.10` was configured on all client-facing gateway sub-interfaces:

* **On the Branch Router (`Gig0/0/0.21`):** Intercepts incoming client broadcasts, converts them into a unicast payload, and routes them across the WAN serial interface to the centralized server at HQ.
* **On the HQ Router (`Gig0/0/0.11`):** Acts as the relay between the Staff PC broadcast domain and the server broadcast domain inside the same building.

---

## 🧪 Operational Verification & Troubleshooting Record

### The Incident
During initial validation, the remote Branch PC successfully claimed a dynamic IP address via the WAN tunnel, but the local HQ PC returned a `DHCP failed. APIPA is being used.` message.

### The Diagnostics & Resolution
* **Switch-Port Boundary Verification:** Running `show interface status` on `HQ-Core-SW` revealed that `Fast0/1` had defaulted into VLAN 1 instead of its intended VLAN 11 assignment, keeping the PC isolated from the router's sub-interface.
* **Action Taken:** The access port was manually re-assigned using the following block:
  ```text
  HQ-Core-SW(config)# interface FastEthernet0/1
  HQ-Core-SW(config-if)# switchport access vlan 11

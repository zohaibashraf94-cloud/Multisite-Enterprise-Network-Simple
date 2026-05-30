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

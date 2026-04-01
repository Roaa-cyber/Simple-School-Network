# School-Wide Network Design & Implementation

## Project Overview

This project details the comprehensive design and implementation of a secure, scalable, and robust network for a multi-branch school organization. The network architecture was developed to meet current operational demands while providing a flexible foundation for future technological growth. Key considerations included high bandwidth availability, layered security, redundancy for critical services, and seamless scalability.

This documentation provides a complete overview of the network topology, hardware specifications, IP addressing scheme, routing protocols, and security policies deployed across the Headquarters (HQ), Branch A, and Branch B.

## Network Topology Overview

The network follows a hub-and-spoke topology, with the Headquarters (HQ) acting as the central point connecting two branch locations and the Internet Service Provider (ISP).

### Headquarters (HQ)
- **Router:** Cisco 2911
- **Switch:** Cisco 2960
- **End Devices:** Admin PCs, HR PCs, Student PCs
- **Servers:** Application Server
- **Printer:** HR Printer
- **WAN Connections:** Dedicated serial links to Branch A, Branch B, and ISP.

### Branch A
- **Router:** Cisco 2911
- **Switch:** Cisco 2960
- **End Devices:** Staff PCs, Student PCs
- **Printer:** Branch A Printer
- **WAN Connection:** Serial link to HQ.

### Branch B
- **Router:** Cisco 2911
- **Switch:** Cisco 2960
- **End Devices:** Staff PCs, Student PCs
- **Printer:** Branch B Printer
- **WAN Connection:** Serial link to HQ.

### ISP Simulation
- **Router:** Cisco 2911
- **Switch:** Cisco 2960
- **Server:** Public Internet Server

---

## IP Addressing Scheme

A structured IP addressing plan was implemented to ensure efficient management, routing, and scalability. VLANs are used for logical segmentation at HQ.

### Network & VLAN Table

| Site / Network | Network (CIDR) | Mask | Gateway | Usable Hosts |
| :--- | :--- | :--- | :--- | :--- |
| **HQ Admin (VLAN10)** | 192.168.10.0/30 | 255.255.255.252 | 192.168.10.1 | .1 – .2 |
| **HQ HR (VLAN20)** | 192.168.20.0/28 | 255.255.255.240 | 192.168.20.1 | .1 – .14 |
| **HQ Servers (VLAN30)** | 192.168.30.0/29 | 255.255.255.248 | 192.168.30.1 | .1 – .6 |
| **HQ Students (VLAN40)** | 192.168.40.0/24 | 255.255.255.0 | 192.168.40.1 | .1 – .254 |
| **Branch A HR** | 192.168.120.0/24 | 255.255.255.0 | 192.168.120.1 | .1 – .254 |
| **Branch A Students** | 192.168.140.0/24 | 255.255.255.0 | 192.168.140.1 | .1 – .254 |
| **Branch B HR** | 192.168.220.0/24 | 255.255.255.0 | 192.168.220.1 | .1 – .254 |
| **Branch B Students** | 192.168.240.0/24 | 255.255.255.0 | 192.168.240.1 | .1 – .254 |
| **WAN HQ–ISP** | 172.16.0.0/30 | 255.255.255.252 | HQ=172.16.0.2 | .1 – .2 |
| **WAN HQ–BA** | 172.16.0.4/30 | 255.255.255.252 | HQ=172.16.0.5 | .5 – .6 |
| **WAN HQ–BB** | 172.16.0.8/30 | 255.255.255.252 | HQ=172.16.0.9 | .9 – .10 |
| **ISP Public** | 203.0.113.0/24 | 255.255.255.0 | ISP=203.0.113.1 | .1 – .254 |

### VLAN Design

| VLAN ID | VLAN Name | Network Range | Usage |
| :--- | :--- | :--- | :--- |
| 10 | Administration | 192.168.10.0/30 | HQ – Admin |
| 20 | HR Management | 192.168.20.0/28 | HQ HR, Branch A Staff, Branch B Staff, Printers |
| 30 | Servers | 192.168.30.0/29 | HQ – Servers |
| 40 | Students | 192.168.40.0/24 | HQ, Branch A, Branch B – Students |

### WAN Addressing Scheme

| Link | Network | HQ IP | Branch/ISP IP |
| :--- | :--- | :--- | :--- |
| HQ – ISP | 172.16.0.0/30 | 172.16.0.2 | 172.16.0.1 (ISP) |
| HQ – Branch A | 172.16.0.4/30 | 172.16.0.5 | 172.16.0.6 |
| HQ – Branch B | 172.16.0.8/30 | 172.16.0.9 | 172.16.0.10 |

### Internet Simulation
- **ISP Network:** 203.0.113.0/24 (TEST-NET-3)
- **ISP Gateway:** 203.0.113.1
- **Internet Server:** 203.0.113.10

---

## Routing Protocols

A hybrid routing approach was employed to optimize for both security and dynamic route management.

- **Static Routing:** A default static route is configured on the HQ router, pointing to the ISP for all outbound internet traffic.
- **OSPF (Open Shortest Path First):** OSPF, in a single area (Area 0), is used for dynamic routing between HQ, Branch A, and Branch B. This ensures automatic failover and reachability across all internal subnets without manual route updates.

---

## DHCP Configuration

Dynamic Host Configuration Protocol (DHCP) is implemented on the routers at HQ and each branch to automate IP address assignment for end devices, significantly reducing administrative overhead.

- **Scope:** Separate DHCP pools are configured for each VLAN.
- **Examples:**
    - **VLAN 20 (HR):** 192.168.20.2 – 192.168.20.14
    - **VLAN 40 (Students):** 192.168.40.2 – 192.168.40.254

---

## Security Measures

A multi-layered security approach is implemented to protect the network from internal and external threats.

### 1. VLAN Segmentation
Network traffic is logically separated into different VLANs (Admin, HR, Servers, Students), isolating sensitive data and limiting the potential impact of a security breach.

### 2. Access Control Lists (ACLs)
Two ACLs are configured on the HQ router to enforce traffic policies:

- **Standard ACL:** Permits specific IP addresses for Network Address Translation (NAT) configuration.
- **Extended ACL:** Applies strict restrictions on student traffic (VLAN 40):
    - Denies access to the external Internet Server (203.0.113.10).
    - Denies inter-VLAN communication.
    - Denies the use of HTTP protocol.
    - Permits all other traffic.

### 3. Port Security
Configured on all switch access ports to prevent MAC address spoofing and unauthorized device connections.
- **Mode:** Restrict to a maximum of 1 MAC address per port.
- **Violation Action:** Shutdown the port upon a violation.

### 4. NAT and PAT
- **Static NAT:** Maps the internal Application Server (192.168.30.x) to a public IP address, making it accessible from the internet.
- **PAT (Port Address Translation) / Overload:** Allows all internal devices to share a single public IP address for outbound internet access, conserving public IP space and enabling simultaneous connections.

---

## Cable Types

The physical infrastructure uses standard cabling for reliability and performance.

| Connection Type | Cable Type |
| :--- | :--- |
| Router ↔ Switch | Copper Straight-Through (Gigabit) |
| Router ↔ Router (WAN) | Serial DCE/DTE Cable |
| Switch ↔ End Devices (PCs, Servers, Printers) | Copper Straight-Through |

---

## Expected Network Behavior

Upon full implementation, the network is designed to function with the following characteristics:

- **Inter-VLAN Communication:** Achieved through a Router-on-a-Stick configuration at HQ and branch routers.
- **Branch Connectivity:** OSPF ensures that Branch A and Branch B can communicate with HQ and each other.
- **Internet Access:** All PCs can access the internet through the HQ router's PAT configuration, with the exception of the restricted student VLAN 40 traffic.
- **Server Hosting:** The application server at HQ is accessible both internally and externally via static NAT.
- **DHCP Dynamic Allocation:** End devices in all VLANs will automatically receive IP addresses from their respective DHCP pools.
- **Access Restrictions:** Student devices will be unable to access the external Internet Server or any other VLAN. HR, Admin, and Server networks will have unrestricted internal and external access as defined by the ACLs.

---

## Conclusion

This network design successfully delivers a balanced solution that prioritizes functionality, scalability, and security. The use of VLANs provides logical separation and security. DHCP simplifies client management, OSPF ensures dynamic and resilient routing, ACLs enforce granular security policies, and NAT/PAT facilitates efficient internet access for all users. The resulting infrastructure is robust enough to handle current operational requirements while remaining flexible and scalable to accommodate future growth and technological advancements.

## Author

Created by Roa'a Alahjeri project demonstrating machine learning in cybersecurity.

---

# enterprise-roas-dhcp-hardening
Cisco Packet Tracer simulation of a secure enterprise network with ROAS, DHCP, and SSHv2 hardening.
# Secure Multi-VLAN Enterprise Network Simulation

## Project Overview
This project models a production-grade enterprise network topology featuring segmented broadcast domains, automated network addressing, inter-VLAN routing, and device hardening. Designed and validated within Cisco Packet Tracer, this architecture isolates organizational traffic zones and secures internal infrastructure management pathways to adhere to strict security and compliance standards.

## Network Architecture & Design
The topology utilizes a Router-on-a-Stick (ROAS) model to route traffic across four distinct functional subnets through a single physical interface line (`Fa0/1` to `Gi0/0`).

### Network Addressing & Segmentation Map

| VLAN ID | Subnet Name   | CIDR Block        | Default Gateway | Allocation Type |
| :---    | :---          | :---              | :---            | :---            |
| **10**  | Staff_Data    | 192.168.10.0/24   | 192.168.10.1    | DHCP Allocated  |
| **20**  | Guest_WiFi    | 192.168.20.0/24   | 192.168.20.1    | DHCP Allocated  |
| **30**  | Corp_Server   | 192.168.30.0/24   | 192.168.30.1    | DHCP Allocated  |
| **99**  | Management    | 192.168.99.0/24   | 192.168.99.1    | Statically Bound|

---

## Core Engineering Features

### 1. Inter-VLAN Routing (ROAS)
* Configured IEEE 802.1Q encapsulation and logical subinterfaces on a Cisco ISR Router to facilitate secure inter-VLAN routing.
* Established an isolated Layer 2 native VLAN environment to prevent data contamination and mitigate VLAN hopping vulnerabilities.

### 2. Automated Network Services (DHCP)
* Deployed localized Cisco IOS DHCP pools (`STAFF_POOL`, `GUEST_POOL`, `SERVER_POOL`) to automate client IP allocation across corporate zones.
* Implemented explicit `ip dhcp excluded-address` blocks (`.1` through `.10`) to safeguard structural network gateways, switches, and static server infrastructure.

### 3. Layer 3 Security Access Controls (ACLs)
* Engineered and applied an extended Access Control List (`BLOCK_GUEST_TO_SERVER`) on the inbound direction of the Guest subinterface (`Gi0/0.20`).
* Enforced strict network segmentation by intercepting and dropping unauthorized inbound traffic originating from the Guest WiFi segment attempting to communicate with the Corporate Server segment (`192.168.30.0/24`).

### 4. Device Hardening & Management Plane Security
* Disabled insecure, plaintext management frameworks (Telnet) and enforced encrypted cryptographic access via **SSHv2** utilizing a 1024-bit RSA key modulus across all virtual terminal lines (`line vty 0 15`).
* Restricted remote management entry across the network by embedding Switch Virtual Interfaces (SVIs) exclusively on a dedicated, statically bound Management VLAN (VLAN 99).
* Configured local authentication with privilege level 15 administrative credentials and protected access methods via an encrypted `enable secret`.

---

## Verification & Testing Records
* **DHCP Verification:** End hosts successfully pulled dynamic leases beginning precisely at `.11` (e.g., Staff PC received `192.168.10.11`), confirming the structural exclusion policies worked perfectly.
* **Security Validation:** Ping tests from `VLAN 10` to `VLAN 30` resolved successfully. ICMP requests from `VLAN 20` to `VLAN 30` dropped immediately at the router subinterface, successfully returning `Destination host unreachable`.
* **SSH Hardening Validation:** Remotely connected to the Switch CLI over port 22 from the dedicated management workstation (PC4) using localized cryptographic credentials via `ssh -l admin 192.168.99.2`.

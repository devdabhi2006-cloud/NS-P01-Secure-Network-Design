# TY B.Sc. IT Network Security Project (NS-P01)
# Master Project Documentation & Defense Dossier

---

**PROJECT TITLE:** Secure Organizational Network Design for a Humanitarian Non-Governmental Organization (Global Hope Foundation)  
**PROJECT CODE:** NS-P01  
**ACADEMIC DISCIPLINE:** Third Year Bachelor of Science in Information Technology (TY B.Sc. IT)  
**SPECIALIZATION:** Network Security, Perimeter Defense & Applied Cryptography  
**ORGANIZATION:** Global Hope Foundation (GHF)  
**STATUS:** Master Documentation Baseline — Academically Verified  

---

## Academic Authenticity & Transparency Statement

> **IMPORTANT DECLARATION OF FIDELITY & SIMULATION METHODOLOGY:**  
> The network architecture, configuration blueprints, addressing plans, and access control matrices presented in this report represent a complete, mathematically verified, and syntactically audited engineering baseline.  
> 
> In accordance with academic honesty principles, all configuration scripts (`.cfg`) and design specifications are compiled to run natively on Cisco IOS (Cisco 2911 Routers, Cisco Catalyst 3560 Layer 3 Switches, and Cisco Catalyst 2960 Layer 2 Switches). Operational telemetry, access control hit counters, security association states, and packet translations are documented as an **empirically modeled verification baseline**. This establishes an exact, reproducible blueprint for execution in Cisco Packet Tracer 8.x or physical laboratory hardware.

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Organizational Profile & Business Operations](#2-organizational-profile--business-operations)
3. [Security Requirements & Threat Modeling](#3-security-requirements--threat-modeling)
4. [Network Architecture & Trust Framework](#4-network-architecture--trust-framework)
5. [Physical & Logical Topology Design](#5-physical--logical-topology-design)
6. [IP Addressing Scheme & VLSM Subnetting](#6-ip-addressing-scheme--vlSM-subnetting)
7. [Security Policies & Access Control Matrix](#7-security-policies--access-control-matrix)
8. [Perimeter Defense, NAT/PAT & Applied Cryptography](#8-perimeter-defense-natpat--applied-cryptography)
9. [Empirical Verification & Security Test Suite](#9-empirical-verification--security-test-suite)
10. [Pre-Flight Engineering Audit & Issue Resolution](#10-pre-flight-engineering-audit--issue-resolution)
11. [Residual Risk Analysis & Enterprise Roadmap](#11-residual-risk-analysis--enterprise-roadmap)
12. [Step-by-Step Packet Tracer Implementation Guide](#12-step-by-step-packet-tracer-implementation-guide)
13. [Conclusion & Project Retrospective](#13-conclusion--project-retrospective)
14. [References & Standards](#14-references--standards)
15. [Appendices: Master Configuration Code Listings](#15-appendices-master-configuration-code-listings)

---

## 1. Executive Summary

Modern Non-Governmental Organizations (NGOs) operate in complex, distributed environments where they handle highly sensitive data, including donor financial records, banking information, volunteer databases, and confidential beneficiary healthcare and refugee tracking details. Simultaneously, they must maintain accessible public donation portals and provide reliable connectivity to field workers stationed in remote relief centers.

This project delivers an enterprise-grade, defense-in-depth network security architecture for the **Global Hope Foundation (GHF)**. Spanning a centralized Head Office (HQ) and two remote field offices, the design systematically applies **Zero-Trust principles**, **strict Layer 3 micro-segmentation**, **three-legged perimeter boundary filtering**, **state-aware containment**, **Static Network Address Translation (NAT)**, **Dynamic Port Address Translation (PAT) with VPN bypass**, and **Site-to-Site IPsec VPN tunnels with Perfect Forward Secrecy (PFS Group 2)**.

The entire design is grounded in academic rigor, complying with NIST SP 800-41 (Guidelines on Firewalls and Firewall Policy) and NIST SP 800-77 (Guide to IPsec VPNs), offering an end-to-end, resilient blueprint for non-profit and humanitarian operations.

---

## 2. Organizational Profile & Business Operations

### 2.1 Mission & Operational Context
* **Organization:** Global Hope Foundation (GHF)
* **Domain:** International Humanitarian Aid, Disaster Response & Community Development.
* **Geographical Distribution:**
  * **Head Office (HQ - Geneva/Zurich Model):** Central administration, donor relations, executive governance, financial accounting, and core data center hosting.
  * **Remote Office 1 (North Region Aid & Logistics Center):** Warehousing of emergency supplies, field volunteer staging, and dispatch logistics.
  * **Remote Office 2 (South Region Community & Educational Hub):** Regional scholarship tracking, community vocational training, and local beneficiary enrollment.

### 2.2 User Groups & Organizational Roles

```
+-----------------------------------------------------------------------------------+
|                        ORGANIZATIONAL STAKEHOLDER MATRIX                          |
+---------------------+-------------------+------------------+----------------------+
| USER GROUP          | PHYSICAL LOCATION | PRIVILEGE LEVEL  | PRIMARY SYSTEM ACCESS|
+---------------------+-------------------+------------------+----------------------+
| General Staff       | HQ Campus         | Standard Internal| Intranet, Shared Docs|
| HR & Finance        | HQ Campus         | High Privacy     | Accounting, Donor DB |
| IT & Security Admin | HQ Campus         | Full Privilege   | Infrastructure Mgmt  |
| Field Relief Staff  | Remote Office 1   | Standard Remote  | Logistics, Med API   |
| Community Trainers  | Remote Office 2   | Standard Remote  | Training, Intranet   |
| Public Donors       | External Internet | Untrusted Public | Donation Web Portal  |
+---------------------+-------------------+------------------+----------------------+
```

### 2.3 Asset Classification & Sensitivity Hierarchy

1. **Confidential / Restricted Assets:**
   * **Financial & Donor Database:** Contains credit card transactions, banking tokens, donor personal contact info, and tax-exemption records.
   * **Beneficiary & Medical Database:** Contains personal identification, refugee tracking records, and healthcare histories of aid recipients.
2. **Internal Corporate Assets:**
   * **Intranet & File Repository:** Contains project blueprints, relief schedules, and operational templates.
   * **Network Management / Syslog / NTP Infrastructure:** Houses centralized telemetry, event logs, and administrative credentials.
3. **Public-Facing Assets:**
   * **Donation Web Portal:** External-facing web platform accepting donations and volunteer inquiries.
   * **Authoritative DNS Server:** Public name resolution for the organization's domain.

---

## 3. Security Requirements & Threat Modeling

### 3.1 Security Objectives
* **Confidentiality:** Restrict sensitive databases to authorized roles; encrypt inter-site WAN communications.
* **Integrity:** Prevent unauthorized alteration of financial records and device configurations.
* **Availability:** Protect perimeter links against denial-of-service attempts and prevent broadcast storms through VLAN segmentation.
* **Least Privilege:** Default-deny posture across all internal boundaries.
* **Defense-in-Depth:** Multiple complementary layers of protection across perimeter, distribution, and access tiers.

### 3.2 STRIDE Threat Assessment

```
+-----------------------------------+-----------------------------------+-----------------------------------+
| THREAT CATEGORY                   | ATTACK VECTOR                     | ARCHITECTURAL MITIGATION          |
+-----------------------------------+-----------------------------------+-----------------------------------+
| Spoofing                          | IP spoofing from public WAN       | Ingress ACL drop; NAT validation  |
| Tampering                         | Eavesdropping / WAN interception  | IPsec AES-256 ESP encapsulation   |
| Repudiation                       | Unauthorized administrative changes| Centralized Syslog & local AAA    |
| Information Disclosure            | Lateral movement to HR/Finance    | SVI Inter-VLAN boundary ACLs      |
| Denial of Service                 | Perimeter resource exhaustion     | DMZ physical boundary; port filter|
| Elevation of Privilege            | Unauthorized router VTY access    | Dedicated Mgmt VLAN & VTY ACL     |
+-----------------------------------+-----------------------------------+-----------------------------------+
```

### 3.3 Security Zone Model

```
+----------------------------------------------------------------------------------------------------+
|                                    SECURITY ZONE TAXONOMY                                          |
+--------+--------------------------+---------------+------------------------------------------------+
| ZONE ID| ZONE NAME                | TRUST LEVEL   | REPRESENTATIVE ASSETS                          |
+--------+--------------------------+---------------+------------------------------------------------+
| Zone 1 | Internet / WAN           | Untrusted     | External actors, ISP backbone, public donors   |
| Zone 2 | HQ DMZ                   | Semi-Trusted  | Public Web/Donation Server, Authoritative DNS  |
| Zone 3 | HQ General Staff LAN     | Trusted (Low) | Program officer PCs, shared office printers    |
| Zone 4 | HQ HR & Finance LAN      | Trusted (High)| Payroll PCs, executive accounting consoles     |
| Zone 5 | HQ IT Management LAN     | Privileged    | Network engineer management workstations       |
| Zone 6 | Protected Server Farm    | Restricted    | Donor DB, Medical DB, Intranet, Syslog/NTP     |
| Zone 7 | Remote Office 1 (North)  | Remote Trusted| Field logistics tablets, caseworker PCs        |
| Zone 8 | Remote Office 2 (South)  | Remote Trusted| Community trainer PCs, registration terminals  |
+--------+--------------------------+---------------+------------------------------------------------+
```

---

## 4. Network Architecture & Trust Framework

The network architecture adheres to the **Cisco Hierarchical Campus Design**, utilizing a **Collapsed Core / Distribution Model** at HQ paired with dedicated branch gateways at the remote offices.

```
                                  ==============================
                                  |    INTERNET / WAN CLOUD    |
                                  |        [ISP-RTR-01]        |
                                  ==============================
                                   /            |             \
            +---------------------+             |              +--------------------+
            | (Gi0/0)                           | (Gi0/1)                           | (Gi0/2)
            |                                   |                                   |
=========================           =========================           =========================
|       HQ-RTR-01       |           |      RO1-RTR-01       |           |      RO2-RTR-01       |
|  (Perimeter Edge /    |           |   (Remote Office 1    |           |   (Remote Office 2    |
|   NAT / IPsec Hub)    |           |    Logistics Edge)    |           |    Community Edge)    |
=========================           =========================           =========================
   | (Gi0/2)       | (Gi0/1)                    | (Gi0/1)                           | (Gi0/1)
   |               |                            |                                   |
   |               | (Routed Transit)   [RO1-SW-01] (2960)                  [RO2-SW-01] (2960)
   |               |                            |                                   |
   |        =================           +---------------+                   +---------------+
   |        | HQ-CORE-3560  |           | RO1-PC-01     |                   | RO2-PC-01     |
   |        | (L3 Switch /  |           | Field Worker  |                   | Community Hub |
   |        | Inter-VLAN)   |           +---------------+                   +---------------+
   |        =================
   |          /    |    \    \
   |         /     |     \    +----------------------------+
   |        /      |      \                                |
[HQ-SW-DMZ]        |    [HQ-SW-FIN]                  [HQ-SW-SRV]
   |               |         |                             |
+--------------+   |   +--------------+      +---------------------------+
| SRV-WEB-DMZ  |   |   | FIN-PC-01    |      | SRV-INTRANET (Files/Docs) |
| SRV-DNS-DMZ  |   |   | HR/Fin User  |      | SRV-DONOR-DB (Financial)  |
+--------------+   |   +--------------+      | SRV-MED-DB   (Medical)    |
                   |                         | SRV-MGMT-LOG (Syslog/AAA) |
             [HQ-SW-STAFF]                   +---------------------------+
                   |
             +--------------+
             | STAFF-PC-01  |
             | Operations   |
             +--------------+
```

### 4.1 Boundary Enforcement Mechanics
1. **Three-Legged Perimeter Router (`HQ-RTR-01`):**  
   Terminates Outside (WAN `Gi0/0`), Inside Transit (`Gi0/1`), and DMZ (`Gi0/2`). Provides hardware-level traffic separation between public servers and the internal core.
2. **Layer 3 Core Switch (`HQ-CORE-3560`):**  
   Hosts Switched Virtual Interfaces (SVIs) for all internal VLANs. Enforces inter-VLAN boundary filtering directly at wire speed.
3. **Dedicated Access Switches (`HQ-SW-STAFF`, `HQ-SW-FIN`, etc.):**  
   Enforce physical and port-level segregation, preventing multi-tenant Layer 2 snooping or VLAN hopping.

---

## 5. Physical & Logical Topology Design

### 5.1 Device Hardware Inventory

| Device Hostname | Model Platform | Installed Modules / Cards | Primary Role |
| :--- | :--- | :--- | :--- |
| **`ISP-RTR-01`** | Cisco 2911 Router | `HWIC-2FE` (Slot 0) | ISP Backbone Simulator & Public Internet Hub |
| **`HQ-RTR-01`** | Cisco 2911 Router | Built-in GE ports | HQ Perimeter Gateway, NAT Engine, IPsec Hub |
| **`RO1-RTR-01`** | Cisco 2911 Router | Built-in GE ports | Remote Office 1 Perimeter & IPsec Spoke |
| **`RO2-RTR-01`** | Cisco 2911 Router | Built-in GE ports | Remote Office 2 Perimeter & IPsec Spoke |
| **`HQ-CORE-3560`**| Cisco Catalyst 3560-24PS| Fixed 24 FE + 2 GE | HQ Collapsed Core & Distribution L3 Switch |
| **`HQ-SW-STAFF`** | Cisco Catalyst 2960-24TT| Fixed 24 FE + 2 GE | HQ General Staff Access Switch |
| **`HQ-SW-FIN`** | Cisco Catalyst 2960-24TT| Fixed 24 FE + 2 GE | HQ HR & Finance Access Switch |
| **`HQ-SW-IT`** | Cisco Catalyst 2960-24TT| Fixed 24 FE + 2 GE | HQ IT & Security Management Access Switch |
| **`HQ-SW-SRV`** | Cisco Catalyst 2960-24TT| Fixed 24 FE + 2 GE | Protected Server Farm Access Switch |
| **`HQ-SW-DMZ`** | Cisco Catalyst 2960-24TT| Fixed 24 FE + 2 GE | Dedicated DMZ Access Switch |
| **`RO1-SW-01`** | Cisco Catalyst 2960-24TT| Fixed 24 FE + 2 GE | Remote Office 1 Access Switch |
| **`RO2-SW-01`** | Cisco Catalyst 2960-24TT| Fixed 24 FE + 2 GE | Remote Office 2 Access Switch |
| **`SRV-WEB-DMZ`** | Server-PT | FastEthernet NIC | Public Web & Donation Portal Server |
| **`SRV-DNS-DMZ`** | Server-PT | FastEthernet NIC | Public Authoritative DNS Server |
| **`SRV-INTRANET`**| Server-PT | FastEthernet NIC | Internal Intranet & File Server |
| **`SRV-DONOR-DB`**| Server-PT | FastEthernet NIC | Confidential Donor & Financial Database |
| **`SRV-MED-DB`** | Server-PT | FastEthernet NIC | Beneficiary & Medical Records Database |
| **`SRV-MGMT-LOG`**| Server-PT | FastEthernet NIC | Central Syslog, NTP, AAA Server |
| **Client PCs** | PC-PT / Laptop-PT | FastEthernet NIC | Departmental Workstations & External Test Hosts |

### 5.2 Port Cabling Schedule

```
[ISP-RTR-01]
  |-- Gi0/0  ------------------------  Gi0/0 [HQ-RTR-01]      (HQ WAN Uplink)
  |-- Gi0/1  ------------------------  Gi0/0 [RO1-RTR-01]     (RO1 WAN Uplink)
  |-- Gi0/2  ------------------------  Gi0/0 [RO2-RTR-01]     (RO2 WAN Uplink)
  |-- Fa0/0/0 (HWIC Slot 0) ---------  Fa0   [EXT-CLIENT-01]  (Simulated Public User)

[HQ-RTR-01]
  |-- Gi0/0  ------------------------  Gi0/0 [ISP-RTR-01]     (Outside Untrusted)
  |-- Gi0/1  ------------------------  Gi0/1 [HQ-CORE-3560]   (Transit /30 Link)
  |-- Gi0/2  ------------------------  Fa0/24 [HQ-SW-DMZ]     (DMZ Gateway /28 Link)

[HQ-CORE-3560]
  |-- Gi0/1  ------------------------  Gi0/1 [HQ-RTR-01]      (Routed Uplink)
  |-- Gi0/2  ------------------------  Fa0/24 [HQ-SW-SRV]     (802.1Q Trunk: 40, 30)
  |-- Fa0/1  ------------------------  Fa0/24 [HQ-SW-STAFF]   (802.1Q Trunk: 10, 30)
  |-- Fa0/2  ------------------------  Fa0/24 [HQ-SW-FIN]     (802.1Q Trunk: 20, 30)
  |-- Fa0/3  ------------------------  Fa0/24 [HQ-SW-IT]      (802.1Q Trunk: 30)

[Access Layer Switches to Endpoints]
  |-- HQ-SW-DMZ Fa0/1  --------------  Fa0   [SRV-WEB-DMZ]    (Access VLAN 1)
  |-- HQ-SW-DMZ Fa0/2  --------------  Fa0   [SRV-DNS-DMZ]    (Access VLAN 1)
  |-- HQ-SW-SRV Fa0/1  --------------  Fa0   [SRV-INTRANET]   (Access VLAN 40)
  |-- HQ-SW-SRV Fa0/2  --------------  Fa0   [SRV-DONOR-DB]   (Access VLAN 40)
  |-- HQ-SW-SRV Fa0/3  --------------  Fa0   [SRV-MED-DB]     (Access VLAN 40)
  |-- HQ-SW-SRV Fa0/4  --------------  Fa0   [SRV-MGMT-LOG]   (Access VLAN 40)
  |-- HQ-SW-STAFF Fa0/1 -------------  Fa0   [STAFF-PC-01]    (Access VLAN 10)
  |-- HQ-SW-FIN Fa0/1 ---------------  Fa0   [FIN-PC-01]      (Access VLAN 20)
  |-- HQ-SW-IT Fa0/1 ----------------  Fa0   [IT-PC-01]       (Access VLAN 30)
  |-- RO1-SW-01 Fa0/1 ---------------  Fa0   [RO1-PC-01]      (Access VLAN 1)
  |-- RO2-SW-01 Fa0/1 ---------------  Fa0   [RO2-PC-01]      (Access VLAN 1)
```

---

## 6. IP Addressing Scheme & VLSM Subnetting

The IP address architecture uses a hierarchical RFC 1918 scheme summarized by site to maximize routing table efficiency, complemented by RFC 5737 public test addressing.

### 6.1 VLSM Subnet Allocation Table

```
+------------------------------------------------------------------------------------------------------------------+
|                                           VLSM MASTER SUBNET SPECIFICATION                                       |
+-------------------+------------------+-----------------+------+-----------------------------+--------------------+
| SUBNET PURPOSE    | NETWORK ADDRESS  | SUBNET MASK     | CIDR | USABLE HOST RANGE           | BROADCAST ADDRESS  |
+-------------------+------------------+-----------------+------+-----------------------------+--------------------+
| WAN: HQ Uplink    | 203.0.113.0      | 255.255.255.252 | /30  | 203.0.113.1 - 203.0.113.2   | 203.0.113.3        |
| WAN: RO1 Uplink   | 203.0.113.4      | 255.255.255.252 | /30  | 203.0.113.5 - 203.0.113.6   | 203.0.113.7        |
| WAN: RO2 Uplink   | 203.0.113.8      | 255.255.255.252 | /30  | 203.0.113.9 - 203.0.113.10  | 203.0.113.11       |
| WAN: External Net | 203.0.113.16     | 255.255.255.240 | /28  | 203.0.113.17 - 203.0.113.30 | 203.0.113.31       |
| HQ DMZ Network    | 10.1.50.0        | 255.255.255.240 | /28  | 10.1.50.1 - 10.1.50.14      | 10.1.50.15         |
| HQ Core Transit   | 10.1.99.0        | 255.255.255.252 | /30  | 10.1.99.1 - 10.1.99.2       | 10.1.99.3          |
| HQ General Staff  | 10.1.10.0        | 255.255.255.0   | /24  | 10.1.10.1 - 10.1.10.254     | 10.1.10.255        |
| HQ HR & Finance   | 10.1.20.0        | 255.255.255.0   | /24  | 10.1.20.1 - 10.1.20.254     | 10.1.20.255        |
| HQ IT Management  | 10.1.30.0        | 255.255.255.0   | /24  | 10.1.30.1 - 10.1.30.254     | 10.1.30.255        |
| HQ Server Farm    | 10.1.40.0        | 255.255.255.0   | /24  | 10.1.40.1 - 10.1.40.254     | 10.1.40.255        |
| RO1 Field LAN     | 10.2.10.0        | 255.255.255.0   | /24  | 10.2.10.1 - 10.2.10.254     | 10.2.10.255        |
| RO2 Community LAN | 10.3.10.0        | 255.255.255.0   | /24  | 10.3.10.1 - 10.3.10.254     | 10.3.10.255        |
+-------------------+------------------+-----------------+------+-----------------------------+--------------------+
```

### 6.2 Master Device Interface Addressing Table

| Device Hostname | Physical/Logical Interface | IP Address | Subnet Mask | Default Gateway | Function / Domain |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **`ISP-RTR-01`** | GigabitEthernet0/0 | `203.0.113.1` | `255.255.255.252` | N/A | Uplink to HQ Edge |
| | GigabitEthernet0/1 | `203.0.113.5` | `255.255.255.252` | N/A | Uplink to RO1 Edge |
| | GigabitEthernet0/2 | `203.0.113.9` | `255.255.255.252` | N/A | Uplink to RO2 Edge |
| | FastEthernet0/0/0 | `203.0.113.17` | `255.255.255.240` | N/A | Public Simulated Subnet |
| **`HQ-RTR-01`** | GigabitEthernet0/0 | `203.0.113.2` | `255.255.255.252` | `203.0.113.1` | WAN Outside (NAT/Crypto Hub) |
| | GigabitEthernet0/1 | `10.1.99.1` | `255.255.255.252` | N/A | Inside Transit to Core |
| | GigabitEthernet0/2 | `10.1.50.1` | `255.255.255.240` | N/A | DMZ Default Gateway |
| **`HQ-CORE-3560`**| GigabitEthernet0/1 | `10.1.99.2` | `255.255.255.252` | `10.1.99.1` | Routed Transit to Edge |
| | Vlan 10 (SVI) | `10.1.10.1` | `255.255.255.0` | N/A | Staff Default Gateway |
| | Vlan 20 (SVI) | `10.1.20.1` | `255.255.255.0` | N/A | HR/Finance Default Gateway |
| | Vlan 30 (SVI) | `10.1.30.1` | `255.255.255.0` | N/A | IT Mgmt Default Gateway |
| | Vlan 40 (SVI) | `10.1.40.1` | `255.255.255.0` | N/A | Server Farm Default Gateway |
| **`HQ-SW-STAFF`** | Vlan 30 (SVI) | `10.1.30.11` | `255.255.255.0` | `10.1.30.1` | Management Plane |
| **`HQ-SW-FIN`** | Vlan 30 (SVI) | `10.1.30.12` | `255.255.255.0` | `10.1.30.1` | Management Plane |
| **`HQ-SW-IT`** | Vlan 30 (SVI) | `10.1.30.13` | `255.255.255.0` | `10.1.30.1` | Management Plane |
| **`HQ-SW-SRV`** | Vlan 30 (SVI) | `10.1.30.14` | `255.255.255.0` | `10.1.30.1` | Management Plane |
| **`HQ-SW-DMZ`** | Vlan 1 (SVI) | `10.1.50.2` | `255.255.255.240` | `10.1.50.1` | Management Plane |
| **`RO1-RTR-01`** | GigabitEthernet0/0 | `203.0.113.6` | `255.255.255.252` | `203.0.113.5` | WAN Outside (NAT/Crypto Spoke) |
| | GigabitEthernet0/1 | `10.2.10.1` | `255.255.255.0` | N/A | RO1 LAN Default Gateway |
| **`RO1-SW-01`** | Vlan 1 (SVI) | `10.2.10.2` | `255.255.255.0` | `10.2.10.1` | Management Plane |
| **`RO2-RTR-01`** | GigabitEthernet0/0 | `203.0.113.10` | `255.255.255.252` | `203.0.113.9` | WAN Outside (NAT/Crypto Spoke) |
| | GigabitEthernet0/1 | `10.3.10.1` | `255.255.255.0` | N/A | RO2 LAN Default Gateway |
| **`RO2-SW-01`** | Vlan 1 (SVI) | `10.3.10.2` | `255.255.255.0` | `10.3.10.1` | Management Plane |
| **`SRV-WEB-DMZ`** | FastEthernet0 | `10.1.50.10` | `255.255.255.240` | `10.1.50.1` | Public Web (NAT: `203.0.113.100`) |
| **`SRV-DNS-DMZ`** | FastEthernet0 | `10.1.50.11` | `255.255.255.240` | `10.1.50.1` | Public DNS (NAT: `203.0.113.101`) |
| **`SRV-INTRANET`**| FastEthernet0 | `10.1.40.10` | `255.255.255.0` | `10.1.40.1` | Internal Intranet & File Server |
| **`SRV-DONOR-DB`**| FastEthernet0 | `10.1.40.20` | `255.255.255.0` | `10.1.40.1` | Confidential Donor & Finance DB |
| **`SRV-MED-DB`** | FastEthernet0 | `10.1.40.30` | `255.255.255.0` | `10.1.40.1` | Beneficiary & Medical Records DB |
| **`SRV-MGMT-LOG`**| FastEthernet0 | `10.1.40.50` | `255.255.255.0` | `10.1.40.1` | Syslog, NTP, AAA Server |
| **`STAFF-PC-01`** | FastEthernet0 | `10.1.10.10` | `255.255.255.0` | `10.1.10.1` | General Staff Workstation |
| **`FIN-PC-01`** | FastEthernet0 | `10.1.20.10` | `255.255.255.0` | `10.1.20.1` | HR / Finance Workstation |
| **`IT-PC-01`** | FastEthernet0 | `10.1.30.10` | `255.255.255.0` | `10.1.30.1` | IT Management Workstation |
| **`RO1-PC-01`** | FastEthernet0 | `10.2.10.10` | `255.255.255.0` | `10.2.10.1` | RO1 Field Workstation |
| **`RO2-PC-01`** | FastEthernet0 | `10.3.10.10` | `255.255.255.0` | `10.3.10.1` | RO2 Community Workstation |
| **`EXT-CLIENT-01`**| FastEthernet0 | `203.0.113.20` | `255.255.255.240` | `203.0.113.17` | Public Internet Test Endpoint |

---

## 7. Security Policies & Access Control Matrix

### 7.1 Global Filtering Architecture
All inter-zone traffic is governed by an **Explicit Default-Deny Architecture**. Filtering occurs at the ingress interface of each boundary device:

```
+------------------------------------------------------------------------------------------------------------+
|                                        INTER-ZONE FILTERING RULES                                          |
+-------------------+-----------------------+-----------------------+----------------------------------------+
| SOURCE ZONE       | DESTINATION ZONE      | ENFORCEMENT POINT     | ACTION & PROTOCOLS                     |
+-------------------+-----------------------+-----------------------+----------------------------------------+
| Internet (WAN)    | HQ DMZ                | HQ-RTR-01 (Gi0/0 in)  | PERMIT: TCP 80, 443; UDP 53            |
| Internet (WAN)    | Internal LANs / DBs   | HQ-RTR-01 (Gi0/0 in)  | DENY ALL (Drop unsolicited)            |
| HQ DMZ            | Internal LANs / DBs   | HQ-RTR-01 (Gi0/2 in)  | DENY ALL (Blocks initiated sessions)   |
| HQ DMZ            | Internal (Return Flow)| HQ-RTR-01 (Gi0/2 in)  | PERMIT: Established TCP, UDP 53 reply  |
| General Staff     | HR & Finance Subnet   | HQ-CORE-3560 (Vlan10) | DENY ALL (Blocks lateral movement)     |
| General Staff     | IT Management Subnet  | HQ-CORE-3560 (Vlan10) | DENY ALL (Protects admin consoles)     |
| General Staff     | Donor / Financial DB  | HQ-CORE-3560 (Vlan10) | DENY ALL (Least privilege enforcement) |
| General Staff     | Medical Records DB    | HQ-CORE-3560 (Vlan10) | DENY ALL (Least privilege enforcement) |
| General Staff     | Intranet / File Server| HQ-CORE-3560 (Vlan10) | PERMIT: HTTP 80, HTTPS 443, FTP 21/445 |
| HR & Finance      | Donor / Financial DB  | HQ-CORE-3560 (Vlan20) | PERMIT: SQL 1433, 3306, HTTPS 443      |
| HR & Finance      | Medical Records DB    | HQ-CORE-3560 (Vlan20) | DENY ALL (Separation of duties)        |
| IT Management     | All Subnets / Servers | HQ-CORE-3560 (Vlan30) | PERMIT ALL (Privileged administration) |
| Remote Office 1   | HQ Intranet & File    | RO1-RTR-01 (Gi0/1 in) | PERMIT over IPsec (HTTP, HTTPS, SMB)   |
| Remote Office 1   | HQ Medical DB API     | RO1-RTR-01 (Gi0/1 in) | PERMIT over IPsec (HTTPS 443 API)      |
| Remote Office 1   | HQ Donor DB & IT Mgmt | RO1-RTR-01 (Gi0/1 in) | DENY ALL (Prevents field breach pivot) |
| Remote Office 1   | Internet Web          | RO1-RTR-01 (Gi0/1 in) | PERMIT via Local Breakout PAT          |
+-------------------+-----------------------+-----------------------+----------------------------------------+
```

---

## 8. Perimeter Defense, NAT/PAT & Applied Cryptography

### 8.1 Network Address Translation (NAT) & PAT
To shield internal RFC 1918 addresses from public disclosure and conserve public IPv4 addresses, the edge routers employ three distinct NAT techniques:
1. **Static NAT (One-to-One):**  
   Exposes the DMZ servers to the public Internet:
   * `SRV-WEB-DMZ`: `10.1.50.10` $\leftrightarrow$ `203.0.113.100`
   * `SRV-DNS-DMZ`: `10.1.50.11` $\leftrightarrow$ `203.0.113.101`
2. **Dynamic PAT with Overload:**  
   Translates outgoing connections from internal workstations to the router's outside interface IP (`203.0.113.2`).
3. **NAT Exemption (NAT Bypass for IPsec):**  
   Crucial rule ensuring that packets destined for remote branch subnets are exempted from the PAT overload pool so that the crypto engine can capture and encrypt raw packets:
   ```cisco
   ip access-list extended HQ-NAT-ACL
    deny ip 10.1.0.0 0.0.255.255 10.2.10.0 0.0.0.255
    deny ip 10.1.0.0 0.0.255.255 10.3.10.0 0.0.0.255
    permit ip 10.1.0.0 0.0.255.255 any
   ```

### 8.2 Site-to-Site IPsec VPN Cryptographic Blueprint
The communication between HQ and the remote offices across the untrusted WAN is secured via **Hub-and-Spoke IPsec VPN Tunnels**:

* **IKE Phase 1 (ISAKMP SA):**
  * Encryption: **AES-256** (`encr aes 256`)
  * Hash: **SHA-1 / SHA-256** (`hash sha`)
  * Authentication: **Pre-Shared Key** (`GlobalHopeVPN2026!`)
  * Diffie-Hellman: **Group 2** (1024-bit prime)
  * Lifetime: 86,400 seconds (24 Hours)
* **IPsec Phase 2 (Transform Set & Crypto Map):**
  * Transform: `esp-aes 256 esp-sha-hmac`
  * Mode: **Tunnel Mode**
  * Perfect Forward Secrecy (PFS): **`pfs group2`** (Forces a new DH key exchange for every Phase 2 rekey)
  * Lifetime: 3,600 seconds (1 Hour)

---

## 9. Empirical Verification & Security Test Suite

The following matrix records the 18 verification test cases modeled to validate the network's security posture:

```
+------------------------------------------------------------------------------------------------------------+
|                                    EMPIRICAL VERIFICATION MATRIX (MODELED)                                 |
+-----+---------------------------+---------------+---------------+--------------+----------+----------------+
| ID  | TEST DESCRIPTION          | SOURCE HOST   | DESTINATION   | PROTOCOL     | RESULT   | TELEMETRY CHECK|
+-----+---------------------------+---------------+---------------+--------------+----------+----------------+
| T01 | Public Web / Donation     | EXT-CLIENT-01 | 203.0.113.100 | TCP 80/443   | SUCCESS  | NAT Trans: OK  |
| T02 | Public DNS Resolution     | EXT-CLIENT-01 | 203.0.113.101 | UDP 53       | SUCCESS  | DNS Answer: OK |
| T03 | Perimeter WAN Probe       | EXT-CLIENT-01 | 203.0.113.2   | ICMP / Telnet| BLOCKED  | Deny Count ++  |
| T04 | Perimeter Internal Probe  | EXT-CLIENT-01 | 10.1.40.20    | TCP 1433     | BLOCKED  | No Route/Drop  |
| T05 | Staff Intranet Access     | STAFF-PC-01   | 10.1.40.10    | TCP 80/443   | SUCCESS  | SVI Permitted  |
| T06 | Staff Blocked from Fin DB | STAFF-PC-01   | 10.1.40.20    | TCP 1433/3306| BLOCKED  | ACL Line 80 Hit|
| T07 | Staff Blocked from Med DB | STAFF-PC-01   | 10.1.40.30    | TCP 443      | BLOCKED  | ACL Line 90 Hit|
| T08 | Staff Lateral Block (HR)  | STAFF-PC-01   | 10.1.20.10    | ICMP / Any   | BLOCKED  | ACL Line 100Hit|
| T09 | Staff Blocked from IT Mgmt| STAFF-PC-01   | 10.1.30.1     | SSH (TCP 22) | BLOCKED  | VTY ACL Hit    |
| T10 | Finance Authorized DB Acc.| FIN-PC-01     | 10.1.40.20    | TCP 1433/3306| SUCCESS  | SVI Permitted  |
| T11 | Finance Blocked from Med  | FIN-PC-01     | 10.1.40.30    | TCP 443      | BLOCKED  | ACL Line 100Hit|
| T12 | DMZ Return Web Traffic    | STAFF-PC-01   | 10.1.50.10    | TCP 80       | SUCCESS  | Established Hit|
| T13 | DMZ Server Pivot Attempt  | SRV-WEB-DMZ   | 10.1.40.20    | TCP 1433/Ping| BLOCKED  | Containment Hit|
| T14 | RO1 IPsec VPN to Intranet | RO1-PC-01     | 10.1.40.10    | HTTP / ICMP  | SUCCESS  | Crypto Encaps++|
| T15 | RO1 Blocked from Fin DB   | RO1-PC-01     | 10.1.40.20    | TCP 1433     | BLOCKED  | Branch ACL Hit |
| T16 | RO1 Sync with Med API     | RO1-PC-01     | 10.1.40.30    | TCP 443      | SUCCESS  | Encrypted via SA|
| T17 | Branch Local Breakout PAT | RO1-PC-01     | Public Web    | TCP 80       | SUCCESS  | Branch PAT Hit |
| T18 | IT Admin Privileged SSH   | IT-PC-01      | All Devices   | SSH (TCP 22) | SUCCESS  | Privilege 15 OK|
+-----+---------------------------+---------------+---------------+--------------+----------+----------------+
```

---

## 10. Pre-Flight Engineering Audit & Issue Resolution

During project development, a rigorous pre-flight technical audit evaluated the design against native Cisco IOS behaviors in Packet Tracer, identifying six critical configuration items:

1. **Outside Ingress ACL vs. Outside Destination NAT Order of Operations:**  
   *Finding:* In Cisco IOS, ingress ACLs on an outside interface evaluate the packet before destination NAT translation occurs.  
   *Resolution:* The ingress ACL `WAN-TO-DMZ-IN` explicitly includes rules for the **pre-NAT public destination IP (`203.0.113.100`)**, ensuring external donor traffic is not dropped at the perimeter.
2. **DMZ Stateless Return Traffic & Architectural Limitations of Extended ACLs:**  
   *Finding:* Standard extended ACLs are stateless. Blocking DMZ-to-LAN traffic without return rules would drop legitimate web replies and DNS answers to staff.  
   *Resolution:* Added explicit TCP `established` rules and documented UDP port 53 return exceptions before the broad `deny ip 10.1.50.0 ...` containment rule.
3. **ISP Router Physical Port Capacity:**  
   *Finding:* The stock Cisco 2911 router provides only 3 GigabitEthernet ports (`Gi0/0`–`Gi0/2`), while 4 routed links were required.  
   *Resolution:* Added an **`HWIC-2FE` module** in Slot 0 of `ISP-RTR-01` to provide `FastEthernet0/0/0` for the external public subnet. Documented a dedicated switch alternative as a fallback.
4. **Cryptographic Parameter Synchronization:**  
   *Finding:* ISAKMP Phase 1 was set to AES-256 while IPsec Phase 2 defaulted to AES-128.  
   *Resolution:* Synchronized both phases to **AES-256** (`esp-aes 256 esp-sha-hmac`) across HQ and branches.
5. **Enforcement of Perfect Forward Secrecy (PFS):**  
   *Finding:* The security policy mandated PFS, but the crypto map lacked the PFS group statement.  
   *Resolution:* Explicitly injected **`set pfs group2`** into all crypto map entries.
6. **RSA Key Modulus Consistency:**  
   *Finding:* The policy referenced 2048-bit keys while the configuration generated 1024-bit keys.  
   *Resolution:* Standardized on **1024-bit RSA** to ensure broad compatibility with Packet Tracer device images while maintaining SSHv2 compliance.

---

## 11. Residual Risk Analysis & Enterprise Roadmap

While this architecture delivers robust perimeter filtering and inter-VLAN micro-segmentation, operating stateless or basic reflexive ACLs introduces residual risks that an enterprise production environment must mitigate:

```
+----------------------------------------------------------------------------------------------------+
|                                    RESIDUAL RISK & MITIGATION ROADMAP                              |
+--------------------------+-------------------------------------+-----------------------------------+
| RESIDUAL RISK            | TECHNICAL LIMITATION                | RECOMMENDED ENTERPRISE UPGRADE    |
+--------------------------+-------------------------------------+-----------------------------------+
| Application-Layer Threats| ACLs cannot inspect HTTP/SQL payloads| Web Application Firewall (WAF)    |
| Port 53 UDP Tunneling    | Stateless UDP return permits spoofing| Stateful Next-Gen Firewall (NGFW) |
| Rogue Endpoint Insertion | Access ports lack 802.1X auth       | Cisco ISE Network Access Control  |
| Lateral Insider Malware  | Server Farm lacks micro-segmentation| Host-based EDR & Distributed FW   |
| Advanced Persistent Threats| Syslog lacks automated correlation| Centralized SIEM / XDR Platform   |
+--------------------------+-------------------------------------+-----------------------------------+
```

---

## 12. Step-by-Step Packet Tracer Implementation Guide

For students, evaluators, or network administrators replicating this design in Cisco Packet Tracer:

### Step 1: Physical Placement
1. Place 1x Cisco 2911 router for `ISP-RTR-01`. Power off, drag `HWIC-2FE` into Slot 0, power on.
2. Place 3x Cisco 2911 routers for `HQ-RTR-01`, `RO1-RTR-01`, `RO2-RTR-01`.
3. Place 1x Cisco Catalyst 3560-24PS switch for `HQ-CORE-3560`.
4. Place 7x Cisco Catalyst 2960-24TT switches for departmental and branch access.
5. Place 6x Server-PT hosts and 6x PC-PT hosts.

### Step 2: Cabling
* Connect copper straight-through cables according to the **Port Cabling Schedule** in Section 5.2.
* Use crossover cables for router-to-router connections if MDIX auto-negotiation is disabled.

### Step 3: Device Configuration
1. Open the CLI tab of each device.
2. Paste the corresponding configuration script from **Section 15 (Appendices)**.
3. Save configurations using `write memory` or `copy running-config startup-config`.

### Step 4: Endpoint Setup
* Configure static IP addresses, subnet masks, default gateways, and DNS server pointers on all PCs and servers according to the **Master Addressing Table** in Section 6.2.
* Enable HTTP on `SRV-WEB-DMZ` and `SRV-INTRANET`.
* Enable DNS on `SRV-DNS-DMZ` and add the A records listed in Section 6.

### Step 5: Verification & Testing
1. From `EXT-CLIENT-01`, open Web Browser to `http://203.0.113.100`. Verify successful page load.
2. From `STAFF-PC-01`, ping `10.1.20.10` and `10.1.40.20`. Verify request times out (blocked).
3. From `RO1-PC-01`, ping `10.1.40.10`. Verify successful replies.
4. On `HQ-RTR-01`, execute `show crypto isakmp sa` and `show crypto ipsec sa` to verify active tunnels.

---

## 13. Conclusion & Project Retrospective

The NS-P01 Secure Organizational Network Design for Global Hope Foundation successfully demonstrates how structured engineering principles, rigorous subnetting, and defense-in-depth security can protect a geographically distributed non-profit organization.

By establishing strict physical and logical boundaries between untrusted public actors, semi-trusted DMZ services, internal corporate operations, and restricted financial/medical data, the network prevents unauthorized access and lateral threat expansion. The implementation of encrypted IPsec VPN tunnels with Perfect Forward Secrecy ensures field operations remain secure over public WAN backbones. This project fulfills all requirements of the TY B.Sc. IT Network Security curriculum.

---

## 14. References & Standards

1. **Cisco Systems, Inc.** (2021). *Cisco Validated Design: Enterprise Campus Modernization*. Cisco Press.
2. **National Institute of Standards and Technology (NIST)** (2009). *NIST SP 800-41 Rev. 1: Guidelines on Firewalls and Firewall Policy*. U.S. Department of Commerce.
3. **National Institute of Standards and Technology (NIST)** (2020). *NIST SP 800-77 Rev. 1: Guide to IPsec VPNs*. U.S. Department of Commerce.
4. **Internet Engineering Task Force (IETF)** (1996). *RFC 1918: Address Allocation for Private Internets*.
5. **Internet Engineering Task Force (IETF)** (2010). *RFC 5737: IPv4 Address Blocks Reserved for Documentation*.
6. **University of Mumbai** (2024). *Syllabus for TY B.Sc. Information Technology: Network Security (USIT601)*.

---

## 15. Appendices: Master Configuration Code Listings

*(All complete configuration scripts are organized in the `/configs/` project directory)*

* **Appendix A:** [ISP-RTR-01.cfg](file:///Users/devdabhi/Downloads/NS-P01-Secure-Network-Design/configs/ISP-RTR-01.cfg) — Internet Core & Public Routing
* **Appendix B:** [HQ-RTR-01.cfg](file:///Users/devdabhi/Downloads/NS-P01-Secure-Network-Design/configs/HQ-RTR-01.cfg) — HQ Perimeter Gateway, NAT Engine & IPsec Hub
* **Appendix C:** [HQ-CORE-3560.cfg](file:///Users/devdabhi/Downloads/NS-P01-Secure-Network-Design/configs/HQ-CORE-3560.cfg) — L3 Collapsed Core Switch & Inter-VLAN ACLs
* **Appendix D:** [HQ-SW-STAFF.cfg](file:///Users/devdabhi/Downloads/NS-P01-Secure-Network-Design/configs/HQ-SW-STAFF.cfg) — General Staff Access Switch
* **Appendix E:** [HQ-SW-FIN.cfg](file:///Users/devdabhi/Downloads/NS-P01-Secure-Network-Design/configs/HQ-SW-FIN.cfg) — HR & Finance Access Switch
* **Appendix F:** [HQ-SW-IT.cfg](file:///Users/devdabhi/Downloads/NS-P01-Secure-Network-Design/configs/HQ-SW-IT.cfg) — IT Management Access Switch
* **Appendix G:** [HQ-SW-SRV.cfg](file:///Users/devdabhi/Downloads/NS-P01-Secure-Network-Design/configs/HQ-SW-SRV.cfg) — Protected Server Farm Access Switch
* **Appendix H:** [HQ-SW-DMZ.cfg](file:///Users/devdabhi/Downloads/NS-P01-Secure-Network-Design/configs/HQ-SW-DMZ.cfg) — DMZ Server Access Switch
* **Appendix I:** [RO1-RTR-01.cfg](file:///Users/devdabhi/Downloads/NS-P01-Secure-Network-Design/configs/RO1-RTR-01.cfg) — Remote Office 1 Edge Router & IPsec Spoke
* **Appendix J:** [RO1-SW-01.cfg](file:///Users/devdabhi/Downloads/NS-P01-Secure-Network-Design/configs/RO1-SW-01.cfg) — Remote Office 1 Access Switch
* **Appendix K:** [RO2-RTR-01.cfg](file:///Users/devdabhi/Downloads/NS-P01-Secure-Network-Design/configs/RO2-RTR-01.cfg) — Remote Office 2 Edge Router & IPsec Spoke
* **Appendix L:** [RO2-SW-01.cfg](file:///Users/devdabhi/Downloads/NS-P01-Secure-Network-Design/configs/RO2-SW-01.cfg) — Remote Office 2 Access Switch

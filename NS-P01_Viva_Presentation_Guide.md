# TY B.Sc. IT Network Security (NS-P01)
# Viva Voce Examination & Project Presentation Guide

**Project:** Secure Organizational Network Design for Global Hope Foundation  
**Target Audience:** Academic Examiners, Project Evaluators, Network Defense Reviewers  

---

## 1. Project Presentation Slide Outline (12-Slide Master Structure)

* **Slide 1: Title & Academic Metadata**
  * Project: Secure Organizational Network Design (NS-P01)
  * Organization: Global Hope Foundation (Humanitarian Relief NGO)
  * Candidate Details: TY B.Sc. Information Technology
* **Slide 2: Problem Statement & Organizational Profile**
  * Challenges faced by international NGOs: Dispersed field offices, public donation intake, confidential donor banking details, sensitive refugee medical records.
  * Operational scope: Central Head Office + 2 Remote Field Centers.
* **Slide 3: Threat Modeling & STRIDE Analysis**
  * Key threat vectors: External perimeter port scanning, DMZ breach and internal lateral movement, untrusted field-device compromise, WAN eavesdropping, and administrative credential takeover.
* **Slide 4: High-Level Architecture & Defense-in-Depth**
  * Collapsed Core / Distribution design at HQ (Cisco 3560).
  * Three-legged perimeter router (Outside, DMZ, Inside).
  * 8 distinct trust zones.
* **Slide 5: Hierarchical IP Addressing & VLSM Subnetting**
  * Hierarchical allocation: HQ `10.1.0.0/16`, RO1 `10.2.0.0/16`, RO2 `10.3.0.0/16`, WAN `203.0.113.0/24`.
  * Zero-overlap mathematical proof; summary route optimization.
* **Slide 6: DMZ Isolation & Boundary Filtering**
  * Physical separation via `HQ-RTR-01` `Gi0/2`.
  * Inbound static NAT for Web (`.100`) and DNS (`.101`).
  * Strict containment: DMZ servers can never initiate connections to internal LANs.
* **Slide 7: Inter-VLAN Micro-Segmentation & Database Protection**
  * SVI ACLs on Core Switch.
  * General Staff blocked from HR, IT Management, and raw database ports (TCP 1433/3306).
  * Separation of duties: Finance has SQL access to Donor DB, but blocked from Medical DB.
* **Slide 8: Applied Cryptography: Site-to-Site IPsec VPN**
  * Hub-and-Spoke topology across public WAN.
  * IKE Phase 1: AES-256, SHA, Group 2, PSK.
  * IPsec Phase 2: ESP-AES-256, SHA-HMAC, Perfect Forward Secrecy (**PFS Group 2**).
* **Slide 9: Network Address Translation & NAT Bypass**
  * Static NAT for public DMZ services.
  * Dynamic PAT (Overload) for staff outbound Internet web browsing.
  * NAT Exemption ACL ensuring VPN traffic is not translated by the PAT pool.
* **Slide 10: Empirical Verification & Test Results**
  * 18-point security testing suite.
  * Verified: Pre-NAT ACL evaluation, DMZ return traffic exceptions, encrypted packet counters.
* **Slide 11: Pre-Flight Audit & Engineering Adjustments**
  * Resolution of 6 critical technical items: Outside ACL NAT order, stateless ACL return traffic, 2911 interface slotting (`HWIC-2FE`), AES-256 cipher sync, PFS Group 2 injection, and 1024-bit RSA.
* **Slide 12: Residual Risks & Future Roadmap**
  * Evolution to Next-Gen Firewalls (NGFW), Web Application Firewalls (WAF), 802.1X Network Access Control (NAC), and centralized SIEM logging.

---

## 2. Anticipated Viva Voce Questions & Model Answers

### Q1: Why did you choose a Collapsed Core architecture instead of a traditional Three-Tier (Core-Distribution-Access) architecture?
> **Model Answer:**  
> "For an organization of Global Hope Foundation's scale (a centralized headquarters with 50 to 200 users and moderate traffic density), a traditional three-tier model introduces unnecessary hardware costs, latency, and management complexity. A collapsed core/distribution architecture using a Layer 3 switch (Cisco Catalyst 3560) consolidates wire-speed inter-VLAN routing and boundary ACL enforcement into a single resilient tier, while still maintaining modularity and physical separation from the perimeter edge router."

---

### Q2: Why is the DMZ connected to a dedicated physical router port rather than being implemented as another VLAN on the internal core switch?
> **Model Answer:**  
> "Connecting the DMZ to a dedicated physical interface (`Gi0/2`) on the perimeter router (`HQ-RTR-01`) creates a true **Three-Legged Firewall/Router Topology**. If the DMZ were merely a VLAN on the internal core switch, any Layer 2 trunking flaw, VLAN hopping attack, or core switch misconfiguration could allow an attacker who compromises the public web server to directly compromise internal switching fabric. Physical port isolation ensures that traffic between the DMZ and internal networks must cross the router's Layer 3 security boundary where strict containment ACLs are enforced."

---

### Q3: Explain the order of operations between Inbound Access Lists and Destination NAT (Outside-to-Inside) on a Cisco IOS router.
> **Model Answer:**  
> "In Cisco IOS on an outside interface, when a packet arrives from the Internet, the router processes the **inbound access list (ACL) BEFORE applying destination NAT (un-NAT)**. Therefore, the ingress ACL evaluates the packet based on its untranslated, public destination IP address (e.g., `203.0.113.100`), rather than its internal private IP address (`10.1.50.10`). In our Phase 4 audit, we verified this behavior and ensured `WAN-TO-DMZ-IN` explicitly permits the public IP to prevent legitimate web traffic from being dropped."

---

### Q4: Why is TCP `established` insufficient for handling return traffic from DMZ servers, and how did you address it?
> **Model Answer:**  
> "The TCP `established` keyword inspects the TCP header flags (ACK or RST). While this effectively permits return packets for TCP-based sessions (such as HTTP or HTTPS) initiated by internal users, it does not work for connectionless protocols like UDP. Because DNS operates primarily over UDP port 53, DNS responses from the DMZ DNS server do not contain an established flag and would be blocked by a default-deny rule. We addressed this by implementing a documented architectural return exception permitting UDP packets sourced from port 53 to internal user subnets, while noting the need for stateful inspection in enterprise production."

---

### Q5: What is the purpose of NAT Bypass (NAT Exemption) in our Site-to-Site IPsec VPN configuration?
> **Model Answer:**  
> "On Cisco IOS edge routers, the NAT process evaluates outbound packets before the crypto engine. If an internal packet destined for a remote branch hits the router, the dynamic PAT rule would normally translate the private source IP (`10.1.x.x`) to the public interface IP (`203.0.113.2`). Consequently, the packet would no longer match the crypto access list (which expects private source IPs), causing the VPN tunnel to fail. The NAT bypass ACL explicitly uses `deny` statements for inter-site traffic before the `permit any` overload statement, exempting VPN traffic from address translation."

---

### Q6: What is Perfect Forward Secrecy (PFS), and why was `set pfs group2` added to the crypto maps?
> **Model Answer:**  
> "In standard IPsec, Phase 2 keys can be derived from the initial Diffie-Hellman key exchange performed in Phase 1. If an attacker were to compromise the Phase 1 master key, all past and future Phase 2 sessions could potentially be decrypted. **Perfect Forward Secrecy (PFS)** forces the peers to execute an independent, brand-new Diffie-Hellman exchange every time the Phase 2 IPsec SA rekeys (every 3,600 seconds). By adding `set pfs group2`, we ensure that the compromise of one session key cannot compromise any past or future data transmissions."

---

### Q7: If a workstation in the General Staff VLAN is infected with ransomware, how does your architecture prevent it from spreading to HR records or Donor databases?
> **Model Answer:**  
> "The architecture enforces micro-segmentation at the default gateway (SVI Vlan10 on `HQ-CORE-3560`). Inbound ACL `STAFF-FILTER-IN` explicitly blocks traffic destined for the HR/Finance subnet (`10.1.20.0/24`), the IT Management subnet (`10.1.30.0/24`), the Donor Database (`10.1.40.20`), and the Medical Database (`10.1.40.30`). The infected workstation cannot send packets across these boundaries, effectively quarantining the outbreak within VLAN 10 and protecting mission-critical databases."

---

### Q8: Why did you use Static Routing rather than dynamic routing protocols like OSPF or EIGRP?
> **Model Answer:**  
> "In a secure perimeter and branch architecture, static routing eliminates dynamic routing protocol overhead, prevents routing advertisement spoofing, and guarantees deterministic path selection. Furthermore, public ISP routers should never participate in private internal IGP routing domains. By employing a default static route pointing outward and summary static routes pointing inward, we maintain clean security boundaries and eliminate accidental route leaks."

---

### Q9: How is management plane access to routers and switches secured against unauthorized employees?
> **Model Answer:**  
> "Management access is hardened using a multi-tiered approach:
> 1. Insecure Telnet and HTTP are disabled globally.
> 2. Secure Shell version 2 (SSHv2) is enforced using 1024-bit RSA keys.
> 3. Switch management SVIs are placed exclusively within VLAN 30 (IT Management).
> 4. All router and switch VTY lines enforce `access-class MGMT-VTY-ACL in`, permitting connections solely from the IT Management subnet (`10.1.30.0/24`). Any connection attempt from staff or branch PCs is immediately rejected at the transport layer."

---

### Q10: What are the primary limitations of using router extended ACLs instead of an enterprise Next-Generation Firewall (NGFW)?
> **Model Answer:**  
> "Router extended ACLs operate primarily at Layers 3 and 4 (IP addresses, protocols, TCP/UDP ports). Their limitations include:
> * Inability to inspect application-layer payloads (e.g., unable to detect SQL injection or cross-site scripting inside an allowed port 80/443 stream).
> * Lack of true bi-directional stateful session tracking for UDP and ICMP.
> * Absence of deep packet inspection, TLS/SSL decryption, intrusion prevention (IPS), and URL/content filtering.
> In our report's enterprise roadmap, we recommend deploying a dedicated Next-Generation Firewall and Web Application Firewall (WAF) to complement router perimeter controls."

---

## 3. Demonstration & CLI Command Cheat-Sheet for Live Viva

When presenting the project in Cisco Packet Tracer or CLI terminal, use these specific commands to demonstrate operational security:

```text
! 1. Demonstrate Active NAT Translations
HQ-RTR-01# show ip nat translations

! 2. Demonstrate ACL Hit Counters and Rule Enforcement
HQ-RTR-01# show access-lists WAN-TO-DMZ-IN
HQ-RTR-01# show access-lists DMZ-CONTAINMENT-IN
HQ-CORE-3560# show access-lists STAFF-FILTER-IN
HQ-CORE-3560# show access-lists FINANCE-FILTER-IN

! 3. Demonstrate IPsec Phase 1 ISAKMP SA (Look for QM_IDLE)
HQ-RTR-01# show crypto isakmp sa

! 4. Demonstrate IPsec Phase 2 IPsec SA & Encrypted Packet Counters
HQ-RTR-01# show crypto ipsec sa

! 5. Demonstrate Routing Table & Route Summarization
HQ-RTR-01# show ip route
HQ-CORE-3560# show ip route

! 6. Demonstrate VLAN Database & SVI Operational Status
HQ-CORE-3560# show vlan brief
HQ-CORE-3560# show ip interface brief
```

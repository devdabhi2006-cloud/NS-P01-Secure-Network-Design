# Phase 5: Verification & Security Test Plan

This document details the exact testing procedures, expected results, and verification CLI commands to validate the Global Hope Foundation Secure Organizational Network.

---

## 1. Test Matrix Overview

| Test ID | Test Category | Source Device | Destination / Target | Protocol / Port | Expected Result | Security Objective Verified |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **TC-01** | Public Web Access | `EXT-CLIENT-01` | `203.0.113.100` | HTTP (TCP 80) | **SUCCESS** (200 OK) | Inbound Static NAT & DMZ Web access |
| **TC-02** | Public DNS Query | `EXT-CLIENT-01` | `203.0.113.101` | DNS (UDP 53) | **SUCCESS** (Resolves) | Inbound Static NAT & DMZ DNS access |
| **TC-03** | Perimeter Inbound Drop | `EXT-CLIENT-01` | `203.0.113.2` | ICMP / Telnet | **BLOCKED / DROPPED** | Default Deny on untrusted WAN |
| **TC-04** | Perimeter Internal Probe| `EXT-CLIENT-01` | `10.1.40.20` | SQL / Any | **BLOCKED / DROPPED** | No routing/access to internal RFC 1918 |
| **TC-05** | Staff Intranet Access | `STAFF-PC-01` | `10.1.40.10` | HTTP (TCP 80) | **SUCCESS** | Internal authorized portal access |
| **TC-06** | Staff DB Block (Donor)| `STAFF-PC-01` | `10.1.40.20` | TCP 1433 / ICMP | **BLOCKED / DROPPED** | Least privilege; DB protected from Staff |
| **TC-07** | Staff DB Block (Med) | `STAFF-PC-01` | `10.1.40.30` | TCP 443 / ICMP | **BLOCKED / DROPPED** | Beneficiary medical data protected |
| **TC-08** | Lateral Isolation | `STAFF-PC-01` | `10.1.20.10` (FIN) | ICMP / Any | **BLOCKED / DROPPED** | Inter-VLAN departmental isolation |
| **TC-09** | Mgmt Plane Isolation | `STAFF-PC-01` | `10.1.30.1` (SVI) | SSH / Telnet | **BLOCKED / DROPPED** | Network consoles restricted from staff |
| **TC-10** | Finance DB Access | `FIN-PC-01` | `10.1.40.20` | TCP 1433 / 3306 | **SUCCESS** | Authorized financial data processing |
| **TC-11** | Finance Med DB Block | `FIN-PC-01` | `10.1.40.30` | TCP 443 / ICMP | **BLOCKED / DROPPED** | Separation of duties (Finance != Medical) |
| **TC-12** | DMZ Return Traffic | `STAFF-PC-01` | `10.1.50.10` (Web) | HTTP (TCP 80) | **SUCCESS** | Return traffic allowed via `established` |
| **TC-13** | DMZ Inward Breach Block| `SRV-WEB-DMZ` | `10.1.40.20` | Any | **BLOCKED / DROPPED** | DMZ containment (no pivoting to LAN) |
| **TC-14** | IPsec VPN RO1 to HQ | `RO1-PC-01` | `10.1.40.10` | HTTP / ICMP | **SUCCESS (ENCRYPTED)**| Site-to-Site VPN tunnel active |
| **TC-15** | RO1 Donor DB Block | `RO1-PC-01` | `10.1.40.20` | TCP 1433 / ICMP | **BLOCKED / DROPPED** | Branch restricted from HQ finances |
| **TC-16** | RO1 Medical API Access | `RO1-PC-01` | `10.1.40.30` | TCP 443 (API) | **SUCCESS (ENCRYPTED)**| Field clinic synchronized over VPN |
| **TC-17** | Branch Local Breakout | `RO1-PC-01` | Internet Web | HTTP (TCP 80) | **SUCCESS (NAT)** | Split-tunnel local Internet access |
| **TC-18** | IT Full Mgmt Authority | `IT-PC-01` | All Routers/SW | SSH (TCP 22) | **SUCCESS** | Authorized administrative control |

---

## 2. Step-by-Step Test Execution & CLI Commands

### 2.1 Test 1: Inbound NAT & Ingress Perimeter Filtering
* **Action:** From `EXT-CLIENT-01` Web Browser, navigate to `http://203.0.113.100`.
* **Expected Result:** Web page "Global Hope Foundation (GHF) - Welcome to our Official Humanitarian Aid and Donation Portal" loads successfully.
* **Verification on `HQ-RTR-01`:**
  ```text
  HQ-RTR-01# show ip nat translations
  Pro Inside global      Inside local       Outside local      Outside global
  tcp 203.0.113.100:80   10.1.50.10:80      203.0.113.20:1025  203.0.113.20:1025
  --- 203.0.113.100      10.1.50.10         ---                ---
  --- 203.0.113.101      10.1.50.11         ---                ---

  HQ-RTR-01# show access-lists WAN-TO-DMZ-IN
  Extended IP access list WAN-TO-DMZ-IN
      10 permit udp host 203.0.113.6 host 203.0.113.2 eq isakmp
      20 permit udp host 203.0.113.6 host 203.0.113.2 eq non500-isakmp
      30 permit esp host 203.0.113.6 host 203.0.113.2
      ...
      70 permit tcp any host 203.0.113.100 eq www (MATCH COUNTER > 0)
      ...
      140 deny ip any any (MATCH COUNTER INCREMENTS ON UNWANTED PROBES)
  ```

---

### 2.2 Test 2: DMZ Containment (Breach Isolation)
* **Action:** From `SRV-WEB-DMZ` Command Prompt, attempt to ping or connect to internal hosts:
  ```text
  ping 10.1.10.10
  ping 10.1.40.20
  ```
* **Expected Result:** `Request timed out` (Packets blocked at `HQ-RTR-01` Gi0/2).
* **Verification on `HQ-RTR-01`:**
  ```text
  HQ-RTR-01# show access-lists DMZ-CONTAINMENT-IN
  Extended IP access list DMZ-CONTAINMENT-IN
      10 permit tcp host 10.1.50.10 eq www 10.1.0.0 0.0.255.255 established
      20 permit tcp host 10.1.50.10 eq 443 10.1.0.0 0.0.255.255 established
      30 permit udp host 10.1.50.11 eq domain 10.1.0.0 0.0.255.255
      40 deny ip 10.1.50.0 0.0.0.15 10.1.0.0 0.0.255.255 (MATCH COUNTER > 0)
  ```

---

### 2.3 Test 3: Inter-VLAN Departmental Isolation & Least Privilege
* **Action 3a:** From `STAFF-PC-01`, ping `FIN-PC-01` (`10.1.20.10`) and `IT-PC-01` (`10.1.30.10`).
  * **Result:** `Destination host unreachable` / `Request timed out`.
* **Action 3b:** From `STAFF-PC-01`, attempt to access `SRV-DONOR-DB` (`10.1.40.20`) and `SRV-MED-DB` (`10.1.40.30`).
  * **Result:** `Connection refused` / `Request timed out`.
* **Action 3c:** From `STAFF-PC-01`, open browser to `http://10.1.40.10` (Intranet).
  * **Result:** Page loads successfully.
* **Verification on `HQ-CORE-3560`:**
  ```text
  HQ-CORE-3560# show access-lists STAFF-FILTER-IN
  Extended IP access list STAFF-FILTER-IN
      ...
      40 permit tcp 10.1.10.0 0.0.0.255 host 10.1.40.10 eq www (MATCHES)
      ...
      80 deny ip 10.1.10.0 0.0.0.255 host 10.1.40.20 (MATCHES: BLOCKED DB ACCESS)
      90 deny ip 10.1.10.0 0.0.0.255 host 10.1.40.30 (MATCHES: BLOCKED MED ACCESS)
      100 deny ip 10.1.10.0 0.0.0.255 10.1.20.0 0.0.0.255 (MATCHES: BLOCKED HR/FIN ACCESS)
      110 deny ip 10.1.10.0 0.0.0.255 10.1.30.0 0.0.0.255 (MATCHES: BLOCKED IT MGMT ACCESS)
  ```

---

### 2.4 Test 4: Site-to-Site IPsec VPN & PFS Verification
* **Action:** From `RO1-PC-01` (`10.2.10.10`), ping `SRV-INTRANET` (`10.1.40.10`).
* **Expected Result:** First 1-2 pings may drop during IKE negotiation, followed by continuous successful replies (`Reply from 10.1.40.10`).
* **Verification on `HQ-RTR-01` & `RO1-RTR-01`:**
  ```text
  HQ-RTR-01# show crypto isakmp sa
  IPv4 Crypto ISAKMP SA
  dst             src             state          conn-id slot status
  203.0.113.2     203.0.113.6     QM_IDLE              1    0 ACTIVE

  HQ-RTR-01# show crypto ipsec sa
  interface: GigabitEthernet0/0
      Crypto map tag: GHF-HQ-MAP, local addr 203.0.113.2
     protected vrf: (none)
     local  ident (addr/mask/prot/port): (10.1.0.0/255.255.0.0/0/0)
     remote ident (addr/mask/prot/port): (10.2.10.0/255.255.255.0/0/0)
     current_peer 203.0.113.6 port 500
       PERMIT, flags={origin_is_acl,}
      #pkts encaps: 4, #pkts encrypt: 4, #pkts digest: 4
      #pkts decaps: 4, #pkts decrypt: 4, #pkts verify: 4
      #send errors 0, #recv errors 0
      local crypto endpt.: 203.0.113.2, remote crypto endpt.: 203.0.113.6
      path mtu 1500, ipsec overhead 74, media mtu 1500
      PFS (Perfect Forward Secrecy): group2 active
  ```

---

### 2.5 Test 5: Management Plane Protection (VTY Restriction)
* **Action 5a:** From `STAFF-PC-01`, open terminal and attempt:
  ```text
  ssh -l admin_ghf 10.1.30.1
  ```
  * **Result:** `Connection refused by remote host` (Dropped by `MGMT-VTY-ACL` or SVI filter).
* **Action 5b:** From `IT-PC-01` (`10.1.30.10`), open terminal and attempt:
  ```text
  ssh -l admin_ghf 10.1.30.1
  ```
  * **Result:** Password prompt appears. Entering `AdminPass2026!` provides full privilege level 15 CLI access.

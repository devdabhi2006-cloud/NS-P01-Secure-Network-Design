# Endpoint & Server Configuration Reference

This document provides the exact static IP settings, default gateways, and application service parameters to configure on all end devices and servers in Cisco Packet Tracer.

---

## 1. DMZ Servers

### `SRV-WEB-DMZ` (Public Web & Donation Portal)
* **IP Configuration:**
  * IP Address: `10.1.50.10`
  * Subnet Mask: `255.255.255.240` (/28)
  * Default Gateway: `10.1.50.1`
  * DNS Server: `10.1.50.11`
* **Services Configuration (Services Tab in Packet Tracer):**
  * **HTTP / HTTPS:** ON
  * Edit `index.html`:
    ```html
    <html>
      <head><title>Global Hope Foundation</title></head>
      <body>
        <h1>Global Hope Foundation (GHF)</h1>
        <p>Welcome to our Official Humanitarian Aid and Donation Portal.</p>
        <p>Status: Secure DMZ Web Server Active</p>
      </body>
    </html>
    ```

### `SRV-DNS-DMZ` (Public Authoritative DNS Server)
* **IP Configuration:**
  * IP Address: `10.1.50.11`
  * Subnet Mask: `255.255.255.240` (/28)
  * Default Gateway: `10.1.50.1`
  * DNS Server: `10.1.50.11` (or `127.0.0.1`)
* **Services Configuration (Services Tab in Packet Tracer):**
  * **DNS Service:** ON
  * **A Records Added:**
    * `globalhope.org` -> `203.0.113.100` (Public IP)
    * `donation.globalhope.org` -> `203.0.113.100` (Public IP)
    * `ns1.globalhope.org` -> `203.0.113.101` (Public IP)
    * `intranet.globalhope.org` -> `10.1.40.10` (Internal IP)

---

## 2. Protected Internal Server Farm (VLAN 40)

### `SRV-INTRANET` (Internal Intranet & File Server)
* **IP Configuration:**
  * IP Address: `10.1.40.10`
  * Subnet Mask: `255.255.255.0` (/24)
  * Default Gateway: `10.1.40.1`
  * DNS Server: `10.1.50.11`
* **Services Configuration:**
  * **HTTP / HTTPS:** ON (Internal Intranet Portal)
  * **FTP:** ON (User `staff_user` / Pass `StaffPass123` with Read/Write permissions)

### `SRV-DONOR-DB` (Confidential Financial & Donor Database)
* **IP Configuration:**
  * IP Address: `10.1.40.20`
  * Subnet Mask: `255.255.255.0` (/24)
  * Default Gateway: `10.1.40.1`
  * DNS Server: `10.1.50.11`
* **Services Configuration:**
  * In Packet Tracer, database simulation is represented via HTTPS/custom HTTP or FTP to verify port filtering (ports 1433, 3306, 443).

### `SRV-MED-DB` (Beneficiary & Medical Database)
* **IP Configuration:**
  * IP Address: `10.1.40.30`
  * Subnet Mask: `255.255.255.0` (/24)
  * Default Gateway: `10.1.40.1`
  * DNS Server: `10.1.50.11`
* **Services Configuration:**
  * HTTP/HTTPS: ON (simulating health records API endpoint).

### `SRV-MGMT-LOG` (Syslog, NTP, AAA Server)
* **IP Configuration:**
  * IP Address: `10.1.40.50`
  * Subnet Mask: `255.255.255.0` (/24)
  * Default Gateway: `10.1.40.1`
  * DNS Server: `10.1.50.11`
* **Services Configuration:**
  * **SYSLOG:** ON (Logs router and switch telemetry).
  * **NTP:** ON (Synchronizes time across all network infrastructure).
  * **AAA:** ON (Optional centralized TACACS+/RADIUS).

---

## 3. Client Workstations & Testing Nodes

| Host Identifier | Subnet & VLAN | IP Address | Subnet Mask | Default Gateway | DNS Server |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **`STAFF-PC-01`** | HQ VLAN 10 | `10.1.10.10` | `255.255.255.0` | `10.1.10.1` | `10.1.50.11` |
| **`FIN-PC-01`** | HQ VLAN 20 | `10.1.20.10` | `255.255.255.0` | `10.1.20.1` | `10.1.50.11` |
| **`IT-PC-01`** | HQ VLAN 30 | `10.1.30.10` | `255.255.255.0` | `10.1.30.1` | `10.1.50.11` |
| **`RO1-PC-01`** | RO1 Branch LAN | `10.2.10.10` | `255.255.255.0` | `10.2.10.1` | `10.1.50.11` |
| **`RO2-PC-01`** | RO2 Branch LAN | `10.3.10.10` | `255.255.255.0` | `10.3.10.1` | `10.1.50.11` |
| **`EXT-CLIENT-01`**| Public WAN Net | `203.0.113.20` | `255.255.255.240`| `203.0.113.17` | `203.0.113.101` |

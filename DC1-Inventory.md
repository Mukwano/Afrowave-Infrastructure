DC1-Inventory
========================

# DC1 – Inventory and System State
Afrowave Domain Controller #1 (Primary)

This document describes the current configuration, network state, services, security policies, and operational details of DC1.  
It serves both as an inventory reference and a baseline for maintenance and future troubleshooting.

---

## 1. Server Identity
- **Hostname:** dc1.afrowave.ltd  
- **Role:** Primary Active Directory Domain Controller  
- **Location:** VPS (USA)  
- **Operating System:** Debian 13.1 “Trixie”  
- **Domain:** AFROWAVE.LTD  
- **Time zone:** (to be filled)  
- **Administrator accounts:** (to be filled)  
- **Last verified:** (date)

> All details below will be updated as we inspect the live server.

---

## 2. Network Configuration Overview

### 2.1 Interfaces
| Interface | Purpose | IP Address | Notes |
|----------|----------|------------|-------|
| `eth0` | Main WAN | (to be filled) | Public IP assigned by provider |
| `wg-link` | Server backbone | (to be filled) | Full trust network, all ports allowed |
| `wg-enroll` | (optional) Domain join VPN | (not configured) | Reserved for domain onboarding |
| `wg-internal` | (optional) Internal client VPN | (not configured) | Reserved |

### 2.2 DNS Configuration
- **DNS mode:** Samba Internal DNS  
- **Authoritative zones:**  
  - `afrowave.ltd`  
  - `_msdcs.afrowave.ltd`  
- **Forwarders:**  
  - (to be filled – expected 1–2 public resolvers)  
- **Required A Records for WG-LINK:**  
  - `dc1-link.afrowave.ltd → <wg-link-IP>`

### 2.3 Routing
- **Default route:** WAN  
- **Backup route (planned):** via WG-LINK (Diblík or DC2)  
- **NAT role:**  
  - Primary NAT exit for servers via WG-LINK

---

## 3. WireGuard Configuration

### 3.1 WG-LINK Interface
- **Purpose:** Trusted backbone for server-to-server communication  
- **Status:** Running  
- **Peer(s):**  
  - Diblík (home server)  
  - Edge server  
  - (future) DC2  
- **IP range:** (to be filled – example: 10.30.0.0/24)  
- **DC1 IP:** (to be filled)

### 3.2 WG-LINK Firewall Zone
- Zone name: `aw-link`  
- Settings:
  - `target=ACCEPT`  
  - No port restrictions  
  - No NAT  

> This zone is used exclusively for trusted server interconnects.

---

## 4. Active Directory Services

### 4.1 Samba AD DC
- **Service:** `samba-ad-dc`  
- **Status:** Active and running  
- **Version:** (to be filled)  
- **Database health:**  
  - `samba-tool dbcheck` → pending  
- **Replication:**  
  - Only DC1 present (no replication yet – DC2 planned)

### 4.2 Kerberos
- **Realm:** `AFROWAVE.LTD`  
- **Ports:** 88/tcp, 88/udp open  
- **KDC Status:** (to be filled)

### 4.3 NTP / Time Services
- **Time service:** `chrony` or `ntpsec` (to be filled)  
- **Domain-signed time:** expected via Samba `ntp_signd`  
- **Upstream NTP servers:** (to be filled)  
- **Current offset:** (to be filled from `timedatectl`)  

NTP correction and configuration will be completed and documented in the next steps.

---

## 5. Firewall Configuration

### 5.1 Zones
| Zone | Purpose | Restrictions |
|------|----------|--------------|
| `public` | WAN | strict |
| `aw-link` | WG-LINK | allow all |
| `internal` | (optional) | for local services |
| others… | (TBD) | |

### 5.2 Required Open Ports for AD
Category | Ports | Status
---------|-------|---------
DNS | 53/tcp, 53/udp | (to be verified)
Kerberos | 88/tcp, 88/udp | (to be verified)
LDAP | 389/tcp, 389/udp | (to be verified)
LDAPS | 636/tcp | (optional)
SMB / RPC | 135/tcp, 139/tcp, 445/tcp | (to be verified)
Kerberos password | 464/tcp, 464/udp | (to be verified)
Global Catalog | 3268/tcp, 3269/tcp | (to be verified)
NTP | 123/udp | (to be verified)

---

## 6. Installed Services
- Samba AD DC  
- Firewalld  
- Chrony or NTPsec (to be confirmed)  
- Cockpit (with valid certificate)  
- SSH server  
- (others to be added)

---

## 7. Logs and Monitoring

### Common log locations:
- `/var/log/samba/`  
- `journalctl -u samba-ad-dc`  
- `/var/log/chrony/` or `/var/log/ntpsec/`  
- `/var/log/firewalld`  
- `journalctl -xe`

---

## 8. Outstanding Tasks (to be completed)

- Verify and fix NTP configuration  
- Add WG-LINK A record in DNS  
- Confirm Kerberos functionality via WG-LINK  
- Audit firewalld zones  
- Add secondary exit routing logic  
- Prepare DC2 replication plan  
- Document all WireGuard peers and keys  

---

## 9. Notes
This document will be updated continuously as we inspect DC1 and complete each subsystem (DNS, NTP, WG-LINK, firewall, Samba, replication, routing).

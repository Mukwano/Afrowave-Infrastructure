# DC1-Inventory.md — Afrowave DC1 (Updated Documentation 2025)

**Server Name:** `afw-dc1`  
**Role:** Primary Samba AD Domain Controller, DNS, KDC, NTP, VPN Hub  
**Status:** Active (production-critical)

---

## **1. Hardware Overview**

| Parameter | Value |
|----------|--------|
| Platform | InterServer VPS |
| CPU | Virtual (shared cores) |
| RAM | 3 GB |
| Storage | SSD-backed virtual disk |
| NIC | `eth0` (public) |

---

## **2. Operating System**

- **OS:** Debian 13.1 ("Trixie")  
- **Kernel:** Debian stock kernel  
- **Init system:** systemd  
- **Firewall:** firewalld (nftables backend)

---

## **3. Network Interfaces**

### **3.1 Physical Interface**

#### `eth0` – WAN / Public Internet  
- IPv4: `69.169.97.200/25`  
- Gateway: `69.169.97.129`  
- IPv6: Global address via ISP  
- Zone: `public`

---

### **3.2 WireGuard Interfaces (Current + Planned)**

This is the **final WireGuard architecture officially adopted** for the Afrowave ecosystem.

---

### 🔵 **wg-link** – primary server-to-server backbone  
- IP: `10.30.0.1/24`  
- Port: `52030/udp`  
- Zone: `aw-link`  
- Purpose: DC1 ↔ EDGE ↔ WebServer / Trusted Mesh  
- NAT: No

---

### 🟦 **wg-link-443** – same as wg-link, but operating over port 443  
- IP: `10.31.0.1/24`  
- Port: `443/udp` *(stealth mode, proxy-friendly)*  
- Zone: `aw-link`  
- Purpose: Server mesh in restricted/corporate network environments  
- NAT: No

---

### 🟢 **wg-internal** – internal corporate VPN (no NAT)  
- IP: `10.20.0.1/16`  
- Zone: `aw-internal`  
- NAT: No  
- Purpose: AD, Kerberos, DNS, file services, management

---

### 🟩 **wg-inet** – internal VPN + Internet egress via Afrowave  
- IP: `10.22.0.1/16`  
- Zone: `aw-inet`  
- NAT: Yes (masquerade)  
- Purpose: For users behind censorship or restricted ISP environments

---

### 🟧 **wg-learn** *(planned)* – training and laboratory network  
- IP Range: `10.24.0.0/16`  
- Zone: `aw-learn`  
- Purpose: Labs, sandbox, education

---

### 🟨 **wg-enroll** *(planned)* – domain enrollment network  
- IP Range: `10.26.0.0/16`  
- Zone: `aw-enroll`  
- Purpose: Short‑term connectivity during device/domain onboarding

---

### 🟫 **wg-guest** *(planned)* – isolated internet‑only VPN  
- IP Range: TBA  
- Zone: `aw-guest`

### 🔴 **wg-rootoz** *(planned)* – high‑security admin network  
- IP Range: TBA  
- Zone: `aw-rootoz`

---

## **4. Firewall Zones (firewalld)**

### `public` (eth0)
- Services: `ssh`  
- Masquerade: yes  
- Purpose: WAN access + secure remote administration

---

### `aw-link` (wg-link + wg-link-443)
- Allowed: DNS, LDAP/LDAPS, Kerberos, Samba, NTP  
- Trusted mesh tunnels  
- NAT: No

---

### `aw-internal` (wg-internal)
- All internal AD services  
- NAT: No

---

### `aw-inet` (wg-inet)
- Internal domain services + Internet NAT  
- Masquerade: yes

---

### `aw-enroll`, `aw-learn`, `aw-guest`, `aw-rootoz`
- Skeleton zones – will be completed after IPAM deployment

---

## **5. Samba / Active Directory**

- **Domain:** `afrowave.ltd`  
- **Functional Level:** Samba AD DC (latest)  
- **Services Provided:** LDAP/LDAPS, Kerberos, DNS, NTP  
- **Resolvers:** 127.0.0.1 / ::1  
- **Health:** Stable and authoritative

---

## **6. DNS Overview**

- Primary zone: **afrowave.ltd**  
- Includes all required SRV/A/AAAA records for Samba AD  
- With IPAM service implemented:  
  - A/PTR records for WG clients will be created on **connect**,  
  - Removed on **disconnect** or **lease expiration**.

---

## **7. WireGuard – Status and Services**

### Active:
- wg-link  
- wg-link-443  
- wg-internal  
- wg-inet  

### Planned / Skeleton:
- wg-learn  
- wg-enroll  
- wg-guest  
- wg-rootoz  

---

## **8. System Services (Status)**

- `sshd` – enabled  
- `samba-ad-dc` – enabled  
- `firewalld` – enabled  
- `systemd-resolved` – disabled  
- WireGuard – managed via `wg-quick`

---

## **9. Routing Overview**

```
default via 69.169.97.129 dev eth0
10.20.0.0/16 dev wg-internal
10.22.0.0/16 dev wg-inet
10.24.0.0/16 dev wg-learn     # planned
10.26.0.0/16 dev wg-enroll    # planned
10.30.0.0/24 dev wg-link
10.31.0.0/24 dev wg-link-443
```

---

## **10. Pending Work / Future Steps**

- [ ] Document all WG skeleton interfaces  
- [ ] Implement **Afrowave.WgAddressDispatcher** (IPAM)  
- [ ] AD Groups `AFW_VPN_*` + access policy mapping  
- [ ] Create additional internal DNS zone entries for wg-link-443  
- [ ] Add monitoring & health checks for all WG tunnels  
- [ ] Add third public-facing server (Web/API) to the WG Mesh  

---

## **Changes Compared to Previous Version**

- Added new WG tunnel **wg-link-443** (port 443/udp, new range 10.31.0.0/24)  
- Confirmed final ranges of all WG networks (20/16, 22/16, 24/16, 26/16, 30/24, 31/24)  
- Updated roles and purposes of individual WG networks based on new architecture  
- Updated firewall zone descriptions and mappings  
- Added structure for the upcoming IPAM service (Afrowave.WgAddressDispatcher)  
- Updated DNS section to support dynamic WG client registration  
- Removed outdated WG-mail references  

---

# **END OF FILE**


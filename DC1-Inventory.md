# Afrowave DC1 – Infrastructure Inventory

**Server Name:** afw-dc1
**Role:** Primary Samba AD DC, DNS, KDC, VPN hub (WG-Link, WG-Internal, WG-Inet)
**Status:** Active, production-critical

---

## 1. Hardware Overview

* **Platform:** VPS (Interserver)
* **CPU:** Virtual (shared cores)
* **RAM:** 3 GB
* **Storage:** SSD-backed virtual disk
* **NIC:** eth0 (public WAN)

---

## 2. Operating System

* **OS:** Debian 13.1 (Trixie)
* **Kernel:** Debian stock kernel (systemd-managed)
* **Init:** systemd
* **Firewall:** firewalld (nftables backend)

---

## 3. Network Interfaces

### Physical

* **eth0** – WAN / public Internet

  * IPv4: `69.169.97.200/25`
  * Gateway: `69.169.97.129`
  * IPv6: global address assigned via ISP
  * Zone: `public`

### Virtual (WireGuard)

* **wg-link**  – Server-to-server trusted mesh (DC1 ↔ home, VPS)

  * IP: `10.30.0.1/24`, `fd10:af:30::1/64`
  * Zone: `aw-link`

* **wg-internal** – Internal AD/Samba/DNS/management VPN

  * IP: `10.20.0.1/16`, `fd10:af:2::1/64`
  * Zone: `aw-internal`

* **wg-inet** – Internet egress VPN (primary IPv4 exit)

  * IP: `10.22.0.1/16`, `fd10:af:22::1/64`
  * Zone: `aw-inet`

* **wg-enroll** – Enrollment-only network (planned)

  * Zone: `aw-enroll`

* **wg-guest** – Guest Internet access (isolated)

  * Zone: `aw-guest`

* **wg-rootoz** – High-security admin tunnel (planned)

  * Zone: `aw-rootoz`

---

## 4. Firewall Zones (firewalld)

### Public (eth0)

* **Allowed services:** ssh, dhcpv6-client
* **Masquerade:** yes
* **Purpose:** WAN access, secure remote admin only

### aw-link (wg-link)

* Purpose: trusted server-to-server mesh
* Allowed: DNS, Kerberos, LDAP, LDAPS, NTP, Samba, high ports

### aw-internal (wg-internal)

* Purpose: internal AD/DNS/Kerberos network
* Allowed: DNS, LDAP/LDAPS, Kerberos, NTP, Samba
* Masquerade: no

### aw-inet (wg-inet)

* Purpose: VPN client Internet egress
* Masquerade: yes (IPv4 NAT)
* Allows domain services for internal routing only

### aw-enroll, aw-guest, aw-rootoz

* Prepared and mapped to interfaces
* Rules to be documented after full deployment

---

## 5. Samba / Active Directory Services

* **Domain:** afrowave.ltd
* **Functions:**

  * Samba AD Domain Controller
  * DNS Server (SAMBA_INTERNAL)
  * LDAP/LDAPS
  * Kerberos + kpasswd
  * NTP (samba-managed)
* **Health:** stable (DC1 is authoritative DNS)

---

## 6. DNS Overview

* **Primary zone:** `afrowave.ltd`
* **Records:**

  * A/AAAA for DC1
  * SRV records for LDAP/Kerberos/DNS
  * WG endpoints resolved externally
* **Resolvers:**

  * `/etc/resolv.conf` points to 127.0.0.1 and ::1 (Samba DNS)
  * systemd-resolved not used

---

## 7. WireGuard Overview

### wg-link

* Role: backbone link to other Afrowave servers
* Port: 52030/udp
* NAT: no
* Status: active

### wg-internal

* Role: internal AD traffic
* Port: 52020/udp
* NAT: no
* Status: active

### wg-inet

* Role: Internet egress for clients
* Port: 52010/udp
* NAT: yes (firewalld)
* Status: active

### Others (planned/skeleton)

* wg-enroll
* wg-guest
* wg-rootoz

---

## 8. System Services

* **sshd** – enabled
* **samba-ad-dc** – enabled
* **firewalld** – enabled
* **WireGuard** – wg-quick used for all tunnels

Cockpit previously installed → removed to prevent interference with network configs.

---

## 9. Routing Overview

```
default via 69.169.97.129 dev eth0
10.20.0.0/16 dev wg-internal
10.22.0.0/16 dev wg-inet
10.30.0.0/24 dev wg-link
```

IPv6 routing through ISP global prefix is functional on DC1.
Client IPv6 over WG-Inet not enabled yet.

---

## 10. To Be Updated / Pending

* [ ] Finalize skeleton interfaces for wg-enroll, wg-guest, wg-rootoz
* [ ] Document full DNS zone structure (including WG-link SRV/A records)
* [ ] Update Live-Checks with all new WG tunnels
* [ ] Add IPv6 future design section

---

# END

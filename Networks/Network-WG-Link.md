# Afrowave Infrastructure – WG-LINK Architecture (2025)

WG-LINK is the primary backbone interconnecting all Afrowave servers across regions. It provides stable, trusted, unrestricted communication for core services such as Active Directory, DNS, Kerberos, CA operations, and internal application traffic.

This document reflects the **updated architecture**, including the unified logical design of **wg-link** and **wg-link-443**.

---

# 1. Purpose

WG-LINK forms the backbone for:
- Active Directory replication (LDAP, Kerberos, GC)
- DNS inter-server communication
- Internal service-to-service links
- Monitoring and management traffic
- Secure routing between datacenters and home servers
- Failover connectivity and fallback tunnels

WG-LINK is designed for maximum trust and minimal filtering.

---

# 2. Logical Topology

WG-LINK operates as a **logical single network**, even though it may be transported over different ports.

```
              +----------------------------+
              |        DC1 (10.30.0.1)     |
              |   Primary hub & AD Root    |
              +---------------+------------+
                              |
                        wg-link / wg-link-443
                              |
          -------------------------------------------------
          |                         |                     |
 +----------------+     +-------------------+   +---------------------+
 | afw-microserver|     |  EDGE Server      |   |  Future Servers     |
 | 10.30.0.10     |     | 10.30.0.20        |   | 10.30.0.x           |
 +----------------+     +-------------------+   +---------------------+
```

**Any-to-any connectivity** is achieved via the hub (DC1), with DNS resolving all server names.

---

# 3. IP Addressing

WG-LINK uses a dedicated /24:
```
10.30.0.0/24
```

Recommended assignments:
- **DC1** → 10.30.0.1
- **afw-microserver** → 10.30.0.10
- **EDGE server** → 10.30.0.20
- **future nodes** → 10.30.0.x

All IPs are static.

---

# 4. Single Logical Network via Dual Transport

WG-LINK may be accessed via two different ports:

### 4.1 Primary transport
```
wg-link → UDP 52030
```

### 4.2 Fallback transport
```
wg-link-443 → UDP 443
```

## Both interfaces carry **the same logical identity**:
- same WG private/public keys (optional but recommended)
- same internal IP addressing (10.30.0.x)
- same routing rules
- same DNS expectations

The difference is **only the transport port**.

This enables connectivity through restrictive firewalls without requiring two separate logical tunnels.

---

# 5. Interfaces and Configuration

Both interfaces belong to the **aw-link** firewalld zone:

```
firewall-cmd --permanent --zone=aw-link --add-interface=wg-link
firewall-cmd --permanent --zone=aw-link --add-interface=wg-link-443
firewall-cmd --reload
```

Zone properties:
- `target=ACCEPT`
- no NAT
- no filtering

---

# 6. DNS Requirements

All WG-LINK nodes must have DNS A records:

```
dc1-link.afrowave.ltd        → 10.30.0.1
afw-microserver.link.afrowave.ltd → 10.30.0.10
edge-link.afrowave.ltd       → 10.30.0.20
```

These are used for:
- Kerberos SPNs
- AD DRS replication
- DNS lookups
- Monitoring
- Internal services

---

# 7. Routing

WG-LINK routing rules:
- each peer announces only its own `/32`
- routing between nodes is performed by DC1 (hub)
- allows stable any-to-any connectivity

Example:
```
AllowedIPs = 10.30.0.10/32
AllowedIPs = 10.30.0.20/32
```

This design is stable, scalable, and predictable.

---

# 8. Monitoring WG-LINK

Commands:
```
wg show wg-link
wg show wg-link-443
ping 10.30.0.1
ping 10.30.0.10
samba-tool drs showrepl
```

Monitoring health of both transports ensures continuity.

---

# 9. Relation to WG-LINK-443

WG-LINK-443 is not a separate network.  
It is simply a *transport alternative* for WG-LINK.

Use cases:
- users behind strict corporate firewalls
- environments with limited UDP port availability
- fallback routing during provider outages

Configuration guidelines:
- WG-LINK and WG-LINK-443 share the **same logical address space**
- DNS records do not differ
- both tunnels merge into the aw-link zone
- DC1 remains the hub for both

---

# Change Log (2025-12-07)
- Unified WG-LINK and WG-LINK-443 into a single logical network.
- Clarified dual-transport architecture using UDP 52030 (primary) and UDP 443 (fallback).
- Added DNS integration model linking all servers via `*.link.afrowave.ltd`.
- Formalized any-to-any interserver communication routed via DC1.
- Added full interface, routing, and zone configuration guidance.

---

# Tags
```
#wireguard #wg-link #architecture #vpn #afrowave #backbone #ad #infrastructure
```
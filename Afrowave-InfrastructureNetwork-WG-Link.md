Afrowave-Infrastructure/Network-WG-Link
========================

# WG-LINK – Afrowave Server Backbone Network
WG-LINK is the core backbone network interconnecting all Afrowave servers.  
It is a trusted, unrestricted, high-throughput channel designed for stability, security, and zero-friction server communication.

This network is the foundation of:
- Active Directory replication
- Kerberos authentication
- Samba internal DNS
- Cross-server service communication
- Monitoring and management
- Backup routing and failover paths

---

# 1. Design Philosophy

WG-LINK follows the **“Internet Principle”**:

> Every server in the backbone is a fully trusted router and neighbor.  
> No firewalls, no NAT, no filtering — only maximum reliability, speed, and transparency.

This ensures:
- Minimal operational complexity  
- Maximum consistency across data centers  
- Ability to handle any protocol without adjustments  
- Predictable troubleshooting  
- Full redundancy between locations  

WG-LINK is the most trusted and highest-priority network segment.

---

# 2. Purpose of WG-LINK

WG-LINK is used for:

### ✔ 2.1 Active Directory (critical)
- LDAP / LDAPS
- Kerberos (UDP/TCP 88)
- Global Catalog (3268/3269)
- DRS replication
- NTP signed time (ntp_signd)
- DNS queries between DCs and servers

### ✔ 2.2 Backend Service Communication
- Internal HTTP/HTTPS services
- File sharing
- Certificates and CA operations
- Monitoring systems

### ✔ 2.3 Inter-server VPN Connectivity
- Server-to-server reachability
- Internal routing and failover
- Cross-location connectivity (USA ↔ EU ↔ home server)

### ✔ 2.4 Backup WAN Routing
WG-LINK may optionally provide:
- Backup internet connectivity for DC1/DC2
- Failover routing between regions
- Controlled fallback for critical updates

---

# 3. IP Address Scheme

WG-LINK uses a **dedicated IPv4 /24**, large enough for:
- All present and future servers
- Temporary testing peers
- Backup servers
- Internal services

Recommended default:
```
WG-LINK IPv4 Network: 10.30.0.0/24
```

### 3.1 Suggested server assignments
| Server | Role | WG-LINK IP | Status |
|--------|------|-------------|--------|
| DC1 | Primary AD DC | 10.30.0.1 | Active |
| DC2 | Secondary AD DC (future) | 10.30.0.2 | Planned |
| Diblík | Home server | 10.30.0.10 | Active |
| Edge | Web/Translate server | 10.30.0.20 | Active |
| Additional servers | Reserved | 10.30.0.x | Planned |

WG-LINK uses **static IPs** for all servers.

---

# 4. WireGuard Configuration Standard

### 4.1 Interface naming
All servers must use:
```
wg-link
```

### 4.2 MTU
Consistent MTU across servers:
```
MTU = 1420
```
(or adjusted per transport provider)

### 4.3 AllowedIPs
WG-LINK is always **/32 per peer**, routing is handled dynamically:

Example:
```
AllowedIPs = 10.30.0.1/32
```

Do **not** route entire subnets through peers — each server is its own endpoint.

---

# 5. Firewall Rules (Firewalld)

WG-LINK is placed in a dedicated zone:

`Zone name: aw-link`

Zone requirements:
- `target=ACCEPT`
- no filtering
- no NAT
- no service restrictions

Example:
```
firewall-cmd --permanent --new-zone=aw-link
firewall-cmd --permanent --zone=aw-link --set-target=ACCEPT
firewall-cmd --permanent --zone=aw-link --add-interface=wg-link
firewall-cmd --reload
```

---

# 6. DNS Requirements

WG-LINK must have valid DNS A records for all servers:

```
dc1-link.afrowave.ltd → 10.30.0.1
dc2-link.afrowave.ltd → 10.30.0.2
dibles-link.afrowave.ltd → 10.30.0.10
edge-link.afrowave.ltd → 10.30.0.20
```

(Names may be adjusted but must be consistent.)

These records allow:
- Kerberos SPN consistency
- AD replication
- Internal service discovery
- Diagnostics

---

# 7. Monitoring and Health Checks

WG-LINK health must be continuously monitored:

### 7.1 Ping checks:
```
ping 10.30.0.1
ping 10.30.0.10
ping 10.30.0.20
```

### 7.2 WireGuard handshake checks:
```
wg show wg-link
```

### 7.3 AD replication tests:
```
samba-tool drs showrepl
```

### 7.4 DNS service tests:
```
host dc1-link.afrowave.ltd
```

Any packet loss or handshake failure is critical.

---

# 8. Backup Routing (Optional)

WG-LINK may provide backup Internet routing between regions.

Rules:
- Only servers (not clients) may receive failover routes.
- NAT must **not** be applied on WG-LINK.
- Failover must be explicit, never automatic.

This ensures stability while providing redundancy.

---

# 9. Future Extensions

WG-LINK will be used for:
- DC2 replication (EU region)
- Secure CA interactions
- Streaming logs / metrics
- Cross-continent service clusters

---

# 10. Cross-References
- [DC1 Inventory](DC1-Inventory.md)
- [DC1 Live Checks](DC1-Live-Checks.md)
- [Troubleshooting](Troubleshooting/)
- [GitHub Sync Guide](GitHub-Sync-Guide.md)

---

# Tags
```
#network #wireguard #wg-link #infrastructure #backbone #vpn #ad #servers
```
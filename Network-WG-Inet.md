# Afrowave Infrastructure – WG-Inet Network

**Version:** Draft 1  
**Scope:** Complete documentation of Afrowave’s WG-Inet network segment (primary Internet VPN egress).

---

## 1. Purpose & Role of WG-Inet
WG-Inet is the **primary secure VPN tunnel for Internet egress** within the Afrowave infrastructure.

It is designed to provide:
- a controlled and encrypted IPv4 Internet exit point for clients,
- secure remote access to the Internet through Afrowave servers,
- a failover/fallback path for intra-infrastructure communication,
- a clean separation of trust levels via firewalld zone isolation,
- high-performance, low-latency encrypted transport.

WG-Inet **does NOT expose domain services to the public Internet**. All domain-related traffic is only available inside WG tunnels.

---

## 2. Addressing Scheme
### IPv4
```
10.22.0.0/16
DC1:    10.22.0.1
Client: 10.22.0.10
```

### IPv6 (ULA – internal only)
```
fd10:af:22::/64
DC1:    fd10:af:22::1
Client: fd10:af:22::10
```

IPv6 is currently **not used** for Internet egress. IPv6 on the server itself works normally.
A future plan will introduce global IPv6 prefix routing.

---

## 3. WireGuard Configuration (Server – DC1)
**`/etc/wireguard/wg-inet.conf`**

```ini
# =========================================
# Afrowave WG-INET – DC1
# Primary IPv4 Internet Exit for Clients
# =========================================

[Interface]
PrivateKey = <DC1-WG-INET-PRIVATE-KEY>
Address = 10.22.0.1/16, fd10:af:22::1/64
ListenPort = 52010

# NAT and forwarding are handled by firewalld (zone: aw-inet):
#   masquerade = yes
#   forward = yes
# No iptables PostUp/PostDown rules are used.

[Peer]
# Slávek — Desktop (primary client)
PublicKey = si+kF9bbaGceRo6c7h2WvXZJpX+1yhezO9kd+MVNZ0Q=
AllowedIPs = 10.22.0.10/32, fd10:af:22::10/128
PersistentKeepalive = 25
```

---

## 4. WireGuard Configuration (Client)
```ini
[Interface]
PrivateKey = <CLIENT-PRIVATE-KEY>
Address = 10.22.0.10/16, fd10:af:22::10/64
DNS = 10.22.0.1
MTU = 1380

[Peer]
PublicKey = <SERVER-PUBLIC-KEY>
AllowedIPs = 0.0.0.0/1, 128.0.0.0/1, ::/1, 8000::/1
Endpoint = ad.afrowave.ltd:52010
PersistentKeepalive = 25
```

### Note on IPv6
Client IPv6 through WG-Inet is intentionally disabled for now.  
Server-side IPv6 works normally.  
Client-side IPv6 egress will be implemented later using either:
- a global IPv6 prefix (preferred), or
- temporary NAT66 (not preferred).

---

## 5. Firewall – Zone `aw-inet`
WG-Inet uses a dedicated firewalld zone to isolate and control traffic.

### Zone Properties
- **target:** ACCEPT
- **interfaces:** `wg-inet`
- **services:** dns, kerberos, kpasswd, ldap, ldaps, rpc-bind, samba, samba-dc, ssh
- **ports opened:** 52010/udp
- **masquerade:** yes (IPv4 NAT for clients)
- **forward:** yes (required for routing 10.22.0.0/16 → Internet)

### Masquerade Activation
```
sudo firewall-cmd --zone=aw-inet --add-masquerade --permanent
```

---

## 6. Routing & Forwarding
### Global forwarding settings (`/etc/sysctl.conf`)
```conf
net.ipv4.ip_forward=1
net.ipv6.conf.all.forwarding=1
```

### Routing Table – DC1
```
10.22.0.0/16 dev wg-inet src 10.22.0.1
fd10:af:22::/64 dev wg-inet
```

### NAT State
```
iptables -t nat -L POSTROUTING
```
→ Must remain **empty**. All NAT is handled by firewalld (nftables backend).

---

## 7. Live Operational Checks
### a) WireGuard State
```
wq show wg-inet
systemctl status wg-quick@wg-inet
```

### b) IPv4 Internet
```
ping 1.1.1.1
curl -4 https://example.com
```

### c) Server-only IPv6 Test
```
ping6 2606:4700:4700::1111
curl -6 https://example.com
```

### d) Firewall Zone Overview
```
firewall-cmd --list-all --zone=aw-inet
```

### e) Routing Table
```
ip r
ip -6 r
```

---

## 8. Security Model
WG-Inet enforces:
- strict zone-based separation (`aw-inet`),
- NAT only inside the VPN zone,
- minimal public exposure (only UDP/52010 and SSH on eth0),
- client isolation via explicit AllowedIPs,
- no domain services exposed publicly.

The public (WAN) interface `eth0` exposes only:
- `ssh`
- `dhcpv6-client`
All other ports and services are blocked.

---

## 9. Future Extensions
- Move primary Internet egress to **Diblík** (home server),
- DC1 becomes secondary/fallback egress,
- Web VPS becomes tertiary egress,
- Introduce global IPv6 prefix for clean IPv6 routing,
- Possible multi-tunnel bonding for failover/aggregation.

---

## 10. Implementation Status
- [x] DC1 implemented and functional
- [x] firewalld NAT functional
- [x] IPv4 Internet access for clients works
- [ ] Add WG-Inet entry to DC1-Inventory
- [ ] Extend DC1-Live-Checks with WG-Inet checks
- [ ] Prepare WG-Inet skeletons for Diblík & VPS servers

---

# END


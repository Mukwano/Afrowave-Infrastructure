Afrowave Infrastructuer - WG-Inet Network
========================

**Version:** Draft 1  
**Scope:** Complete documentation of Afrowave’s WG-Inet network segment (Internet VPN egress).

---

## 1. Purpose & Role of WG-Inet
WG-Inet je **primární zabezpečená VPN síť pro přístup do internetu** přes Afrowave infrastrukturu.

Je navržena jako:
- hlavní výstupní bod do internetu pro klienty,
- rozšíření interní sítě o zabezpečený IPv4 tunel,
- plně řízený firewalld zónou `aw-inet`,
- síť s NATem pro IPv4 (bez IPv6 NAT),
- nízká latence, vysoká spolehlivost, vysoká úroveň šifrování.

Služby dostupné přes WG-Inet:
- veškerý IPv4 internetový provoz,
- AD / DNS / management traffic směrem na DC1,
- fallback kanál pro komunikaci server–server.

WG-Inet **není** určena pro veřejný přístup do DC1 – DC1 poskytuje služby pouze přes WG tunely.

---

## 2. IP Addressing Scheme
### IPv4 tunel:
```
10.22.0.0/16
DC1:    10.22.0.1
Client: 10.22.0.10
```

### IPv6 (ULA only, no internet routing):
```
fd10:af:22::/64
DC1:    fd10:af:22::1
Client: fd10:af:22::10
```

IPv6 se zatím nepoužívá pro odchozí internet. Budoucí plán: globální prefix → čisté routování.

---

## 3. WireGuard Configuration (Server – DC1)
**/etc/wireguard/wg-inet.conf**

```ini
# =========================================
# Afrowave WG-INET - DC1
# IPv4 Internet Exit for Clients
# =========================================

[Interface]
PrivateKey = <DC1-WG-INET-PRIVATE-KEY>
Address = 10.22.0.1/16, fd10:af:22::1/64
ListenPort = 52010

# NAT + forwarding řeší firewalld:
#   - Zone: aw-inet
#   - masquerade=yes
#   - forward=yes
# Proto zde nejsou žádné PostUp/PostDown iptables.

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

**Poznámka:** IPv6 přes tunel je zatím blokováno ISP / absencí NAT66. IPv6 bude řešeno později.

---

## 5. Firewall – Zóna `aw-inet`
Významná část WG-Inet infrastruktury.

### Konfigurace zóny
- **target:** ACCEPT
- **interfaces:** wg-inet
- **services:** dns, kerberos, kpasswd, ldap, ldaps, rpc-bind, samba, samba-dc, ssh
- **ports opened:** 52010/udp
- **masquerade:** yes (IPv4 NAT)
- **forward:** yes (nutné pro routing 10.22.0.0/16 → internet)

### Masquerade
Řízeno **firewalld (nftables)**:
```
sudo firewall-cmd --zone=aw-inet --add-masquerade --permanent
```

---

## 6. Routing & Forwarding
### Globální forwarding (`/etc/sysctl.conf`)
```conf
net.ipv4.ip_forward=1
net.ipv6.conf.all.forwarding=1
```

### Routing tabulka DC1
```
10.22.0.0/16 dev wg-inet src 10.22.0.1
fd10:af:22::/64 dev wg-inet
```

### NAT tabulka – pouze firewalld
`
iptables -t nat -L POSTROUTING
`
→ **musí být prázdná** (jen policy ACCEPT). Vše řeší nft.

---

## 7. Live Checks (Operational)
### a) WireGuard
```
wg show wg-inet
systemctl status wg-quick@wg-inet
```

### b) IPv4 Internet
```
ping 1.1.1.1
curl -4 https://example.com
```

### c) IPv6 server-only test
```
ping6 2606:4700:4700::1111
curl -6 https://example.com
```

### d) Firewall
```
firewall-cmd --list-all --zone=aw-inet
```

### e) Routing
```
ip r
ip -6 r
```

---

## 8. Security Model
WG-Inet je chráněna:
- oddělenou zónou `aw-inet`,
- NATem pouze uvnitř zóny,
- nulovou expozicí doménových služeb veřejně,
- endpointem dostupným pouze na UDP 52010,
- klienti jsou explicitně definováni pomocí AllowedIPs.

Public interface (eth0) obsahuje pouze:
- ssh
- dhcpv6-client

Vše ostatní je blokováno.

---

## 9. Future Extensions
- přesun primárního internetového egressu na Diblíka (server doma),
- DC1 jako sekundární/failover exit,
- Web VPS jako terciární exit,
- globální IPv6 prefix → čisté IPv6 routování bez NAT66,
- možnost bondingu vícero WG-INET tunelů.

---

## 10. Stav Implementace
- [x] DC1 funkční
- [x] NAT přes firewalld
- [x] IPv4 internet routován pro klienty
- [ ] Dokumentace v DC1-Inventory
- [ ] Dokumentace v Live-Checks přidat
- [ ] Přidat WG-Inet skeletony na dalších serverech (Diblík, VPS)

---

# END



# Afrowave Infrastructure – WG-Internal Network


**Version:** Draft 1  
**Scope:** Detailed technical documentation of the WG-Internal VPN segment used inside Afrowave infrastructure.

---

## 1. Purpose & Role of WG-Internal
WG-Internal is the **private, non-routable internal transport layer** mezi Afrowave DC1 (Debian AD Controller) a vybranými interními zařízeními (například Slávek – Desktop).

Je navržena jako síť:
- výhradně pro **privátní interní služby**,
- bez NATu,
- s plným L3 routováním,
- s absolutně minimální expozicí ven,
- s garantovaným šifrováním a stabilitou.

Používá se zejména pro:
- AD traffic (LDAP, LDAPS, Kerberos)
- DNS
- RPC/Samba interní volání
- interní management a provisioning

WG-Internal **není** určena pro internetové připojení, NAT nebo veřejné služby.

---

## 2. IP Adresní Plán
### IPv4:
```
10.20.0.0/16
DC1: 10.20.0.1
Client (desktop): 10.20.0.10
```

### IPv6 (ULA):
```
fd10:af:2::/64
DC1: fd10:af:2::1
Client: fd10:af:2::10
```

IPv6 zde slouží jako budoucí-proofing – není používán pro externí routing.

---

## 3. WireGuard Konfigurace (Server – DC1)
**/etc/wireguard/wg-internal.conf**

```rust

```



```ini
[Interface]
# Afrowave DC1 – WG-Internal
Address = 10.20.0.1/16, fd10:af:2::1/64
ListenPort = 52020
PrivateKey = <DC1-WG-INTERNAL-PRIVATE-KEY>

# Žádný NAT, žádný PostUp – forwarding je globální v /etc/sysctl.conf

[Peer]
# Slávek – desktop (internal)
PublicKey = si+kF9bbaGceRo6c7h2WvXZJpX+1yhezO9kd+MVNZ0Q=
AllowedIPs = 10.20.0.10/32, fd10:af:2::10/128
PersistentKeepalive = 25
```

---

## 4. WireGuard Konfigurace (Client)

```ini
[Interface]
PrivateKey = <CLIENT-KEY>
Address = 10.20.0.10/16, fd10:af:2::10/64
DNS = 10.20.0.1
MTU = 1380

[Peer]
PublicKey = <SERVER-PUBKEY>
AllowedIPs = 10.20.0.0/16, fd10:af:2::/64
Endpoint = ad.afrowave.ltd:52020
PersistentKeepalive = 25
```

---

## 5. Firewall – Zóna aw-internal
WG-Internal je mapována do vlastní firewalld zóny:

```sh
sudo firewall-cmd --zone=aw-internal --add-interface=wg-internal --permanent
```

### Výsledná zóna (stav DC1):
- **target:** ACCEPT
- **interfaces:** wg-internal
- **services:** dns, kerberos, ldap, ldaps, ntp, samba
- **masquerade:** no
- **forward:** yes

### Proč ACCEPT?
Protože WG-Internal je síť **vysoké důvěry** – pouze mezi DC1 a vybranými interními stanicemi.

---

## 6. Forwarding & Routing
WG-Internal používá čisté routování:

### `/etc/sysctl.conf`

```conf
net.ipv4.ip_forward=1
net.ipv6.conf.all.forwarding=1
```

### Routing tabulka na DC1
```
10.20.0.0/16 dev wg-internal src 10.20.0.1
fd10:af:2::/64 dev wg-internal
```

Žádný NAT. Každá strana vidí druhou jako plnohodnotný interní subnet.

---

## 7. Kontrola Funkčnosti (Live Checks)
### a) WireGuard stav

```sh
sudo wg show wg-internal
systemctl status wg-quick@wg-internal
```

### b) Pingy

```sh
ping 10.20.0.10
ping6 fd10:af:2::10
```

### c) Firewall

```sh
sudo firewall-cmd --list-all --zone=aw-internal
```

---

## 8. Budoucí Rozšíření
- přidání dalších interních serverů (Edge, Mirror-DC, VM hosty)
- možnost připojení management VLAN přes bridged WG adapter
- rozšíření IPv6 (globální prefix) bez NAT

---

## 9. Stav Implementace
- [x] DC1: kompletně funkční
- [x] Client: funkční
- [x] Routování OK
- [ ] Doplnit dokumentaci do DC1-Inventory
- [ ] Doplnit do DC1-Live-Checks.md

---

# END
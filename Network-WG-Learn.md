Afrowave Infrastructure – WG-Learn Network
========================


**Version:** Draft 1
**Scope:** Concept + baseline design for the WG-Learn VPN used for teaching / lab environments.

---

## 1. Purpose & Role of WG-Learn

WG-Learn is a **learning / teaching VPN** designed to give students secure remote access to their personal lab environment, without exposing the full internal infrastructure.

Key properties:

* Students connect from the Internet via WireGuard.
* Over WG-Learn they reach **RDP on one or more “Learning Servers”** (Linux with xrdp / similar).
* They log in using **standard Afrowave domain accounts** (roaming profiles, developer tools, etc.).
* All normal work (web browsing, git, compilers, editors, etc.) runs **inside the RDP session**, not from the student’s local machine.
* WG-Learn itself **does not provide general Internet egress** for the client; Internet is consumed from inside the VM session.

Effectively: the VPN is a secure “pipe” from the student’s device to the RDP port of the learning environment.

---

## 2. Trust & Separation Model

* **Student identity:** standard Afrowave domain users (e.g. `AFROWAVE\\bolaji`).
* **Network separation:**

  * WG-Learn clients do *not* see the whole internal LAN.
  * They see only:

    * DC1 (for DNS / Kerberos / LDAP **if needed**), and
    * RDP endpoints on Learning Servers.
* **Permissions separation:**

  * What a user can do is controlled primarily via AD (groups, policies, file shares).
  * Network-level rules keep the blast radius small if a client is compromised.

Optionally, later we can add:

* Guest / non-domain users via a dedicated Learning AD OU or local accounts.
* A separate learning domain, but re-using the same WG-Learn tunnel.

---

## 3. Addressing Scheme (Proposal)

To keep the addressing clean and consistent with existing ranges:

### IPv4

```text
WG-LEARN IPv4:   10.24.0.0/16
DC1 address:     10.24.0.1
Learning server: 10.24.0.20 (example)
Clients:         10.24.0.100+ (per student)
```

### IPv6 (ULA, internal only)

```text
WG-LEARN IPv6:   fd10:af:24::/64
DC1 address:     fd10:af:24::1
Learning server: fd10:af:24::20
Clients:         fd10:af:24::100+
```

This is **internal-only** space; no NAT from WG-Learn to the Internet. Students get Internet *inside* their RDP session.

---

## 4. WireGuard Interface Standard

* Interface name: `wg-learn`
* MTU: `1420` (aligned with other WG tunnels)
* Keys: standard WireGuard keypair per peer.

### 4.1 DC1 – `wg-learn` (server)

**Planned configuration skeleton:**

```ini
[Interface]
# DC1 – WG-Learn
Address    = 10.24.0.1/16, fd10:af:24::1/64
ListenPort = 52040
PrivateKey = <DC1-WG-LEARN-PRIVATE-KEY>

# No NAT here – WG-Learn is internal-only.

# Example peer – Student 1
[Peer]
# Student-1 laptop
PublicKey  = <STUDENT1-PUBKEY>
AllowedIPs = 10.24.0.100/32, fd10:af:24::100/128
PersistentKeepalive = 25

# (More students: 10.24.0.101, 10.24.0.102, ...)
```

Learning Server can:

* either sit directly in 10.24.0.0/16, using a WG interface on that host, or
* be on another internal network, with a **static route** on DC1 pointing to it.

---

## 5. Firewall – Zone `aw-learn` (New)

WG-Learn gets its own firewalld zone for strong isolation.

### 5.1 Zone basics

* **Zone name:** `aw-learn`
* **Interfaces:** `wg-learn`
* **Target:** `default` (not ACCEPT — we whitelist services explicitly)
* **Masquerade:** `no` (no Internet egress via WG-Learn)
* **Forward:** `yes` (allow routing to Learning Servers & DC1)

### 5.2 Allowed traffic (high-level)

From WG-Learn clients:

* To DC1:

  * DNS (tcp/udp 53) – optional; may be used for SRV lookups.
  * Kerberos / kpasswd – only if we ever need direct auth over WG-Learn.
* To Learning Servers:

  * RDP (tcp/3389)
  * optionally SSH (tcp/22) for maintenance / advanced training.

Everything else is **blocked** by default in `aw-learn`.

Later we will add explicit `firewall-cmd` examples once the exact hostnames / IPs of Learning Servers are defined.

---

## 6. Client Configuration Template

Example WireGuard client config for a student:

```ini
[Interface]
PrivateKey = <STUDENT-PRIVATE-KEY>
Address    = 10.24.0.100/16, fd10:af:24::100/64
DNS        = 10.24.0.1   # optional – only if we want DNS over WG-Learn
MTU        = 1380

[Peer]
PublicKey  = <DC1-WG-LEARN-PUBKEY>
AllowedIPs = 10.24.0.0/16, fd10:af:24::/64
Endpoint   = ad.afrowave.ltd:52040
PersistentKeepalive = 25
```

The client sees only the WG-Learn address space and any internal endpoints that DC1 routes and firewalld permits.

---

## 7. RDP / Learning Server Concept

Each student gets access to one or more **Learning Servers**:

* OS: Linux (e.g., Debian/Ubuntu) with `xrdp` or similar.
* Joined to Afrowave domain.
* Roaming profiles stored on a central file share.
* Full developer stack (IDEs, compilers, tools) pre-installed.

Network-wise:

* Learning Server must be reachable from DC1 and from WG-Learn clients.
* RDP port 3389 exposed only inside **internal / WG** networks, never to public Internet.
* AD policies can restrict what users can do (e.g., no local admin).

---

## 8. Integration with Existing Design

WG-Learn fits into the existing VPN family:

* `wg-link`   – backbone between servers
* `wg-internal` – internal admin / AD traffic
* `wg-inet`   – client Internet egress
* `wg-guest`  – guest Internet-only (future)
* `wg-enroll` – enrollment / first join (future)
* `wg-rootoz` – high-security admin (future)
* `wg-learn`  – **teaching / lab VPN** (this document)

All follow the same principles:

* Dedicated subnet per tunnel.
* Dedicated firewalld zone per tunnel.
* Clear NAT vs non-NAT policy.

---

## 9. Next Steps (Implementation Plan)

1. Confirm chosen subnet: `10.24.0.0/16` + `fd10:af:24::/64` (or adjust if needed).
2. Generate DC1 `wg-learn` keys and create `/etc/wireguard/wg-learn.conf` from the skeleton.
3. Create firewalld zone `aw-learn`, attach `wg-learn`, add minimal rules (DNS + RDP to Learning Servers).
4. Decide where the first Learning Server will live (IP, hostname) and open RDP from `aw-learn` only.
5. Prepare a **student WG profile template** (.conf) with placeholders for keys.
6. Update:

   * DC1-Inventory (add WG-Learn section)
   * DC1-Live-Checks (add WG-Learn checks)
   * Learning Server documentation (separate file).

---

# END

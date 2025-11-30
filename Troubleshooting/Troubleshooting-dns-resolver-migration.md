Troubleshooting-dns-resolver-migration
========================

# DNS Resolver Migration – Troubleshooting Guide

## Overview

This document explains the symptoms, root cause, diagnosis, and permanent fix for DNS failures after changing WireGuard address ranges (e.g., 10.7.x.x → 10.30.x.x). It also explains how `systemd-resolved` interacts with `/etc/resolv.conf`, why tools like `dig` may work while normal applications fail, and the correct long‑term configuration for stable DNS operation.

---

## 1. Symptoms

After changing VPN subnets or DNS IP addresses, affected systems (e.g., Debian server "Diblík") may show:

* Internet via **IP works**:

  ```bash
  ping 1.1.1.1     # works
  ```
* DNS tools like `dig` work:

  ```bash
  dig @10.30.0.1 google.com
  ```
* But applications **cannot resolve domains**:

  ```bash
  ping google.com     # fails: Temporary failure in name resolution
  apt update          # fails
  curl https://...    # fails
  ```

This indicates that:

* Networking is functional
* DNS server is reachable
* **But local DNS resolver is misconfigured**

---

## 2. Root Cause

The system uses **systemd-resolved**, which provides a *local DNS stub* at:

```
127.0.0.53
```

And manages `/etc/resolv.conf` automatically.

When the old DNS server (e.g., 10.7.x.x) was removed and replaced with 10.30.0.1, the local resolver did **not** update its upstream DNS list. As a result:

* `dig` works → it bypasses systemd-resolved and queries DNS directly
* applications fail → they query 127.0.0.53, which tries contacting a non‑existent old DNS IP

This mismatch causes a complete DNS outage for system applications even though DNS servers are reachable.

---

## 3. Diagnosis Steps

To confirm the mismatch:

### 3.1 Check direct DNS

```
dig @10.30.0.1 google.com
```

If this works → DNS server is OK.

### 3.2 Check default route

```
ip r
```

Should show a valid gateway.

### 3.3 Check current resolver mode

```
cat /etc/resolv.conf
```

If you see:

```
# This is /run/systemd/resolve/stub-resolv.conf
```

then the system uses systemd‑resolved and ignores manual edits.

### 3.4 Check effective DNS servers

```
resolvectl status
```

If you see old DNS servers → that is the cause.

---

## 4. Permanent Fix (systemd‑resolved)

To correctly configure DNS and ensure it is stable:

### 4.1 Edit resolver configuration

```
sudo nano /etc/systemd/resolved.conf
```

Add:

```
[Resolve]
DNS=10.30.0.1 1.1.1.1
FallbackDNS=1.0.0.1 8.8.8.8
Domains=afrowave.ltd
LLMNR=no
MulticastDNS=no
```

### 4.2 Restart resolver

```
sudo systemctl restart systemd-resolved
```

### 4.3 Replace resolv.conf symlink

```
sudo rm /etc/resolv.conf
sudo ln -s /run/systemd/resolve/resolv.conf /etc/resolv.conf
```

### 4.4 Validate configuration

```
resolvectl status
ping google.com
host microsoft.com
dig google.com
```

All should work.

---

## 5. Why Previous Edits Failed

Direct edits to `/etc/resolv.conf` do **not** persist because the file is a symlink to:

```
/run/systemd/resolve/stub-resolv.conf
```

Systemd-resolved overwrites it every time network state changes.

Only settings placed in:

```
/etc/systemd/resolved.conf
```

or drop-ins under:

```
/etc/systemd/resolved.conf.d/
```

are permanent.

---

## 6. Prevention

To avoid resolver mismatches in the future:

* Always update `resolved.conf` after changing VPN or DNS ranges
* Avoid manually editing `/etc/resolv.conf` unless replacing systemd-resolved entirely
* Document DNS changes in infrastructure notes
* Verify DNS routing after every WireGuard address migration

---

## 7. Summary

| Layer                 | Status                       |
| --------------------- | ---------------------------- |
| Internet connectivity | ✔ OK                         |
| WireGuard routing     | ✔ OK                         |
| DNS server (DC1)      | ✔ OK                         |
| Local system resolver | ❌ Broken (old DNS) → ✔ Fixed |

After correction, DNS functions normally:

* IPv4 + IPv6
* Local domain resolution
* System services (`apt`, `curl`, browser)
* VPN‑routed name resolution

---

## 8. Status: **FIXED**

This incident is now fully resolved and documented.

All systems now correctly use:

```
Primary DNS:   10.30.0.1 (DC1)
Secondary DNS: 1.1.1.1 (Cloudflare)
Fallback DNS:  1.0.0.1, 8.8.8.8
Domain:        afrowave.ltd
```

This configuration is stable, persistent, and suitable for enterprise‑level infrastructure.

---

## End of Document

# Afrowave DC1 – Live Operational Checks

**Server:** afw-dc1  
**Purpose:** This checklist validates that all critical services, networking, VPN tunnels, DNS, and domain functionality are operational.  
Run after reboot, updates, network changes, or troubleshooting.

---

# 1. Basic System Health
### Check system uptime & load
```bash
uptime
```

### Check memory & swap
```bash
free -h
```

### Check disk space
```bash
df -h
```

---

# 2. Network Interfaces
### List all interfaces
```bash
ip a
```

### Confirm active WireGuard tunnels
```bash
wg show
```

### Confirm routing table
```bash
ip r
ip -6 r
```

Expected routes:
```
10.20.0.0/16 → wg-internal
10.22.0.0/16 → wg-inet
10.30.0.0/24 → wg-link
```

---

# 3. WireGuard Tunnel Checks
Each tunnel should be **active**, with recent handshake.

## WG-Link
```bash
wg show wg-link
systemctl status wg-quick@wg-link
```

## WG-Internal
```bash
wg show wg-internal
systemctl status wg-quick@wg-internal
```

## WG-Inet
```bash
wg show wg-inet
systemctl status wg-quick@wg-inet
```

## Other (planned)
Skeletons only — may show inactive:
```bash
wg show wg-enroll
wg show wg-guest
wg show wg-rootoz
```

---

# 4. Connectivity Tests
## Ping Local Networks (WG)
```bash
ping -c4 10.20.0.10      # internal client
ping -c4 10.22.0.10      # inet client
ping -c4 10.30.0.10      # link peer
```

## Ping Internet
### IPv4
```bash
ping -c4 1.1.1.1
curl -4 https://example.com
```

### IPv6 (server only)
```bash
ping6 -c4 2606:4700:4700::1111
curl -6 https://example.com
```

---

# 5. Firewall Status
### List active zones
```bash
firewall-cmd --get-active-zones
```

### Show public WAN zone
```bash
firewall-cmd --list-all --zone=public
```

### WG zones
```bash
firewall-cmd --list-all --zone=aw-link
firewall-cmd --list-all --zone=aw-internal
firewall-cmd --list-all --zone=aw-inet
firewall-cmd --list-all --zone=aw-enroll
firewall-cmd --list-all --zone=aw-guest
firewall-cmd --list-all --zone=aw-rootoz
```

### NAT state (firewalld-managed)
```bash
sudo firewall-cmd --query-masquerade --zone=aw-inet
```
Expected: `yes`

### Ensure no iptables NAT rules
```bash
iptables -t nat -L -n -v
```
Expected: **empty** (policy ACCEPT only)

---

# 6. DNS & Resolver Checks
### Check resolv.conf
```bash
cat /etc/resolv.conf
```
Expected:
```
nameserver 127.0.0.1
nameserver ::1
```

### DNS resolution via Samba DNS
```bash
dig @127.0.0.1 ad.afrowave.ltd
```

### SRV records
```bash
dig @127.0.0.1 _ldap._tcp.afrowave.ltd SRV
```
```bash
dig @127.0.0.1 _kerberos._tcp.afrowave.ltd SRV
```

---

# 7. Samba / AD Domain Health
### Check Samba status
```bash
systemctl status samba-ad-dc
```

### Check database consistency
```bash
samba-tool dbcheck --cross-ncs
```

### Check AD services
```bash
samba-tool domain info 127.0.0.1
```

### Check replication (single DC → no partners yet)
Expected: OK

---

# 8. Kerberos Health
```bash
kinit administrator
klist
```

Expected: ticket issued

### Test kpasswd
```bash
kpasswd administrator
```

---

# 9. Time Synchronization (NTP)
Samba provides NTP.

```bash
ntpq -p
```
Expected peers + no errors.

---

# 10. Logs & Errors
### Samba DNS errors
```bash
journalctl -u samba-ad-dc -b | grep -i dns
```

### WireGuard errors
```bash
journalctl -u wg-quick@wg-link -b
journalctl -u wg-quick@wg-internal -b
journalctl -u wg-quick@wg-inet -b
```

### Firewall errors
```bash
journalctl -u firewalld -b
```

---

# 11. Post-Reboot Validation Summary
Everything is **OK** when:
- All WG tunnels show recent handshake
- Routes exist for 10.20.x / 10.22.x / 10.30.x
- DNS resolves correctly
- Kerberos tickets can be issued
- eth0 zone = public (SSH only)
- No unexpected services exposed
- Firewalld NAT active only on aw-inet
- IPv4 Internet connectivity works through WG-Inet

---

# END
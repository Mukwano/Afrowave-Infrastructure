DC1-Live-Checks
========================

# DC1 – Live System Checks
This document contains real-time system commands and outputs that describe the current state of DC1.  
It is used to verify the inventory, detect inconsistencies, and guide configuration repair.

Each section includes:
- Commands to run
- Expected behavior
- Space for collected output (added manually)

---

## 1. Basic System Information

### Commands:
```
hostnamectl
lsb_release -a
timedatectl
uptime
```



### Notes:
Record OS version, kernel, timezone, time sync status, and uptime.

---

## 2. Network Interfaces and IP Addresses

### Commands:
```
ip a
ip r
ip -6 r
```


### Notes:
Verify:
- WAN IP
- WG-LINK IP
- Routing table correctness
- No unwanted default routes

---

## 3. WireGuard Status

### Commands:
```
wg show
wg show wg-link
systemctl status wg-quick@wg-link
```

### Notes:
Check:
- Peers visible and handshake timestamps
- Correct WG-LINK IP
- Endpoint connectivity

---

## 4. DNS / Samba Internal DNS

### Commands:
```
host dc1.afrowave.ltd 127.0.0.1
host -t SRV _kerberos._udp.afrowave.ltd 127.0.0.1
samba-tool dns query localhost afrowave.ltd dc1 A
samba-tool dns zoneinfo localhost afrowave.ltd
```

### Notes:
Confirm:
- A records exist
- SRV records exist
- Internal DNS is authoritative

---

## 5. Samba AD DC Status

### Commands:
```
systemctl status samba-ad-dc
samba-tool dbcheck
samba-tool drs showrepl
```

### Notes:
Expect:
- Service active
- No DB corruption
- No replication (only one DC for now)

---

## 6. Kerberos

### Commands:
```
kinit administrator
klist
```

### Notes:
Check:
- TGT issued correctly
- No clock skew issues
- Realm = AFROWAVE.LTD

---

## 7. NTP / Time Sync

### Commands:
```
timedatectl
chronyc sources
chronyc tracking
ls -ld /var/lib/samba/ntp_signd/
```

### Notes:
Verify:
- Chrony communicating with upstream servers
- Samba ntp_signd socket exists
- Time is synchronized

---

## 8. Firewall (Firewalld)

### Commands:
```
firewall-cmd --get-active-zones
firewall-cmd --list-all --zone=aw-link
firewall-cmd --list-all --zone=public
```

### Notes:
Confirm:
- wg-link interface is in `aw-link`
- Zone target is ACCEPT
- No hidden port filters

---

## 9. System Logs Summary

### Commands:
```
journalctl -u samba-ad-dc -b | tail -n 50
journalctl -u firewalld -b | tail -n 50
journalctl -u chrony -b | tail -n 50
```

### Notes:
Look for:
- DNS update errors
- NTP sync issues
- Firewall reload messages

---

## 10. Outstanding Items for DC1
- Sync time service with correct upstream NTP servers  
- Verify DNS forwarders  
- Add WG-LINK A record (dc1-link.afrowave.ltd)  
- Confirm WG-LINK routing stability  
- Prepare DC2 replication readiness  

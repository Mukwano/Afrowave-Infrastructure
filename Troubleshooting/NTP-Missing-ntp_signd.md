NTP-Missing-ntp_signd
========================

# NTP: Missing ntp_signd Socket
Category: NTP / Samba / Kerberos  
Severity: High  
Affected systems: DC1 (Primary Domain Controller)

---

## 1. Issue Summary
Samba Active Directory requires a signed NTP socket (`ntp_signd`) to provide secure domain time to Windows clients and Kerberos.  
Without it:
- Domain clients may fail to sync time,
- Kerberos tickets may be rejected,
- Domain join operations may fail,
- Time drift errors appear in logs.

The issue occurs when the expected directory or socket for `ntp_signd` is missing, not created, or inaccessible to the NTP daemon.

---

## 2. Symptoms

Observations:
- Running `timedatectl` shows `System clock synchronized: no`.
- Kerberos TGT requests fail intermittently.
- Windows clients report:
  > “The Kerberos client received a KRB_AP_ERR_SKEW error”
- AD DC logs include messages about NTP signing failures.

Common log output:
```
Unable to open /var/lib/samba/ntp_signd/socket: No such file or directory
```

NTP status:
```
chronyc sources
No valid time sources or refused servers
```

## 3. Commands and Evidence

### Checking socket directory:
```
ls -ld /var/lib/samba/ntp_signd/
ls -l /var/lib/samba/ntp_signd/
```

### Checking samba-ad-dc service:
```
systemctl status samba-ad-dc
journalctl -u samba-ad-dc -b | grep -i ntp
```

### Checking NTP daemon:
```
systemctl status chrony
chronyc tracking
```

---

## 4. Root Cause Analysis

Possible causes identified:
1. **Directory missing:**  
   `/var/lib/samba/ntp_signd/` does not exist or was removed.

2. **Permissions incorrect:**  
   Chrony or NTPsec cannot access the directory.

3. **Wrong NTP daemon:**  
   Incorrect daemon installed (e.g., default `systemd-timesyncd` instead of `chrony`).

4. **Chrony not configured** to use Samba's signed socket:
   Required directive missing from `/etc/chrony/chrony.conf`:
```
bindcmdaddress /var/lib/samba/ntp_signd/socket
```

5. **Race condition on startup:**  
Chrony starts before Samba created the socket.

---

## 5. Resolution Steps

### 💡 Step 1 — Ensure the directory exists
```
sudo mkdir -p /var/lib/samba/ntp_signd/
sudo chown root:root /var/lib/samba/ntp_signd/
sudo chmod 755 /var/lib/samba/ntp_signd/
```

### 💡 Step 2 — Check if Samba created the socket
The socket file should appear after Samba starts:
```
/var/lib/samba/ntp_signd/socket
```

### 💡 Step 3 — Configure Chrony for Samba AD
Add to `/etc/chrony/chrony.conf`:
Required for Samba AD signed time
```
ntpsigndsocket /var/lib/samba/ntp_signd/socket
```

Ensure upstream servers exist:
```
pool pool.ntp.org iburst
```

### 💡 Step 4 — Restart services in correct order
```
systemctl restart samba-ad-dc
systemctl restart chrony
```

### 💡 Step 5 — Validate time sync
```
timedatectl
chronyc tracking
chronyc sources
```

Expect:
- `System clock synchronized: yes`
- Chrony actively using upstream NTP sources
- No NTP-related Samba warnings in logs

---

## 6. Validation

### Check socket:
```
ls -l /var/lib/samba/ntp_signd/
```

### Check Samba logs:
```
journalctl -u samba-ad-dc -b | grep -i ntp
```

### Check domain time validity (Kerberos):
```
kinit administrator
klist
```

Kerberos must issue TGT without time skew errors.

---

## 7. Prevention Recommendations

- Document all NTP-related configurations in DC1 inventory.
- Keep `chrony` as the only NTP daemon.
- Ensure no other NTP service (like `systemd-timesyncd`) is active.
- Restart Chrony AFTER Samba during planned maintenance.
- Monitor time drift and NTP source reliability.
- Schedule regular checks using a monitoring script.

---

## 8. Cross-References

- [DC1 Inventory](../DC1-Inventory.md)
- [DC1 Live Checks](../DC1-Live-Checks.md)
- [WG-LINK Network](../Network-WG-Link.md)
- [Main Documentation](../README.md)

---

## 9. Tags
```
#ntp #samba #kerberos #dc1 #timesync #chrony #error
```

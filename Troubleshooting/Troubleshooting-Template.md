Troubleshooting-Template
========================
# Troubleshooting Template
This document provides the standard structure for documenting issues within the Afrowave infrastructure.

Each troubleshooting entry should follow this format, ensuring clear understanding, repeatability, and long-term maintainability.

---

## 1. Issue Summary
**Short title:**  
(e.g., “NTP: Missing ntp_signd socket”)

**Category:**  
- NTP / DNS / Kerberos / Firewall / WireGuard / Samba / System / Networking / Other

**Severity:**  
Low / Medium / High / Critical

**Affected systems:**  
(e.g., DC1, Diblík, Edge, WG-LINK, AD, DNS)

---

## 2. Symptoms
Describe what was observed, including logs or outputs.  
Examples:
- Errors in system logs
- Services failing to start
- Network instability
- DNS lookup failures
- Kerberos tickets not issuing

---

## 3. Commands and Evidence
Record real system output.

```
(command that was used)
(output)
```

Add multiple blocks as needed.

---

## 4. Root Cause Analysis
Explain:
- Why the issue occurred  
- Which subsystem was responsible  
- Whether it was config drift, missing package, misrouting, firewall rule, etc.

---

## 5. Resolution Steps
Clear, actionable, reproducible steps to fix the problem.

Example:

1. Install missing package  
2. Restart affected service  
3. Adjust configuration  
4. Validate the fix  

---

## 6. Validation
Commands used to confirm the issue is resolved.
```
(command)
(expected output)
```

---

## 7. Prevention Recommendations
Describe how to avoid the issue in the future:
- Additional monitoring
- Documentation corrections
- Firewall rule changes
- Synchronization tasks
- Automated scripts
- Procedural notes

---

## 8. Cross-References  
Links to related documentation:

- [DC1 Inventory](../DC1-Inventory.md)  
- [Live Checks](../DC1-Live-Checks.md)  
- [WG-LINK Network](../Network-WG-Link.md)  
- [Main Documentation](../README.md)

Add more links as needed.

---

## 9. Tags
```
#ntp #dns #kerberos #wg #firewall #samba #system #network
```

(Use appropriate tags for filtering inside QOwnNotes.)


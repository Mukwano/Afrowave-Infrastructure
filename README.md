# Afrowave Infrastructure Documentation

Welcome to the **Afrowave Infrastructure** repository — the central, authoritative knowledge base for all Afrowave servers, networking, VPN architecture, domain services, and operational standards.

This repository contains complete documentation of the Afrowave system, including:

* Active Directory Domain Controllers (DC1, DC2)
* WireGuard multi-tier VPN architecture (WG-LINK, WG-INTERNAL, WG-INET, WG-GUEST)
* Firewall architecture and zone definitions
* Server roles and service topology
* Time synchronization (NTP, ntp_signd)
* DNS, Kerberos, and Samba AD configurations
* Troubleshooting procedures and knowledge base
* GitHub sync guide for administrators

All documents follow strict structure and professional standards to ensure long-term maintainability and ease of onboarding for new administrators.

---

## 📘 Repository Structure

```
Afrowave-Infrastructure/
│
├── README.md                        ← You are here
├── DC1-Inventory.md                 ← Full inventory of Domain Controller #1
├── DC1-Live-Checks.md               ← Commands + real-time verification for DC1
├── GitHub-Sync-Guide.md             ← Managing documentation with two GitHub accounts
│
├── Troubleshooting/                 ← Full enterprise troubleshooting knowledge base
│   ├── Troubleshooting-Template.md  ← Standard template for each issue
│   └── <individual issue files>.md  ← Example: NTP-Missing-ntp_signd.md
│
└── (More documents will be added as infrastructure expands)
```

---

## 🌍 Architectural Overview

Afrowave infrastructure is designed around the **"Internet Principle"**:

> *Every server acts as a fully trusted hub on the backbone network (WG-LINK), maximizing redundancy, throughput, reliability, and simplicity of administration.*

Key design goals:

* Full trust + full connectivity between servers via WG-LINK
* Fully documented, reproducible infrastructure
* Enterprise-quality AD, DNS, Kerberos, and NTP configuration
* Multi-tier VPN for employees, internal services, and guests
* Clear separation between server backbone and user networks
* Easy onboarding for new administrators (documentation + scripts)

---

## 🖥️ Current Core Components

### **DC1 – Afrowave Domain Controller #1**

* Debian 13.1
* Samba AD DC
* DNS, Kerberos, LDAP
* WG-LINK backbone endpoint
* Time service (chrony + ntp_signd)

### **Diblík – Home Server**

* Domain member
* Secondary WG-LINK endpoint
* Planned backup WAN route provider

### **EDGE Server (Translate / Web)**

* LibreTranslate
* Reverse proxy
* Application services

### **DC2 (planned)**

* Future secondary domain controller
* Geographic redundancy

---

## 🛡️ Network Architecture

### **WG-LINK (Server Backbone)**

* Full trust network
* No firewall filtering
* No NAT
* All ports allowed
* Used for:

  * AD replication
  * Kerberos
  * DNS
  * Backend service communication
  * Cross-server routing and monitoring

### **WG-INTERNAL**

* Internal employee VPN
* No NAT
* Strict access control

### **WG-INET**

* VPN with internet access
* NAT enabled

### **WG-GUEST**

* Internet-only VPN
* No internal access
* NAT enabled

---

## 🧰 Troubleshooting Knowledge Base

All troubleshooting follows a professional, structured template:

* Symptoms
* Evidence (commands + outputs)
* Root cause
* Resolution steps
* Validation
* Prevention
* Cross-references

Example entry:

* `Troubleshooting/NTP-Missing-ntp_signd.md`

---

## 🔄 GitHub Sync Workflow

Documents are maintained using two GitHub accounts:

* **afrowaveltd** (personal)
* **mukwano** (community)

SSH configuration enables automatic account switching using:

```
github.com-afrowave
github.com-mukwano
```

Full guide: `GitHub-Sync-Guide.md`

---

## 🧭 Goals of This Repository

* Provide a complete, version-controlled source of truth
* Support transparency and teamwork in the Afrowave community
* Preserve knowledge for future administrators and contributors
* Reduce errors and configuration drift
* Maintain professional-grade infrastructure documentation

---

## 🧩 Contributing

Contributions are welcome from:

* Administrators
* Developers
* Community collaborators

Please follow existing document structures and templates.

---

## 📄 License

This repository is licensed under the **MIT License**, allowing free use, modification, and distribution.

---

## 💛 Maintained by

**Afrowave Community**
[https://afrowave.ltd](https://afrowave.ltd)

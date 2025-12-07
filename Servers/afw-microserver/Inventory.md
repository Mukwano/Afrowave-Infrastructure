Afrowave Microserver – Infrastructure Inventory (Draft 2025)
========================

**Server Name:** `afw-microserver`  
**Friendly Name:** "Diblík"  
**Role (planned):** On‑prem training & development host, secondary AD DC, primary Internet egress for WG‑Inet  
**Status:** Active (hardware online, services in planning / early deployment)

---

## 1. Hardware Overview

| Parameter | Value |
|----------|-------|
| Platform | Mini PC / Ryzen 7 host |
| CPU | AMD Ryzen 7 (8 cores / 16 threads, base 3.5 GHz) |
| RAM | 16 GB DDR5 (expandable up to 64 GB) |
| Primary Storage | 512 GB NVMe SSD |
| Secondary Storage | 2 TB USB disk (planned for `/home` / data) |
| NIC | 1× Ethernet (home LAN uplink) |

---

## 2. Planned Operating System & Hypervisor

> Exact distribution / stack can be confirmed later; this section captures the intended direction.

- **Host OS:** Linux, Debian‑based (e.g. Debian 12/13)  
- **Hypervisor:** KVM/QEMU with libvirt *(or Proxmox VE on top, TBD)*  
- **Firewall on host:** firewalld or nftables‑based rules (aligned with DC1 conventions)  
- **Domain Membership:** Will be joined to `afrowave.ltd` and later promoted as **additional Samba AD DC**.

---

## 3. High‑Level Roles

1. **Virtualization Host for the Classroom / Training Lab**  
   - Runs at least two primary VMs for students/contributors.  
2. **Local Internet Egress for WG‑Inet**  
   - Acts as the main Internet exit for Afrowave WireGuard clients (unmetered home line).  
   - DC1 (VPS) remains the central control plane and routing hub.  
3. **Secondary Domain Controller (planned)**  
   - Provides redundancy for AD, DNS and Kerberos for `afrowave.ltd`.  
4. **Multi‑user Linux Development Environment**  
   - Shared VM for concurrent users working on Afrowave codebases.

---

## 4. Planned Virtual Machines

### 4.1 VM1 – Windows 11 Development Workstation

- **Name (proposed):** `afw-devwin01`  
- **OS:** Windows 11 Pro / Enterprise, domain‑joined to `afrowave.ltd`  
- **Usage Model:**
  - Single‑user at any given time (no concurrent interactive sessions expected).  
  - Reserved for cases where a Linux alternative is not possible or practical.  
- **Primary Purpose:**
  - .NET / Visual Studio development  
  - Windows‑specific tools and testing  
  - RSAT, management tools, GUI‑only applications
- **Resource Sizing (initial proposal):**
  - vCPU: 4 vCPUs  
  - RAM: 8 GB  
  - Disk: 128–160 GB virtual disk on NVMe  

---

### 4.2 VM2 – Debian Multi‑User Development Server

- **Name (proposed):** `afw-devlin01`  
- **OS:** Debian (stable)  
- **Usage Model:**
  - Multi‑user, concurrent SSH / remote desktop sessions.  
  - Shared environment for contributors and students.  
- **Primary Purpose:**
  - Git, CI helpers, build tools  
  - Python, .NET SDK, Node.js and other Afrowave tooling  
  - Editors and remote IDEs (e.g. VS Code Server, SSH‑based workflows)
- **Resource Sizing (initial proposal):**
  - vCPU: 4–6 vCPUs  
  - RAM: 8–12 GB (depending on host upgrades)  
  - Disk: ~200 GB virtual disk (optionally using the 2 TB USB as backing storage)

---

## 5. Storage Layout (Planned)

### 5.1 Host Level

- **NVMe 512 GB**
  - `/` (root + host OS)  
  - `/var/lib/libvirt` or hypervisor VM storage  
  - Critical host tooling

- **USB 2 TB**
  - Planned as data / home storage:  
    - `/home` for local shell users  
    - Optional `/srv/vmdata` for large VM disks / lab environments  
    - Backup targets / snapshots

> Exact partitioning scheme and filesystem choice (e.g. ext4, xfs, zfs) are still to be decided and documented.

---

## 6. Network & WireGuard Integration (Concept)

### 6.1 Home Network Role

- `afw-microserver` is located on the home LAN with **unmetered Internet**, making it ideal for student workloads and heavy development traffic.
- It will serve as the **primary Internet egress** for VPN clients using **WG‑Inet**, reducing data consumption on VPS‑based servers.

### 6.2 WireGuard Participation (Planned)

`afw-microserver` will be a peer in the Afrowave WG topology coordinated by DC1. In addition to its routing and DC roles, it is designated as the **temporary primary hub for the WG‑Learn network**.

#### **wg-link**  
- Purpose: Secure backbone tunnel between DC1 and `afw-microserver`.  
- Addressing: microserver receives an IP in `10.30.0.0/24` (TBD).

#### **wg-internal**  
- Purpose: Internal AD, DNS, Kerberos, and management traffic.  
- Addressing: microserver receives an IP in `10.20.0.0/16` (TBD).

#### **wg-inet**  
- Purpose: Primary VPN Internet exit for clients.  
- Traffic path: client → DC1 (wg-inet) → `afw-microserver` (via wg-link) → home ISP.  
- NAT handled locally on `afw-microserver`.

#### **wg-learn** (Primary classroom hub)
- Purpose: Acts as the **main hub for student machines**, virtual lab stations, and multi-user development nodes.  
- Temporary central role (expected to move to a future dedicated Classroom/EDGE server).  
- Addressing: Within `10.24.0.0/16` (exact assignment TBD).

> The microserver's role as the hub for WG‑Learn is intentionally flexible: it provides full classroom capabilities today, while allowing seamless migration to a future separated "Afrowave Classroom Server" when needed.

---

## 7. Security & Domain Integration (Planned)

- `afw-microserver` will:
  - Join `afrowave.ltd` as a domain member.  
  - Later be promoted to a **secondary Samba AD DC**.  
  - Host a local copy of DNS/LDAP/Kerberos for resilience when the VPS is unreachable.  
- Admin access will be restricted to Afrowave admin groups (e.g. `AFW_Role_GlobalAdmin`, `AFW_Role_OrgAdmin`, etc.).

---

## 8. Relationship to DC1 (afw-dc1)

- DC1 remains the **authoritative AD DC and central routing hub**.  
- `afw-microserver` acts as:
  - On‑prem extension for classroom and training VMs.  
  - Resilient AD/DNS/Kerberos replica.  
  - Primary Internet exit for WG‑Inet traffic.  
- All WireGuard design and IPAM rules stay **centrally defined** on DC1 (via Afrowave.WgAddressDispatcher concept), with `afw-microserver` treated as a managed node.

---

## 9. Open Items / To‑Do

- [ ] Decide on exact host OS (Debian version, or Proxmox vs. plain KVM).  
- [ ] Define final hypervisor layout and VM naming convention.  
- [ ] Design and document partitioning and filesystem layout for NVMe + 2 TB USB.  
- [ ] Assign concrete IP addresses for `afw-microserver` on `wg-link`, `wg-internal` and `wg-inet`.  
- [ ] Join host to `afrowave.ltd` and later promote to AD DC.  
- [ ] Configure NAT and firewall rules for WG‑Inet egress via home Internet.  
- [ ] Install and configure tooling inside VM1 (Win 11) and VM2 (Debian dev) according to Afrowave developer standards.  
- [ ] Integrate with the future **Afrowave.WgAddressDispatcher** (IPAM) service.

---

# END OF FILE


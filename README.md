# 🖥️ BW Enterprise Lab — R320

> Professional Enterprise Homelab on Dell PowerEdge R320  
> **Blue Team · Red Team · CompTIA · CCNA · Azure · AWS**

---

## 📌 About

This repository documents the design, build, and evolution of a professional homelab built on a **Dell PowerEdge R320** running **Proxmox VE**.

The goal is to replicate a real enterprise environment — firewall, NAS, backups, SIEM, Active Directory, monitoring, and security labs — while generating hands-on experience aligned with:

- 🔐 **CompTIA** A+ · Network+ · Security+
- 🌐 **Cisco CCNA**
- ☁️ **Azure** · **AWS**
- 🛡️ **Blue Team** (Wazuh SIEM, IDS/IPS, AD hardening)
- ⚔️ **Red Team** (Kali Purple, Parrot Security, Cowrie honeypot)

Every decision is documented. Every failure is documented too.

---

## 🔧 Hardware

| Component | Detail |
|---|---|
| Server | Dell PowerEdge R320 |
| CPU | Intel Xeon E5-2450 @ 2.10 GHz |
| RAM | 32 GB ECC (upgrade to 64 GB planned) |
| RAID Controller | PERC H710 |
| NIC 1 (eno1) | WAN → Movistar router |
| NIC 2 (eno2) | LAN → Cisco managed switch |
| Disk sda | ~837 GB — Proxmox VE + ISOs |
| Disk sdb | ~1.6 TB — TrueNAS pool `bw_pool` |
| Disk sdc | ~837 GB — VM disks (LVM-Thin) |
| Wi-Fi | 2 × Cisco Catalyst 9120AXI-E (one runs the Embedded Wireless Controller, EWC) |

---

## 🗺️ Network Architecture

```
Internet
   │
Router Movistar (192.168.1.1)
   │
   ├── eno1 (WAN) ──── Dell R320 / Proxmox (192.168.1.100)
   │                         │
   │                     eno2 (LAN)
   │                         │
   └──────────── Cisco Switch ──── Lab VMs, Wi-Fi APs, admin PC (10.10.10.0/24)
```

| Network | Subnet | Role |
|---|---|---|
| WAN | `192.168.1.0/24` | Movistar router → pfSense WAN |
| LAN Lab | `10.10.10.0/24` | All lab VMs, APs and admin PC |
| Proxmox secondary | `10.10.10.254` | Proxmox ↔ VM communication |

### Remote access

Remote access to the lab goes through **Tailscale**, running in the LXC container `bw-tailscale` (CT 105, `10.10.10.5`). It lives inside the R320: with the server off there is no remote access.

---

## 🖧 VM Inventory

| ID | Name | IP | Role | Status |
|---|---|---|---|---|
| 110 | bw-pfsense | `10.10.10.1` | Firewall, NAT, DHCP, DNS | ✅ Operational |
| 105 | bw-tailscale (LXC) | `10.10.10.5` | Remote access (Tailscale) | ✅ Operational |
| 120 | bw-truenas | `10.10.10.120` | NAS, SMB, storage | ✅ Operational |
| 130 | bw-pbs | `10.10.10.130` | Proxmox Backup Server | ✅ Operational |
| 100 | bw-infra-debian | `10.10.10.100` | Docker, Portainer, Ollama, Open WebUI (OpenVAS, Cowrie pending) | ✅ Operational |
| 150 | bw-winserver | `10.10.10.150` | Windows Server 2025, AD (`bw.lab`), DNS, GPO | ✅ Operational |
| 160 | bw-win11 | `10.10.10.160` | Windows 11 domain client, Packet Tracer | 🔄 Installed — network and domain join pending |
| 700 | linux-mint | `10.10.10.190` | Distro test VM | ✅ Installed |
| 140 | bw-wazuh | `10.10.10.140` | Wazuh SIEM (Manager + Indexer + Dashboard) | 🔄 Pending (RAM) |
| 181 | bw-kali-purple | `10.10.10.181` | Red team, Wireshark | 🔄 Pending |
| 170 | bw-gns3 | `10.10.10.170` | GNS3 Server — CCNA topologies | 🔄 Pending |
| 172 | bw-parrot | `10.10.10.172` | Parrot Security — red team complement | 🔄 Pending |

---

## ⚙️ Services

| Service | VM | Address | Status |
|---|---|---|---|
| Proxmox WebUI | dell-r320-lab | `https://192.168.1.100:8006` | ✅ |
| pfSense WebUI | bw-pfsense | `https://10.10.10.1` | ✅ |
| TrueNAS WebUI | bw-truenas | `http://10.10.10.120` | ✅ |
| SMB Share | bw-truenas | `\\10.10.10.120\bw-nas` | ✅ |
| PBS WebUI | bw-pbs | `https://10.10.10.130:8007` | ✅ |
| Portainer | bw-infra-debian | `https://10.10.10.100:9443` | ✅ |
| Open WebUI (Ollama) | bw-infra-debian | `http://10.10.10.100:3000` | on demand |
| Active Directory / DNS | bw-winserver | `10.10.10.150` (domain `bw.lab`) | ✅ |
| Cisco EWC WebUI | AP 9120AXI-E (not a VM) | `https://10.10.10.240` | ✅ |
| Wazuh Dashboard | bw-wazuh | `https://10.10.10.140` | 🔄 Pending |

---

## 🔁 Auto-start order

After a power-on, Proxmox starts these guests by itself, in this order:

| Order | ID | Name | Wait before next |
|---|---|---|---|
| 1 | VM 110 | bw-pfsense | 60 s |
| 2 | CT 105 | bw-tailscale | 15 s |
| 3 | VM 120 | bw-truenas | 60 s |
| 4 | VM 130 | bw-pbs | 30 s |
| 5 | VM 100 | bw-infra-debian | — |

`bw-winserver`, `bw-win11` and `linux-mint` are started manually (RAM limit). Without `bw-winserver` running, the `bw.lab` domain does not resolve.

---

## 💾 Backup Strategy

Managed by **Proxmox Backup Server 4.2.1** (`bw-pbs`, VMID 130).

| Item | Detail |
|---|---|
| Schedule | Weekly, Sundays 01:00 |
| Guests in the job | 100 (bw-infra-debian), 110 (bw-pfsense), 120 (bw-truenas). bw-pbs (130) is not in the job |
| Retention | keep last 2 + daily 7 |
| Not yet in the job | bw-winserver (150), bw-win11 (160), bw-tailscale (105) |
| Outside PBS | Cisco EWC (not a VM) — local config copy in the AP's bootflash |

> **Note:** bw-pbs is not backed up as a whole VM — its disk holds the datastore, so it would mean backing up backups inside backups.  
> Recovery procedure documented in [`docs/pbs-config.md`](docs/pbs-config.md).

---

## 📁 Repository Structure

```
bw-enterprise-lab-r320/
├── README.md                    ← You are here
├── LICENSE
├── docs/
│   ├── 01-INFRAESTRUCTURA.md    ← Hardware, hypervisor, disk layout, auto-start
│   ├── 02-RED-Y-FIREWALL.md     ← Network topology, pfSense config, remote access
│   ├── 03-MAQUINAS-VIRTUALES.md ← Full VM specs and notes
│   ├── 04-SERVICIOS.md          ← Services, ports, access
│   ├── 05-BACKUP-Y-MONITORIZACION.md ← Backup policy, Wazuh plan
│   ├── 06-INVENTARIO-Y-CHANGELOG.md  ← Hardware/software inventory + changelog
│   └── pbs-config.md            ← PBS recovery reference
├── scripts/                     ← Automation and setup scripts (planned)
└── diagrams/                    ← Network diagrams (planned)
```

---

## 📺 LinkedIn Series — BlancoWhite Lab

This build is documented publicly as a LinkedIn episode series.

| # | Title | Technical milestone |
|---|---|---|
| T1E01 | El Servidor Olvidado | R320 diagnosis. Decision to reinstall from scratch. |
| T1E02 | El nacimiento del laboratorio | Clean Proxmox VE install. First hypervisor boot. |
| T1E03 | El fallo que nadie esperaba | Boot screen trap + DNS resolution failure and fix. |
| T1E04 | Tres discos, tres decisiones | Storage architecture design. |
| T1E05 | *(upcoming)* | pfSense — full config and network migration. |
| T1E06 | *(upcoming)* | PBS — scheduled backups for all VMs. |
| T1E07 | *(upcoming)* | TrueNAS — NAS and SMB operational. |
| T1E08 | *(upcoming)* | bw-infra-debian — Docker, Portainer, Ollama, Open WebUI. |
| T1E09 | *(upcoming)* | Wazuh — SIEM install. Agents deployed across all VMs. |
| T1E10 | *(upcoming)* | Windows Server 2025 — AD, DNS, GPO. |
| T1E11 | *(upcoming)* | bw-win11 — domain client joined and validated. |
| Special | Certificación obtenida | Cisco *Introduction to Cybersecurity* — theory + practice. |

---

## 🧑‍💻 Author

**Emilio Durán — Blanco White Tech**  
Technical Support N4 · Cybersecurity · Infrastructure  
[LinkedIn](https://linkedin.com/in/fj-duran)

---

> *"In technology, you often learn more by fixing a failure than by following a manual."*

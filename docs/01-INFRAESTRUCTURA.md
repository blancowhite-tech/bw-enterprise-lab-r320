# BlancoWhite Lab — 01 Infraestructura

> Documento actualizado 2026-09-02.

---

## Hardware

| Campo | Valor |
|---|---|
| Servidor | Dell PowerEdge R320 |
| Controladora | PERC H710 |
| CPU | Intel Xeon E5-2450 @ 2.10 GHz |
| RAM | ECC (capacidad total por confirmar en reinstalación) |

---

## Hipervisor

| Campo | Valor |
|---|---|
| Software | Proxmox VE (última versión estable al instalar) |
| Hostname | `dell-r320-lab` |
| IP de gestión | `192.168.1.100/24` |
| IP secundaria (LAN lab) | `10.10.10.254/24` (post-up en vmbr0) |
| Gateway | `192.168.1.1` |
| DNS | `192.168.1.1` |

---

## Discos físicos

| Disco | Tamaño | Uso |
|---|---|---|
| sda | ~837 GB | Sistema Proxmox + ISOs |
| sdb | ~1.6 TB | Datos TrueNAS (NAS pool bw_pool) |
| sdc | ~837 GB | Discos de VMs |

> **Importante:** sdb contiene el pool TrueNAS `bw_pool`. No formatear.

---

## Storages Proxmox

| Nombre | Disco | Tipo | Uso |
|---|---|---|---|
| `local` | sda | dir | ISOs y backups pequeños |
| `local-lvm` | sda | LVM-thin | Discos de VMs en sda |
| `vm-lab-blancowhite` | sdc | LVM-thin | Discos principales de VMs |
| `pbs` | — | PBS | Proxmox Backup Server en `10.10.10.130` |

---

## Máquinas virtuales

| VMID | Nombre | IP | Función | Estado |
|---|---|---|---|---|
| 110 | bw-pfsense | `10.10.10.1` (LAN) / WAN DHCP | Firewall, NAT, DHCP, DNS, Suricata | ✅ Operativa |
| 120 | bw-truenas | `10.10.10.120` | NAS, SMB, almacenamiento | ✅ Operativa |
| 130 | bw-pbs | `10.10.10.130` | Proxmox Backup Server | ✅ Operativa |
| 100 | bw-infra-debian | `10.10.10.100` | Docker, Portainer, OpenVAS, Cowrie, AWS CLI | ✅ Operativa |
| 140 | bw-wazuh | `10.10.10.140` | SIEM, monitorización | Pendiente |
| 150 | bw-winserver | `10.10.10.150` | Windows Server 2025, AD, DNS, GPO | Pendiente |
| 160 | bw-win11 | `10.10.10.160` | Cliente Windows, unido al dominio, Packet Tracer | Pendiente |
| 181 | bw-kali-purple | `10.10.10.181` | Red team, Wireshark | Pendiente |
| 170 | bw-gns3 | `10.10.10.170` | GNS3 Server, CCNA | Pendiente |
| 172 | bw-parrot | `10.10.10.172` | Red team complementario | Pendiente |

---

## Reglas operativas

1. Revisar el estado real antes de cambiar.
2. Un cambio cada vez.
3. Validar el resultado antes de continuar.
4. Documentar únicamente lo confirmado.
5. No modificar servicios operativos sin razón concreta.
6. Hacer backup antes de tocar cualquier VM en producción.
7. Cada nueva VM se documenta antes de pasar a la siguiente.

---

## Changelog

| Fecha | Acción |
|---|---|
| 2026-08-28 | Decisión de reinstalación limpia de Proxmox. |
| 2026-08-28 | eno1 conectado al router Movistar (WAN). eno2 al switch (LAN). |
| 2026-08-28 | Instalación de Proxmox VE limpia en sda. |
| 2026-09-01 | Migración de red LAN de `192.168.1.0/24` a `10.10.10.0/24`. Causa: WAN y LAN de pfSense en la misma subred — pfSense no podía enrutar. |
| 2026-09-01 | IP secundaria `10.10.10.254/24` añadida a vmbr0 en Proxmox (post-up permanente). |
| 2026-09-01 | Docker 29.7.2 y Portainer CE instalados en bw-infra-debian. |
| 2026-09-01 | Infraestructura final definida: 9 VMs (bw-winserver VMID 150 y bw-win11 VMID 160 añadidas al plan). |
| 2026-09-02 | Infraestructura revisada. bw-omarchy eliminada (redundante). bw-gns3 (VMID 170) y bw-parrot (VMID 172) añadidas. |
| 2026-09-02 | bw-infra-debian RAM ampliada a 8 GB. Ollama y Open WebUI instalados (inactivos por RAM). |
| 2026-09-02 | Objetivo del lab ampliado: Blue Team + Red Team · CompTIA A+, Network+, Security+ · CCNA · Azure · AWS. |

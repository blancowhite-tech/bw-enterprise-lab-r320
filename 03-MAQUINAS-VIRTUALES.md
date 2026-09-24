# BlancoWhite Lab — 03 Máquinas Virtuales

> Documento actualizado 2026-09-02.
> Las VMs se instalan en el orden del plan. Cada una se documenta y valida antes de continuar con la siguiente.

---

## Resumen general

| VMID | Nombre | IP | Rol | Estado | Prioridad |
|---|---|---|---|---|---|
| 110 | bw-pfsense | `10.10.10.1` | Firewall, NAT, DHCP, DNS, Suricata | ✅ Operativa | 1 |
| 130 | bw-pbs | `10.10.10.130` | Backups | ✅ Operativa | 2 |
| 120 | bw-truenas | `10.10.10.120` | NAS, SMB, almacenamiento | ✅ Operativa | 3 |
| 100 | bw-infra-debian | `10.10.10.100` | Docker, Portainer, OpenVAS, Cowrie, AWS CLI | ✅ Operativa | 4 |
| 140 | bw-wazuh | `10.10.10.140` | SIEM, monitorización | Pendiente | 5 |
| 150 | bw-winserver | `10.10.10.150` | Windows Server 2025, AD, DNS, GPO | Pendiente | 6 |
| 160 | bw-win11 | `10.10.10.160` | Cliente Windows, unido al dominio, Packet Tracer | Pendiente | 7 |
| 181 | bw-kali-purple | `10.10.10.181` | Red team, Wireshark | Pendiente | 8 |
| 170 | bw-gns3 | `10.10.10.170` | GNS3 Server, CCNA | Pendiente | 9 |
| 172 | bw-parrot | `10.10.10.172` | Red team complementario | Pendiente | 10 |

---

## VMID 110 — bw-pfsense

| Campo | Valor |
|---|---|
| Nombre | `bw-pfsense` |
| IP LAN | `10.10.10.1/24` |
| IP WAN | DHCP desde router Movistar |
| RAM | 4 GB |
| CPU | 2 vCPU |
| Disco | 32 GB |
| Machine | Q35 |
| Firmware | OVMF / UEFI |
| net0 (LAN) | VirtIO → `vmbr0` |
| net1 (WAN) | VirtIO → `vmbr1` |
| Storage | `vm-lab-blancowhite` |
| ISO | `netgate-installer-v1.2-RELEASE-amd64.iso` |
| Estado | ✅ Operativa |

### Notas
- LAN migrada de `192.168.1.110` a `10.10.10.1` el 2026-09-01.
- NAT Outbound en modo Hybrid para `10.10.10.0/24`.
- DHCP LAN activo: rango `10.10.10.200-10.10.10.250`.

---

## VMID 130 — bw-pbs

| Campo | Valor |
|---|---|
| Nombre | `bw-pbs` |
| IP | `10.10.10.130/24` |
| Gateway | `10.10.10.1` |
| DNS | `1.1.1.1` / `8.8.8.8` |
| RAM | 4 GB |
| CPU | 2 vCPU |
| Disco sistema | ~18.5 GB |
| Machine | Q35 |
| Firmware | OVMF / UEFI |
| net0 | VirtIO → `vmbr0` |
| OS | Proxmox Backup Server 4.2.0 |
| Datastore | `bw-backup` |
| Estado | ✅ Operativa |

### Notas
- IP migrada de `192.168.1.130` a `10.10.10.130` el 2026-09-01.
- Storage `pbs` actualizado en Proxmox con nueva IP.
- Backup de bw-pbs completado el 2026-09-01 sin errores.

---

## VMID 120 — bw-truenas

| Campo | Valor |
|---|---|
| Nombre | `bw-truenas` |
| IP | `10.10.10.120/24` |
| Gateway | `10.10.10.1` |
| DNS | `1.1.1.1` / `8.8.8.8` |
| RAM | 4 GB |
| CPU | 4 vCPU, type: host |
| Disco sistema | 20 GB (scsi0 → `local-lvm`) |
| Disco datos | 1.6 TB (virtio1 → `/dev/sdb` passthrough) |
| Machine | Q35 |
| Firmware | SeaBIOS |
| net0 | VirtIO → `vmbr0` |
| OS | TrueNAS SCALE 25.10.6 Goldeye |
| Usuario admin | `truenas_admin` |
| Estado | ✅ Operativa |

### Configuración aplicada
- Pool `bw_pool` en `vda` (1.57 TiB disponibles)
- Datasets: `bw-nas` y `bw-backup`, ambos con preset SMB
- SMB activo, usuario `blancowhite`
- IP migrada de `192.168.1.120` a `10.10.10.120` el 2026-09-01

---

## VMID 100 — bw-infra-debian

| Campo | Valor |
|---|---|
| Nombre | `bw-infra-debian` |
| IP | `10.10.10.100/24` |
| Gateway | `10.10.10.1` |
| DNS | `1.1.1.1` / `8.8.8.8` |
| RAM | 8 GB |
| CPU | 2 vCPU, type: host |
| Disco | 40 GB |
| Machine | Q35 |
| Firmware | OVMF / UEFI |
| net0 | VirtIO → `vmbr0` |
| Storage | `vm-lab-blancowhite` |
| OS | Debian 13 |
| Usuario | `blancowhite` |
| Estado | ✅ Operativa |

### Servicios instalados
- Docker 29.7.2
- Portainer CE (última estable) — `https://10.10.10.100:9443`
- Ollama — instalado, desactivado (pendiente ampliación RAM)
- Open WebUI — instalado, bajo demanda — `http://10.10.10.100:3000`
- llama3.2 — descargado, inactivo
- QEMU Guest Agent
- SSH
- GitHub repo clonado en `/root/bw-enterprise-lab-r320`

### Servicios pendientes
- OpenVAS (Greenbone) — pendiente RAM
- Cowrie honeypot
- AWS CLI

---

## VMID 140 — bw-wazuh

| Campo | Valor |
|---|---|
| Nombre | `bw-wazuh` |
| IP | `10.10.10.140/24` |
| Gateway | `10.10.10.1` |
| DNS | `1.1.1.1` / `8.8.8.8` |
| RAM | 8 GB |
| CPU | 4 vCPU |
| Disco | 100 GB |
| Machine | Q35 |
| Firmware | OVMF / UEFI |
| net0 | VirtIO → `vmbr0` |
| Storage | `vm-lab-blancowhite` |
| OS | Ubuntu 22.04 LTS |
| Estado | Pendiente |

---

## VMID 150 — bw-winserver

| Campo | Valor |
|---|---|
| Nombre | `bw-winserver` |
| IP | `10.10.10.150/24` |
| Gateway | `10.10.10.1` |
| DNS | `10.10.10.150` (propio) / `1.1.1.1` (secundario) |
| RAM | 8 GB |
| CPU | 4 vCPU |
| Disco | 80 GB |
| Machine | Q35 |
| Firmware | OVMF / UEFI |
| net0 | VirtIO → `vmbr0` |
| Storage | `vm-lab-blancowhite` |
| OS | Windows Server 2025 |
| Rol | Active Directory, DNS, GPO |
| Estado | Pendiente |

### Notas de planificación
- Será el Domain Controller principal del lab.
- Dominio a definir (e.g. `bw.lab`).
- pfSense DNS Resolver deberá reenviar queries del dominio AD a `10.10.10.150`.
- bw-win11 se unirá a este dominio.

---

## VMID 160 — bw-win11

| Campo | Valor |
|---|---|
| Nombre | `bw-win11` |
| IP | `10.10.10.160/24` |
| Gateway | `10.10.10.1` |
| DNS | `10.10.10.150` (AD DC) |
| RAM | 4 GB |
| CPU | 2 vCPU |
| Disco | 60 GB |
| Machine | Q35 |
| Firmware | OVMF / UEFI |
| net0 | VirtIO → `vmbr0` |
| Storage | `vm-lab-blancowhite` |
| OS | Windows 11 |
| Estado | Pendiente |

### Notas de planificación
- Se instala después de bw-winserver y AD operativo.
- Se unirá al dominio definido en bw-winserver.
- Objetivo: practicar GPO, gestión de usuarios AD, integración con Wazuh.

---

## VMID 181 — bw-kali-purple

| Campo | Valor |
|---|---|
| Nombre | `bw-kali-purple` |
| IP | `10.10.10.181/24` |
| Gateway | `10.10.10.1` |
| RAM | 8 GB |
| CPU | 4 vCPU |
| Disco | 60 GB |
| Machine | Q35 |
| Firmware | OVMF / UEFI |
| net0 | VirtIO → `vmbr0` |
| Storage | `vm-lab-blancowhite` |
| ISO | `kali-linux-2026.2-installer-purple-amd64.iso` |
| Rol | Red team, Wireshark |
| Estado | Pendiente |

---

## VMID 170 — bw-gns3

| Campo | Valor |
|---|---|
| Nombre | `bw-gns3` |
| IP | `10.10.10.170/24` |
| Gateway | `10.10.10.1` |
| DNS | `1.1.1.1` / `8.8.8.8` |
| RAM | 4 GB |
| CPU | 4 vCPU |
| Disco | 40 GB |
| Machine | Q35 |
| Firmware | OVMF / UEFI |
| net0 | VirtIO → `vmbr0` |
| Storage | `vm-lab-blancowhite` |
| Rol | GNS3 Server — topologías CCNA, OSPF, EIGRP, ACLs |
| Estado | Pendiente |

---

## VMID 172 — bw-parrot

| Campo | Valor |
|---|---|
| Nombre | `bw-parrot` |
| IP | `10.10.10.172/24` |
| Gateway | `10.10.10.1` |
| RAM | 4 GB |
| CPU | 2 vCPU |
| Disco | 40 GB |
| Machine | Q35 |
| Firmware | OVMF / UEFI |
| net0 | VirtIO → `vmbr0` |
| Storage | `vm-lab-blancowhite` |
| ISO | `Parrot-security-7.3.iso` |
| Rol | Red team complementario |
| Estado | Pendiente |

---

## Changelog

| Fecha | VM | Acción |
|---|---|---|
| 2026-08-28 | Todas | Decisión de reinstalación limpia. VMs anteriores eliminadas. |
| 2026-08-30 | bw-pfsense | Instalada y validada con WAN y LAN operativos. |
| 2026-08-30 | bw-pbs | Instalada. Datastore `bw-backup` operativo. Backup de bw-pfsense completado. |
| 2026-08-30 | bw-truenas | Instalada. Pool `bw_pool` creado. SMB activo. |
| 2026-09-01 | bw-pfsense | LAN migrada a `10.10.10.1`. NAT Hybrid configurado. |
| 2026-09-01 | bw-infra-debian | IP migrada a `10.10.10.100`. Docker y Portainer instalados. |
| 2026-09-01 | bw-pbs | IP migrada a `10.10.10.130`. Storage PBS actualizado en Proxmox. |
| 2026-09-01 | bw-truenas | IP migrada a `10.10.10.120`. |
| 2026-09-01 | bw-winserver | VMID 150, IP `10.10.10.150` añadidos al plan de infraestructura. |
| 2026-09-01 | bw-win11 | VMID 160, IP `10.10.10.160` añadidos al plan de infraestructura. |
| 2026-09-02 | bw-infra-debian | RAM ampliada de 2 GB a 8 GB. Ollama y Open WebUI instalados (inactivos). llama3.2 descargado. |
| 2026-09-02 | bw-omarchy | Eliminada del plan — redundante con bw-infra-debian y bw-win11. |
| 2026-09-02 | bw-gns3 | VMID 170, IP `10.10.10.170` añadidos al plan. GNS3 Server para CCNA. |
| 2026-09-02 | bw-parrot | VMID 172, IP `10.10.10.172` añadidos al plan. Red team complementario. |

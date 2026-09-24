# BlancoWhite Lab — 06 Inventario y Changelog

> Documento actualizado 2026-09-02.
> Registro completo del proyecto BW Enterprise Lab.

---

## Sobre el proyecto

**BW Enterprise Lab** es un proyecto de homelab enterprise construido sobre un Dell PowerEdge R320 con Proxmox VE. El objetivo es montar un entorno que simule una red empresarial real: firewall, NAS, backups, SIEM, Active Directory, monitorización y laboratorio de seguridad.

El proyecto se documenta públicamente en LinkedIn como una serie por temporadas.

> *"Todo gran laboratorio empieza con una decisión."*

---

## Inventario hardware

| Componente | Detalle |
|---|---|
| Servidor | Dell PowerEdge R320 |
| Controladora RAID | PERC H710 |
| CPU | Intel Xeon E5-2450 @ 2.10 GHz |
| NIC 1 | eno1 — WAN (router Movistar) |
| NIC 2 | eno2 — LAN (switch Cisco) |
| Switch | Cisco gestionable |
| Router | Movistar |
| Disco sistema | sda ~837 GB — Proxmox VE |
| Disco NAS | sdb ~1.6 TB — TrueNAS pool bw_pool |
| Disco VMs | sdc ~837 GB — discos de VMs |

---

## Inventario software

| Software | Versión | VM | Rol | Estado |
|---|---|---|---|---|
| Proxmox VE | última estable | dell-r320-lab | Hipervisor | ✅ Operativo |
| pfSense | netgate-installer-v1.2-RELEASE | bw-pfsense (110) | Firewall, NAT, DHCP, DNS | ✅ Operativo |
| TrueNAS SCALE | 25.10.6 Goldeye | bw-truenas (120) | NAS, SMB | ✅ Operativo |
| Proxmox Backup Server | 4.2.0 | bw-pbs (130) | Backups | ✅ Operativo |
| Docker | 29.7.2 | bw-infra-debian (100) | Contenedores | ✅ Operativo |
| Portainer CE | última estable | bw-infra-debian (100) | Gestión Docker | ✅ Operativo |
| Ollama | — | bw-infra-debian (100) | LLM local | Instalado, inactivo |
| Open WebUI | — | bw-infra-debian (100) | Interfaz Ollama | Instalado, bajo demanda |
| Wazuh | 4.x | bw-wazuh (140) | SIEM | Pendiente |
| Windows Server 2025 | — | bw-winserver (150) | AD, DNS, GPO | Pendiente |
| Windows 11 | — | bw-win11 (160) | Cliente de dominio | Pendiente |
| Kali Purple | 2026.2 | bw-kali-purple (181) | Red team, Wireshark | Pendiente |
| GNS3 | — | bw-gns3 (170) | Topologías CCNA | Pendiente |
| Parrot Security | 7.3 | bw-parrot (172) | Red team complementario | Pendiente |

---

## Inventario VMs

| VMID | Nombre | IP | Rol | Estado |
|---|---|---|---|---|
| 110 | bw-pfsense | `10.10.10.1` | Firewall, NAT, DHCP, DNS, Suricata | ✅ Operativa |
| 120 | bw-truenas | `10.10.10.120` | NAS, SMB, almacenamiento | ✅ Operativa |
| 130 | bw-pbs | `10.10.10.130` | Backups | ✅ Operativa |
| 100 | bw-infra-debian | `10.10.10.100` | Docker, Portainer, OpenVAS, Cowrie, AWS CLI | ✅ Operativa |
| 140 | bw-wazuh | `10.10.10.140` | SIEM, monitorización | Pendiente |
| 150 | bw-winserver | `10.10.10.150` | Windows Server 2025, AD, DNS, GPO | Pendiente |
| 160 | bw-win11 | `10.10.10.160` | Cliente Windows, unido al dominio, Packet Tracer | Pendiente |
| 181 | bw-kali-purple | `10.10.10.181` | Red team, Wireshark | Pendiente |
| 170 | bw-gns3 | `10.10.10.170` | GNS3 Server, CCNA | Pendiente |
| 172 | bw-parrot | `10.10.10.172` | Red team complementario | Pendiente |

---

## ISOs

| ISO | Storage | Uso | Estado |
|---|---|---|---|
| `proxmox-ve_*.iso` | local | Instalación Proxmox | Usada |
| `netgate-installer-v1.2-RELEASE-amd64.iso` | local | bw-pfsense | Usada |
| `TrueNAS-SCALE-25.10.6.iso` | local | bw-truenas | Usada |
| `proxmox-backup-server_*.iso` | local | bw-pbs | Usada |
| `debian-13.*-amd64-netinst.iso` | local | bw-infra-debian | Usada |
| `ubuntu-22.04.*-live-server-amd64.iso` | local | bw-wazuh | Pendiente |
| `WinServer_*.iso` | local | bw-winserver | Pendiente |
| `Win11_*.iso` | local | bw-win11 | Pendiente |
| `kali-linux-2026.2-installer-purple-amd64.iso` | local | bw-kali-purple | Pendiente |
| `Parrot-security-7.3.iso` | local | bw-parrot | Pendiente |
| `SystemRescue-*.iso` | local | herramienta | Conservar |
| `gparted-live-*.iso` | local | herramienta | Conservar |

---

## Episodios publicados

### Temporada 1

| Episodio | Título | Fecha | Hito técnico |
|---|---|---|---|
| T1E01 | El Servidor Olvidado | 2026-08-28 | Diagnóstico del R320. Decisión de reinstalar desde cero. |
| T1E02 | El nacimiento del laboratorio | 2026-08-28 | Instalación limpia de Proxmox VE. |
| T1E03 | El fallo que nadie esperaba | 2026-08-28 | Problemas de arranque y DNS. |
| T1E04 | Tres discos, tres decisiones | — | Arquitectura de almacenamiento. |
| T1E05 | — | — | Pendiente (pfSense + migración de red) |
| T1E06 | — | — | Pendiente (PBS) |
| T1E07 | — | — | Pendiente (TrueNAS) |
| T1E08 | — | — | Pendiente (bw-infra-debian: Docker, Portainer, Ollama) |
| T1E09 | — | — | Pendiente (Wazuh) |
| T1E10 | — | — | Pendiente (Windows Server 2025 + AD) |
| T1E11 | — | — | Pendiente (bw-win11 + dominio) |

---

## Changelog técnico

### 2026-08-28 — T1E01 · El Servidor Olvidado
- Diagnóstico del R320. pfSense mal configurado (eno1 sin cable).
- Decisión: reinstalación limpia de Proxmox en sda. sdb conservado.

### 2026-08-28 — T1E02 · El nacimiento del laboratorio
- Proxmox VE reinstalado limpio.
- Hostname: `dell-r320-lab`, IP: `192.168.1.100/24`
- Bridges: `vmbr0` (eno2/LAN), `vmbr1` (eno1/WAN)
- Storages: `local`, `local-lvm`, `vm-lab-blancowhite`

### 2026-08-28 — T1E03 · El fallo que nadie esperaba
- Pantalla de contraseña en primer arranque → Ctrl+O para modo normal.
- DNS no resolvía → `1.1.1.1` y `8.8.8.8` añadidos en `/etc/network/interfaces`.

### 2026-08-30 — bw-pfsense operativo
- VMID 110, Q35/OVMF, LAN `192.168.1.110`, WAN DHCP.

### 2026-08-30 — bw-pbs operativo
- PBS 4.2.0, datastore `bw-backup`, backup de bw-pfsense completado.

### 2026-08-30 — bw-truenas operativo
- TrueNAS SCALE 25.10.6, pool `bw_pool`, datasets `bw-nas` y `bw-backup`, SMB activo.

### 2026-09-01 — Migración de red LAN a 10.10.10.0/24
**Causa:** WAN (`192.168.1.x`) y LAN (`192.168.1.110`) de pfSense en la misma subred. pfSense no podía enrutar entre interfaces en la misma red.

**Cambios aplicados:**
- pfSense LAN: `192.168.1.110` → `10.10.10.1/24`
- NAT Outbound: modo Hybrid, regla automática para `10.10.10.0/24`
- DHCP LAN: rango `10.10.10.200-10.10.10.250`
- bw-infra-debian: `192.168.1.100` → `10.10.10.100/24`
- bw-pbs: `192.168.1.130` → `10.10.10.130/24`
- bw-truenas: `192.168.1.120` → `10.10.10.120/24`
- Proxmox: IP secundaria `10.10.10.254/24` añadida a `vmbr0` (post-up permanente)
- Storage `pbs` en Proxmox reconfigurado con IP `10.10.10.130`

### 2026-09-01 — bw-infra-debian operativa
- Docker 29.7.2 instalado.
- Portainer CE instalado en `https://10.10.10.100:9443`.
- Salida a internet validada.

### 2026-09-01 — Infraestructura final definida
- 9 VMs en total planificadas.
- bw-winserver (VMID 150, `10.10.10.150`): Windows Server 2025, AD, DNS, GPO.
- bw-win11 (VMID 160, `10.10.10.160`): cliente Windows unido al dominio.
- bw-infra-debian ampliada con Ollama y Open WebUI (pendiente).

### 2026-09-02 — Revisión de infraestructura y objetivo del lab
- Objetivo ampliado: Blue Team + Red Team · CompTIA A+, Network+, Security+ · CCNA · Azure · AWS.
- bw-omarchy eliminada del plan — redundante con bw-infra-debian y bw-win11.
- bw-gns3 (VMID 170, `10.10.10.170`) añadida: GNS3 Server para CCNA.
- bw-parrot (VMID 172, `10.10.10.172`) añadida: red team complementario.
- Infraestructura final: 10 VMs.

### 2026-09-02 — bw-infra-debian — servicios
- RAM ampliada de 2 GB a 8 GB.
- Ollama instalado y desactivado (systemd disabled) — pendiente ampliación RAM.
- Open WebUI instalado — `http://10.10.10.100:3000` — bajo demanda.
- llama3.2 (2 GB) descargado. Mistral eliminado por insuficiencia de RAM.
- GitHub repo `bw-enterprise-lab-r320` clonado en `/root/bw-enterprise-lab-r320`.

---

## Pendientes de contenido

| Episodio | Acción |
|---|---|
| T1E05 | pfSense — configuración completa y migración de red. |
| T1E06 | PBS — backups programados para todas las VMs. |
| T1E07 | TrueNAS — NAS y SMB operativos. |
| T1E08 | bw-infra-debian — Docker, Portainer, Ollama, Open WebUI. |
| T1E09 | Wazuh — SIEM instalado. Agentes desplegados. |
| T1E10 | Windows Server 2025 — AD, DNS, GPO. |
| T1E11 | bw-win11 — cliente unido al dominio. |

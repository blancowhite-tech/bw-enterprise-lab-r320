# BlancoWhite Lab — 03 Máquinas Virtuales

> Documento actualizado 2026-10-24.
> Las VMs se instalan en el orden del plan. Cada una se documenta y valida antes de continuar con la siguiente.
> Los datos marcados como "por confirmar" no se han verificado todavía.
> Los valores de RAM y disco de arranque de las VMs en marcha se contrastaron con `qm list` el 2026-10-24.

---

## Resumen general

| VMID | Nombre | IP | Rol | Estado | Prioridad |
|---|---|---|---|---|---|
| 110 | bw-pfsense | `10.10.10.1` | Firewall, NAT, DHCP, DNS | ✅ Operativa | 1 |
| 130 | bw-pbs | `10.10.10.130` | Backups | ✅ Operativa | 2 |
| 120 | bw-truenas | `10.10.10.120` | NAS, SMB, almacenamiento | ✅ Operativa | 3 |
| 100 | bw-infra-debian | `10.10.10.100` | Docker, Portainer, Ollama, Open WebUI (OpenVAS, Cowrie, AWS CLI pendientes) | ✅ Operativa | 4 |
| 105 | bw-tailscale (LXC) | `10.10.10.5` / Tailscale `100.103.34.120` | Acceso remoto al lab | ✅ Operativo | — |
| 150 | bw-winserver | `10.10.10.150` | Windows Server 2025, AD (`bw.lab`), DNS, GPO | ✅ Operativa (AD en marcha). Apagada el 2026-10-24 | 5 |
| 160 | bw-win11 | `10.10.10.160` | Cliente Windows, unido al dominio, Packet Tracer | Instalada, falta red y dominio. Apagada el 2026-10-24 | 6 |
| 140 | bw-wazuh | `10.10.10.140` | SIEM, monitorización | Pendiente (RAM) | 7 |
| 181 | bw-kali-purple | `10.10.10.181` | Red team, Wireshark | Pendiente | 8 |
| 170 | bw-gns3 | `10.10.10.170` | GNS3 Server, CCNA | Pendiente | 9 |
| 172 | bw-parrot | `10.10.10.172` | Red team complementario | Pendiente | 10 |
| 700 | linux-mint | `10.10.10.190` (fija) | VM de prueba de distros | ✅ Instalada. Apagada el 2026-10-24 | — |

> Orden cambiado respecto al plan original: bw-winserver y bw-win11 se crearon antes que bw-wazuh. Wazuh espera a la ampliación de RAM.
> En Proxmox, el host (`192.168.1.100`) no es la VM 100. La VM 100 es bw-infra-debian.

---

## VMID 110 — bw-pfsense

| Campo | Valor |
|---|---|
| Nombre | `bw-pfsense` |
| IP LAN | `10.10.10.1/24` |
| IP WAN | DHCP desde router Movistar |
| RAM | 2 GB según `qm list` (2026-10-24). El documento anterior decía 4 GB. **Por confirmar** |
| CPU | 2 vCPU |
| Disco | 20 GB según `qm list` (2026-10-24). El documento anterior decía 32 GB. **Por confirmar** |
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
- DHCP LAN activo: rango observado `10.10.10.200-10.10.10.240` (2026-10-24). El documento anterior decía `.250`. **Por confirmar**.
- DNS Resolver: Domain Override `bw.lab` → `10.10.10.150`, con `domain-insecure` y `private-domain` para `bw.lab` en Custom Options (2026-10-03).
- Sirve NTP al EWC de Cisco.
- Suricata figura como rol previsto; no está confirmado que esté instalado.

---

## VMID 130 — bw-pbs

| Campo | Valor |
|---|---|
| Nombre | `bw-pbs` |
| IP | `10.10.10.130/24` |
| Gateway | `10.10.10.1` |
| DNS | `1.1.1.1` / `8.8.8.8` |
| RAM | 2 GB |
| CPU | 2 vCPU (x86-64-v2-AES) |
| Disco | scsi0 — 240 GB (234 GB útiles) en `vm-lab-blancowhite` |
| Machine | Q35 |
| Firmware | OVMF / UEFI |
| net0 | VirtIO → `vmbr0` |
| OS | Proxmox Backup Server 4.2.1 |
| Datastore | `bw-backup` |
| Estado | ✅ Operativa |

### Notas
- IP migrada de `192.168.1.130` a `10.10.10.130` el 2026-09-01.
- Storage `pbs` actualizado en Proxmox con nueva IP.
- No se hace backup de la VM completa: el disco contiene el datastore. La recuperación se documenta en `pbs-config.md`.
- Job de backup semanal los domingos a las 01:00 (keep last 2 + daily 7). La VM 130 **no** está en el job (confirmado el 2026-10-25).

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
- Open WebUI — instalado, desactivado/bajo demanda — `http://10.10.10.100:3000`
- llama3.2 — descargado, inactivo
- QEMU Guest Agent
- SSH
- GitHub repo `bw-enterprise-lab-r320` clonado en `/root/bw-enterprise-lab-r320` (`docs/`, `scripts/`, `diagrams/`)

### Servicios pendientes
- OpenVAS (Greenbone) — pendiente RAM
- Cowrie honeypot
- AWS CLI

---

## CTID 105 — bw-tailscale (contenedor LXC)

| Campo | Valor |
|---|---|
| Nombre | `bw-tailscale` |
| Tipo | Contenedor LXC (se gestiona con `pct`, no con `qm`) |
| IP LAN | `10.10.10.5/24` (fija) |
| Gateway | `10.10.10.1` |
| Red | net0 → eth0 → `vmbr0`, firewall de Proxmox desactivado |
| IP Tailscale | `100.103.34.120` (se ve con `pct exec 105 -- tailscale ip -4`) |
| RAM / CPU / Disco | por confirmar |
| Uso | Conectarse al Dell R320 desde redes externas. Tailscale instalado en los equipos del usuario |
| Arranque automático | Sí: `onboot 1`, `order=2,up=15` |
| Backup | Sin backup a PBS. Pendiente |
| Estado | ✅ Operativo |

### Notas
- La IP `10.10.10.5` está fuera del rango DHCP de pfSense.
- Con el R320 apagado no hay acceso remoto: corre dentro del servidor.
- Si abre o no la LAN completa (subnet router): por confirmar.

---

## VMID 150 — bw-winserver

| Campo | Valor |
|---|---|
| Nombre | `bw-winserver` |
| IP | `10.10.10.150/24` |
| Gateway | `10.10.10.1` |
| DNS | `10.10.10.150` (propio) / `1.1.1.1` (secundario) |
| RAM | 8 GB según `qm list` (2026-10-24). El documento anterior decía 4 GB (decisión propia). **Por confirmar** si se subió después |
| CPU | por confirmar (plan: 4 vCPU) |
| Disco | 40 GB (el plan decía 80 GB), en `vm-lab-blancowhite` |
| Machine | Q35 |
| Firmware | OVMF / UEFI |
| net0 | VirtIO → `vmbr0` |
| OS | Windows Server 2025 Datacenter, Desktop Experience (ISO EVAL en inglés) |
| Acceso | RDP (la consola noVNC no permite pegar) |
| Rol | Active Directory, DNS, GPO |
| Estado | ✅ Operativa. Apagada el 2026-10-24 (`stopped`) |

### Dominio
- Dominio: `bw.lab` (NetBIOS `BW`, nivel funcional Windows2025Domain).
- Controlador de dominio principal del lab.
- NTDS, DNS y Netlogon en Running (2026-10-03).
- pfSense reenvía `bw.lab` a `10.10.10.150` (ver bw-pfsense).
- `nslookup bw.lab 10.10.10.1` responde bien desde bw-infra-debian.
- Si la VM está apagada, el dominio y la resolución de `bw.lab` no funcionan.

### Estructura de AD (creada el 2026-10-03)
- Diseño tipo centro logístico: sedes, tiering y grupos GG_/DL_.
- 49 OUs bajo `OU=BW`: Admin (Tier0/1/2), Servidores, Servicio, Corporativo, Sedes (SVQ1, MAD7, BCN4), Grupos (Roles, Recursos, Distribución) y Deshabilitados.
- 37 grupos (GG_ y DL_).
- 60 usuarios operativos `e1000xx`.
- Cuentas `t0-admin`, `t1-admin` y `t2-admin`, todavía sin privilegios.
- Objetivo: practicar altas de usuario, permisos, privilegios altos, delegación y GPO.

### Pendiente
- Ajustar hora/NTP.
- Borrar el snapshot `pre-ad` (creado antes de promover el DC), una vez haya backup en PBS.
- Backup a PBS (aún no está en el job).
- SSH.

---

## VMID 160 — bw-win11

| Campo | Valor |
|---|---|
| Nombre | `bw-win11` |
| IP prevista | `10.10.10.160/24` |
| Gateway previsto | `10.10.10.1` |
| DNS previsto | `10.10.10.150` (AD DC) |
| RAM | 4 GB |
| CPU | 2 cores, type: host |
| Disco | scsi0 — 70 GiB en `vm-lab-blancowhite` |
| Machine | Q35 |
| Firmware | OVMF / UEFI, con TPM 2.0 |
| net0 | VirtIO → `vmbr0` |
| ISOs | `Windows11_Client_x64_es-es_26300_9457.iso`, `virtio-win.iso` |
| OS | Windows 11 Pro 26H2 (compilación 26300.9457) |
| Equipo / cuenta local | `bw-win11` / `blancowhite` |
| Estado | Instalada. Pendiente de red y dominio. Apagada el 2026-10-24 |

### Pendiente
- Red `10.10.10.160/24` con DNS `10.10.10.150`.
- VirtIO guest tools.
- Backup en PBS.
- Unirla al dominio `bw.lab`.
- Objetivo posterior: practicar GPO, gestión de usuarios AD e integración con Wazuh.

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

### Notas de planificación
- Con la RAM actual (32 GB) no cabe junto al resto de VMs encendidas. Espera a la ampliación a 64 GB (sin comprar) o a arrancarla con menos RAM.

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

## VMs de prueba de distros (pool `bw-distros`, serie 700)

- Pool `bw-distros` (comentario TEST-DISTROS). Sin "Start at boot".
- Todas en `vmbr0`, dentro de `10.10.10.0/24`.
- Orden previsto: Linux Mint, Zorin OS 18.1 Core, Winux 11, CachyOS 260809 (ISOs ya subidas).

### VMID 700 — linux-mint

| Campo | Valor |
|---|---|
| Nombre | `linux-mint` |
| Hostname | `bwLinuxMint` |
| Usuario | `blancowhite` |
| IP | `10.10.10.190/24` (fija) |
| Gateway | `10.10.10.1` |
| DNS | `10.10.10.1` |
| Disco | 40 GB en `vm-lab-blancowhite` |
| RAM | 4 GB (según `qm list`, 2026-10-24) |
| CPU | por confirmar |
| OS | Linux Mint 22.3 Cinnamon |
| Estado | ✅ Instalada. Apagada el 2026-10-24 |

### Notas
- Al arrancar cogió `192.168.1.198` del router Movistar en lugar del DHCP de pfSense, así que se le puso IP fija. Ping a `10.10.10.1` y a internet validados.
- Acceso por la consola de Proxmox. xrdp y SSH pendientes.

---

## Arranque automático (configurado el 2026-10-25)

| Orden | ID | Nombre | Comando aplicado |
|---|---|---|---|
| 1 | VM 110 | bw-pfsense | `qm set 110 --onboot 1 --startup order=1,up=60` |
| 2 | CT 105 | bw-tailscale | `pct set 105 --onboot 1 --startup order=2,up=15` |
| 3 | VM 120 | bw-truenas | `qm set 120 --onboot 1 --startup order=3,up=60` |
| 4 | VM 130 | bw-pbs | `qm set 130 --onboot 1 --startup order=4,up=30` |
| 5 | VM 100 | bw-infra-debian | `qm set 100 --onboot 1 --startup order=5` |

- `up` = segundos que espera Proxmox antes de arrancar la siguiente.
- En la web: seleccionar la VM → **Options** → **Start at boot** y **Start/Shutdown order**.
- Sin arranque automático: 150 (bw-winserver), 160 (bw-win11) y 700 (linux-mint), por RAM. Se arrancan a mano con `qm start <VMID>`.
- Prueba de arranque pendiente para el 2026-10-26 (detalle en `01-INFRAESTRUCTURA.md`).

---

## Ideas en estudio (sin VMID ni IP)

- **Kasm Workspaces:** valorado el 2026-10-07. Se montaría sobre Ubuntu Server en una VM propia.

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
| 2026-10-01 | bw-winserver | Creada con 40 GB de disco. Windows Server 2025 Datacenter Desktop Experience instalado. Disco movido de `local-lvm` a `vm-lab-blancowhite`. |
| 2026-10-03 | bw-winserver | Dominio `bw.lab` operativo. Snapshot `pre-ad` previo a la promoción. |
| 2026-10-03 | bw-pfsense | Domain Override `bw.lab` → `10.10.10.150` en el DNS Resolver. |
| 2026-10-03 | bw-winserver | Estructura de AD creada: 49 OUs, 37 grupos, 60 usuarios y cuentas `t0/t1/t2-admin`. |
| 2026-10-04 | bw-win11 | Creada (4 GB, 70 GiB, TPM 2.0). Windows 11 Pro 26H2 instalado. |
| 2026-10-05 | linux-mint | VM 700 instalada. IP fija `10.10.10.190`. |
| 2026-10-07 | — | Kasm Workspaces valorado (VM sobre Ubuntu Server, sin crear). |
| 2026-10-24 | Todas | Estado contrastado con `qm list`: 100, 110, 120 y 130 en marcha; 150, 160 y 700 paradas. RAM/disco de pfSense y RAM de winserver difieren del documento anterior (por confirmar). |
| 2026-10-25 | bw-tailscale | Documentado el contenedor LXC 105: `10.10.10.5`, Tailscale `100.103.34.120`. |
| 2026-10-25 | 110, 105, 120, 130, 100 | Arranque automático configurado con orden y esperas. Servidor apagado para probarlo al encender. |

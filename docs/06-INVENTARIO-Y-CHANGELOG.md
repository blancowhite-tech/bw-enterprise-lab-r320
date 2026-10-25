# BlancoWhite Lab — 06 Inventario y Changelog

> Documento actualizado 2026-10-24.
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
| RAM | 32 GB ECC DDR3 (4 × 8 GB, 1333 MHz). Slots libres: DIMM_A3 y DIMM_A6. Ampliación a 64 GB pendiente de comprar |
| NIC 1 | eno1 — WAN (router Movistar) |
| NIC 2 | eno2 — LAN (switch Cisco) |
| Switch | Cisco gestionable |
| Router | Movistar |
| Puntos de acceso | 2 × Cisco Catalyst 9120AXI-E (Wi-Fi 6), comprados de segunda mano. Requieren PoE+ (802.3at); el Catalyst 3750 clásico solo da 802.3af y no basta (se resolvió con un inyector PoE+). Uno hace de controladora (EWC) y el otro está unido por CAPWAP |
| Disco sistema | sda ~837 GB — Proxmox VE |
| Disco NAS | sdb ~1.6 TB — TrueNAS pool bw_pool |
| Disco VMs | sdc ~837 GB — discos de VMs |
| PC de administración | `BlancoWhiteHack`, `10.10.10.216` por cable (y `10.10.10.200` por wifi) |

---

## Inventario software

| Software | Versión | VM | Rol | Estado |
|---|---|---|---|---|
| Proxmox VE | última estable | dell-r320-lab | Hipervisor | ✅ Operativo |
| pfSense | netgate-installer-v1.2-RELEASE | bw-pfsense (110) | Firewall, NAT, DHCP, DNS | ✅ Operativo |
| TrueNAS SCALE | 25.10.6 Goldeye | bw-truenas (120) | NAS, SMB | ✅ Operativo |
| Proxmox Backup Server | 4.2.1 | bw-pbs (130) | Backups | ✅ Operativo |
| Docker | 29.7.2 | bw-infra-debian (100) | Contenedores | ✅ Operativo |
| Portainer CE | última estable | bw-infra-debian (100) | Gestión Docker | ✅ Operativo |
| Ollama | — | bw-infra-debian (100) | LLM local | Instalado, inactivo |
| Open WebUI | — | bw-infra-debian (100) | Interfaz Ollama | Instalado, bajo demanda |
| Tailscale | por confirmar | bw-tailscale (CT 105) | Acceso remoto al lab | ✅ Operativo |
| Wazuh | 4.x | bw-wazuh (140) | SIEM | Pendiente |
| Windows Server 2025 Datacenter | EVAL, Desktop Experience | bw-winserver (150) | AD `bw.lab`, DNS, GPO | ✅ Operativo |
| Windows 11 Pro | 26H2 (compilación 26300.9457) | bw-win11 (160) | Cliente de dominio | Instalado, falta red y dominio |
| Linux Mint | 22.3 Cinnamon | linux-mint (700) | VM de prueba | ✅ Instalado |
| Kali Purple | 2026.2 | bw-kali-purple (181) | Red team, Wireshark | Pendiente |
| GNS3 | — | bw-gns3 (170) | Topologías CCNA | Pendiente |
| Parrot Security | 7.3 | bw-parrot (172) | Red team complementario | Pendiente |
| Cisco EWC (Embedded Wireless Controller) | 17.15.6.0.770 | AP Cisco 9120AXI-E (no es VM) | Controladora wifi integrada en el propio AP | ✅ Operativo |

---

## Inventario VMs

| VMID | Nombre | IP | Rol | Estado |
|---|---|---|---|---|
| 100 | bw-infra-debian | `10.10.10.100` | Docker, Portainer, Ollama, Open WebUI (OpenVAS, Cowrie, AWS CLI pendientes) | ✅ Operativa |
| 105 | bw-tailscale (LXC) | `10.10.10.5` / Tailscale `100.103.34.120` | Acceso remoto al lab | ✅ Operativo |
| 110 | bw-pfsense | `10.10.10.1` | Firewall, NAT, DHCP, DNS (Suricata por confirmar) | ✅ Operativa |
| 120 | bw-truenas | `10.10.10.120` | NAS, SMB, almacenamiento | ✅ Operativa |
| 130 | bw-pbs | `10.10.10.130` | Backups | ✅ Operativa |
| 140 | bw-wazuh | `10.10.10.140` | SIEM, monitorización | Pendiente |
| 150 | bw-winserver | `10.10.10.150` | Windows Server 2025, AD, DNS, GPO | ✅ Operativa (apagada el 2026-10-24) |
| 160 | bw-win11 | `10.10.10.160` | Cliente Windows, unido al dominio, Packet Tracer | Instalada, falta red y dominio (apagada el 2026-10-24) |
| 170 | bw-gns3 | `10.10.10.170` | GNS3 Server, CCNA | Pendiente |
| 172 | bw-parrot | `10.10.10.172` | Red team complementario | Pendiente |
| 181 | bw-kali-purple | `10.10.10.181` | Red team, Wireshark | Pendiente |
| 700 | linux-mint | `10.10.10.190` (fija) | VM de prueba de distros | ✅ Instalada (apagada el 2026-10-24) |

---

## ISOs

| ISO | Storage | Uso | Estado |
|---|---|---|---|
| `proxmox-ve_*.iso` | local | Instalación Proxmox | Usada |
| `netgate-installer-v1.2-RELEASE-amd64.iso` | local | bw-pfsense | Usada |
| `TrueNAS-SCALE-25.10.6.iso` | local | bw-truenas | Usada |
| `proxmox-backup-server_4.2-1.iso` | local | bw-pbs | Usada |
| `debian-13.*-amd64-netinst.iso` | local | bw-infra-debian | Usada |
| Windows Server 2025 EVAL (inglés) | local | bw-winserver | Usada |
| `Windows11_Client_x64_es-es_26300_9457.iso` | local | bw-win11 | Usada |
| `virtio-win.iso` | local | bw-win11 | Usada |
| Linux Mint 22.3 Cinnamon | local | linux-mint | Usada |
| `ubuntu-22.04.*-live-server-amd64.iso` | local | bw-wazuh | Pendiente |
| `kali-linux-2026.2-installer-purple-amd64.iso` | local | bw-kali-purple | Pendiente |
| `Parrot-security-7.3.iso` | local | bw-parrot | Pendiente |
| Zorin OS 18.1 Core, Winux 11, CachyOS 260809 | local | VMs de prueba de distros | Subidas, pendientes |
| `SystemRescue-*.iso` | local | herramienta | Conservar |
| `gparted-live-*.iso` | local | herramienta | Conservar |

---

## Imágenes Cisco (descargadas en el PC de administración, no en Proxmox)

| Archivo | Uso | Estado |
|---|---|---|
| `C9800-AP-universalk9.17.12.08.zip` | Conversión del AP a EWC (contiene `ap1g7` y `C9800-AP-iosxe-wlc.bin`), servido por TFTP desde `C:\tftp` | Usada |
| Imagen 17.15.6.0.770 | Actualización del EWC por WebUI (Desktop HTTP) | Usada |
| `C9800-CL-universalk9.17.12.07a.qcow2` | WLC virtual 9800-CL para Proxmox | Descargada, VM no creada (no necesaria al usar EWC) |

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

### Episodio especial

| Episodio | Título | Hito |
|---|---|---|
| Especial | Certificación obtenida | Certificación Cisco *Introduction to Cybersecurity*, publicada como complemento teoría + práctica de la serie. |

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
- PBS, datastore `bw-backup`, backup de bw-pfsense completado.

### 2026-08-30 — bw-truenas operativo
- TrueNAS SCALE 25.10.6, pool `bw_pool`, datasets `bw-nas` y `bw-backup`, SMB activo.

### 2026-09-01 — Migración de red LAN a 10.10.10.0/24
**Causa:** WAN (`192.168.1.x`) y LAN (`192.168.1.110`) de pfSense en la misma subred. pfSense no podía enrutar entre interfaces en la misma red.

**Cambios aplicados:**
- pfSense LAN: `192.168.1.110` → `10.10.10.1/24`
- NAT Outbound: modo Hybrid, regla automática para `10.10.10.0/24`
- DHCP LAN activado
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
- bw-winserver (VMID 150, `10.10.10.150`): Windows Server 2025, AD, DNS, GPO.
- bw-win11 (VMID 160, `10.10.10.160`): cliente Windows unido al dominio.

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

### 2026-10-01 — bw-winserver creada
- VMID 150, 40 GB de disco, Windows Server 2025 Datacenter Desktop Experience (ISO EVAL en inglés).
- Disco movido de `local-lvm` (sda) a `vm-lab-blancowhite` (sdc) por rendimiento de IO.
- Acceso por RDP: la consola noVNC no permite pegar.

### 2026-10-03 — Active Directory `bw.lab`
- Dominio `bw.lab` (NetBIOS `BW`, nivel Windows2025Domain) operativo. NTDS, DNS y Netlogon en Running.
- Snapshot `pre-ad` creado antes de promover el DC.
- pfSense DNS Resolver: Domain Override `bw.lab` → `10.10.10.150`, con `domain-insecure` y `private-domain`.
- Estructura de AD tipo centro logístico: 49 OUs bajo `OU=BW` (Admin Tier0/1/2, Servidores, Servicio, Corporativo, Sedes SVQ1/MAD7/BCN4, Grupos, Deshabilitados), 37 grupos GG_/DL_, 60 usuarios `e1000xx` y cuentas `t0/t1/t2-admin` sin privilegios.

### 2026-10-04 — bw-win11 creada
- VMID 160: Q35/OVMF, TPM 2.0, 4 GB, 70 GiB. Windows 11 Pro 26H2 instalado.
- Pendiente: red `10.10.10.160/24`, VirtIO guest tools, backup y unión al dominio.

### 2026-10-05 — AP Cisco Catalyst 9120AXI-E convertido a EWC
- Equipo de segunda mano. Alimentación: necesita PoE+ (802.3at); con PoE 802.3af (Catalyst 3750 clásico) se apagaba en bucle.
- Estado inicial: software `8.10.130.0` (CAPWAP), ciclo de LEDs estable buscando controladora.
- Imagen: `C9800-AP-universalk9.17.12.08.zip` (Cisco Software Download, rama 17.12).
- Método: Tftpd64 en el PC de administración (`10.10.10.216`, carpeta `C:\tftp`), consola serie (PuTTY, COM3, 9600) y comando `ap-type ewc-ap` con las dos imágenes. El AP tenía `10.10.10.225` por DHCP antes de convertir.
- Tras el reinicio arrancó como controladora (`Installation mode is BUNDLE`, 100 APs soportados) con la IP `192.168.1.195` (red Movistar). Causa sin explicar.
- Acceso web por `mywifi.cisco.com`, resuelto con una entrada en el `hosts` del PC.
- La WLC virtual 9800-CL se descargó pero no se desplegó: no hace falta con EWC.
- Aviso: Cisco lista la versión 17.12.8 como afectada por un aviso de seguridad (bugs CSCwv93265 y CSCwv98483), con APSP disponible.

### 2026-10-05 — VM linux-mint
- VMID 700 instalada (Mint 22.3 Cinnamon, 40 GB). Cogió `192.168.1.198` del router Movistar en lugar del DHCP de pfSense, así que se le puso IP fija `10.10.10.190/24`.

### 2026-10-07 — Kasm Workspaces
- Valorado para el lab, sobre Ubuntu Server en una VM propia. Sin crear.

### 2026-10-08 — Cisco EWC: IP, versión y WLANs
- IP de gestión `10.10.10.240/24` (Gi0), gateway `10.10.10.1`, WebUI `https://10.10.10.240`. Ya no usa `192.168.1.195`.
- Software 17.12.8.4 → 17.15.6.0.770 por el bug CSCwv93265, vía WebUI (Desktop HTTP) y `install commit`. Aviso de seguridad de 17.12.8 resuelto.
- NTP contra pfSense, zona CET/CEST.
- WLANs `Blanco_White_Tech_5G` (ID 1, solo 5 GHz) y `Blanco_White_Tech_2.4G` (ID 2, solo 2.4 GHz), WPA2-PSK AES. Policy Profile `Blanco_White_Tech_Policy`, Policy Tag `Tag_Blanco_White_Tech`.
- Segundo AP 9120AXI-E unido por CAPWAP (`1006.ed52.906c`), con `wireless management trustpoint CISCO_IDEVID_CMCA3_SUDI`.
- Backup local en bootflash: `EWC_backup_final.cfg`.

### 2026-10-24 — Revisión de documentación
- Comprobación con `qm list`: VMs 100, 110, 120 y 130 en marcha; 150, 160 y 700 paradas.
- Incidencia: el EWC no abría por `mywifi.cisco.com` porque el `hosts` del PC seguía apuntando a `192.168.1.195`. La IP correcta es `10.10.10.240`. El AP (`10.10.10.225`) no sirve web.
- Documentación corregida (EWC, estado de VMs, PBS 4.2.1, inventarios).
- Discrepancias detectadas y **por confirmar**: RAM y disco de bw-pfsense (`qm list`: 2 GB / 20 GB; documento: 4 GB / 32 GB), RAM de bw-winserver (`qm list`: 8 GB; documento: 4 GB), rango DHCP (`.200-.240` observado frente a `.200-.250`). Resuelto el 2026-10-25: la VM 130 **no** está en el job de backup.

---

### 2026-10-25 — Tailscale y arranque automático
- Documentado el contenedor LXC 105 `bw-tailscale`: `10.10.10.5/24` fija (gateway `10.10.10.1`), IP Tailscale `100.103.34.120`. Se usa para conectarse al lab desde redes externas, con Tailscale instalado en los equipos del usuario.
- Arranque automático configurado (`onboot 1` y `startup`): pfSense (1, 60 s) → bw-tailscale (2, 15 s) → TrueNAS (3, 60 s) → PBS (4, 30 s) → bw-infra-debian (5). Sin arranque automático: 150, 160 y 700 (RAM).
- Verificado con `qm config 110` y `pct config 105`; los otros tres comandos respondieron sin errores.
- Apagado ordenado del servidor para probar el arranque automático al encenderlo el 2026-10-26. Resultado: por registrar.

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

---

## Pendientes técnicos (resumen)

- Reservar IP fija para el EWC (`10.10.10.240`) y para el AP (`10.10.10.225`) en pfSense.
- Actualizar el `hosts` del PC de administración a `10.10.10.240`.
- Backup de bw-winserver y bw-win11 en PBS. Después, borrar el snapshot `pre-ad`.
- Sacar copia del backup del EWC fuera del AP.
- Red, dominio y guest tools de bw-win11.
- Ampliación de RAM a 64 GB y despliegue de bw-wazuh.
- OpenVAS, Cowrie y AWS CLI en bw-infra-debian.
- Confirmar las discrepancias de la entrada del 2026-10-24.
- Registrar el resultado de la prueba de arranque automático (2026-10-26).
- Backup de bw-tailscale (CT 105) en PBS.
- Confirmar versión de Tailscale y si `bw-tailscale` actúa como subnet router.
- Confirmar el encendido automático del R320 tras un corte de luz (BIOS/iDRAC).

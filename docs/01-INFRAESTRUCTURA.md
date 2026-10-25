# BlancoWhite Lab — 01 Infraestructura

> Documento actualizado 2026-10-24.
> Los datos marcados como "por confirmar" no se han verificado todavía.

---

## Hardware

| Campo | Valor |
|---|---|
| Servidor | Dell PowerEdge R320 |
| Controladora | PERC H710 |
| CPU | Intel Xeon E5-2450 @ 2.10 GHz |
| RAM | 32 GB ECC DDR3 RDIMM (4 × 8 GB, 1333 MHz PC3-10600R) |
| Slots RAM | 6 en total. Libres: DIMM_A3 y DIMM_A6 |
| Ampliación prevista | 2 × 16 GB PC3-12800R ECC Registered (≈ 40 €) → 64 GB. Sin comprar todavía |

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

> **Ojo con los números:** `192.168.1.100` es el host Proxmox. La VM con VMID 100 es `bw-infra-debian` (`10.10.10.100`). No son lo mismo.

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

## Equipos de red

| Equipo | Función | IP | Estado |
|---|---|---|---|
| Cisco Catalyst 9120AXI-E (segunda mano) | AP wifi con controladora integrada (EWC 17.15.6.0.770). No es una VM | `10.10.10.240` (gestión del EWC) / `10.10.10.225` (IP DHCP del AP) | ✅ Operativo |
| Segundo Cisco 9120AXI-E | AP unido al EWC en modo CAPWAP (MAC `1006.ed52.906c`) | por confirmar | ✅ Operativo |
| Switch Cisco | LAN del lab (eno2) | — | ✅ Operativo |
| Router Movistar | Salida a internet (WAN de pfSense, eno1) | `192.168.1.1` | ✅ Operativo |

> Detalle de red en `02-RED-Y-FIREWALL.md`, del servicio en `04-SERVICIOS.md` y de la conversión en `06-INVENTARIO-Y-CHANGELOG.md`.

---

## Máquinas virtuales

| VMID | Nombre | IP | Función | Estado |
|---|---|---|---|---|
| 100 | bw-infra-debian | `10.10.10.100` | Docker, Portainer, Ollama, Open WebUI (OpenVAS, Cowrie, AWS CLI pendientes) | ✅ Operativa |
| 105 | bw-tailscale (contenedor LXC) | `10.10.10.5` (LAN) / `100.103.34.120` (Tailscale) | Acceso remoto al lab desde redes externas | ✅ Operativo |
| 110 | bw-pfsense | `10.10.10.1` (LAN) / WAN DHCP | Firewall, NAT, DHCP, DNS (Suricata por confirmar) | ✅ Operativa |
| 120 | bw-truenas | `10.10.10.120` | NAS, SMB, almacenamiento | ✅ Operativa |
| 130 | bw-pbs | `10.10.10.130` | Proxmox Backup Server | ✅ Operativa |
| 140 | bw-wazuh | `10.10.10.140` | SIEM, monitorización | Pendiente (RAM) |
| 150 | bw-winserver | `10.10.10.150` | Windows Server 2025, AD `bw.lab`, DNS, GPO | ✅ Operativa (apagada el 2026-10-24) |
| 160 | bw-win11 | `10.10.10.160` | Cliente Windows, unido al dominio, Packet Tracer | Instalada. Falta red y dominio (apagada el 2026-10-24) |
| 170 | bw-gns3 | `10.10.10.170` | GNS3 Server, CCNA | Pendiente |
| 172 | bw-parrot | `10.10.10.172` | Red team complementario | Pendiente |
| 181 | bw-kali-purple | `10.10.10.181` | Red team, Wireshark | Pendiente |
| 700 | linux-mint | `10.10.10.190` (fija) | VM de prueba de distros (pool `bw-distros`) | ✅ Instalada (apagada el 2026-10-24) |

> Estado de arranque comprobado con `qm list` el 2026-10-24 a las 20:23: 100, 110, 120 y 130 `running`; 150, 160 y 700 `stopped`.
> Detalle de cada VM en `03-MAQUINAS-VIRTUALES.md`.

---

## Arranque automático (configurado el 2026-10-25)

Al encender el R320, Proxmox arranca solo estas guests, en este orden y con esta espera antes de pasar a la siguiente:

| Orden | VMID | Nombre | Tipo | Espera (`up`) |
|---|---|---|---|---|
| 1 | 110 | bw-pfsense | VM | 60 s |
| 2 | 105 | bw-tailscale | LXC | 15 s |
| 3 | 120 | bw-truenas | VM | 60 s |
| 4 | 130 | bw-pbs | VM | 30 s |
| 5 | 100 | bw-infra-debian | VM | — |

- Todas con `onboot: 1`. Tiempo total estimado hasta tenerlo todo arriba: 3-4 minutos.
- Orden pensado así: pfSense primero (red y DHCP) y Tailscale justo después, porque necesita red para conectarse.
- **No arrancan solas:** bw-winserver (150), bw-win11 (160) y linux-mint (700). Con 32 GB de RAM no caben todas a la vez.
- Comandos aplicados: `qm set <VMID> --onboot 1 --startup order=N,up=S` para las VMs y `pct set 105 --onboot 1 --startup order=2,up=15` para el contenedor.
- Verificado con `qm config 110` y `pct config 105`. Las otras tres (120, 130, 100) respondieron `update VM` sin errores.
- Para comprobar: `qm config <VMID> | grep -E "onboot|startup"` (VMs) o `pct config 105 | grep -E "onboot|startup"`.
- Por confirmar: encendido automático del propio R320 tras un corte de luz (BIOS/iDRAC, "AC Power Recovery" en "Last" u "On").

### Prueba de arranque (pendiente, 2026-10-26)
1. Apagado ordenado con `shutdown -h now` el 2026-10-25 (Proxmox apaga las guests solo).
2. Encender el R320 al día siguiente y no tocar nada.
3. A los ~4 minutos, desde el PC: `ping` a `10.10.10.1`, `10.10.10.5`, `10.10.10.120`, `10.10.10.130` y `10.10.10.100`.
4. En el shell de Proxmox: `qm list` debe mostrar 100, 110, 120 y 130 en `running`, y `pct list` el 105.
5. Resultado: por registrar.

---

## Reglas operativas

1. Revisar el estado real antes de cambiar.
2. Un cambio cada vez.
3. Validar el resultado antes de continuar.
4. Documentar únicamente lo confirmado.
5. No modificar servicios operativos sin razón concreta.
6. Hacer backup antes de tocar cualquier VM en producción.
7. Cada nueva VM se documenta antes de pasar a la siguiente.
8. Antes de dar por perdido un equipo, comprobar su IP actual en pfSense (Status → DHCP Leases) y revisar este documento: las IPs cambian y la documentación tiene que seguirlas.

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
| 2026-09-01 | Infraestructura final definida: bw-winserver (VMID 150) y bw-win11 (VMID 160) añadidas al plan. |
| 2026-09-02 | bw-omarchy eliminada (redundante). bw-gns3 (VMID 170) y bw-parrot (VMID 172) añadidas. |
| 2026-09-02 | bw-infra-debian RAM ampliada a 8 GB. Ollama y Open WebUI instalados (inactivos por RAM). |
| 2026-09-02 | Objetivo del lab ampliado: Blue Team + Red Team · CompTIA A+, Network+, Security+ · CCNA · Azure · AWS. |
| 2026-10-01 | bw-winserver (150) creada e instalada con Windows Server 2025 Datacenter. |
| 2026-10-03 | Dominio `bw.lab` operativo. Estructura de AD creada. |
| 2026-10-04 | bw-win11 (160) creada e instalada. |
| 2026-10-05 | AP Cisco 9120AXI-E convertido a EWC 17.12.08. La WLC virtual 9800-CL se descargó pero no se desplegó. VM `linux-mint` (700) instalada. |
| 2026-10-08 | EWC movido a la LAN lab (`10.10.10.240`) y actualizado a 17.15.6.0.770. WLANs `Blanco_White_Tech_5G` y `_2.4G` creadas. Segundo AP unido por CAPWAP. |
| 2026-10-24 | Documentación revisada y alineada con el estado real. |
| 2026-10-25 | Documentado el contenedor `bw-tailscale` (LXC 105, `10.10.10.5`). Configurado el arranque automático de 110, 105, 120, 130 y 100. Apagado del servidor para probarlo al encender. |

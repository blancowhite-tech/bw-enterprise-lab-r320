# BlancoWhite Lab — 02 Red y Firewall

> Documento actualizado 2026-10-24.
> Los datos marcados como "por confirmar" no se han verificado todavía.

---

## Topología física

```
Internet
   │
Router Movistar (192.168.1.1)
   │
   ├── eno1 (WAN) ──── Dell R320
   │                       │
   │                   eno2 (LAN)
   │                       │
   └──────────── Switch Cisco ────── resto de dispositivos (PC admin, AP Cisco)
```

> **eno1** → conectado directamente al router Movistar. WAN de pfSense.
> **eno2** → conectado al switch. LAN del lab. Sobre `vmbr0`.

---

## Bridges Proxmox

| Bridge | Interfaz física | Uso |
|---|---|---|
| `vmbr0` | eno2 | LAN del lab — todas las VMs |
| `vmbr1` | eno1 | WAN de pfSense — salida a internet |

---

## Subredes

| Red | Subred | Uso |
|---|---|---|
| WAN | `192.168.1.0/24` | Router Movistar → pfSense WAN |
| LAN lab | `10.10.10.0/24` | Todas las VMs del laboratorio, el AP wifi y el PC de administración |

> **Nota:** La LAN se migró de `192.168.1.0/24` a `10.10.10.0/24` el 2026-09-01.
> Causa: WAN (`192.168.1.x`) y LAN (`192.168.1.110`) de pfSense estaban en la misma subred — pfSense no podía enrutar entre ellas.

---

## Esquema IP

### Proxmox (fuera de la LAN lab)
| IP | Hostname | Descripción |
|---|---|---|
| `192.168.1.100` | `dell-r320-lab` | Proxmox VE — gestión (red Movistar) |
| `10.10.10.254` | `dell-r320-lab` | IP secundaria en la LAN lab (ping verificado el 2026-10-24) |

> `192.168.1.100` es el host Proxmox. No confundir con la VM 100 (`10.10.10.100`, bw-infra-debian).

### LAN lab — 10.10.10.0/24
| IP | Hostname | Rol |
|---|---|---|
| `10.10.10.1` | `bw-pfsense` | Firewall, NAT, DHCP, DNS |
| `10.10.10.5` | `bw-tailscale` | Contenedor LXC 105. Acceso remoto por Tailscale. IP fija |
| `10.10.10.100` | `bw-infra-debian` | Docker, Portainer, Ollama, Open WebUI |
| `10.10.10.120` | `bw-truenas` | NAS, SMB, almacenamiento |
| `10.10.10.130` | `bw-pbs` | Proxmox Backup Server |
| `10.10.10.140` | `bw-wazuh` | Wazuh SIEM (pendiente) |
| `10.10.10.150` | `bw-winserver` | Windows Server 2025, AD `bw.lab`, DNS, GPO |
| `10.10.10.160` | `bw-win11` | Cliente Windows, unido al dominio (pendiente de red y dominio) |
| `10.10.10.170` | `bw-gns3` | GNS3 Server, CCNA (pendiente) |
| `10.10.10.172` | `bw-parrot` | Red team complementario (pendiente) |
| `10.10.10.181` | `bw-kali-purple` | Red team, pentest (pendiente) |
| `10.10.10.190` | `bwLinuxMint` | VM de prueba, IP fija |
| `10.10.10.200-240` | — | Rango DHCP de pfSense (ver nota) |
| `10.10.10.216` | `BlancoWhiteHack` | PC de administración (cable), DHCP |
| `10.10.10.225` | `AP488B-0A86-D1D0` | IP DHCP del AP Cisco 9120AXI-E (MAC `48:8b:0a:86:d1:d0`) |
| `10.10.10.240` | Cisco EWC | IP de gestión del controlador wifi (`https://10.10.10.240`) |

> `bw-omarchy` (`10.10.10.101`) se retiró del esquema: eliminada del plan el 2026-09-02.
> **Nota DHCP:** la pantalla de leases de pfSense del 2026-10-24 muestra el rango `10.10.10.200-240` (41 direcciones). El documento anterior decía `.200-.250`. **Por confirmar** cuál es el real. En cualquier caso `10.10.10.240` (EWC) está en el borde del rango y conviene reservarla.

---

## pfSense — configuración actual

| Campo | Valor |
|---|---|
| VMID | 110 |
| IP LAN | `10.10.10.1/24` |
| IP WAN | DHCP desde router Movistar (`192.168.1.x`) |
| Interfaz WAN | `vtnet1` → `vmbr1` → `eno1` → Router Movistar |
| Interfaz LAN | `vtnet0` → `vmbr0` → `eno2` → Switch |
| Versión | netgate-installer-v1.2-RELEASE |
| DHCP LAN | Activo — rango `10.10.10.200-10.10.10.240` (ver nota arriba) |
| NAT Outbound | Hybrid — regla automática para `10.10.10.0/24` |
| DNS Resolver | Domain Override `bw.lab` → `10.10.10.150`; `domain-insecure` y `private-domain` para `bw.lab` en Custom Options (2026-10-03) |
| NTP | Sirve la hora al EWC (`10.10.10.1`) |

### Reglas base pfSense
- No exponer puerto 8006 de Proxmox a internet.
- Acceso remoto via Tailscale (ver sección "Acceso remoto").
- LAN tiene acceso completo a internet via NAT.
- WAN bloquea todo el tráfico entrante no solicitado.

---

## Punto de acceso Cisco (EWC)

| Campo | Valor |
|---|---|
| Equipo | Cisco Catalyst 9120AXI-E en modo EWC, versión 17.15.6.0.770 |
| Alimentación | PoE+ (802.3at) |
| IP de gestión del EWC | `10.10.10.240/24` (Gi0), gateway `10.10.10.1` |
| IP del propio AP | `10.10.10.225`, DHCP de pfSense. Sin servidor web (80, 443 y 8443 cerrados) |
| Acceso web | `https://10.10.10.240` (certificado no confiable: el navegador avisa) |
| `mywifi.cisco.com` | Solo resuelve si el archivo `hosts` del PC apunta a `10.10.10.240`. El 2026-10-24 seguía apuntando a `192.168.1.195`, que ya no existe |
| SSIDs | `Blanco_White_Tech_5G` (solo 5 GHz) y `Blanco_White_Tech_2.4G` (solo 2.4 GHz), WPA2-PSK |
| Clientes wifi | Reciben IP y DNS de pfSense (`10.10.10.x`, DNS `10.10.10.1`) |
| NTP | `10.10.10.1`, zona CET/CEST |

### Historial de red del EWC
- Antes de convertirlo (2026-10-05): `10.10.10.225` por DHCP.
- Tras la conversión (2026-10-05): `192.168.1.195`, red Movistar. Causa sin explicar.
- Desde 2026-10-08: IP de gestión `10.10.10.240` en la LAN lab.

### Pendiente
- Reservar `10.10.10.240` (Static Mapping en pfSense o fuera del rango DHCP) para que no cambie.
- Reservar también la IP del AP (`10.10.10.225`, MAC `48:8b:0a:86:d1:d0`).
- Actualizar el `hosts` del PC de administración a `10.10.10.240`, o crear un Host Override en pfSense para `mywifi.cisco.com`.

> Más detalle del servicio en `04-SERVICIOS.md` y de la conversión en `06-INVENTARIO-Y-CHANGELOG.md`.

---

## Acceso remoto — Tailscale

| Campo | Valor |
|---|---|
| Dónde corre | Contenedor LXC 105 `bw-tailscale` en Proxmox |
| IP LAN | `10.10.10.5/24`, fija, en `vmbr0` (eth0), gateway `10.10.10.1` |
| IP Tailscale | `100.103.34.120` |
| Clientes | Tailscale instalado en los equipos del usuario, para conectarse al lab desde redes externas |
| Arranque | Automático (orden 2, después de pfSense). Ver `01-INFRAESTRUCTURA.md` |
| Dependencia | Corre dentro del R320: con el servidor apagado no hay acceso remoto |
| Estado | ✅ Operativo |

### Pendiente
- Documentar si `bw-tailscale` actúa como subnet router y qué rutas anuncia (por confirmar).
- Backup del contenedor (no está en el job de PBS).

---

## Active Directory (bw-winserver)

- Dominio `bw.lab` operativo en `10.10.10.150` (2026-10-03).
- pfSense DNS Resolver reenvía `bw.lab` a `10.10.10.150`.
- Los clientes del dominio (bw-win11) usarán `10.10.10.150` como DNS primario.
- Wazuh recibirá eventos de seguridad de AD y clientes Windows vía agentes (cuando esté desplegado).

---

## Seguridad general

- Proxmox accesible desde `192.168.1.x` (red Movistar) y `10.10.10.x` (LAN lab).
- Acceso remoto seguro mediante Tailscale (`bw-tailscale`, LXC 105). VPN propia en pfSense: sin configurar.
- Wazuh monitorizará todas las VMs cuando esté operativo.
- Validar conectividad siempre antes y después de cambios de red.
- Las credenciales no se guardan en estos documentos.

---

## Changelog

| Fecha | Acción |
|---|---|
| 2026-08-28 | Identificado problema: pfSense tenía vmbr1 sin interfaz física (eno1 sin cable). |
| 2026-08-28 | eno1 conectado al router Movistar. Topología física correcta establecida. |
| 2026-08-28 | Reinstalación limpia. vmbr0 y vmbr1 configurados desde cero. |
| 2026-08-30 | pfSense instalado y validado con WAN real. |
| 2026-09-01 | LAN migrada de `192.168.1.0/24` a `10.10.10.0/24`. pfSense LAN: `10.10.10.1`. |
| 2026-09-01 | NAT Outbound configurado en modo Hybrid para `10.10.10.0/24`. |
| 2026-09-01 | DHCP LAN activado. |
| 2026-09-01 | IP secundaria `10.10.10.254` añadida a Proxmox en vmbr0. |
| 2026-09-01 | Esquema IP final definido. IPs reservadas para bw-winserver (150) y bw-win11 (160). |
| 2026-10-03 | pfSense DNS Resolver: Domain Override `bw.lab` → `10.10.10.150`. |
| 2026-10-05 | AP Cisco 9120AXI-E: `10.10.10.225` antes de convertir a EWC y `192.168.1.195` después. Causa sin explicar. VM `linux-mint` con IP fija `10.10.10.190`. |
| 2026-10-08 | EWC con IP de gestión `10.10.10.240` en la LAN lab. WLANs `Blanco_White_Tech_5G` y `_2.4G` creadas. |
| 2026-10-24 | AP visible en pfSense con `10.10.10.225`. Documento corregido: IP del EWC, esquema IP y rango DHCP observado. |
| 2026-10-25 | Documentado `bw-tailscale` (LXC 105): `10.10.10.5/24` fija, Tailscale `100.103.34.120`. |

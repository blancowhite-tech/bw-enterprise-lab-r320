# BlancoWhite Lab — 02 Red y Firewall

> Documento actualizado 2026-09-01.

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
   └──────────── Switch Cisco ────── resto de dispositivos
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
| LAN lab | `10.10.10.0/24` | Todas las VMs del laboratorio |

> **Nota:** La LAN se migró de `192.168.1.0/24` a `10.10.10.0/24` el 2026-09-01.
> Causa: WAN (`192.168.1.x`) y LAN (`192.168.1.110`) de pfSense estaban en la misma subred — pfSense no podía enrutar entre ellas.

---

## Esquema IP

### Proxmox (fuera de la LAN lab)
| IP | Hostname | Descripción |
|---|---|---|
| `192.168.1.100` | `dell-r320-lab` | Proxmox VE — gestión |
| `10.10.10.254` | `dell-r320-lab` | IP secundaria para comunicación con VMs |

### LAN lab — 10.10.10.0/24
| IP | Hostname | Rol |
|---|---|---|
| `10.10.10.1` | `bw-pfsense` | Firewall, NAT, DHCP, DNS |
| `10.10.10.100` | `bw-infra-debian` | Docker, Portainer, Ollama, Open WebUI |
| `10.10.10.101` | `bw-omarchy` | Escritorio Linux (pendiente) |
| `10.10.10.120` | `bw-truenas` | NAS, SMB, almacenamiento |
| `10.10.10.130` | `bw-pbs` | Proxmox Backup Server |
| `10.10.10.140` | `bw-wazuh` | Wazuh SIEM (pendiente) |
| `10.10.10.150` | `bw-winserver` | Windows Server 2025, AD, DNS, GPO (pendiente) |
| `10.10.10.160` | `bw-win11` | Cliente Windows, unido al dominio (pendiente) |
| `10.10.10.181` | `bw-kali-purple` | Hacking ético, pentest, IA integrada (pendiente) |
| `10.10.10.200-250` | — | Rango DHCP (equipos temporales) |

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
| DHCP LAN | Activo — rango `10.10.10.200-10.10.10.250` |
| NAT Outbound | Hybrid — regla automática para `10.10.10.0/24` |

### Reglas base pfSense
- No exponer puerto 8006 de Proxmox a internet.
- Acceso remoto via VPN (pendiente de configurar).
- LAN tiene acceso completo a internet via NAT.
- WAN bloquea todo el tráfico entrante no solicitado.

---

## Consideraciones — Active Directory (bw-winserver)

Cuando bw-winserver esté operativo como DC:
- pfSense DNS Resolver deberá reenviar las queries del dominio AD (`bw.lab` o el que se defina) a `10.10.10.150`.
- Los clientes del dominio (bw-win11) apuntarán a `10.10.10.150` como DNS primario.
- Wazuh recibirá eventos de seguridad de AD y clientes Windows vía agentes.

---

## Seguridad general

- Proxmox accesible desde `192.168.1.x` (red Movistar) y `10.10.10.x` (LAN lab).
- VPN pendiente de configurar para acceso remoto seguro.
- Wazuh monitorizará todas las VMs cuando esté operativo.
- Validar conectividad siempre antes y después de cambios de red.

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
| 2026-09-01 | DHCP LAN activado: rango `10.10.10.200-10.10.10.250`. |
| 2026-09-01 | IP secundaria `10.10.10.254` añadida a Proxmox en vmbr0. |
| 2026-09-01 | Esquema IP final definido. IPs reservadas para bw-winserver (150) y bw-win11 (160). |

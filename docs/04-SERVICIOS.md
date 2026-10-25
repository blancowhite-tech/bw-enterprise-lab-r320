# BlancoWhite Lab — 04 Servicios

> Documento actualizado 2026-10-24.
> Todo servicio nuevo debe documentar: ubicación, IP/puerto, dependencias, persistencia, backup y recuperación.

---

## Resumen de servicios

| Servicio | VM | IP:Puerto | Estado |
|---|---|---|---|
| Proxmox WebUI | dell-r320-lab (host, no es la VM 100) | `https://192.168.1.100:8006` (también `https://10.10.10.254:8006` desde la LAN lab; ping OK, web por confirmar) | ✅ Operativo |
| pfSense WebUI | bw-pfsense | `https://10.10.10.1` | ✅ Operativo |
| TrueNAS WebUI | bw-truenas | `http://10.10.10.120` | ✅ Operativo |
| SMB | bw-truenas | `\\10.10.10.120` | ✅ Operativo |
| PBS WebUI | bw-pbs | `https://10.10.10.130:8007` | ✅ Operativo |
| Portainer | bw-infra-debian | `https://10.10.10.100:9443` | ✅ Operativo |
| Docker | bw-infra-debian | — | ✅ Operativo |
| Tailscale | bw-tailscale (LXC 105) | `10.10.10.5` (LAN) / `100.103.34.120` (Tailscale) | ✅ Operativo |
| Open WebUI | bw-infra-debian | `http://10.10.10.100:3000` | Instalado, bajo demanda |
| Active Directory / DNS (`bw.lab`) | bw-winserver | `10.10.10.150` (acceso por RDP) | ✅ Operativo (VM apagada el 2026-10-24) |
| Wazuh Dashboard | bw-wazuh | `https://10.10.10.140` | Pendiente |
| Cisco EWC WebUI | AP 9120AXI-E (no es VM) | `https://10.10.10.240` | ✅ Operativo |

---

## pfSense

| Campo | Valor |
|---|---|
| VM | bw-pfsense (VMID 110) |
| Acceso | `https://10.10.10.1` |
| Usuario | admin |
| Función | Firewall, router, DHCP, NAT, DNS Resolver |
| WAN | DHCP desde router Movistar (`192.168.1.x`) |
| LAN | `10.10.10.1/24` |
| NAT Outbound | Hybrid — regla automática para `10.10.10.0/24` |
| DHCP LAN | Rango observado `10.10.10.200 - 10.10.10.240` (2026-10-24). Por confirmar frente al `.250` documentado antes |
| DNS | Domain Override `bw.lab` → `10.10.10.150` |
| Estado | ✅ Operativo |

### Utilidad en diagnóstico
- Status → DHCP Leases muestra la IP actual de cada equipo que usa DHCP (PC de administración, AP Cisco, clientes wifi).

---

## TrueNAS

| Campo | Valor |
|---|---|
| VM | bw-truenas (VMID 120) |
| Acceso | `http://10.10.10.120` |
| Usuario admin | `truenas_admin` |
| Pool | `bw_pool` (1.57 TiB disponibles) |
| SMB | ✅ Operativo |
| Dataset NAS | `bw_pool/bw-nas` |
| Dataset backup | `bw_pool/bw-backup` |
| Estado | ✅ Operativo |

### Acceso SMB
- **Windows:** `\\10.10.10.120\bw-nas`
- **Mac:** `smb://10.10.10.120/bw-nas`

---

## Proxmox Backup Server (PBS)

| Campo | Valor |
|---|---|
| VM | bw-pbs (VMID 130) |
| Acceso | `https://10.10.10.130:8007` |
| Versión | 4.2.1 |
| Datastore | `bw-backup` |
| Storage en Proxmox | `pbs` (configurado con IP `10.10.10.130`) |
| Estado | ✅ Operativo |

---

## Docker

| Campo | Valor |
|---|---|
| VM | bw-infra-debian (VMID 100) |
| Versión | 29.7.2 |
| Datos | `/var/lib/docker` |
| Backup | via PBS |
| Estado | ✅ Operativo |

---

## Portainer

| Campo | Valor |
|---|---|
| VM | bw-infra-debian (VMID 100) |
| Acceso | `https://10.10.10.100:9443` |
| Despliegue | contenedor Docker |
| Persistencia | volumen Docker `portainer_data` |
| Backup | via PBS |
| Estado | ✅ Operativo |

### Instalación aplicada
```bash
docker volume create portainer_data
docker run -d \
  -p 8000:8000 -p 9443:9443 \
  --name portainer --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce:latest
```

---

## Ollama y Open WebUI

| Campo | Valor |
|---|---|
| VM | bw-infra-debian (VMID 100) |
| Ollama | Instalado, desactivado (systemd disabled) por RAM |
| Modelo | llama3.2 (≈ 2 GB). Mistral eliminado por RAM insuficiente |
| Open WebUI | `http://10.10.10.100:3000`, bajo demanda |
| Backup | via PBS |
| Estado | Instalado, inactivo |

---

## Active Directory y DNS (bw.lab)

| Campo | Valor |
|---|---|
| VM | bw-winserver (VMID 150) |
| IP | `10.10.10.150` |
| Dominio | `bw.lab` (NetBIOS `BW`, Windows2025Domain) |
| Servicios | NTDS, DNS, Netlogon |
| Acceso | RDP (la consola noVNC no permite pegar) |
| Dependencias | pfSense reenvía `bw.lab` a `10.10.10.150`. La VM debe estar encendida para que el dominio resuelva |
| Persistencia | Disco de la VM (40 GB en `vm-lab-blancowhite`) |
| Backup | Sin backup a PBS todavía. Solo existe el snapshot `pre-ad` |
| Estado | ✅ Operativo. VM apagada el 2026-10-24 |

---

## Tailscale (acceso remoto)

| Campo | Valor |
|---|---|
| Dónde corre | Contenedor LXC 105 `bw-tailscale` en Proxmox |
| IP LAN | `10.10.10.5/24` (fija), gateway `10.10.10.1` |
| IP Tailscale | `100.103.34.120` |
| Cómo se instala / accede | Tailscale instalado en los equipos del usuario; con él activo se conecta al lab desde fuera |
| Consola de administración | `https://login.tailscale.com/admin/machines` |
| Comprobar la IP | `pct exec 105 -- tailscale ip -4` |
| Dependencias | Red de pfSense (`10.10.10.1`). Arranca después de pfSense |
| Persistencia | Disco del contenedor y cuenta de Tailscale |
| Backup | Sin backup a PBS. Pendiente |
| Recuperación | Recrear el contenedor, instalar Tailscale y volver a autenticarlo con la cuenta; fijar `10.10.10.5/24` |
| Estado | ✅ Operativo |

### Notas
- Corre dentro del R320: con el servidor apagado no hay acceso remoto.
- Si anuncia la subred `10.10.10.0/24` (subnet router): por confirmar.

---

## Arranque automático

| Orden | ID | Servicio | Espera |
|---|---|---|---|
| 1 | VM 110 | pfSense | 60 s |
| 2 | CT 105 | Tailscale | 15 s |
| 3 | VM 120 | TrueNAS | 60 s |
| 4 | VM 130 | PBS | 30 s |
| 5 | VM 100 | bw-infra-debian (Docker, Portainer) | — |

- Configurado el 2026-10-25 con `onboot 1` y `startup order=N,up=S`.
- No arrancan solas: AD (VM 150), bw-win11 (160) y linux-mint (700). **Ojo:** si no se arranca la VM 150, `bw.lab` no resuelve.
- Tiempo estimado hasta tener todo arriba: 3-4 minutos.
- Detalle y prueba de arranque (2026-10-26) en `01-INFRAESTRUCTURA.md`.

---

## Wazuh

| Campo | Valor |
|---|---|
| VM | bw-wazuh (VMID 140) |
| Acceso | `https://10.10.10.140` |
| Instalación | all-in-one (Manager + Indexer + Dashboard) |
| Agentes | todas las VMs del lab |
| Estado | Pendiente (RAM) |

---

## Cisco EWC (Embedded Wireless Controller)

| Campo | Valor |
|---|---|
| Dónde corre | Dentro del AP Cisco Catalyst 9120AXI-E (no es una VM de Proxmox) |
| Versión | 17.15.6.0.770 (actualizado desde 17.12.8.4 el 2026-10-08 por el bug CSCwv93265; método: WebUI, modo Desktop HTTP, `install commit`) |
| IP de gestión | `10.10.10.240/24` (Gi0), gateway `10.10.10.1`. Ya no es `192.168.1.195` |
| IP del propio AP | `10.10.10.225` por DHCP de pfSense (MAC `48:8b:0a:86:d1:d0`, nombre `AP488B-0A86-D1D0`). No sirve web: 80, 443 y 8443 cerrados |
| Acceso | `https://10.10.10.240/webui/#/accessPoints`. Certificado no confiable: el navegador avisa |
| `mywifi.cisco.com` | Solo si el `hosts` del PC apunta a `10.10.10.240`. El 2026-10-24 el `hosts` seguía con `192.168.1.195` |
| Hora | NTP contra pfSense (`10.10.10.1`), zona CET/CEST |
| WLANs | `Blanco_White_Tech_5G` (ID 1, solo 5 GHz) y `Blanco_White_Tech_2.4G` (ID 2, solo 2.4 GHz). Ambas WPA2-PSK AES (elegido por compatibilidad) |
| Perfiles | Policy Profile `Blanco_White_Tech_Policy`, Policy Tag `Tag_Blanco_White_Tech` |
| Segundo AP | 9120AXI-E unido en modo CAPWAP, MAC `1006.ed52.906c`. Error DTLS resuelto con `wireless management trustpoint CISCO_IDEVID_CMCA3_SUDI`; policy tag asignada por CLI |
| Capacidad | Hasta 100 APs (según el arranque del EWC) |
| Dependencias | PoE+ (802.3at) para el AP; DHCP y NTP de pfSense |
| Persistencia | Configuración almacenada en el propio AP |
| Backup | Copia local en bootflash: `EWC_backup_final.cfg`. Copia fuera del AP: pendiente. PBS no lo cubre porque no es una VM |
| Estado | ✅ Operativo |

### Instalación aplicada (conversión a EWC, 2026-10-05)
Desde la consola serie del AP (PuTTY, COM3, 9600), con Tftpd64 sirviendo `C:\tftp` desde el PC `10.10.10.216`:
```
ap-type ewc-ap tftp://10.10.10.216/ap1g7 tftp://10.10.10.216/C9800-AP-iosxe-wlc.bin
```
El AP descargó las dos imágenes, las instaló y se reinició como controladora (con la versión 17.12.08).

### Recuperación si falla
1. Reset de fábrica con el botón físico del AP (mantener 20-30 s).
2. Repetir la conversión con el procedimiento de arriba y el zip `C9800-AP-universalk9.17.12.08.zip`.
3. Actualizar a 17.15.6 por WebUI y restaurar la configuración desde `EWC_backup_final.cfg` (por confirmar el procedimiento exacto).

### Si no abre la web
1. Mirar en pfSense (Status → DHCP Leases) por si el AP o el EWC han cambiado de IP.
2. Comprobar `Test-NetConnection 10.10.10.240 -Port 443`.
3. Comprobar que el `hosts` del PC apunta a `10.10.10.240` si se usa `mywifi.cisco.com`.

### Notas
- Credenciales y PSK de las WLAN: no se guardan en este documento.
- La `10.10.10.240` está en el borde del rango DHCP de pfSense. Pendiente: reservarla o sacarla del rango.

---

## Regla operativa

Todo servicio nuevo debe documentar antes de pasar al siguiente:
1. VM y IP:puerto donde corre
2. Cómo se instala
3. Cómo se accede
4. Qué persiste y dónde
5. Cómo se hace backup
6. Cómo se recupera si falla

---

## Changelog

| Fecha | Servicio | Acción |
|---|---|---|
| 2026-08-28 | Todos | Reinicio desde cero. |
| 2026-08-30 | pfSense | Instalado y validado. |
| 2026-08-30 | PBS | Instalado. Datastore `bw-backup` operativo. |
| 2026-08-30 | TrueNAS + SMB | Operativo. Pool `bw_pool` creado. |
| 2026-09-01 | pfSense | LAN migrada a `10.10.10.1`. NAT Hybrid activo. |
| 2026-09-01 | PBS | IP actualizada a `10.10.10.130`. Storage PBS reconfigurado en Proxmox. |
| 2026-09-01 | TrueNAS | IP actualizada a `10.10.10.120`. |
| 2026-09-01 | Docker | Instalado en bw-infra-debian. Versión 29.7.2. |
| 2026-09-01 | Portainer | Instalado y operativo en `https://10.10.10.100:9443`. |
| 2026-09-02 | Ollama / Open WebUI | Instalados en bw-infra-debian, inactivos por RAM. |
| 2026-10-03 | Active Directory | Dominio `bw.lab` operativo en `10.10.10.150`. pfSense reenvía `bw.lab`. |
| 2026-10-05 | Cisco EWC | AP 9120AXI-E convertido a EWC 17.12.08. Operativo en `192.168.1.195`. |
| 2026-10-08 | Cisco EWC | IP de gestión movida a `10.10.10.240`. Actualizado a 17.15.6.0.770. NTP y WLANs configuradas. Segundo AP unido por CAPWAP. Backup local `EWC_backup_final.cfg`. |
| 2026-10-24 | Documento | Corregida la IP del EWC (`10.10.10.240`) y la versión. Añadidos AD, Ollama y Open WebUI. PBS a 4.2.1. |
| 2026-10-25 | Tailscale | Documentado el contenedor `bw-tailscale` (LXC 105). |
| 2026-10-25 | Arranque automático | Configurado para 110, 105, 120, 130 y 100. |

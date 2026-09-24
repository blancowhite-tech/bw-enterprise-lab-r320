# BlancoWhite Lab — 04 Servicios

> Documento actualizado 2026-09-01.
> Todo servicio nuevo debe documentar: ubicación, IP/puerto, dependencias, persistencia, backup y recuperación.

---

## Resumen de servicios

| Servicio | VM | IP:Puerto | Estado |
|---|---|---|---|
| Proxmox WebUI | dell-r320-lab | `https://192.168.1.100:8006` | ✅ Operativo |
| pfSense WebUI | bw-pfsense | `https://10.10.10.1` | ✅ Operativo |
| TrueNAS WebUI | bw-truenas | `http://10.10.10.120` | ✅ Operativo |
| SMB | bw-truenas | `\\10.10.10.120` | ✅ Operativo |
| PBS WebUI | bw-pbs | `https://10.10.10.130:8007` | ✅ Operativo |
| Portainer | bw-infra-debian | `https://10.10.10.100:9443` | ✅ Operativo |
| Docker | bw-infra-debian | — | ✅ Operativo |
| Wazuh Dashboard | bw-wazuh | `https://10.10.10.140` | Pendiente |

---

## pfSense

| Campo | Valor |
|---|---|
| VM | bw-pfsense (VMID 110) |
| Acceso | `https://10.10.10.1` |
| Usuario | admin |
| Función | Firewall, router, DHCP, NAT |
| WAN | DHCP desde router Movistar (`192.168.1.x`) |
| LAN | `10.10.10.1/24` |
| NAT Outbound | Hybrid — regla automática para `10.10.10.0/24` |
| DHCP LAN | `10.10.10.200 - 10.10.10.250` |
| Estado | ✅ Operativo |

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
| Versión | 4.2.0 |
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

## Wazuh

| Campo | Valor |
|---|---|
| VM | bw-wazuh (VMID 140) |
| Acceso | `https://10.10.10.140` |
| Instalación | all-in-one (Manager + Indexer + Dashboard) |
| Agentes | todas las VMs del lab |
| Estado | Pendiente |

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

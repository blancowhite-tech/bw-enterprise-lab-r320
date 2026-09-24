# BlancoWhite Lab — 05 Backup y Monitorización

> Documento actualizado 2026-09-01.

---

## Proxmox Backup Server (PBS)

### Datos de la VM

| Campo | Valor |
|---|---|
| VMID | 130 |
| Nombre | `bw-pbs` |
| IP | `10.10.10.130` |
| Acceso WebUI | `https://10.10.10.130:8007` |
| OS | Proxmox Backup Server 4.2.0 |
| RAM | 4 GB |
| CPU | 2 vCPU |
| Disco sistema | ~18.5 GB |
| Firmware | OVMF / UEFI |
| Datastore | `bw-backup` |
| Estado | ✅ Operativo |

### Estado actual
- Datastore `bw-backup` operativo.
- Backup de bw-pfsense completado sin errores.
- Backup de bw-pbs completado el 2026-09-01 sin errores.
- Storage `pbs` configurado en Proxmox con IP `10.10.10.130`.

### Integración con Proxmox

PBS añadido como storage en Proxmox:

1. Proxmox WebUI → Datacenter → Storage → Add → Proxmox Backup Server
2. ID: `pbs`
3. Server: `10.10.10.130`
4. Datastore: `bw-backup`
5. Usuario: `root@pam`

---

## Política de backups

### Regla general
- Hacer backup antes de cualquier cambio importante en una VM.
- Verificar que el backup es válido antes de continuar.
- Nunca trabajar sobre una VM sin backup reciente.

### Programación automática

| VM | Frecuencia | Retención diaria | Retención semanal | Retención mensual |
|---|---|---|---|---|
| bw-pfsense (110) | Diaria | 7 | 4 | 2 |
| bw-truenas (120) | Diaria | 7 | 4 | 2 |
| bw-pbs (130) | Diaria | 7 | 4 | 2 |
| bw-infra-debian (100) | Diaria | 7 | 4 | 2 |
| bw-wazuh (140) | Diaria | 7 | 4 | 1 |
| bw-winserver (150) | Diaria | 7 | 4 | 2 |
| bw-win11 (160) | Diaria | 7 | 4 | 1 |
| bw-omarchy (101) | Semanal | — | 4 | 1 |
| bw-kali-purple (181) | Semanal | — | 4 | 1 |

---

## Monitorización — Wazuh (SIEM)

| Campo | Valor |
|---|---|
| VMID | 140 |
| Nombre | `bw-wazuh` |
| IP | `10.10.10.140` |
| Acceso WebUI | `https://10.10.10.140` |
| Instalación | All-in-one (Manager + Indexer + Dashboard) |
| OS | Ubuntu 22.04 LTS |
| Estado | Pendiente |

### Agentes previstos

| VM | OS | Método |
|---|---|---|
| bw-pfsense (110) | FreeBSD/pfSense | Plugin pfSense o syslog |
| bw-truenas (120) | TrueNAS SCALE | Syslog → Wazuh |
| bw-pbs (130) | Debian | Agente Wazuh |
| bw-infra-debian (100) | Debian | Agente Wazuh |
| bw-winserver (150) | Windows Server 2025 | Agente Wazuh |
| bw-win11 (160) | Windows 11 | Agente Wazuh |
| bw-omarchy (101) | Linux | Agente Wazuh |
| bw-kali-purple (181) | Kali Linux | Agente Wazuh |

---

## Changelog

| Fecha | Acción |
|---|---|
| 2026-08-30 | PBS 4.2.0 instalado y operativo. Datastore `bw-backup` configurado. |
| 2026-08-30 | Primer backup de bw-pfsense completado sin errores. |
| 2026-09-01 | IP de PBS migrada a `10.10.10.130`. Storage `pbs` reconfigurado en Proxmox. |
| 2026-09-01 | Backup de bw-pbs completado sin errores tras la migración. |
| 2026-09-01 | Política de backups ampliada con bw-winserver (150) y bw-win11 (160). |
| 2026-09-01 | Tabla de agentes Wazuh previstos actualizada con infraestructura final. |

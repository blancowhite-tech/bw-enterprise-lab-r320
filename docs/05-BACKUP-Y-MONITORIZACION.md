# BlancoWhite Lab — 05 Backup y Monitorización

> Documento actualizado 2026-10-24.
> Los datos marcados como "por confirmar" no se han verificado todavía.

---

## Proxmox Backup Server (PBS)

### Datos de la VM

| Campo | Valor |
|---|---|
| VMID | 130 |
| Nombre | `bw-pbs` |
| IP | `10.10.10.130` |
| Acceso WebUI | `https://10.10.10.130:8007` |
| OS | Proxmox Backup Server 4.2.1 |
| RAM | 2 GB |
| CPU | 2 vCPU (x86-64-v2-AES) |
| Disco | scsi0 — 240 GB en `vm-lab-blancowhite` (234 GB útiles; 17 GB usados el 2026-09-21) |
| Firmware | OVMF / UEFI |
| Datastore | `bw-backup` (`/mnt/bw-backup`, GC diario) |
| Estado | ✅ Operativo |

### Estado actual
- Datastore `bw-backup` operativo.
- Storage `pbs` configurado en Proxmox con IP `10.10.10.130`.
- Backup de bw-pfsense completado sin errores (2026-08-30).
- Configuración de PBS respaldada en `pbs-config.md` (ver más abajo).

### Integración con Proxmox

PBS añadido como storage en Proxmox:

1. Proxmox WebUI → Datacenter → Storage → Add → Proxmox Backup Server
2. ID: `pbs`
3. Server: `10.10.10.130`
4. Datastore: `bw-backup`
5. Usuario: `root@pam`

### Recuperación de bw-pbs
- No se hace backup de la VM completa: su disco contiene el datastore y respaldar backups dentro de backups no tiene sentido.
- En su lugar, `pbs-config.md` guarda el hardware de la VM, `/etc/network/interfaces`, `datastore.cfg` y los pasos para reconstruirla.
- **Por confirmar:** si la VM 130 está incluida o no en el job semanal (ver abajo). Este documento y `pbs-config.md` se contradicen: `03-MAQUINAS-VIRTUALES.md` y el changelog de este documento del 2026-10-06 la incluyen; `pbs-config.md` dice que no se respalda la VM completa.

---

## Política de backups

### Regla general
- Hacer backup antes de cualquier cambio importante en una VM.
- Verificar que el backup es válido antes de continuar.
- Nunca trabajar sobre una VM sin backup reciente.

### Job real (aplicado)

| Campo | Valor |
|---|---|
| Frecuencia | Semanal, domingos 01:00 |
| VMs incluidas | 100 (bw-infra-debian), 110 (bw-pfsense), 120 (bw-truenas), 130 (bw-pbs, por confirmar) |
| Retención | keep last 2 + daily 7 |
| Destino | storage `pbs` |

### Pendiente de incluir en el job
- bw-winserver (150): sin backup a PBS todavía. De momento solo tiene el snapshot `pre-ad`, que hay que borrar una vez haya backup. Tiene el AD del lab, así que es la prioridad.
- bw-win11 (160): sin backup a PBS todavía.
- linux-mint (700): VM de prueba, sin backup previsto.
- bw-tailscale (CT 105): contenedor de acceso remoto, sin backup a PBS todavía. Pendiente decidir si se incluye en el job (los contenedores LXC también se respaldan con PBS).

### Plan original (no aplicado)
La política anterior preveía backups diarios con retención 7/4/2 (o 7/4/1) para las VMs de infraestructura y semanales para Kali Purple. No se ha aplicado; el job real es el semanal de arriba. Si se quiere volver a ese esquema, hay que decidirlo y registrarlo aquí.

### Fuera del alcance de PBS
- **Cisco EWC (AP 9120AXI-E):** no es una VM, PBS no lo cubre. Existe una copia local en el bootflash del AP (`EWC_backup_final.cfg`), que se pierde si el AP se resetea. Pendiente: sacar una copia a otro sitio (por ejemplo, el NAS).
- **Credenciales:** no se guardan en estos documentos.
- **Configuración de pfSense:** exportación del XML por confirmar.

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
| RAM prevista | 8 GB |
| Estado | Pendiente |

> Pendiente de la ampliación de RAM (el lab tiene 32 GB; la ampliación a 64 GB está sin comprar). Con las VMs actuales encendidas no cabe.

### Agentes previstos

| VM | OS | Método |
|---|---|---|
| bw-pfsense (110) | FreeBSD/pfSense | Plugin pfSense o syslog |
| bw-truenas (120) | TrueNAS SCALE | Syslog → Wazuh |
| bw-pbs (130) | Debian | Agente Wazuh |
| bw-infra-debian (100) | Debian | Agente Wazuh |
| bw-winserver (150) | Windows Server 2025 | Agente Wazuh (eventos de seguridad de AD) |
| bw-win11 (160) | Windows 11 | Agente Wazuh |
| bw-kali-purple (181) | Kali Linux | Agente Wazuh |

> bw-omarchy (101) se ha quitado de la lista: eliminada del plan el 2026-09-02.

---

## Changelog

| Fecha | Acción |
|---|---|
| 2026-08-30 | PBS instalado y operativo. Datastore `bw-backup` configurado. |
| 2026-08-30 | Primer backup de bw-pfsense completado sin errores. |
| 2026-09-01 | IP de PBS migrada a `10.10.10.130`. Storage `pbs` reconfigurado en Proxmox. |
| 2026-09-01 | Tabla de agentes Wazuh previstos actualizada con infraestructura final. |
| 2026-09-21 | `pbs-config.md` generado: configuración de PBS y pasos de recuperación. |
| 2026-10-06 | Documento revisado: PBS a 4.2.1, 2 GB de RAM y disco de 240 GB. Job semanal real (domingos 01:00, VMs 100, 110, 120 y 130) sustituye a la política diaria del plan. bw-omarchy retirada. |
| 2026-10-08 | EWC con copia local de configuración en bootflash (`EWC_backup_final.cfg`). |
| 2026-10-24 | Documento revisado: backup del EWC fuera de PBS, prioridad de respaldar bw-winserver y contradicción sobre la VM 130 señalada. |
| 2026-10-25 | Anotado que bw-tailscale (CT 105) no está en el job de backup. Arranque automático de 110, 105, 120, 130 y 100 configurado (ver `01-INFRAESTRUCTURA.md`). |

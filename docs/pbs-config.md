# BlancoWhite Lab — PBS Configuration Backup
> Documento generado: 2026-09-21
> Propósito: recuperación rápida de bw-pbs si la VM falla. No se hace backup de la VM completa porque el disco de 234 GB contiene el datastore — respaldar backups dentro de backups no tiene sentido.

---

## Hardware de la VM (VMID 130)

| Campo | Valor |
|---|---|
| Nombre | bw-pbs |
| VMID | 130 |
| IP | 10.10.10.130/24 |
| RAM | 2 GB |
| CPU | 2 vCPU (x86-64-v2-AES) |
| Disco | scsi0 — 240 GB en vm-lab-blancowhite |
| EFI Disk | 4M |
| BIOS | OVMF (UEFI) |
| Machine | q35 |
| OS | Proxmox Backup Server 4.2.1 |
| ISO instalación | proxmox-backup-server_4.2-1.iso |

---

## /etc/network/interfaces

```
auto lo
iface lo inet loopback

auto nic0
iface nic0 inet static
        address 10.10.10.130/24
        gateway 10.10.10.1
        dns-nameservers 1.1.1.1 8.8.8.8

source /etc/network/interfaces.d/*
```

---

## /etc/proxmox-backup/datastore.cfg

```
datastore: bw-backup
        comment
        gc-schedule daily
        notification-mode notification-system
        path /mnt/bw-backup
```

---

## Espacio en disco

| Filesystem | Size | Used | Avail | Use% |
|---|---|---|---|---|
| /dev/mapper/pbs-root | 234G | 17G | 208G | 8% |

---

## Pasos de recuperación si falla la VM

1. Crear nueva VM en Proxmox con los mismos parámetros de hardware
2. Instalar Proxmox Backup Server 4.2.1 desde ISO
3. Configurar red con los valores de `/etc/network/interfaces` de este documento
4. Montar el disco de datos si se conservó, o crear datastore nuevo
5. Restaurar configuración del datastore desde `/etc/proxmox-backup/datastore.cfg`
6. Reconfigurar storage `pbs` en Proxmox WebUI apuntando a `10.10.10.130`

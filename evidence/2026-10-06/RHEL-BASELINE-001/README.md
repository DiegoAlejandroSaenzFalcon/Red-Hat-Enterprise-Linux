# RHEL-BASELINE-001 — Baseline técnica y auditoría inicial

**Fecha:** 2026-10-06  
**Estado:** EVIDENCED / DOCUMENTED  
**Alcance:** observación y diagnóstico read-only del RHEL físico actual mediante Desktop Commander Remote.

## Objetivo

Establecer una línea base verificable del sistema antes de realizar cambios de arquitectura, seguridad, acceso remoto o AICCP.

## Método

La inspección se realizó directamente sobre el host RHEL 10.2 mediante Desktop Commander Remote. No se modificó almacenamiento, boot, firewall, SELinux, SSH, systemd ni configuración del agente durante esta auditoría.

## Resultado

- RHEL 10.2 (Coughlan)
- Kernel 6.12.0-211.62.1.el10_2.x86_64
- multi-user.target
- 0 unidades systemd fallidas
- SELinux Enforcing
- firewalld activo, zona public
- SSH activo en TCP/22
- Wi-Fi operativo
- Node.js 22.23.2
- npm 10.9.8
- Python 3.12.14
- Desktop Commander Remote online
- Memoria disponible aproximada: 6.1 GiB de 7.2 GiB
- Raíz: 5% usada
- /home: 3% usado
- Swap: 7.6 GiB, sin uso

## Hallazgos pendientes

1. RTC configurado en hora local; requiere evaluación antes de modificarlo, especialmente por dual boot.
2. El medio de instalación continúa montado en /mnt/rheliso y /mnt/installimg; no desmontar ni modificar sin autorización.
3. DNF informa que los repositorios de Subscription Management no están actualizados; requiere auditoría de registro/repositorios.
4. SSH está activo y escuchando en TCP/22; requiere revisión de exposición y política antes de habilitar acceso remoto móvil.
5. Desktop Commander está ejecutándose en TTY2 de forma interactiva; no se ha creado todavía un servicio persistente.

## Regla

Esta línea base describe estado observado el 2026-10-06. No representa una garantía permanente del sistema.

# RHEL / Linux Command Reference

> **Purpose:** aprender a leer, ejecutar y verificar comandos reales de RHEL/Linux. Esta referencia no pretende enseñar a copiar bloques a ciegas: cada comando debe entenderse antes de automatizarlo.

## Regla de operación

**COMANDO → RESULTADO → INTERPRETACIÓN → VERIFICACIÓN → EVIDENCIA**

- Ejecutar primero como usuario normal cuando sea posible.
- Usar `sudo` solamente cuando la operación lo requiera.
- No ejecutar comandos destructivos sin autorización explícita.
- Antes de combinar comandos, entender cada componente.
- En producción, validar el alcance, la reversibilidad y el resultado.

---

## 1. Orientación en la terminal

| Comando | Para qué sirve | Ejemplo |
|---|---|---|
| `pwd` | muestra el directorio actual | `pwd` |
| `ls` | lista archivos/directorios | `ls -la` |
| `cd` | cambia de directorio | `cd /etc` |
| `tree` | muestra estructura de directorios | `tree -L 2` |
| `clear` | limpia la pantalla | `clear` |
| `history` | muestra historial de comandos | `history` |
| `man` | manual de un comando | `man ls` |
| `info` | documentación Info | `info coreutils` |
| `which` | localiza un ejecutable | `which python3` |
| `type` | identifica cómo interpreta Bash un nombre | `type cd` |
| `help` | ayuda de comandos internos de Bash | `help cd` |

### Primeras diferencias importantes

- `cd` es un **builtin de Bash**, no un ejecutable externo.
- `whoami` es un comando que consulta la identidad efectiva.
- `echo` imprime texto o valores.
- `man` permite estudiar un comando antes de utilizarlo.

---

## 2. Identidad, usuarios y permisos

| Comando | Para qué sirve |
|---|---|
| `whoami` | usuario efectivo |
| `id` | UID, GID y grupos |
| `groups` | grupos del usuario |
| `passwd` | cambiar contraseña |
| `sudo -l` | permisos disponibles mediante sudo |
| `sudo <comando>` | ejecutar un comando con privilegios |
| `su -` | cambiar de usuario |
| `last` | historial de sesiones |
| `w` | usuarios conectados y actividad |
| `who` | sesiones actuales |

**Seguridad:** nunca compartir contraseñas, tokens, claves privadas ni secretos con documentación, IA o Git.

---

## 3. Archivos y directorios

| Comando | Para qué sirve |
|---|---|
| `touch` | crea un archivo vacío o actualiza su fecha |
| `mkdir` | crea directorios |
| `cp` | copia |
| `mv` | mueve o renombra |
| `rm` | elimina |
| `rmdir` | elimina directorios vacíos |
| `ln` | crea enlaces |
| `file` | identifica el tipo de archivo |
| `stat` | muestra metadatos |
| `find` | busca archivos |
| `locate` | búsqueda indexada, si está disponible |

### Lectura de archivos

| Comando | Para qué sirve |
|---|---|
| `cat` | muestra contenido |
| `less` | lectura paginada |
| `head` | primeras líneas |
| `tail` | últimas líneas |
| `tail -f` | seguimiento de cambios |
| `wc` | cuenta líneas, palabras y bytes |

---

## 4. Texto y procesamiento

| Comando | Para qué sirve |
|---|---|
| `echo` | imprime texto/variables |
| `printf` | salida formateada |
| `grep` | busca patrones |
| `sort` | ordena líneas |
| `uniq` | elimina/reporta duplicados consecutivos |
| `cut` | extrae columnas/campos |
| `tr` | transforma caracteres |
| `sed` | transformación de texto por flujo |
| `awk` | procesamiento estructurado por campos |
| `tee` | muestra y escribe simultáneamente |

Estas herramientas son fundamentales para comprender los comandos complejos generados por automatizaciones e IA.

---

## 5. Variables, entorno y Bash

| Construcción | Significado |
|---|---|
| `VAR=valor` | asigna una variable |
| `echo $VAR` | muestra su valor |
| `printenv` | muestra variables de entorno |
| `env` | muestra/ejecuta con entorno |
| `export VAR=valor` | exporta una variable |
| `$HOME` | directorio personal |
| `$USER` | usuario |
| `$PATH` | rutas donde Bash busca ejecutables |
| `$?` | código de salida del comando anterior |
| `$(comando)` | sustitución de comando |
| `&&` | ejecuta lo siguiente si lo anterior tuvo éxito |
| `||` | ejecuta lo siguiente si lo anterior falló |
| `;` | separa comandos |
| `>` | redirige/sobrescribe salida |
| `>>` | redirige/anexa salida |
| `2>` | redirige errores |
| `|` | pipe: conecta salida con entrada |

### Ejemplo didáctico

```bash
echo "Usuario: $USER"
```

Aquí `echo` es el comando y `$USER` es una variable de entorno.

---

## 6. Procesos

| Comando | Para qué sirve |
|---|---|
| `ps` | procesos |
| `ps aux` | vista amplia de procesos |
| `top` | procesos y consumo en tiempo real |
| `free -h` | memoria |
| `uptime` | tiempo activo y carga |
| `pgrep` | busca procesos por nombre |
| `pkill` | solicita terminación por nombre |
| `kill` | envía una señal a un proceso |
| `nice` / `renice` | prioridad de procesos |

**Precaución:** `kill`, `pkill` y cambios de prioridad pueden afectar servicios. Verificar primero.

---

## 7. Sistema y hardware

| Comando | Para qué sirve |
|---|---|
| `uname -a` | kernel y arquitectura |
| `hostnamectl` | identidad del sistema |
| `hostname` | nombre del host |
| `lscpu` | CPU |
| `lsmem` | memoria |
| `lsblk` | dispositivos de bloque |
| `lspci` | dispositivos PCI |
| `lsusb` | dispositivos USB |
| `dmidecode` | información DMI/firmware; normalmente requiere sudo |
| `df -hT` | espacio y tipos de filesystem |
| `du -sh` | tamaño ocupado |
| `free -h` | memoria |
| `date` | fecha/hora |
| `timedatectl` | configuración de hora |

---

## 8. Paquetes RHEL — DNF

| Comando | Para qué sirve |
|---|---|
| `dnf search <texto>` | busca paquetes |
| `dnf info <paquete>` | información |
| `dnf list installed` | paquetes instalados |
| `dnf repolist` | repositorios |
| `dnf provides <archivo>` | encuentra qué paquete proporciona un archivo |
| `dnf install <paquete>` | instala |
| `dnf remove <paquete>` | elimina |
| `dnf update` | actualiza paquetes |
| `dnf history` | historial de transacciones |

**Regla del laboratorio:** no instalar, eliminar ni actualizar paquetes durante una auditoría sin registrar antes el motivo y el estado.

---

## 9. Servicios — systemd

| Comando | Para qué sirve |
|---|---|
| `systemctl status <servicio>` | estado |
| `systemctl start <servicio>` | iniciar |
| `systemctl stop <servicio>` | detener |
| `systemctl restart <servicio>` | reiniciar |
| `systemctl enable <servicio>` | habilitar al arranque |
| `systemctl disable <servicio>` | deshabilitar al arranque |
| `systemctl is-enabled <servicio>` | comprobar habilitación |
| `systemctl is-active <servicio>` | comprobar actividad |
| `systemctl --failed` | unidades fallidas |
| `systemctl list-units` | unidades cargadas |
| `systemctl list-unit-files` | archivos de unidades |

---

## 10. Logs — journald

| Comando | Para qué sirve |
|---|---|
| `journalctl` | consulta el journal |
| `journalctl -b` | logs del arranque actual |
| `journalctl -p err` | errores |
| `journalctl -u <servicio>` | logs de un servicio |
| `journalctl -f` | seguimiento en tiempo real |
| `journalctl --since "1 hour ago"` | ventana temporal |

---

## 11. Red

| Comando | Para qué sirve |
|---|---|
| `ip addr` | interfaces y direcciones |
| `ip link` | estado de interfaces |
| `ip route` | tabla de rutas |
| `nmcli general status` | estado de NetworkManager |
| `nmcli device status` | dispositivos |
| `nmcli connection show` | conexiones |
| `ping` | conectividad ICMP |
| `ss -tulpn` | sockets/puertos escuchando |
| `curl` | solicitudes HTTP |
| `dig` | consultas DNS, si está instalado |
| `resolvectl` | diagnóstico DNS cuando corresponda |

---

## 12. Firewall y seguridad

| Comando | Para qué sirve |
|---|---|
| `firewall-cmd --state` | estado de firewalld |
| `firewall-cmd --get-active-zones` | zonas activas |
| `firewall-cmd --list-all` | configuración de zona |
| `getenforce` | estado SELinux |
| `sestatus` | estado detallado SELinux |
| `ausearch` | búsqueda en audit logs |
| `auditctl` | consulta/configuración de auditoría; requiere cuidado |

**Nunca desactivar SELinux o firewalld para “hacer que algo funcione” sin analizar primero la causa.**

---

## 13. SSH

| Comando | Para qué sirve |
|---|---|
| `ssh usuario@host` | conexión SSH |
| `ssh-keygen` | genera claves |
| `ssh-copy-id` | instala clave pública, si está disponible |
| `ssh -v` | diagnóstico detallado |
| `sshd -t` | valida configuración de sshd |
| `systemctl status sshd` | estado del servicio |

Las claves privadas nunca deben entrar en Git, documentación pública, capturas ni prompts.

---

## 14. Almacenamiento

| Comando | Para qué sirve |
|---|---|
| `lsblk -f` | discos, particiones y filesystem |
| `blkid` | UUID/tipos de filesystem |
| `findmnt` | montajes |
| `df -hT` | espacio |
| `du -sh` | consumo |
| `mount` | montajes actuales / operación de montaje |
| `umount` | desmontaje |
| `vgs` | grupos LVM |
| `lvs` | volúmenes lógicos |
| `pvs` | volúmenes físicos |

**Particionado, formateo, montaje persistente y operaciones LVM son operaciones de alto riesgo.**

---

## 15. Git

| Comando | Para qué sirve |
|---|---|
| `git status` | estado del repositorio |
| `git log` | historial |
| `git diff` | cambios |
| `git branch` | ramas |
| `git switch` | cambiar/crear ramas |
| `git add` | preparar cambios |
| `git commit` | crear commit |
| `git pull` | traer cambios |
| `git push` | publicar cambios |
| `git remote -v` | remotos |
| `git restore` | restaurar archivos |
| `git stash` | guardar cambios temporalmente |

---

## 16. Python y desarrollo

| Comando | Para qué sirve |
|---|---|
| `python3 --version` | versión |
| `python3 -m venv .venv` | entorno virtual |
| `source .venv/bin/activate` | activa entorno |
| `deactivate` | sale del entorno |
| `python3 -m pip` | gestor pip asociado a Python |
| `git` | control de versiones |
| `ssh` | acceso seguro |
| `tar` | archivado |
| `gzip` | compresión |

---

## 17. Contenedores — Podman

| Comando | Para qué sirve |
|---|---|
| `podman --version` | versión |
| `podman images` | imágenes |
| `podman ps` | contenedores activos |
| `podman ps -a` | todos los contenedores |
| `podman pull` | descarga imagen |
| `podman run` | crea/ejecuta contenedor |
| `podman stop` | detiene |
| `podman start` | inicia |
| `podman logs` | logs |
| `podman inspect` | detalles |
| `podman exec` | ejecuta dentro de un contenedor |

---

## 18. Compresión y transferencia

| Comando | Para qué sirve |
|---|---|
| `tar -tf archivo.tar` | lista contenido |
| `tar -xf archivo.tar` | extrae |
| `tar -czf archivo.tar.gz directorio/` | crea tar.gz |
| `gzip` | comprime |
| `gunzip` | descomprime |
| `rsync` | sincronización eficiente |
| `scp` | copia mediante SSH |

---

## 19. Automatización Bash

Conceptos que deben dominarse antes de escribir automatizaciones:

1. comandos y argumentos;
2. variables;
3. códigos de salida;
4. redirecciones;
5. pipes;
6. condiciones;
7. bucles;
8. funciones;
9. parámetros;
10. manejo de errores;
11. permisos;
12. logs;
13. rollback.

### Estructura mínima

```bash
#!/bin/bash

echo "Inicio"

comando

echo "Fin"
```

Una IA puede generar scripts mucho más complejos, pero el operador debe poder explicar qué hace cada línea antes de ejecutarlos.

---

## 20. Diagnóstico rápido

Cuando algo falla, no empezar cambiando cosas al azar.

Primero preguntar:

- ¿Qué cambió?
- ¿Qué servicio está afectado?
- ¿Qué dicen los logs?
- ¿Qué proceso está ejecutándose?
- ¿Qué puertos están abiertos?
- ¿Qué configuración está activa?
- ¿Hay un problema de permisos?
- ¿SELinux registró un rechazo?
- ¿Hay conectividad?
- ¿Existe una operación reciente en `dnf history`?

Comandos habituales:

```text
systemctl --failed
journalctl -p err
df -hT
free -h
ip addr
ip route
ss -tulpn
getenforce
sestatus
dnf history
```

---

## 21. Sintaxis que debemos aprender a leer

Una orden compleja de IA no debe tratarse como una caja negra.

Ejemplo:

```bash
comando --opcion valor | grep texto > resultado.txt
```

Se puede descomponer como:

1. `comando` — programa principal.
2. `--opcion valor` — argumentos.
3. `|` — pipe.
4. `grep texto` — filtra la salida.
5. `>` — guarda la salida en un archivo.

El objetivo del laboratorio es llegar a poder **leer esta gramática sin depender de copiar/pegar**.

---

## 22. Comandos de alto riesgo

No ejecutar automáticamente:

- `rm -rf`
- `mkfs`
- `dd`
- `fdisk`
- `parted`
- `wipefs`
- `mount` / `umount` sobre sistemas críticos
- modificaciones de bootloader
- cambios de particiones
- eliminación masiva de paquetes
- desactivación de SELinux/firewalld
- cambios de sudoers
- comandos obtenidos de fuentes no confiables

**El hecho de que un comando aparezca en una respuesta de IA no constituye autorización para ejecutarlo.**

---

## 23. Metacomandos para aprender cualquier comando

Antes de usar un comando nuevo:

```bash
comando --help
man comando
type comando
which comando
```

No todos aplican a todos los comandos. La idea es aprender a **investigar primero y ejecutar después**.

---

## 24. Ruta de aprendizaje

### Nivel 1 — Supervivencia
`pwd`, `ls`, `cd`, `cat`, `less`, `cp`, `mv`, `mkdir`, `rm`, `man`

### Nivel 2 — Administración
`sudo`, `id`, `dnf`, `systemctl`, `journalctl`, `hostnamectl`

### Nivel 3 — Red
`ip`, `nmcli`, `ss`, `ping`, `curl`, DNS

### Nivel 4 — Seguridad
SELinux, firewalld, SSH, permisos, audit

### Nivel 5 — Automatización
Bash, pipes, redirecciones, variables, condiciones, loops

### Nivel 6 — Infraestructura
LVM, storage, systemd, Podman, backups, monitoring

### Nivel 7 — IA
Interpretar comandos generados por IA, revisar riesgos, ejecutar con mínimo privilegio, verificar resultados y generar evidencia.

---

## Relación con el laboratorio

Esta referencia es material de aprendizaje **y** una base para las operaciones del laboratorio.

Cada práctica real debe enlazar:

**comando → objetivo → resultado esperado → verificación → evidencia → documentación**

La IA debe ayudar a comprender y automatizar, no sustituir la comprensión del administrador.

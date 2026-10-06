# SOLUCIONATIA-001 — Recuperación de Wi-Fi en RHEL 10.2

## Estado

**Status:** VERIFIED / EVIDENCED / DOCUMENTED  
**Fecha:** 2026-10-06  
**Objetivo:** recuperar la conectividad Wi-Fi de RHEL 10.2 en un sistema mínimo/headless sin asumir que el hardware estaba defectuoso.

## Alcance

Este registro cubre exclusivamente la incidencia de Wi-Fi. No incluye Node.js, Desktop Commander Remote ni otros trabajos posteriores.

## Síntoma

La interfaz Intel Wi-Fi `wlp0s20f3` existía pero NetworkManager la mostraba como **sin gestión (unmanaged)** y no había conectividad por Wi-Fi.

## Diagnóstico observado

- Adaptador PCI Intel CNVi Wi-Fi, dispositivo `00:14.3`, ID observado `0274`.
- Kernel driver in use: `iwlwifi`.
- Módulo: `iwlwifi`.
- Interfaz: `wlp0s20f3`.
- `wpa_supplicant`: no instalado.
- `NetworkManager-wifi`: no instalado.
- Repositorios configurados: BaseOS y AppStream.
- Sin conectividad inicial: no había ruta de red ni resolución DNS hacia los repositorios.
- USB tethering del teléfono proporcionó conectividad temporal y DNS funcional.

## Estrategias consideradas

### Método A — recuperación oficial con conectividad temporal

Se utilizó el teléfono mediante USB tethering como enlace temporal. Con esa conectividad se instalaron los paquetes oficiales:

```bash
sudo dnf install -y wpa_supplicant NetworkManager-wifi
sudo systemctl restart NetworkManager
```

Resultado observado: instalación completada y, tras reiniciar NetworkManager, `wlp0s20f3` quedó operativo.

### Método B — contingencia offline con el medio RHEL

Se verificó y documentó el procedimiento offline disponible en la guía Wi-Fi del repositorio:

1. USB RHEL reconocida como `/dev/sda`, aproximadamente 14.6 GiB.
2. Medio con estructura ISO9660 y partición EFI.
3. Montaje de la ISO en modo lectura.
4. Presencia de `images/install.img`.
5. Montaje de `install.img` como SquashFS en solo lectura.
6. Verificación de `wpa_supplicant`.
7. Verificación del plugin `libnm-device-plugin-wifi.so`.
8. Se identificó que el plugin disponible en `install.img` correspondía a `1.56.0-1.el10`, mientras el NetworkManager instalado se observó como `1.56.0-1.el10_2.x86_64`.
9. Por control de integridad y versionado, **no se utilizó la copia manual como solución definitiva en esta incidencia**.

El método offline queda como contingencia documentada; la recuperación efectiva de esta incidencia se realizó mediante RPM oficiales con DNF.

## Verificación final

Después de instalar `wpa_supplicant` y `NetworkManager-wifi`:

```bash
sudo systemctl restart NetworkManager
nmcli device status
```

Resultado observado: estado Wi-Fi correcto y funcionamiento de `wlp0s20f3`.

## Decisión técnica

La recuperación mediante DNF fue preferida porque mantiene la gestión de paquetes de RHEL y evita dejar binarios copiados manualmente fuera del inventario RPM.

La recuperación desde `install.img` se conserva como procedimiento de emergencia para escenarios realmente offline.

## Riesgo residual

- La evidencia de ejecución de esta sesión se basa en resultados observados durante la intervención y resumidos en `output.txt`; no se inventan capturas ni salidas completas que no fueron preservadas literalmente.
- El procedimiento offline de copia de archivos tiene una brecha de gestión RPM y debe considerarse contingencia, no sustituto de la instalación formal mediante paquetes.

## Rollback

No se realizó eliminación de paquetes ni modificación de almacenamiento/boot. La intervención aplicada fue la instalación de los dos paquetes de soporte Wi-Fi y el reinicio de NetworkManager.

## Criterio de cierre

Wi-Fi recuperada y verificada mediante NetworkManager. Incidencia cerrada.

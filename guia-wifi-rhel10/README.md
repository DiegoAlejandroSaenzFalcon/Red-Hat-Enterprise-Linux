# Guía: Conectar WiFi en RHEL 10.2 sin internet

### Diego Alejandro Saenz Falcon · Red Hat Enterprise Linux 10 (RHEL 10.2)

> **Escenario profesional:** estación de trabajo RHEL 10.2 en entorno aislado, **sin
> acceso a internet**. Se resuelve 100% offline utilizando únicamente el **ISO de
> instalación de RHEL 10.2** (el mismo medio con el que se instaló el sistema).

---

## 1. Diagnóstico (por qué no funcionaba)

El hardware de red es correcto; el problema es de software. Compruébalo tú mismo:

```bash
# 1) El chip WiFi y su driver están bien
lspci -k -s 00:14.3            # "Kernel driver in use: iwlwifi" -> OK
dmesg | grep iwlwifi | tail    # "loaded firmware ... so-a0-hr-b0-89.ucode" -> OK
rfkill list                   # WiFi: Soft/Hard blocked: no -> OK

# 2) El problema real: NetworkManager no gestiona la WiFi
nmcli device status            # wlp0s20f3  wifi  sin gestión   <- PROBLEMA
rpm -q wpa_supplicant NetworkManager-wifi   # "not installed"   <- CAUSA
```

**Causa raíz:** a RHEL le faltan dos piezas que permiten a NetworkManager manejar y
autenticar la WiFi:

- el binario **`wpa_supplicant`** (realiza la autenticación WPA/WPA2), y
- el plugin **`libnm-device-plugin-wifi.so`** de NetworkManager.

Sin ellos, NetworkManager muestra la tarjeta como **"sin gestión"** y la deja apagada.

> En una instalación con acceso a repositorios esto se resuelve con
> `dnf install wpa_supplicant NetworkManager-wifi`. Aquí **no hay internet**, así que
> los recuperamos del ISO de instalación.

---

## 2. De dónde salen los archivos

El ISO de instalación de RHEL 10.2 (el que usaste para instalar el sistema) contiene,
en su interior, el entorno del instalador. Dentro de ese ISO está el archivo
`images/install.img` (un sistema de archivos `squashfs`), que incluye los binarios
del instalador — entre ellos, justo los dos que el sistema ya instalado no tiene.

```
ISO de instalación de RHEL 10.2
 └─ images/install.img   (squashfs del instalador)
     ├─ usr/sbin/wpa_supplicant
     └─ usr/lib64/NetworkManager/.../libnm-device-plugin-wifi.so
```

Es una recuperación a nivel de archivos (no un `rpm -ivh`), y es válida porque el
`install.img` es de la **misma versión** de RHEL que el sistema instalado.

---

## 3. Solución paso a paso (todo offline)

Ejecuta como **root** (`su -` o `sudo -i`).

### Paso 1 — Montar el ISO de instalación de RHEL 10.2
```bash
mkdir -p /mnt/rhelcdrom
mount -o loop /ruta/al/rhel-10.2-x86_64-*.iso /mnt/rhelcdrom
```
> Si el ISO está en un USB, monta primero la unidad USB y usa la ruta real del archivo
> (por ejemplo `/mnt/usb/rhel-10.2-x86_64-boot.iso`).

### Paso 2 — Montar `install.img` (el entorno del instalador)
```bash
mkdir -p /mnt/installimg
mount -o loop,ro -t squashfs /mnt/rhelcdrom/images/install.img /mnt/installimg
```

### Paso 3 — Copiar los archivos faltantes al sistema
```bash
# binario wpa_supplicant + su configuración y servicio
cp /mnt/installimg/usr/sbin/wpa_supplicant /usr/sbin/wpa_supplicant
cp /mnt/installimg/usr/lib/systemd/system/wpa_supplicant.service /usr/lib/systemd/system/
cp /mnt/installimg/etc/dbus-1/system.d/wpa_supplicant.conf /etc/dbus-1/system.d/
cp -r /mnt/installimg/etc/wpa_supplicant /etc/

# plugin WiFi de NetworkManager (al directorio versionado que usa el sistema)
PLUGDIR=$(ls -d /usr/lib64/NetworkManager/*el10*)
cp /mnt/installimg/usr/lib64/NetworkManager/*el10*/libnm-device-plugin-wifi.so "$PLUGDIR/"

chmod 755 /usr/sbin/wpa_supplicant "$PLUGDIR/libnm-device-plugin-wifi.so"
```

### Paso 4 — Reiniciar NetworkManager
```bash
systemctl restart NetworkManager
sleep 3
nmcli device status        # wlp0s20f3 ahora aparece "desconectado" (¡gestionado!)
```

### Paso 5 — Conectar a la red WiFi
```bash
nmcli dev wifi list                             # ver redes disponibles
nmcli dev wifi connect "NOMBRE_SSID" password "CLAVE_WIFI"
```

---

## 4. Verificación

```bash
nmcli radio wifi            # debe decir "enabled"
pgrep -a wpa_supplicant     # debe aparecer el proceso corriendo
nmcli device status         # wlp0s20f3 ya NO dice "sin gestión"
```

Si los tres comandos confirman lo anterior, la WiFi está operativa.

---

## 5. Notas para documentar

- Esta es una **recuperación a nivel de archivos** (no una instalación formal con `rpm`).
  Funciona porque el `install.img` es del mismo RHEL 10.2 que el sistema instalado.
- Los archivos copiados **persisten tras reiniciar** (están en el disco del sistema).
- Si en el futuro se actualiza NetworkManager a otra versión, el plugin puede quedar
  desactualizado; en ese caso se repite el Paso 3 apuntando al nuevo directorio.
- Para una instalación "de libro" **con internet**, basta con:
  `dnf install -y wpa_supplicant NetworkManager-wifi`. Esta guía cubre el caso
  **100% offline** usando solo el ISO de instalación.
- Al terminar puedes desmontar los medios:
  `umount /mnt/installimg /mnt/rhelcdrom`

---

*Autor: Diego Alejandro Saenz Falcon* · https://github.com/DiegoAlejandroSaenzFalcon

# Guía: Optimización de Rendimiento en RHEL 10.2

### Diego Alejandro Saenz Falcon · Red Hat Enterprise Linux 10 (RHEL 10.2) · Administración de Sistemas

> Ajustes de rendimiento **sin instalar nada**: se usan `sysctl`, `nmcli`, `tuned`
> y configuración nativa. Pensado para una estación de trabajo con Intel Core
> i3-N305 (8 núcleos, 1 hilo/núcleo) y 8 GB de RAM.

---

## 1. Diagnóstico inicial

```bash
nproc                                  # 8 CPUs (sin hyperthreading)
free -h                                # ~7.2 GiB disponibles (8 GB físicos)
lscpu | grep -E "Model name|MHz"       # i3-N305, hasta 3800 MHz
grep ^MemTotal /proc/meminfo           # memoria total del sistema
cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor   # performance
sysctl net.ipv4.tcp_congestion_control net.core.default_qdisc
swapon --show; cat /proc/sys/vm/swappiness
```

Nota: 8 GB se muestran como ~7.2 GiB por la conversión GiB y ~570 MB reservados
por el hardware (visible en `dmesg` como memoria reservada).

---

## 2. Red: BBR + DNS rápido (mayor velocidad de descarga/subida)

`/etc/sysctl.d/99-performance.conf`:

```ini
net.core.default_qdisc = fq
net.ipv4.tcp_congestion_control = bbr
```

Aplicar: `sudo sysctl --system`. Verificar: `sysctl net.ipv4.tcp_congestion_control`
(debe decir `bbr`).

DNS resolvers rápidos (ej. Cloudflare + Google):

```bash
sudo nmcli connection modify $(sudo nmcli -t -f NAME c show --active) \
  ipv4.dns "1.1.1.1 8.8.8.8" ipv4.ignore-auto-dns yes
sudo nmcli connection down $(sudo nmcli -t -f NAME c show --active); \
sudo nmcli connection up $(sudo nmcli -t -f NAME c show --active)
```

---

## 3. Memoria: menos swappiness

Para una estación con 8 GB (sin pretender servidor), reducir el uso de swap:

```bash
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.d/99-performance.conf
sudo sysctl --system
```

(opcional) Para sistemas con SSD NVMe rápido y mucha RAM, `swappiness=10` evita
intercambios innecesarios sin eliminar la red de seguridad de la swap.

---

## 4. WiFi: apagar ahorro de energía (estabilidad de velocidad)

```bash
sudo nmcli con modify $(sudo nmcli -t -f NAME c show --active) \
  802-11-wireless.powersave 2     # 2 = disable
sudo nmcli connection down $(sudo nmcli -t -f NAME c show --active); \
sudo nmcli connection up $(sudo nmcli -t -f NAME c show --active)
```

`powersave 2` desactiva el ahorro de energía de la radio WiFi → mejor flujo
constante (importa para videollamadas/transferencias grandes).

---

## 5. Desactivar Bluetooth (no usado)

```bash
sudo systemctl disable --now bluetooth.service
```

---

## 6. Verificación final

```bash
sysctl net.ipv4.tcp_congestion_control net.core.default_qdisc vm.swappiness
resolvectl status | grep -E "DNS Servers"
nmcli -f 802-11-wireless.powersave con show $(sudo nmcli -t -f NAME c show --active)
systemctl is-enabled bluetooth.service    # disabled
```

---

*Autor: Diego Alejandro Saenz Falcon* · https://github.com/DiegoAlejandroSaenzFalcon

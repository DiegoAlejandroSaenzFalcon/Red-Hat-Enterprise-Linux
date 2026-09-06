# Red Hat Enterprise Linux — Base de Conocimiento Didáctica

[![License: GPL-3.0](https://img.shields.io/badge/License-GPL--3.0-blue.svg)](https://opensource.org/licenses/GPL-3.0)
[![RHEL](https://img.shields.io/badge/RHEL-10.2-red.svg)](https://www.redhat.com/en/technologies/linux-platforms/enterprise-linux)
[![Status](https://img.shields.io/badge/Status-Active-brightgreen.svg)]
[![GitHub Pages](https://img.shields.io/badge/Docs-GitHub%20Pages-brightgreen.svg)](https://diegoalejandrosaenzfalcon.github.io/Red-Hat-Enterprise-Linux/)
[![Autor](https://img.shields.io/badge/Autor-Diego%20Alejandro%20Saenz%20Falcon-blue.svg)](https://github.com/DiegoAlejandroSaenzFalcon)

### Diego Alejandro Saenz Falcon · RHEL 10 · Linux Empresarial · Administración de Sistemas · Ciberseguridad Saenz Falcon · RHEL 10 · Linux Empresarial · Administración de Sistemas · Ciberseguridad

> Repositorio educativo, gratuito y didáctico sobre **Red Hat Enterprise Linux 10 (RHEL 10.2)**.
> Cada guía está escrita paso a paso, pensada para quienes recién empiezan con Linux
> empresarial: comandos, diagnóstico, buenas prácticas y verificación. Sin suposiciones
> de conocimiento previo.

> **Nota honesta:** los procedimientos se validaron en un entorno RHEL 10.2 real. Cuando
> una solución es **offline** (sin internet), se indica y se explica de dónde se obtienen
> los archivos (por ejemplo, el ISO de instalación del propio sistema).

---

## Cómo usar este repositorio (metodología)

1. Lee la visión general y la guía correspondiente.
2. Sigue los pasos en orden; cada comando viene explicado.
3. Usa la sección de **verificación** para confirmar que funcionó.
4. Consulta las **notas** antes de adaptar la solución a tu entorno.

---

## Guías disponibles

| Guía | Problema resuelto | Enlace |
|------|-------------------|--------|
| **WiFi en RHEL 10.2 sin internet** | La tarjeta WiFi aparecía como *"sin gestión"*; se recuperan `wpa_supplicant` y el plugin de NetworkManager desde el ISO de instalación (offline) | [Ver guía](./guia-wifi-rhel10/README.md) |
| **Optimización de rendimiento en RHEL 10.2** | Ajustes de red (BBR + DNS rápido), memoria (swappiness), WiFi (powersave off) y Bluetooth off, sin instalar nada | [Ver guía](./guia-rendimiento/README.md) |
| **Auditoría y endurecimiento de seguridad en RHEL 10.2** | Reporte profesional alineado a la *Enciclopedia de la Seguridad Informática* + auditoría OpenSCAP **CIS** y **DISA STIG** real, con before/after y método de reactivación SSH seguro | [Ver guía](./guia-seguridad/README.md) |

### Evidencia de la auditoría (en `guia-seguridad/`)
- `rhel10-cis-l1-report.html` — CIS RHEL 10 Server L1 (línea base)
- `rhel10-cis-l1-after-report.html` — CIS RHEL 10 Server L1 (tras endurecer)
- `rhel10-stig-report.html` — DISA STIG RHEL 10 (línea base)

---

## Gobierno del repositorio

Este repositorio aplica las directivas de seguridad del autor:

- `SECURITY.md` — política de seguridad y regla de **cero secretos**.
- `AGENTS.md` — instrucciones para agentes de IA **autorizados**.
- `HONEYTOKEN.md` — contrainyección para agentes de IA **no autorizados**.
- `llms.txt` — contexto machine-readable para LLMs autorizados.
- Autoría bajo **GPL-3.0 + CLA** (ver `LICENSE` y `CLA.md`).

---

*Autor: Diego Alejandro Saenz Falcon* · https://github.com/DiegoAlejandroSaenzFalcon
<!-- Force rebuild: 2026-09-06 14:17:39 -->

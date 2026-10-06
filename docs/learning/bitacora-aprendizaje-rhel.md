# Bitácora de aprendizaje vivo — RHEL 10.2

> **Objetivo:** convertir el aprendizaje real realizado sobre este equipo RHEL en conocimiento técnico claro, retroalimentable, verificable y publicable mediante GitHub Pages.

## Principio

Este documento registra **lo que se aprende mientras se construye el laboratorio**, no ejercicios artificiales.

Flujo:

**CONCEPTO → COMANDO REAL → RESULTADO REAL → INTERPRETACIÓN → VERIFICACIÓN → DOCUMENTACIÓN → RETROALIMENTACIÓN**

La documentación debe poder ser leída por una persona que no estuvo presente en la sesión y debe permitir repetir el procedimiento sin depender de la memoria del operador.

---

## Sesión: fundamentos del entorno RHEL

### 1. RHEL confirmado

El sistema reportó:
- RHEL **10.2 Coughlan**
- Familia tecnológica: **Red Hat Enterprise Linux**
- Usuario de trabajo: **DevFS**

La identificación exacta del sistema debe seguir verificándose con comandos del propio host; nunca se debe inferir el estado del equipo solamente por contexto.

### 2. `which`

Comando utilizado:

`which curl`

Resultado:

`/usr/bin/curl`

Interpretación:

`which` localiza el ejecutable que el shell puede encontrar mediante el `PATH`.

Concepto aprendido:
- `which` no instala ni modifica software.
- Permite descubrir qué ejecutable se utilizará cuando escribimos su nombre.
- Es especialmente útil antes de instalar otra copia de una herramienta.

### 3. `curl`

Se confirmó:

`/usr/bin/curl`

`curl` es una herramienta para realizar transferencias de datos mediante protocolos de red. En este laboratorio interesa porque puede formar parte de procedimientos de instalación, consulta o automatización.

**Regla:** disponer de `curl` no significa que debamos ejecutar automáticamente una instalación remota. Primero se debe identificar y verificar el procedimiento y su procedencia.

### 4. `wget`

Se comprobó que `wget` no está disponible mediante `which wget`.

Esto no constituye un fallo del sistema.

Aprendizaje:
> No debemos instalar una herramienta simplemente porque otra persona la utiliza en un ejemplo. Primero comprobamos qué existe y si realmente es necesaria.

### 5. `dnf`

Se confirmó:

`/usr/bin/dnf`

Versión observada:

`4.20.0`

`dnf` es el gestor de paquetes utilizado por RHEL para administrar software y dependencias mediante repositorios.

Usos principales:
- consultar paquetes;
- instalar software;
- actualizar software;
- eliminar paquetes;
- resolver dependencias;
- consultar repositorios.

Regla operacional:
> Antes de instalar una dependencia, descubrir qué repositorios están habilitados y por qué se necesita la dependencia.

### 6. Bash

**Bash (Bourne Again SHell)** es el shell/intérprete de comandos que permite interpretar órdenes y ejecutar scripts.

Bash puede:
- ejecutar programas;
- trabajar con variables;
- encadenar comandos;
- redirigir entrada y salida;
- utilizar tuberías (`|`);
- evaluar condiciones (`&&`, `||`);
- realizar sustitución de comandos (`$()`);
- automatizar tareas mediante scripts.

No debe confundirse:
- **terminal:** interfaz donde interactuamos;
- **shell:** programa que interpreta comandos;
- **Bash:** uno de los shells más utilizados;
- **comando:** programa o función que Bash ejecuta.

### 7. `PATH`

Se consultó:

`echo "$PATH"`

Resultado real de la sesión:

`/home/DevFS/.local/bin:/home/DevFS/bin:/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin`

`PATH` es una variable de entorno que contiene directorios separados por `:`.

Cuando se escribe:

`curl`

Bash busca el ejecutable en los directorios de `PATH`, en orden.

En esta sesión encontró:

`/usr/bin/curl`

El orden importa: si existieran dos ejecutables con el mismo nombre, normalmente se utilizaría el primero encontrado.

### 8. Estructura de rutas

La raíz del sistema de archivos es:

`/`

No existen solamente dos directorios principales. `/home` y `/usr` son dos de muchos directorios que cuelgan de `/`.

### 9. `/home/DevFS`

`/home/DevFS` es el directorio personal del usuario `DevFS`.

Conceptualmente se parece al espacio de usuario de `C:\Users\<usuario>` en Windows.

Dentro pueden existir directorios como `.config`, `.cache` y `.local`.

### 10. `/usr`

`/usr` contiene gran parte del software, bibliotecas y datos utilizados por el sistema y las aplicaciones.

Ejemplos relevantes:
- `/usr/bin` → muchos ejecutables de uso general;
- `/usr/sbin` → herramientas de administración del sistema;
- `/usr/local/bin` → software instalado localmente fuera del conjunto principal gestionado por la distribución;
- `/usr/local/sbin` → equivalente administrativo local.

La simplificación **"/home = usuario, /usr = sistema"** sirve como primera aproximación, pero no describe toda la arquitectura de Linux.

### 11. ¿Por qué `.local` empieza por punto?

En Unix/Linux, los nombres que comienzan por `.` se consideran normalmente **ocultos para los listados normales**.

Ejemplos: `.local`, `.config`, `.cache`.

"Oculto" no significa cifrado ni secreto.

`ls` normalmente omite esos nombres, mientras que `ls -a` los incluye. La `a` significa **all**.

### 12. `/home/DevFS/.local/bin`

Esta ruta combina varios conceptos:
- `/home` → directorios personales;
- `DevFS` → usuario;
- `.local` → datos/recursos locales del usuario, ocultos en listados normales;
- `bin` → ubicación destinada habitualmente a ejecutables.

Como `/home/DevFS/.local/bin` aparece al principio del `PATH`, las herramientas instaladas allí para `DevFS` pueden ser encontradas directamente por Bash.

Esto permite distinguir entre:
- **Herramienta del usuario:** `/home/DevFS/.local/bin`
- **Herramienta disponible a nivel del sistema:** `/usr/bin`

No significa que una ubicación sea automáticamente "mejor"; la elección depende del alcance, seguridad, mantenimiento y método de instalación.

---

## Conocimientos consolidados

Al finalizar esta sesión, el operador puede explicar:
- qué es `/`;
- qué es `/home`;
- qué es `/usr`;
- qué es un directorio;
- qué es un ejecutable;
- qué hace `which`;
- qué es Bash;
- qué es una variable de entorno;
- qué es `PATH`;
- cómo Bash encuentra un ejecutable;
- qué significa un nombre que comienza por `.`;
- diferencia conceptual entre `.local/bin`, `/usr/local/bin` y `/usr/bin`;
- qué es `dnf`;
- por qué se debe descubrir el entorno antes de instalar software.

## Retroalimentación y próximos pasos

Esta bitácora debe crecer con cada operación real del laboratorio.

Cada nueva entrada debe registrar:
1. **Qué concepto aprendimos.**
2. **Qué comando real lo demostró.**
3. **Qué resultado produjo el equipo.**
4. **Qué significa ese resultado.**
5. **Qué riesgo o límite existe.**
6. **Cómo se verifica.**
7. **Dónde queda la evidencia.**
8. **Qué conocimiento previo habilita.**
9. **Qué siguiente paso depende de él.**

### Regla de calidad
> No documentar solamente "Se ejecutó el comando X". Documentar qué se quería comprobar, qué devolvió el sistema, qué demuestra y qué falta verificar.

Así la documentación se convierte en **memoria técnica del laboratorio**, no en un simple historial de comandos.

## Relación con GitHub Pages

Esta bitácora pertenece al repositorio existente:

`DiegoAlejandroSaenzFalcon/Red-Hat-Enterprise-Linux`

y debe publicarse mediante la documentación MkDocs del laboratorio.

La documentación pública debe distinguir claramente entre conocimiento conceptual, estado real del host, procedimientos, evidencia, resultados, hipótesis y elementos todavía no verificados.

Nunca presentar una hipótesis como hecho.

## Estado de esta sesión

**Estado:** EN CURSO

**Nivel de evidencia:** E2 — ejecución real observada durante la sesión.

**Pendiente:** continuar con la preparación del entorno para Desktop Commander, verificando primero el procedimiento oficial y las dependencias necesarias antes de instalar o modificar componentes.
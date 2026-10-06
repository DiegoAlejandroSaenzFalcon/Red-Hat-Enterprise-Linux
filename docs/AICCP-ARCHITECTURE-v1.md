# AICCP — Architecture v1

**Estado:** SPECIFIED  
**Fecha:** 2026-10-06  
**Host objetivo:** RHEL 10.2 Minimal / headless  
**Objetivo:** establecer un canal gobernado entre ChatGPT, Desktop Commander, OpenCode, auditoría, Git local/GitHub y los proyectos de trabajo.

## 1. Principios
1. RHEL es nodo operativo; GitHub es fuente durable de verdad.
2. TTY1 queda reservado para recuperación/administración física.
3. Desktop Commander no se convierte en servicio hasta verificar persistencia, autenticación y rollback.
4. OpenCode se inicia inicialmente con una sola instancia controlada.
5. No se instalarán LLM locales, Docker/Podman, bases externas, Redis, PostgreSQL ni otros servicios pesados sin justificación.
6. No se almacenan API keys ni secretos en repositorios, logs, prompts, evidencias o variables documentadas.
7. Ninguna instancia se considera terminada por declaración propia: AICCP debe verificarla.
8. Cada cambio material requiere verificación y evidencia.

## 2. Capacidad observada
- RAM total: ~7.2 GiB.
- RAM disponible durante inventario: ~6.0 GiB.
- Swap: 7.6 GiB, 0 usado.
- `/`: 70G, 5% usado.
- `/home`: 159G, 3% usado.
- Node.js: 22.23.2.
- Python: 3.12.14.
- Git: no instalado al momento del inventario.
- GitHub CLI (`gh`): no encontrado.
- OpenCode: no encontrado entre los procesos activos.
- AICCP: no encontrado entre los procesos activos.
- Desktop Commander: activo en TTY2; procesos observados PID 12806/13085/13109.

## 3. Topología
```
Diego (autoridad final)
        |
        v
ChatGPT — arquitectura/gobierno/revisión
        |
        v
AICCP — registry/estado/heartbeat/evidencia
        |
   +----+---------+
   |              |
   v              v
OpenCode       Auditor
   |              |
   +------+-------+
          |
          v
      Git local
          |
          v
       GitHub
```

## 4. Workspace
Los repositorios se alojarán bajo una raíz dedicada, propuesta: `/home/DevFS/workspaces/`.
El runtime de AICCP no se mezclará con los repositorios:
`/opt/aiccp/{bin,config,runtime,logs,evidence,state,tasks,policies}`.
La ruta final se confirmará antes de crearla.

## 5. Registro de instancias
El registro AICCP deberá conservar como mínimo: instance_id, role, project, pid/parent_pid, terminal/session, state, task, current_step, started_at, last_heartbeat, cpu, memory, last_error, repository, branch y commit.
Estados: `CREATED, STARTING, READY, RUNNING, WAITING, BLOCKED, ERROR, STOPPING, STOPPED, COMPLETED, VERIFIED`.
Una instancia sin heartbeat válido será marcada `STALE`, no asumida como activa.

## 6. Persistencia
Primera implementación: SQLite para estado; JSON/texto para eventos y heartbeats cuando aporte legibilidad; logs rotados; evidencia por ejecución; sin servidor de base de datos externo.

## 7. OpenCode
Primera etapa: instalar/verificar; descubrir providers/modelos; determinar qué puede ejecutarse sin API key; prueba mínima; medir RAM/CPU; registrar configuración sin secretos; después habilitar tareas reales.
No se presupone ningún modelo gratuito o autenticación disponible.

## 8. Auditoría
El auditor debe comprobar independientemente proceso real, estado AICCP, cambios de filesystem, Git, diff, tests, evidencia, commit y sincronización GitHub.
Regla: `DECLARED COMPLETE -> VERIFY -> VERIFIED`.

## 9. Git
Repositorios prioritarios: Soluciona-Inteligencia-Artificial, Directivas-de-Seguridad, Trading-Ciencia, Red-Hat-Enterprise-Linux, diegoalejandrosaenzfalcon.github.io, Automatizacion-de-Datos y Tecnologias-de-la-Informacion.
Los repositorios privados se tratarán como privados. No se copiarán secretos ni credenciales.
Antes de clonar cada repositorio se registrará nombre, visibilidad, rama por defecto, remoto, estado local, tamaño, presencia de AGENTS/contratos y divergencia GitHub/local cuando exista checkout previo.

## 10. Presupuesto de recursos
Regla inicial: una instancia OpenCode; auditor bajo demanda; AICCP ligero; ningún servicio residente adicional salvo necesidad demostrada.
Cada nueva instancia debe justificar su consumo.

## 11. Gates
- Gate A — Inventario: EVIDENCED.
- Gate B — Diseño: SPECIFIED.
- Gate C — Bootstrap: instalar solamente Git y dependencias mínimas.
- Gate D — Workspace: clonar y validar repositorios.
- Gate E — OpenCode: instalar, descubrir proveedor/modelo y medir.
- Gate F — AICCP Core: implementar registry, heartbeat, state machine y evidencia.
- Gate G — Auditor: validación independiente.
- Gate H — Soluciona IA: primera ejecución real bajo gobierno AICCP.

## 12. Fuera de alcance inmediato
- acceso móvil por Internet;
- exposición adicional de SSH;
- servicio systemd para Desktop Commander;
- LLM local;
- Docker/Podman;
- Kubernetes;
- infraestructura de bases de datos pesada.
Se tratarán como decisiones separadas y autorizadas.
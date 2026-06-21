---
layout: default
title: Comando patch-status
parent: Referencia de Comandos
nav_order: 15
---

# Comando patch-status
{: .no_toc }

## Tabla de Contenidos
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Introducción

`patch-status` es un comando de diagnóstico de solo lectura para comprobar si el proceso objetivo está parcheado por gevent/eventlet, si primitivas stdlib como socket/thread/time todavía coinciden con las implementaciones nativas capturadas por Runtime Primitive Layer (RPL), y qué loop asyncio y modelo de hilos están activos.

El comando no corrige ni revierte monkey patches. Solo informa el estado para ayudar a razonar sobre attach, watch, trace y otros diagnósticos en entornos de runtime complejos.

## Sintaxis

```bash
peeka-cli attach <pid>
peeka-cli patch-status [--pid <pid>]
```

| Parámetro | Descripción |
|-----------|-------------|
| `--pid` | Parámetro opcional de compatibilidad; la implementación actual lo ignora e informa sobre la sesión adjunta activa |

Primero debes ejecutar `attach`. Si no hay una sesión activa, el comando devuelve un error que solicita ejecutar `peeka-cli attach <pid>`.

## Campos de Salida

Cuando se ejecuta correctamente, `patch-status` devuelve un mensaje `result` cuyo dato interno contiene:

| Campo | Descripción |
|-------|-------------|
| `schema_version` | Versión del schema de salida, actualmente `"1"` |
| `pid` | PID del proceso objetivo |
| `timestamp` | Marca temporal de muestreo |
| `monkey_patch` | Estado de importación y activación de gevent/eventlet, y módulos parcheados cuando están disponibles |
| `stdlib_origin` | IDs de primitivas stdlib actuales comparados con los IDs nativos capturados por RPL |
| `asyncio_loop` | Estado del loop, policy y clase de loop |
| `thread_model` | Hilo principal, total de hilos, hilos daemon y clasificación |
| `rpl_integrity` | Si las primitivas capturadas por RPL están intactas y si hay drift |

## Ejemplo

```bash
peeka-cli attach 12345
peeka-cli patch-status | jq '.data.data'
```

Salida de ejemplo:

```json
{
  "schema_version": "1",
  "pid": 12345,
  "monkey_patch": {
    "gevent": {
      "status": "active",
      "patched_modules": ["socket", "threading"]
    },
    "eventlet": "not_imported"
  },
  "stdlib_origin": {
    "socket.socket": {
      "matches": false
    }
  },
  "asyncio_loop": {
    "running": true,
    "policy": "DefaultEventLoopPolicy",
    "loop_class": "SelectorEventLoop"
  },
  "thread_model": {
    "total_threads": 4,
    "classification": "multi_threaded_with_daemons"
  },
  "rpl_integrity": {
    "ok": true
  }
}
```

## Usos Típicos

- Confirmar el estado de monkey patch antes o después de adjuntar a servicios gevent o eventlet
- Investigar problemas de attach u observación causados por primitivas socket, threading o time reemplazadas
- Verificar que RPL aún tenga primitivas nativas socket/thread/time disponibles para rutas internas de Peeka
- Comprobar si el proceso objetivo ejecuta un loop asyncio y si el modelo de hilos es el esperado

## Interpretación

- `monkey_patch.gevent.status == "active"` o `monkey_patch.eventlet.status == "active"` significa que hay monkey patching activo.
- `stdlib_origin.*.matches == false` significa que el objeto stdlib actual difiere del objeto nativo capturado por RPL.
- `rpl_integrity.ok == true` significa que las primitivas nativas capturadas por RPL siguen disponibles para la ruta interna de diagnóstico de Peeka.
- `patch-status` no restaura funciones nativas. Para eliminar inyecciones de observación de Peeka, usa [reset]({% link commands/reset.md %}) o [detach]({% link commands/detach.md %}).

## Historial de Versiones

| Versión | Fecha de Lanzamiento | Cambios |
|---------|----------------------|---------|
| 0.1.14 | 2026-05-24 | Añadido el comando de diagnóstico de runtime `patch-status` |

## Comandos Relacionados

- [attach]({% link commands/attach.md %}) - Adjuntar al proceso objetivo
- [watch]({% link commands/watch.md %}) - Observar llamadas a funciones
- [trace]({% link commands/trace.md %}) - Rastrear cadenas de llamadas
- [reset]({% link commands/reset.md %}) - Restablecer inyección

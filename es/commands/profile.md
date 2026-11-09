---
layout: default
title: Comando profile
parent: Referencia de Comandos
nav_order: 22
---

# Comando profile
{: .no_toc }

## Tabla de Contenidos
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Introducción

El comando `profile` **crea, consulta y exporta sesiones de profiling** (ProfileSession) en el proceso objetivo bajo demanda. A diferencia de [`top`]({% link commands/top.md %}), que muestra datos en vivo, una sesión de profile conserva sus muestras en el agente, de modo que puedes tomar una instantánea en cualquier momento o exportarla en JSON, texto, [speedscope](https://www.speedscope.app/) o [Perfetto](https://ui.perfetto.dev/) para analizarla sin conexión y compartirla.

`top` también muestrea en una ProfileSession temporal. Por defecto se limpia cuando `top` termina; usa `top --keep-profile` para conservarla y exportarla con `profile`.

## Sintaxis

```bash
peeka-cli attach <pid>    # Primero adjúntate al proceso objetivo
peeka-cli profile <subcommand> [options]
```

## Subcomandos

| Subcomando | Descripción |
|------------|-------------|
| `start` | Inicia una sesión de profiling y devuelve su `profile_id` |
| `stop --profile-id <id>` | Detiene la sesión y conserva los datos recogidos |
| `snapshot --profile-id <id>` | Toma una instantánea de los datos actuales |
| `list` | Lista todas las sesiones de profile |
| `cleanup` | Elimina sesiones antiguas |
| `export --profile-id <id>` | Exporta los datos de la sesión, opcionalmente a un archivo |

### Opciones de start

| Opción | Descripción | Valor por defecto |
|--------|-------------|-------------------|
| `--backend sampling/exact` | Backend de profiling | `sampling` |
| `--experimental` | Permite el backend experimental `exact` | desactivado |
| `--interval <segundos>` | Intervalo de muestreo | `0.01` |
| `--duration <segundos>` | Duración opcional; la sesión se detiene sola al expirar | sin límite |
| `--max-samples <N>` | Límite opcional de muestras; la sesión se detiene sola al alcanzarlo | `100000` |

### Opciones de snapshot / export

| Opción | Descripción | Valor por defecto |
|--------|-------------|-------------------|
| `--profile-id <id>` | ID de la sesión (obligatorio) | - |
| `--format json/text/speedscope/perfetto` | Formato de salida | `json` |
| `--output <path>` | Solo `export`: escribe el contenido exportado en este archivo | salida estándar |

### Opciones de cleanup

| Opción | Descripción |
|--------|-------------|
| `--older-than <segundos>` | Elimina sesiones creadas hace más de N segundos |
| `--max-sessions <N>` | Conserva como máximo N sesiones |

## Backends

- **sampling (por defecto)**: un hilo en segundo plano muestrea las pilas de llamadas de todos los hilos cada `--interval`. Bajo overhead, apto para ejecuciones largas en procesos de producción.
- **exact (experimental)**: registra cada llamada a función a partir de los eventos de llamada de Python. Requiere Python 3.12+ (`threading.setprofile_all_threads`), tiene un overhead notablemente mayor y debe iniciarse con `--experimental`. El número de eventos está limitado; la sesión termina sola al alcanzar el límite.

`exact` y `sampling` son mutuamente excluyentes: mientras uno se ejecuta, el otro no puede iniciarse, para que el overhead del hook exact no distorsione los resultados del muestreo. Si el proceso objetivo ya tiene su propio profiler instalado (`sys.setprofile` / `threading.setprofile`), `exact` se niega a iniciar en lugar de reemplazarlo.

## Ejemplos

```bash
# Muestrear durante 30 segundos
peeka-cli profile start --duration 30

# Listar sesiones
peeka-cli profile list

# Exportar como speedscope y abrirlo en https://www.speedscope.app/
peeka-cli profile export --profile-id <profile_id> --format speedscope --output ./profile.speedscope.json

# Detener y limpiar
peeka-cli profile stop --profile-id <profile_id>
peeka-cli profile cleanup --max-sessions 5
```

Añadir una sesión de profile a un caso DX:

```bash
peeka-cli dx add-profile --dx-case dx_123 --profile-id <profile_id>
```

## Uso en la TUI

En la TUI, pulsa `0` para abrir la pestaña **Performance** y cambia a la subpestaña **Profiles** para ver la lista de sesiones (`profile_id`, `backend`, `status`, `started_at`, `sample_count`). Selecciona una fila y haz clic en **Export** para exportarla; pulsa `r` para refrescar. Consulta la [Guía de la TUI]({% link tui.md %}).

## Notas

- Las sesiones de profile viven en el agente del proceso objetivo. Se conservan hasta 50 sesiones; a partir de ahí se descartan las más antiguas. `detach` detiene todos los profiles activos.
- La exportación `speedscope` genera un profile separado por hilo.
- En producción, prefiere el backend por defecto `sampling`.

## Historial de Versiones

| Versión | Fecha de lanzamiento | Cambios |
|---------|----------------------|---------|
| 0.1.21 | 2026-11-09 | Nuevo grupo de comandos `profile` (start / stop / snapshot / list / cleanup / export) con backends sampling y exact experimental y formatos JSON / text / speedscope / Perfetto |

## Comandos Relacionados

- [top]({% link commands/top.md %}) - Muestreo de rendimiento de funciones en vivo
- [dx]({% link commands/dx.md %}) - Paquetes de casos de diagnóstico
- [detach]({% link commands/detach.md %}) - Desconectar y limpiar recursos

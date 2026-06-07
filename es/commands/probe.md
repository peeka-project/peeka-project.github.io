---
layout: default
title: Comando probe
parent: Referencia de Comandos
nav_order: 19
---

# Comando probe
{: .no_toc }

Gestiona ejecuciones de probe. Comandos de observación como `watch`, `trace`, `monitor` y `top` pueden registrar probes dentro de un target; `probe` lista, inspecciona, detiene y limpia esas ejecuciones.
{: .fs-6 .fw-300 }

## Tabla de Contenidos
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Sintaxis

```bash
peeka-cli probe <subcommand> [options]
```

## Subcomandos

| Subcomando | Descripción |
|------------|-------------|
| `list` | Lista ejecuciones de probe |
| `status --probe <id>` | Muestra el estado del probe |
| `inspect --probe <id>` | Muestra detalles y eventos recientes |
| `stop --probe <id>` | Detiene un probe en ejecución |
| `cleanup` | Limpia probes antiguos |

Opciones comunes:

| Opción | Descripción |
|--------|-------------|
| `--target <id>` | Filtra o ubica el target propietario |
| `--type <type>` | Filtra `list` por tipo de probe, como `watch` o `trace` |
| `--status <status>` | Filtra `list` por estado |
| `--events <n>` | Número de eventos recientes devueltos por `inspect`, predeterminado `100` |
| `--all` | En `cleanup`, también limpia probes created/paused; nunca probes active |
| `--older-than <duration>` | En `cleanup`, limpia probes más antiguos que la duración; predeterminado `10m` |
| `--format table/json` | Formato de salida, predeterminado `table` |

## Ejemplos

```bash
peeka-cli probe list --target target_abcd1234
peeka-cli probe list --type watch --status active
peeka-cli probe inspect --probe probe_123 --events 20 --format json
peeka-cli probe stop --probe probe_123
peeka-cli probe cleanup --older-than 1h
```

## Historial de Versiones

| Versión | Fecha de lanzamiento | Cambios |
|---------|----------------------|---------|
| 0.1.16 | 2026-06-07 | Añadido el grupo de comandos `probe` |

## Comandos Relacionados

- [watch]({% link commands/watch.md %}) - Crear observaciones de funciones
- [trace]({% link commands/trace.md %}) - Crear trazas de cadenas de llamadas
- [job]({% link commands/job.md %}) - Gestionar jobs de comando

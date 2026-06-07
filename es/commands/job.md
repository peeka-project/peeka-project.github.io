---
layout: default
title: Comando job
parent: Referencia de Comandos
nav_order: 18
---

# Comando job
{: .no_toc }

Gestiona jobs de comando asíncronos: revisa su ciclo de vida, inspecciona metadatos de resultados, interrumpe trabajo en ejecución y limpia jobs antiguos.
{: .fs-6 .fw-300 }

## Tabla de Contenidos
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Sintaxis

```bash
peeka-cli job <subcommand> [options]
```

## Subcomandos

| Subcomando | Descripción |
|------------|-------------|
| `list` | Lista jobs de comando |
| `status --job <id>` | Muestra el estado de un job |
| `inspect --job <id>` | Muestra detalles completos del job |
| `interrupt --job <id>` | Interrumpe un job en ejecución |
| `cleanup` | Limpia jobs antiguos |
| `pull --job <id> --consumer <name>` | Interfaz reservada para extraer resultados; actualmente es un stub de Phase 5 |

Filtros y opciones comunes:

| Opción | Descripción |
|--------|-------------|
| `--target <id>` | Filtra o ubica el target propietario |
| `--client <id>` | Filtra `list` por sesión cliente |
| `--status <status>` | Filtra `list` por estado del job |
| `--completed` | En `cleanup`, limpia solo jobs completados |
| `--older-than <duration>` | En `cleanup`, limpia jobs más antiguos que la duración; predeterminado `10m` |
| `--format table/json` | Formato de salida, predeterminado `table` |

`--older-than` acepta segundos o duraciones como `30s`, `10m` o `2h`.

## Ejemplos

```bash
peeka-cli job list --target target_abcd1234
peeka-cli job status --job job_123 --format json
peeka-cli job inspect --job job_123 --target target_abcd1234
peeka-cli job interrupt --job job_123
peeka-cli job cleanup --completed --older-than 30m
```

## Historial de Versiones

| Versión | Fecha de lanzamiento | Cambios |
|---------|----------------------|---------|
| 0.1.16 | 2026-06-07 | Añadido el grupo de comandos `job` |

## Comandos Relacionados

- [probe]({% link commands/probe.md %}) - Gestionar ejecuciones de probe
- [consumer]({% link commands/consumer.md %}) - Leer resultados en buffer
- [target]({% link commands/target.md %}) - Seleccionar un target

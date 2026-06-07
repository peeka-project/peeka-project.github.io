---
layout: default
title: Comando consumer
parent: Referencia de Comandos
nav_order: 20
---

# Comando consumer
{: .no_toc }

Gestiona consumidores de resultados. Un consumer crea un buffer acotado para salida de job, probe o target, de modo que clientes CLI, TUI, MCP o API puedan leer flujos de resultados de diagnóstico.
{: .fs-6 .fw-300 }

## Tabla de Contenidos
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Sintaxis

```bash
peeka-cli consumer <subcommand> [options]
```

## Subcomandos

| Subcomando | Descripción |
|------------|-------------|
| `create` | Crea un consumidor de resultados |
| `list` | Lista consumidores de resultados |
| `status --consumer <id>` | Muestra el estado del consumidor |
| `drain --consumer <id>` | Lee registros en buffer |
| `close --consumer <id>` | Cierra un consumidor |
| `cleanup` | Limpia consumidores closed o failed |

## Opciones de create

| Opción | Descripción |
|--------|-------------|
| `--target <id>` | Target propietario |
| `--source cli/tui/mcp/api/internal` | Fuente de la solicitud |
| `--scope-type job/probe/target` | Tipo de alcance a consumir |
| `--scope-id <id>` | ID de job, probe o target |
| `--client <id>` | Cliente propietario opcional |
| `--max-buffer-size <n>` | Registros máximos en buffer, predeterminado `1000` |
| `--backpressure-policy drop_oldest/drop_newest/fail` | Política cuando el buffer está lleno, predeterminado `drop_oldest` |

## Opciones de drain

| Opción | Descripción |
|--------|-------------|
| `--limit <n>` | Registros máximos a devolver, predeterminado `100` |
| `--after-sequence <n>` | Devuelve registros con sequence mayor que este valor |
| `--timeout-ms <n>` | Espera hasta estos milisegundos por nuevos registros, predeterminado `0` |

Todos los subcomandos soportan `--format table` o `--format json`.

## Ejemplos

```bash
peeka-cli consumer create \
  --target target_abcd1234 \
  --source cli \
  --scope-type probe \
  --scope-id probe_123 \
  --format json

peeka-cli consumer drain --consumer consumer_123 --limit 50 --format json
peeka-cli consumer close --consumer consumer_123
peeka-cli consumer cleanup --target target_abcd1234
```

## Historial de Versiones

| Versión | Fecha de lanzamiento | Cambios |
|---------|----------------------|---------|
| 0.1.16 | 2026-06-07 | Añadido el grupo de comandos `consumer` |

## Comandos Relacionados

- [client]({% link commands/client.md %}) - Gestionar sesiones cliente
- [job]({% link commands/job.md %}) - Gestionar jobs de comando
- [probe]({% link commands/probe.md %}) - Gestionar ejecuciones de probe

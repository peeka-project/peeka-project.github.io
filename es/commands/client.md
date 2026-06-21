---
layout: default
title: Comando client
parent: Referencia de Comandos
nav_order: 17
---

# Comando client
{: .no_toc }

Crea y gestiona sesiones cliente vinculadas a agentes target. Las sesiones cliente distinguen llamadas desde CLI, TUI, MCP, API o componentes internos.
{: .fs-6 .fw-300 }

## Tabla de Contenidos
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Introducción

El comando `client` **crea y gestiona sesiones de cliente vinculadas a un target agent**, distinguiendo a los llamadores CLI, TUI, MCP, API e internos. Cada sesión tiene su propio ID, lo que permite aislar por cliente los resultados de [job]({% link commands/job.md %}) y [consumer]({% link commands/consumer.md %}).

Identifica al llamador con `--source` y usa `list`, `status` y `close` para seguir el ciclo de vida de la sesión, lo cual resulta útil cuando varios clientes ejecutan diagnósticos simultáneamente sobre el mismo target.

## Sintaxis

```bash
peeka-cli client <subcommand> [options]
```

## Subcomandos

| Subcomando | Descripción |
|------------|-------------|
| `create --target <id> --source <source>` | Crea una sesión cliente |
| `list [--target <id>]` | Lista sesiones cliente, opcionalmente filtradas por target |
| `status --client <id>` | Muestra el estado de la sesión cliente |
| `close --client <id>` | Cierra una sesión cliente |

`--source` puede ser `cli`, `tui`, `mcp`, `api` o `internal`. `create` también acepta `--user <id>` como identificador de usuario opcional.

Todos los subcomandos soportan `--format table` (predeterminado) o `--format json`.

## Ejemplos

```bash
# Crear un cliente para automatización CLI
peeka-cli client create --target target_abcd1234 --source cli --format json

# Listar clientes de un target
peeka-cli client list --target target_abcd1234

# Inspeccionar y cerrar un cliente
peeka-cli client status --client client_123
peeka-cli client close --client client_123
```

## Usos típicos

- Dar identidad separada a cada llamador cuando varias herramientas se conectan al mismo target.
- Asociar operaciones de job, probe, consumer o dx con un cliente concreto.
- Crear primero un cliente y pasar `--client` cuando la automatización requiere propiedad o control de acceso explícito.

## Historial de Versiones

| Versión | Fecha de lanzamiento | Cambios |
|---------|----------------------|---------|
| 0.1.16 | 2026-06-07 | Añadido el grupo de comandos `client` |

## Comandos Relacionados

- [target]({% link commands/target.md %}) - Gestionar targets
- [job]({% link commands/job.md %}) - Gestionar jobs de comando
- [consumer]({% link commands/consumer.md %}) - Gestionar consumidores de resultados

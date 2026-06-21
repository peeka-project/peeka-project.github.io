---
layout: default
title: Comando target
parent: Referencia de Comandos
nav_order: 16
---

# Comando target
{: .no_toc }

Gestiona agentes target de Peeka: descubre targets en el host actual, inspecciona su estado, limpia marcadores stale y separa por target ID.
{: .fs-6 .fw-300 }

## Tabla de Contenidos
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Introducción

El comando `target` **gestiona los target agents de Peeka**: descubre targets de Peeka en el host actual, consulta su estado, limpia marcadores obsoletos y desvincula targets por ID. Es la puerta de entrada al resto de comandos de diagnóstico: solo cuando un target está confirmado como `alive`, comandos como `watch`, `trace` o `inspect` pueden enrutarse correctamente al proceso objetivo.

Con `list`, `current`, `status`, `inspect`, `cleanup` y `detach` se puede indicar con precisión el target sobre el que operar cuando varios coexisten en el mismo host, y recuperar a tiempo los targets obsoletos cuyos marcadores de socket quedaron tras la salida del proceso.

## Sintaxis

```bash
peeka-cli target <subcommand> [options]
```

`target` opera sobre procesos inyectados o gestionados por Peeka. Los estados de target incluyen `alive`, `stale`, `unknown`, `attaching`, `failed` y `detached`.

`session` sigue disponible como alias compatible y obsoleto:

```bash
peeka-cli session list
```

Usa `peeka-cli target ...` en scripts nuevos.

## Subcomandos

| Subcomando | Descripción |
|------------|-------------|
| `list` | Lista los agentes target descubiertos |
| `current` | Devuelve el target actual cuando existe exactamente un target `alive` |
| `status --target <id>` | Muestra un resumen del estado del target |
| `inspect --target <id>` | Muestra detalles completos, capacidades y próximas acciones |
| `cleanup [--target <id>] [--dry-run]` | Limpia marcadores stale; la limpieza solo de stale es el comportamiento predeterminado |
| `detach --target <id> [--force]` | Separa un target; los targets alive requieren `--force` |

Todos los subcomandos soportan:

| Opción | Descripción |
|--------|-------------|
| `--format table` | Salida en tabla por defecto |
| `--format json` | Eventos JSONL para scripts |

## Ejemplos

```bash
# Listar todos los targets
peeka-cli target list

# Devolver el target si hay exactamente un target alive
peeka-cli target current

# Inspeccionar detalles
peeka-cli target inspect --target target_abcd1234 --format json

# Previsualizar limpieza de marcadores stale
peeka-cli target cleanup --dry-run

# Separar un target alive
peeka-cli target detach --target target_abcd1234 --force
```

## Códigos de salida de current

| Código | Significado |
|--------|-------------|
| `0` | Existe exactamente un target alive |
| `1` | No existe ningún target alive |
| `2` | Existen varios targets alive; elige uno explícitamente |

## Historial de Versiones

| Versión | Fecha de lanzamiento | Cambios |
|---------|----------------------|---------|
| 0.1.16 | 2026-06-07 | Añadido el grupo de comandos `target`; `session` queda como alias compatible obsoleto |

## Comandos Relacionados

- [attach]({% link commands/attach.md %}) - Adjuntar a un proceso target
- [detach]({% link commands/detach.md %}) - Separar la sesión actual
- [client]({% link commands/client.md %}) - Gestionar sesiones cliente

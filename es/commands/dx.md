---
layout: default
title: Comando dx
parent: Referencia de Comandos
nav_order: 21
---

# Comando dx
{: .no_toc }

Crea, organiza y exporta paquetes de casos de diagnóstico. Un caso DX reúne secciones de target, client, job, probe, consumer, error, nota y resumen en un contexto exportable.
{: .fs-6 .fw-300 }

## Tabla de Contenidos
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Sintaxis

```bash
peeka-cli dx <subcommand> [options]
```

## Subcomandos

| Subcomando | Descripción |
|------------|-------------|
| `create --target <id> --title <text>` | Crea un caso DX |
| `list` | Lista casos DX |
| `status --dx-case <id>` | Muestra el estado del caso |
| `add --dx-case <id>` | Añade una sección |
| `summary --dx-case <id>` | Genera un resumen |
| `export --dx-case <id>` | Exporta un caso DX |
| `close --dx-case <id>` | Cierra un caso DX |

Opciones comunes:

| Opción | Descripción |
|--------|-------------|
| `--target <id>` | Target propietario; obligatorio en `create`, opcional en otros subcomandos |
| `--client <id>` | Cliente propietario opcional para control de acceso |
| `--format table/json` | Formato de salida, predeterminado `table` |
| `--output-path <path>` | Ruta de destino opcional para `export` |

## Añadir secciones

```bash
peeka-cli dx add \
  --dx-case dx_123 \
  --section-type note \
  --title "Investigation note" \
  --payload-json '{"text":"slow query reproduced"}'
```

`--section-type` acepta `target`, `client`, `job`, `probe`, `consumer`, `note`, `error` y `summary`. Usa `--object-ref-type` y `--object-ref-id` para enlazar un objeto existente.

## Ejemplos

```bash
peeka-cli dx create --target target_abcd1234 --title "Slow request"
peeka-cli dx list --target target_abcd1234
peeka-cli dx summary --dx-case dx_123
peeka-cli dx export --dx-case dx_123 --output-path ./slow-request.dx.json
peeka-cli dx close --dx-case dx_123
```

## Historial de Versiones

| Versión | Fecha de lanzamiento | Cambios |
|---------|----------------------|---------|
| 0.1.16 | 2026-06-07 | Añadido el grupo de comandos `dx` |

## Comandos Relacionados

- [target]({% link commands/target.md %}) - Gestionar targets
- [job]({% link commands/job.md %}) - Recopilar información de jobs de comando
- [probe]({% link commands/probe.md %}) - Recopilar eventos de probe

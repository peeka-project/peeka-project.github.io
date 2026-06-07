---
layout: default
title: Comando top
parent: Referencia de Comandos
nav_order: 12
---

# Comando top
{: .no_toc }

## Tabla de Contenidos
{: .no_toc .text-delta }

1. TOC
{:toc}

## Introducción

El comando `top` es un **perfilador de rendimiento por muestreo a nivel de función**, similar al comando `top` de Linux y a `py-spy top`. Mediante el muestreo periódico de la pila de llamadas de todos los hilos, estadística el uso de CPU de cada función, ayudando a los desarrolladores a localizar rápidamente cuellos de botella en el rendimiento.

**Características de Diseño**:
- **Bajo sobrecosto**: Modo de muestreo, impacto en el rendimiento < 5%
- **Estadísticas en tiempo real**: Salida en flujo de datos de rendimiento, soporta monitoreo en tiempo real
- **Granularidad por función**: Preciso a nivel de función, muestra own time y total time
- **Filtrado automático**: Por defecto filtra los hilos de Peeka itself, evitando interferencias

## Escenarios de Uso

- **Localización de cuellos de botella de rendimiento**: Encontrar las funciones con mayor consumo de CPU
- **Análisis de funciones punto caliente**: Estadística frecuencia de llamada y distribución de consumo de tiempo
- **Monitoreo en tiempo real**: Muestreo continuo para observar cambios en el rendimiento del programa
- **Verificación de optimizaciones**: Comparar datos de rendimiento antes y después de optimizaciones

## Formato del Comando

```bash
peeka-cli attach <pid>    # Primero adjuntar al proceso objetivo
peeka-cli top [options]
```

### Descripción de Parámetros

| Parámetro               | Descripción                                           | Valor por defecto | Ejemplo                                   |
|-------------------------|-------------------------------------------------------|-------------------|-------------------------------------------|
| `-i, --interval`        | Intervalo de muestreo (segundos)                      | `0.01`            | `-i 0.02` (muestreo cada 20ms)            |
| `-c, --cycles`          | Cantidad de ciclos de muestra (-1 = infinito)         | `-1`              | `-c 10` (se detiene automáticamente después de 10 ciclos) |
| `--sort`                | Columna de ordenación (own / total / own-time / total-time) | `own`        | `--sort total`                            |
| `--no-filter-peeka`     | Deshabilitar filtrado de hilos de Peeka (habilitado por defecto) | `false` | `--no-filter-peeka` (mostrar todos los hilos) |

### Descripción de Métricas de Rendimiento

| Métrica         | Descripción                                          | Cálculo                                   |
|-----------------|------------------------------------------------------|-------------------------------------------|
| `own_pct`       | Porcentaje de CPU exclusivo (consumo de tiempo de la función misma) | `own_count / total_samples * 100`       |
| `total_pct`     | Porcentaje total de CPU (incluye funciones hijas llamadas) | `total_count / total_samples * 100`    |
| `own_time`      | Tiempo exclusivo (segundos)                          | `own_count * interval`                   |
| `total_time`    | Tiempo total (segundos, incluye subfunciones)        | `total_count * interval`                 |
| `own_count`     | Cantidad de muestreos exclusivos (veces que la función está en la cima de la pila) | Estadística directa            |
| `total_count`   | Cantidad total de muestreos (veces que la función está en la pila de llamadas) | Estadística sin duplicados        |

## Uso Básico

### 1. Iniciar Análisis de Rendimiento

```bash
# Primero adjuntar al proceso objetivo
peeka-cli attach 12345

# Iniciar comando top (intervalo de muestreo predeterminado 10ms)
peeka-cli top
```

**Ejemplo de Salida** (salida en flujo, actualización una vez por segundo):

```json
{
  "type": "top_snapshot",
  "top_id": "top_a1b2c3d4",
  "total_samples": 1000,
  "sample_interval": 0.01,
  "functions": [
    {
      "name": "compute_matrix",
      "filename": "/app/algorithm.py",
      "line": 156,
      "own_pct": 45.3,
      "total_pct": 58.7,
      "own_time": 0.453,
      "total_time": 0.587,
      "own_count": 453,
      "total_count": 587
    },
    {
      "name": "multiply",
      "filename": "/app/math_utils.py",
      "line": 42,
      "own_pct": 12.8,
      "total_pct": 13.4,
      "own_time": 0.128,
      "total_time": 0.134,
      "own_count": 128,
      "total_count": 134
    },
    {
      "name": "log_result",
      "filename": "/app/logger.py",
      "line": 89,
      "own_pct": 8.2,
      "total_pct": 8.9,
      "own_time": 0.082,
      "total_time": 0.089,
      "own_count": 82,
      "total_count": 89
    }
  ]
}
```

### 2. Ajustar Intervalo de Muestreo

```bash
# Intervalo de muestreo 20ms (menor sobrecosto, pero disminuye precisión)
peeka-cli top -i 0.02

# Intervalo de muestreo 5ms (mayor precisión, pero aumenta el sobrecosto)
peeka-cli top -i 0.005
```

**Recomendaciones para seleccionar el intervalo de muestreo**:
- **Entorno de producción**: 10-20ms (predeterminado 10ms), impacto en rendimiento < 5%
- **Entorno de desarrollo**: 5-10ms, mayor precisión
- **Monitoreo prolongado**: 20-50ms, reduce el sobrecosto

### 3. Limitar Cantidad de Ciclos

```bash
# Se detiene automáticamente después de 10 ciclos
peeka-cli top -c 10

# Se detiene automáticamente después de 60 ciclos (aproximadamente 1 minuto)
peeka-cli top -c 60
```

### 4. Ordenar por Diferentes Campos

```bash
# Ordenar por porcentaje de CPU exclusivo (predeterminado)
peeka-cli top --sort own

# Ordenar por porcentaje total de CPU
peeka-cli top --sort total

# Ordenar por tiempo exclusivo
peeka-cli top --sort own-time

# Ordenar por tiempo total
peeka-cli top --sort total-time
```

### 5. Incluir Hilos de Peeka

```bash
# Mostrar todos los hilos, incluyendo los de Peeka itself
peeka-cli top --no-filter-peeka
```

**Nota**: Habilitar `--no-filter-peeka` mostrará el Agente de Peeka y el hilo de muestreo mismos, usualmente se usa para depurar Peeka o verificar la lógica de filtrado.

## Formato de Salida

### Salida en Flujo (una instantánea por segundo)

```json
{
  "type": "top_snapshot",
  "top_id": "top_a1b2c3d4",
  "total_samples": 1000,
  "sample_interval": 0.01,
  "functions": [
    {
      "name": "compute_matrix",
      "filename": "/app/algorithm.py",
      "line": 156,
      "own_pct": 45.3,
      "total_pct": 58.7,
      "own_time": 0.453,
      "total_time": 0.587,
      "own_count": 453,
      "total_count": 587
    }
  ]
}
```

**Descripción de Campos**:

| Campo              | Descripción                                         | Valor de Ejemplo          |
|--------------------|-----------------------------------------------------|---------------------------|
| `top_id`           | ID de sesión de análisis de rendimiento             | `"top_a1b2c3d4"`          |
| `total_samples`    | Cantidad total de muestreos                          | `1000`                    |
| `sample_interval`  | Intervalo de muestreo (segundos)                    | `0.01`                    |
| `functions`       | Lista de estadísticas de funciones (orden descendente por own_pct) | `[...]`             |

**Elementos del arreglo functions**:

| Campo         | Descripción                                  | Valor de Ejemplo       |
|---------------|----------------------------------------------|------------------------|
| `name`        | Nombre de la función                         | `"compute_matrix"`     |
| `filename`    | Ruta del archivo fuente                      | `"/app/algorithm.py"`  |
| `line`        | Número de línea de definición de la función | `156`                  |
| `own_pct`     | Porcentaje de CPU exclusivo                  | `45.3`                 |
| `total_pct`   | Porcentaje total de CPU (incluye subfunciones) | `58.7`               |
| `own_time`    | Tiempo exclusivo (segundos)                  | `0.453`                |
| `total_time`  | Tiempo total (segundos, incluye subfunciones) | `0.587`               |
| `own_count`   | Cantidad de muestreos exclusivos             | `453`                  |
| `total_count` | Cantidad total de muestreos (sin duplicados)| `587`                  |

## Metadatos de runtime gevent

Desde v0.1.16, `top_snapshot` incluye un objeto `meta` que describe la política de compatibilidad de runtime activa:

| Campo | Descripción |
|-------|-------------|
| `meta.gevent_state` | Estado de gevent: `none`, `imported`, `patched` o `active_hub` |
| `meta.backend` | Backend de muestreo: normalmente `frame_walk`; puede ser `greenlet_aware_sampling` en runtimes gevent patched/active hub |
| `meta.greenlet_blind` | Indica si greenlet puede ocultar ejecución suspendida al muestreo de frames |
| `meta.degraded_reason` | Motivo de degradación, o `null` si no hay degradación |
| `greenlet_events` | Presente con `greenlet_aware_sampling`; contiene contadores de switch/throw de greenlet |

Cuando los trace hooks de greenlet no están disponibles, `top` vuelve a `frame_walk` y reporta el motivo en `degraded_reason`.

## Ejemplos de Monitoreo en Tiempo Real

### Usar jq para filtrar las 5 funciones punto caliente principales

```bash
peeka-cli top | jq 'select(.type == "top_snapshot") | .functions[:5]'
```

### Estadística de consumo de CPU de un módulo específico

```bash
peeka-cli top | jq '
  select(.type == "top_snapshot") |
  .functions[] |
  select(.filename | contains("/app/")) |
  {name, own_pct}
'
```

### Exportar datos de rendimiento a un archivo

```bash
# Detenerse después de recolectar 100 instantáneas, guardar en un archivo
peeka-cli top -c 100 > performance.jsonl
```

## Uso en TUI

En modo TUI, el comando `top` proporciona una interfaz visual de análisis de rendimiento:

1. Iniciar TUI:
   ```bash
   peeka  # o python -m peeka.tui
   ```

2. Conectarse al proceso objetivo (ingresa PID o selecciona un proceso)

3. Presiona la tecla **`0`** para cambiar a la **vista Top - Perfilador de funciones**

4. Funcionalidades:
   - Mostrar en tiempo real la clasificación de rendimiento de funciones
   - Soporte para ordenar por own_pct / total_pct / own_time / total_time
   - Codificación por colores: rojo (alto CPU), amarillo (medio), verde (bajo)
   - Muestra estadísticas de muestreo (total_samples, interval)
   - Soporte para pausar/reanudar muestreo

5. Atajos de teclado:
   - `r`: Actualizar datos
   - `s`: Cambiar campo de ordenación
   - `Enter`: Iniciar análisis de rendimiento
   - `Delete`: Detener análisis de rendimiento
   - `c`: Vaciar datos estadísticos (reset)

## Principio de Funcionamiento

### Flujo de Muestreo

1. **Iniciar hilo de muestreo**: Ejecuta un hilo en segundo plano con el intervalo especificado (predeterminado 10ms)
2. **Obtener instantánea de hilos**: Llama a `sys._current_frames()` para obtener el marco de pila actual de todos los hilos
3. **Filtrar hilos**: Excluye los hilos de Peeka itself (a menos que se use `--no-filter-peeka`)
4. **Recorrer pila de llamadas**: Estadística marco por marco desde la cima de la pila (leaf frame) hasta la base (root frame)
5. **Actualizar estadísticas**:
   - Marco de la cima: `own_count += 1` (la función se está ejecutando a sí misma)
   - Todos los marcos: `total_count += 1` (sin duplicados, incluye la cadena de llamada)
6. **Generar instantánea**: Genera una instantánea una vez por segundo y la envía al cliente

### Estrategia de Filtrado

Por defecto se filtran los siguientes hilos (`--no-filter-peeka` lo deshabilita):
- Hilos cuyo nombre empieza con `peeka-`
- Hilos cuya ruta de ejecución del código está en el directorio del paquete Peeka
- El propio hilo de muestreo

### Cálculo de Métricas Estadísticas

```python
# Porcentaje exclusivo
own_pct = (own_count / total_samples) * 100

# Porcentaje total
total_pct = (total_count / total_samples) * 100

# Tiempo exclusivo
own_time = own_count * sample_interval

# Tiempo total
total_time = total_count * sample_interval
```

## Notas Importantes

1. **Precisión de muestreo**:
   - El modo de muestreo no puede capturar todas las llamadas a funciones, solo estadística los marcos de pila en el momento del muestreo
   - Las funciones que se ejecutan en poco tiempo pueden ser omitidas
   - Es adecuado para análisis de rendimiento de larga duración, no para microbenchmarks

2. **Sobrecosto de rendimiento**:
   - Intervalo predeterminado de 10ms: impacto en rendimiento < 5%
   - Intervalo de 5ms: impacto en rendimiento aproximadamente 5-10%
   - Intervalo de 1ms: impacto en rendimiento 10-20% (no recomendado)

3. **Filtrado de hilos**:
   - Por defecto se filtran los hilos de Peeka para evitar contaminación de los datos estadísticos
   - Puedes usar `--no-filter-peeka` para ver todos los hilos (incluyendo Peeka)

4. **Frecuencia de salida**:
   - Modo CLI: salida de una instantánea por segundo (fijo)
   - El intervalo de muestreo y la frecuencia de salida son independientes

5. **Forma de detener**:
   - Ctrl+C: detener normalmente, salida de la instantánea final
   - Parámetro `--cycles`: detener automáticamente
   - Modo TUI: presiona la tecla `Delete` para detener

6. **Diferencia con el comando trace**:
   - `top`: modo de muestreo, bajo sobrecosto, adecuado para monitoreo prolongado
   - `trace`: modo de instrumentación, alta precisión, adecuado para análisis de cadena de llamada de corta duración
   - `top` estadística todas las funciones, `trace` solo rastrea funciones especificadas

7. **Requisitos de permisos**:
   - Necesitas usar primero el comando `attach` para adjuntar al proceso objetivo
   - Los requisitos de permisos para adjuntar se describen en la [documentación del comando attach](attach)

## Historial de Versiones

| Versión | Fecha de lanzamiento | Cambios |
|---------|----------------------|---------|
| 0.1.16 | 2026-06-07 | Añadido `meta` de compatibilidad runtime a `top_snapshot`; runtimes gevent patched/active hub soportan muestreo greenlet-aware y motivos de degradación |

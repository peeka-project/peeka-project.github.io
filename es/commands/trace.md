---
layout: default
title: Comando trace
parent: Referencia de Comandos
nav_order: 3
---

# Comando trace
{: .no_toc }

## Tabla de Contenidos
{: .no_toc .text-delta }

1. TOC
{:toc}

## Introducción

El comando `trace` se usa para rastrear los **llamados directos** de funciones Python y su tiempo de ejecución, mostrando como lista plana la información agregada de las subllamadas de un solo nivel bajo la función objetivo. Es una potente herramienta de análisis de rendimiento que ayuda a identificar rápidamente cuellos de botella y puntos calientes de ejecución.

**Cambio importante en v0.1.20**: `trace` ya no configura niveles de profundidad ni emite árboles de llamadas recursivos anidados. El backend solo captura y agrega los llamados directos (direct callees) de la función objetivo, con una salida más estable y un sobrecoste más predecible.

## Uso en TUI

En modo TUI, presiona la tecla **`3`** para cambiar a la **Vista Trace**, que proporciona las siguientes características interactivas:

- **Entrada de Patrón**: Soporta autocompletado de nombres de funciones (obtenido en tiempo real desde el proceso objetivo)
- **Configuración de Parámetros**: Configuración visual de duración mínima, número de observaciones, expresiones condicionales, skip-builtin
- **Lista de rastreos activos**: La parte superior muestra el estado y conteo de las tareas de trace actuales
- **Visualización de árbol de llamadas**: Muestra en una estructura interactiva los llamados directos de cada observación, con nodos callee agregados
- **Panel de estadísticas detalladas**: Muestra duración, conteo, porcentaje y excepciones de la observación o callee seleccionado
- **Temporización con Codificación de Colores**:
  - 🟢 Verde: < 10ms (rápido)
  - 🟡 Amarillo: 10-100ms (medio)
  - 🔴 Rojo: >= 100ms (lento)
- **Operaciones Rápidas**:
  - Presiona Enter después de ingresar el patrón para iniciar el rastreo
  - Selecciona una fila en Active Traces para ver el árbol de llamadas de ese pattern
  - Selecciona un nodo callee en el árbol de llamadas y presiona `t` para iniciar un trace de profundización (Drill Trace)
  - Presiona `c` para limpiar registros de rastreo y detener traces en ejecución
  - Presiona Delete para detener todos los traces

![Vista Trace de Peeka]({{ site.url }}/assets/images/screenshots/peeka-trace.png)

**Comandos CLI equivalentes**: Todos los ejemplos a continuación usan comandos CLI para demostración. TUI proporciona la misma funcionalidad con una interfaz gráfica.

## Escenarios de Uso

- **Identificación de cuellos de botella de rendimiento**: Encuentra rápidamente las subllamadas más lentas bajo la función objetivo
- **Identificación de funciones calientes**: Agrega duración total, conteo, duración mínima y máxima de llamados directos
- **Rastreo de rutas de ejecución de código**: Observar rutas de ejecución de código bajo diferentes condiciones
- **Análisis de temporización de sub-funciones**: Analizar la distribución de tiempo entre sub-funciones
- **Diagnóstico de llamadas con excepción**: Rastrear llamadas que lanzan excepciones y ver el tipo de excepción

## Formato del Comando

```bash
peeka-cli attach <pid>    # Primero adjuntar al proceso objetivo
peeka-cli trace <pattern> [options]
```

### Parámetros

| Parámetro | Descripción | Valor por defecto | Ejemplo |
|-----------|-------------|-------------------|---------|
| `pattern` | Patrón de coincidencia de función | - | `module.Class.method` |
| `-n, --times` | Número de observaciones (-1 para ilimitado) | `-1` | `-n 10` |
| `--condition` | Expresión de condición (soporta variable `cost`) | Ninguno | `--condition "cost > 50"` |
| `--client` | ID de sesión cliente existente; si se omite, se crea un cliente efímero automáticamente | Automático | `--client client_123` |
| `--skip-builtin` | Omitir funciones integradas y de la biblioteca estándar | `true` | `--skip-builtin=false` |
| `--min-duration` | Filtro de duración mínima (milisegundos), solo registra llamados directos con duración mayor o igual a este valor | `0` | `--min-duration 10` |

**Notas**:
- `--skip-builtin` está habilitado por defecto para reducir el ruido de salida
- La variable `cost` en expresiones de condición representa la duración total de la llamada (milisegundos)
- Desde v0.1.20 la salida se limita a llamados directos agregados

### Patrón de Coincidencia de Función (pattern)

Soporta los siguientes formatos:

```python
# 1. Función a nivel de módulo
"mymodule.my_function"

# 2. Método de clase
"mymodule.MyClass.my_method"

# 3. Método de clase anidada
"mypackage.mymodule.OuterClass.InnerClass.method"

# 4. Ruta de módulo
"package.subpackage.module.function"
```

**Nota**: Debe usar la ruta completa del módulo (desde la raíz de importación). La versión actual no soporta coincidencia con comodines.

## Uso Básico

### 1. Rastrear llamadas de función

```bash
# Primero adjuntar al proceso objetivo
peeka-cli attach 12345

# Rastrear 5 llamadas
peeka-cli trace "calculator.Calculator.calculate" -n 5
```

**Ejemplo de Salida**:

```json
{
  "type": "observation",
  "watch_id": "trace_abc123",
  "timestamp": 1705586200.123,
  "func_name": "calculator.Calculator.calculate",
  "location": "AtExit",
  "call_tree": [
    {
      "function": "calculator.Calculator._validate",
      "filename": "/app/calculator.py",
      "lineno": 18,
      "count": 1,
      "total_ms": 2.1,
      "min_ms": 2.1,
      "max_ms": 2.1
    },
    {
      "function": "calculator.Calculator._compute",
      "filename": "/app/calculator.py",
      "lineno": 25,
      "count": 1,
      "total_ms": 98.2,
      "min_ms": 98.2,
      "max_ms": 98.2
    },
    {
      "function": "calculator.Logger.info",
      "filename": "/app/logger.py",
      "lineno": 10,
      "count": 1,
      "total_ms": 15.7,
      "min_ms": 15.7,
      "max_ms": 15.7
    }
  ],
  "total_duration_ms": 125.3,
  "self_time_ms": 9.3,
  "callee_count": 3,
  "node_count": 4,
  "thread_id": 140234567890,
  "thread_name": "MainThread"
}
```

**Descripción de Campos**:

| Campo | Descripción | Ejemplo |
|-------|-------------|---------|
| `watch_id` | ID de observación | `"trace_abc123"` |
| `timestamp` | Marca de tiempo | `1705586200.123` |
| `func_name` | Nombre de la función objetivo | `"calculator.calculate"` |
| `location` | Ubicación de observación | `"AtExit"` |
| `call_tree` | Lista de llamados directos (agregación plana) | `[...]` |
| `total_duration_ms` | Tiempo total de ejecución (milisegundos) | `125.3` |
| `self_time_ms` | Tiempo consumido por la función objetivo en sí (milisegundos) | `9.3` |
| `callee_count` | Número de tipos de llamados directos | `3` |
| `node_count` | Conteo total de nodos (función objetivo + llamados directos) | `4` |
| `thread_id` | ID del hilo | `140234567890` |
| `thread_name` | Nombre del hilo | `"MainThread"` |
| `exception` | Información de excepción (si se lanza) | `"ValueError: ..."` |
| `runtime_meta` | Metadatos de runtime (backend, gevent, etc.) | `{...}` |

**Campos de Nodo de Árbol de Llamadas**:

| Campo | Descripción | Ejemplo |
|-------|-------------|---------|
| `function` | Nombre completo de la función | `"module.Class.method"` |
| `filename` | Ruta del archivo | `"/app/module.py"` |
| `lineno` | Número de línea | `42` |
| `count` | Número de llamadas durante este ciclo de observación | `5` |
| `total_ms` | Tiempo total de ejecución (milisegundos) | `125.3` |
| `min_ms` | Duración mínima (milisegundos) | `10.5` |
| `max_ms` | Duración máxima (milisegundos) | `95.1` |

### 2. Árbol de llamadas visual (TUI)

En modo TUI, la vista Trace usa un diseño vertical:

![Árbol de llamadas Trace de Peeka]({{ site.url }}/assets/images/screenshots/peeka-trace.png)

**Explicación**:
- Active Traces en la parte superior muestra las tareas actuales (Pattern / Status / Count)
- Call Tree en la parte inferior izquierda muestra nodos de observación, nodos callee agregados y el porcentaje de tiempo de cada callee
- Stats en la parte inferior derecha muestra estadísticas detalladas de la observación o callee seleccionado
- Los colores resaltan distintos rangos de tiempo
- Tras seleccionar un callee, presiona `t` para iniciar rápidamente un nuevo trace de profundización

### 3. Filtrar por duración mínima

```bash
# Registrar solo llamados directos con duración >= 10ms
peeka-cli trace "service.process" --min-duration 10
```

Esto reduce el ruido de funciones auxiliares de alta frecuencia y corta duración, enfocándose en subllamadas con consumo real de recursos.

### 4. Filtrado Condicional

```bash
# Solo rastrear llamadas que exceden 50ms
peeka-cli trace "api.handler" --condition "cost > 50"

# Combinar condiciones de parámetros y temporización
peeka-cli trace "service.query" --condition "cost > 100 and params[0] > 1000"
```

### 5. Omitir Funciones Integradas

```bash
# Comportamiento predeterminado: omitir funciones integradas (reducir ruido de salida)
peeka-cli trace "mymodule.func"

# Mostrar todas las llamadas (incluyendo funciones integradas)
peeka-cli trace "mymodule.func" --skip-builtin=false
```

**Ejemplos de Funciones Integradas**:
- Funciones integradas de Python: `len()`, `str()`, `isinstance()`, `print()`
- Funciones de biblioteca estándar: `json.dumps()`, `os.path.join()`, `datetime.now()`

## Tecnología de Implementación

El comando `trace` de Peeka selecciona automáticamente la implementación óptima según la versión de Python:

### Principios de Implementación

El comando `trace` de Peeka selecciona automáticamente la implementación óptima según la versión de Python:

| Versión de Python | Implementación | Sobrecoste de Rendimiento | Notas |
|----------------|----------------|----------------------|-------|
| 3.12+ | sys.monitoring | < 5% | API oficial de PEP 669, rendimiento óptimo |
| 3.8.1-3.11 | sys.settrace | < 20% | Buena compatibilidad, habilitado automáticamente |

**Semántica de llamados directos (v0.1.20+)**: todos los backends capturan solo los llamados directos de la función objetivo y agregan, dentro del mismo ciclo de observación, llamadas con el mismo `(function, filename, lineno)`, emitiendo `count / total_ms / min_ms / max_ms`. Esto evita la incertidumbre de rendimiento y el crecimiento de datos de los árboles profundos.

**Compatibilidad con gevent (v0.1.15+)**: cuando el proceso objetivo tiene monkey patching de gevent o un hub activo, `trace` se degrada al backend `wrapper_only` para evitar que `sys.settrace` rompa invariantes de la pila de frames. Este modo sigue reportando observaciones de la función objetivo, pero no proporciona una lista de llamados directos.

**Implementación sys.monitoring (Python 3.12+)**:

- Basado en la API oficial de monitoreo de [PEP 669](https://peps.python.org/pep-0669/)
- Usa eventos `PY_START` y `PY_RETURN` para capturar llamadas
- Sobrecoste de rendimiento < 5%, recomendado para entornos de producción
- Asigna automáticamente tool_id, sin conflictos con múltiples observaciones

**Implementación sys.settrace (Python 3.8.1-3.11)**:

- Usa el mecanismo incorporado `sys.settrace()` de Python
- Habilitado solo durante la ejecución de la función objetivo (rastreo local)
- Sobrecoste de rendimiento < 20%, completamente utilizable en la mayoría de escenarios

**Mecanismo de Filtrado skip-builtin**:

- Verifica `code.co_filename.startswith('<')` para filtrar funciones integradas (ej., `<built-in>`)
- Verifica rutas de la biblioteca estándar de Python para filtrar funciones de la biblioteca estándar
- Habilitado por defecto, reduce los nodos de salida en más de un 50%

## Impacto en el Rendimiento

### Sobrecoste de Rendimiento

| Escenario | Sobrecoste | Notas |
|----------|----------|-------|
| **Funciones simples** | < 5% | Python 3.12+ |
| **Funciones simples** | < 20% | Python 3.8.1-3.11 |
| **Subllamadas de alta frecuencia** | 10-30% | Depende de la versión de Python y de `--min-duration` |
| **Llamadas de alta frecuencia (>1000 QPS)** | 20-50% | Recomienda limitar el número de observaciones |

**Explicación**:
- Python 3.12+ usa `sys.monitoring`, reduciendo significativamente el sobrecoste
- Desde v0.1.20 solo se rastrea un nivel de llamados directos, con sobrecoste más estable y predecible
- Se recomienda usar filtrado condicional y límites de conteo en producción

### Recomendaciones de Optimización de Rendimiento

1. **Usar filtro de duración mínima**
   ```bash
   # Registrar solo llamados directos con duración >= 10ms
   peeka-cli trace "func" --min-duration 10
   ```

2. **Omitir funciones integradas**
   ```bash
   # Habilitado por defecto, reduce los nodos en más de un 50%
   peeka-cli trace "func" --skip-builtin
   ```

3. **Usar filtrado condicional**
   ```bash
   # Solo rastrear llamadas lentas
   peeka-cli trace "func" --condition "cost > 100"
   ```

4. **Limitar número de observaciones**
   ```bash
   # Solo observar 10 veces
   peeka-cli trace "func" -n 10
   ```

## Ejemplos de Uso

### 1. Identificar Cuellos de Botella de Rendimiento

```bash
# Rastrear endpoints lentos, encontrar sub-llamadas con mayor duración
peeka-cli trace "api.handler.process_request" --condition "cost > 100"
```

**Salida**:
```
`---[1250ms] api.handler.process_request()
    +---[10ms] api.validator.check_params()
    +---[1200ms] database.query.execute()  ← ¡Cuello de botella aquí!
    `---[20ms] api.formatter.to_json()
```

**Conclusión**: La consulta de base de datos consume el 96% del tiempo, necesita optimización SQL o adición de índice.

### 2. Agregar llamadas de alta frecuencia

```bash
# Rastrear una función dentro de un bucle para observar subllamadas repetidas
peeka-cli trace "algorithm.process_batch" -n 5
```

**Salida de ejemplo**:
```json
{
  "func_name": "algorithm.process_batch",
  "call_tree": [
    {
      "function": "database.query.fetch",
      "count": 100,
      "total_ms": 850.5,
      "min_ms": 5.1,
      "max_ms": 25.3
    }
  ]
}
```

**Conclusión**: `process_batch` disparó 100 consultas de base de datos en una sola ejecución; considera optimizar con consultas por lotes.

### 3. Comprender Ruta de Ejecución de Código

```bash
# Rastrear rutas de ejecución de ramas condicionales
peeka-cli trace "service.business_logic" -n 1
```

**Escenario A (flujo normal)**:
```
`---[50ms] service.business_logic()
    +---[5ms] service.validate_input()
    +---[30ms] service.process_data()
    `---[10ms] service.save_result()
```

**Escenario B (flujo de excepción)**:
```
`---[20ms] service.business_logic()
    +---[5ms] service.validate_input()
    +---[10ms] service.handle_invalid_input()
    `---[3ms] service.log_error()
```

### 4. Comparar Rendimiento Antes/Después de Optimización

```bash
# Antes de optimización
peeka-cli trace "converter.parse_json" -n 10 > before.jsonl

# Después de optimización
peeka-cli trace "converter.parse_json" -n 10 > after.jsonl

# Analizar cambios de temporización
jq '.total_duration_ms' before.jsonl | awk '{sum+=$1; count++} END {print "Before:", sum/count, "ms"}'
jq '.total_duration_ms' after.jsonl | awk '{sum+=$1; count++} END {print "After:", sum/count, "ms"}'
```

### 5. Integrar en CI/CD

```bash
# Prueba de regresión de rendimiento
#!/bin/bash
THRESHOLD=100  # Duración máxima permitida 100ms

peeka-cli attach $PID
RESULT=$(peeka-cli trace "critical.function" -n 50 | \
  jq -s 'map(select(.type == "observation")) | map(.total_duration_ms) | add / length')

if (( $(echo "$RESULT > $THRESHOLD" | bc -l) )); then
  echo "Regresión de rendimiento detectada: ${RESULT}ms > ${THRESHOLD}ms"
  exit 1
fi
```

## Procesamiento y Análisis de Datos

### Procesar JSON con jq

```bash
# 1. Extraer lista de llamados directos
peeka-cli trace "func" | jq '.call_tree'

# 2. Calcular duración promedio
peeka-cli trace "func" -n 100 | jq '.total_duration_ms' | \
  awk '{sum+=$1; count++} END {print "avg:", sum/count, "ms"}'

# 3. Encontrar la subllamada más lenta
peeka-cli trace "func" | jq '.call_tree | sort_by(.total_ms) | reverse | .[0]'

# 4. Contar frecuencia de llamadas (agregando el campo count)
peeka-cli trace "func" -n 100 | jq -s '[.[] | .call_tree[] | {function, count}] | group_by(.function) | map({function: .[0].function, total_count: map(.count) | add}) | sort_by(.total_count) | reverse'

# 5. Generar datos para flame graph
peeka-cli trace "func" -n 1000 | jq -r '.call_tree[] | "\(.function) \(.total_ms)"' > flamegraph.txt
```

### Análisis de Datos con Python

```python
import json
import sys
from collections import defaultdict

# Contar duración total y ocurrencias de llamados directos
stats = defaultdict(lambda: {"count": 0, "total_ms": 0})

for line in sys.stdin:
    data = json.loads(line)
    if data["type"] == "observation":
        for callee in data.get("call_tree", []):
            func = callee.get("function")
            if func:
                stats[func]["count"] += callee.get("count", 1)
                stats[func]["total_ms"] += callee.get("total_ms", 0)

# Ordenar por duración total
sorted_stats = sorted(stats.items(), key=lambda x: x[1]["total_ms"], reverse=True)

print("Top 10 Funciones que Consumen Tiempo:")
print(f"{'Función':<60} {'Conteo':>10} {'Total (ms)':>15} {'Promedio (ms)':>12}")
print("-" * 100)

for func, stat in sorted_stats[:10]:
    avg_ms = stat["total_ms"] / stat["count"]
    print(f"{func:<60} {stat['count']:>10} {stat['total_ms']:>15.2f} {avg_ms:>12.2f}")
```

**Ejecutar**:
```bash
peeka-cli trace "module.func" -n 100 | python analyze_trace.py
```

**Salida**:
```
Top 10 Funciones que Consumen Tiempo:
Función                                                      Conteo     Total (ms)      Promedio (ms)
----------------------------------------------------------------------------------------------------
database.query.execute                                          100       12500.00       125.00
api.handler.process_request                                     100       15000.00       150.00
json.dumps                                                      500        1000.00         2.00
...
```

## Problemas Comunes

### 1. ¿Por qué no aparecen llamadas más profundas?

**Problema**: El árbol solo muestra los llamados directos de la función objetivo, no los llamados de las subllamadas.

**Causa**: Desde v0.1.20, `trace` captura y agrega solo direct callees para ofrecer rendimiento más estable y salida más predecible.

**Solución**:

```bash
# Si necesitas observar el interior de una subllamada, inicia un trace separado sobre esa función
peeka-cli trace "module.sub_module.slow_func" -n 10
```

### 2. Demasiados Datos de Salida

**Problema**: Contiene muchas llamadas a funciones integradas, salida difícil de leer

**Solución**:

```bash
# Omitir funciones integradas (habilitado por defecto)
peeka-cli trace "module.func" --skip-builtin

# Registrar solo llamadas > 10ms
peeka-cli trace "module.func" --min-duration 10

# Usar filtrado condicional
peeka-cli trace "module.func" --condition "cost > 50"
```

### 3. Sobrecoste de Rendimiento Excesivo

**Problema**: La respuesta de la aplicación se ralentiza después de habilitar el rastreo

**Solución**:

```bash
# 1. Aumentar el umbral de duración mínima para registrar menos nodos
peeka-cli trace "module.func" --min-duration 10

# 2. Limitar número de observaciones
peeka-cli trace "module.func" -n 10

# 3. Usar filtrado condicional, rastrear solo llamadas lentas
peeka-cli trace "module.func" --condition "cost > 100"

# 4. Considerar actualizar a Python 3.12+ para mejor rendimiento
```

### 4. Sin Datos Observados

**Posibles Causas**:
- Función no llamada
- Error de ortografía en nombre de función
- Expresión de condición demasiado estricta
- Límite de número de observaciones alcanzado (parámetro -n)

**Pasos de Solución de Problemas**:

```bash
# 1. Confirmar que el nombre de la función es correcto
python3 -c "import mymodule; print(mymodule.MyClass.my_method)"

# 2. Eliminar expresión de condición, observar una vez primero
peeka-cli trace "mymodule.func" -n 1

# 3. Verificar si el proceso existe
ps aux | grep <pid>
```

## Consejos Avanzados

### 1. Generar Flame Graph

```bash
# Recolectar datos de rastreo
peeka-cli trace "module.func" -n 1000 > trace.jsonl

# Convertir a formato flame graph (llamados directos plegados por total_ms)
jq -r '.call_tree[] | "\(.function);\(.total_ms)"' trace.jsonl \
  > folded.txt

# Generar flame graph (requiere instalación de flamegraph.pl)
flamegraph.pl folded.txt > flamegraph.svg
```

### 2. Comparar Rendimiento entre Versiones

```bash
# Versión A
git checkout v1.0
peeka-cli trace "module.func" -n 100 > trace_v1.jsonl

# Versión B
git checkout v2.0
peeka-cli trace "module.func" -n 100 > trace_v2.jsonl

# Comparar duración promedio
echo "v1.0: $(jq -s 'map(.total_duration_ms) | add / length' trace_v1.jsonl) ms"
echo "v2.0: $(jq -s 'map(.total_duration_ms) | add / length' trace_v2.jsonl) ms"
```

### 3. Monitoreo Automatizado de Rendimiento

```python
#!/usr/bin/env python3
"""Script de monitoreo de regresión de rendimiento"""
import json
import subprocess
import time

THRESHOLD = 100  # Duración máxima permitida (ms)
CHECK_INTERVAL = 3600  # Intervalo de verificación (segundos)

def check_performance(pid, pattern):
    cmd = ["peeka-cli", "trace", pattern, "-n", "50"]
    proc = subprocess.Popen(cmd, stdout=subprocess.PIPE, text=True)

    durations = []
    for line in proc.stdout:
        data = json.loads(line)
        if data["type"] == "observation":
            durations.append(data["total_duration_ms"])

    avg_duration = sum(durations) / len(durations) if durations else 0

    if avg_duration > THRESHOLD:
        send_alert(f"Regresión de rendimiento: {avg_duration:.2f}ms > {THRESHOLD}ms")

    return avg_duration

def send_alert(message):
    # Enviar alerta (email, Slack, DingTalk, etc.)
    print(f"ALERTA: {message}")

if __name__ == "__main__":
    pid = int(sys.argv[1])
    pattern = sys.argv[2]

    while True:
        duration = check_performance(pid, pattern)
        print(f"[{time.strftime('%Y-%m-%d %H:%M:%S')}] Duración promedio: {duration:.2f}ms")
        time.sleep(CHECK_INTERVAL)
```

### 4. Integrar con Prometheus

```python
from prometheus_client import Histogram, start_http_server
import json
import subprocess

# Definir métricas
trace_duration = Histogram('trace_duration_ms', 'Duración de rastreo de función', ['function'])

# Iniciar servidor Prometheus
start_http_server(8000)

# Recolectar datos de rastreo
proc = subprocess.Popen(
    ["peeka-cli", "trace", "module.func"],
    stdout=subprocess.PIPE,
    text=True
)

for line in proc.stdout:
    data = json.loads(line)
    if data["type"] == "observation":
        # Procesar lista de llamados directos
        for callee in data.get("call_tree", []):
            func = callee.get("function")
            total_ms = callee.get("total_ms", 0)
            if func:
                trace_duration.labels(function=func).observe(total_ms)
```

## Referencias

- [PEP 669: Low Impact Monitoring for CPython](https://peps.python.org/pep-0669/)
- [Diseño de Arquitectura de Peeka](../architecture.md)

## Registro de Cambios

| Versión | Fecha | Actualizaciones |
|---------|------|---------|
| 0.2.0 | 2026-02 | Agregada documentación del comando trace |
| 0.1.0 | 2025-01 | Versión inicial |

## Historial de cambios

| Versión | Fecha | Cambios |
|---------|-------|---------|
| 0.1.20 | 2026-07-05 | Trace captura y agrega solo los llamados directos (direct callees) de la función objetivo; `call_tree` pasa a ser una lista plana y añade `self_time_ms` y `callee_count`; la vista Trace de TUI usa lista superior Active Traces + panel inferior de árbol/estadísticas, con tecla `t` para profundizar en un callee y nodos callee agregados |
| 0.1.18 | 2026-06-24 | CLI `--times` ahora cuenta solo observaciones cuyo `stream_id` coincida con el flujo activo, evitando que flujos concurrentes se sumen al límite del trace actual; el wrapper `run` detiene el flujo de trace al alcanzar el límite |
| 0.1.17 | 2026-06-13 | Las respuestas de trace incluyen `runtime_meta` (`startup_backend`, `effective_backend`, `downgrade_reason`) cuando se degradan al backend `wrapper_only`; la vista Trace de la TUI muestra Backend / Gevent en el panel de estadísticas |
| 0.1.16 | 2026-06-07 | Añadido `--client` |
| 0.1.15 | 2026-05-27 | Runtimes gevent patched/active hub se degradan al backend `wrapper_only` de trace |
| 0.1.12 | 2026-05-08 | Sistema de paneles TUI unificado, diseños responsivos refinados (commit 50c4af4) |
| 0.1.11 | 2026-05-07 | Etiquetado de clientes con fuentes estables (commit 965ff22), diagnósticos de actividad enriquecidos (commit b1b0412) |
| 0.1.10 | 2026-05-04 | Normalización de colores de botones en TUI (commit fd6a0a1), mejora de legibilidad del ajuste de línea del registro de actividad (commit 5f46ae8) |

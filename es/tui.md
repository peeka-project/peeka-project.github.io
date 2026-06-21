---
layout: default
title: Guía de Uso de TUI
nav_order: 6
---

# Guía de Uso de TUI
{: .no_toc }

## Tabla de Contenidos
{: .no_toc .text-delta }

1. TOC
{:toc}

## Introducción

Peeka proporciona una **Interfaz de Usuario de Texto (TUI)** completa, construida sobre el framework [Textual](https://textual.textualize.io/). El modo TUI proporciona una experiencia de interacción más intuitiva que CLI, y soporta flujo de datos en tiempo real, operaciones interactivas, salida en color y navegación con atajos de teclado.

**Escenarios de aplicación**:
- **Diagnóstico interactivo**: Necesitas cambiar frecuentemente de comandos y ver datos en tiempo real
- **Monitoreo en tiempo real**: Observar métricas de rendimiento, salida de registros, cambios de memoria
- **Análisis visual**: Estructura de árbol para mostrar cadena de llamadas, datos de rendimiento codificados por color
- **Depuración exploratoria**: No estás seguro de qué comando usar, explora paso a paso a través de TUI

## Iniciar TUI

### Método 1: Inicio directo

```bash
peeka
```

### Método 2: Como módulo Python

```bash
python -m peeka.tui
```

### Opciones de inicio

```bash
# Usar tema personalizado
peeka --theme dracula

# Listar temas disponibles
peeka --list-themes
```

**Temas disponibles**:
- `default` - Tema por defecto de Textual
- `nord` - Esquema de colores Nord
- `dracula` - Esquema de colores Dracula
- `gruvbox` - Esquema de colores Gruvbox
- `monokai` - Esquema de colores Monokai
- `solarized-light` - Solarized Light
- `solarized-dark` - Solarized Dark

## Selector de procesos

Desde v0.1.16, el selector inicial muestra tanto agentes target gestionados por Peeka como procesos Python adjuntables:

- Los targets `alive` se muestran por target ID; los procesos Python normales aparecen como `attachable`
- En terminales anchas se muestran Name, PID, State, Python, Command y Created; en terminales estrechas se conservan Name, State y Created
- El filtro busca por nombre, comando, PID o estado
- `r` refresca la lista, `Enter` selecciona la fila actual, `y` copia el attach log y `escape` / `q` sale
- Las filas stale, failed, detached y unknown se muestran para confirmar estado, pero no son targets seleccionables para adjuntar

## Diseño de la Interfaz TUI

Después de iniciar TUI, la interfaz se divide en las siguientes áreas:

![Vista Dashboard de Peeka TUI]({{ site.url }}/assets/images/screenshots/peeka-dashboard.png)

### Descripción de Áreas

- **Header**: Muestra el título de TUI y la hora actual
- **PID Status**: Muestra el PID del proceso objetivo actualmente adjuntado
- **Tab Bar**: Muestra todas las etiquetas de vistas disponibles
- **View Content**: Área de contenido de la vista actualmente activada
- **Footer**: Muestra sugerencias de atajos de teclado globales

## Atajos de Teclado Globales

| Atajo     | Función                | Descripción                      |
|---------|-------------------|-------------------------------|
| `1`     | Vista Dashboard     | Cambiar al panel                  |
| `2`     | Vista Watch         | Cambiar a observación de funciones                 |
| `3`     | Vista Trace         | Cambiar a rastreo de cadena de llamadas                |
| `4`     | Vista Stack         | Cambiar a captura de pila de llamadas                |
| `5`     | Vista Monitor       | Cambiar a monitoreo de rendimiento                 |
| `6`     | Vista Memory        | Cambiar a análisis de memoria                 |
| `7`     | Vista Logger        | Cambiar a gestión de registros                 |
| `8`     | Vista Inspect       | Cambiar a inspección de objetos                 |
| `9`     | Vista Threads       | Cambiar a gestión de hilos                 |
| `0`     | Vista Top           | Cambiar a Perfilador de Funciones (Perfilador de funciones)                |
| `?`     | Ayuda Help          | Muestra información de ayuda (disponible en algunas vistas)        |
| `escape` / `q` | Volver / Salir | Sale en Dashboard, vuelve a Dashboard en otras vistas |

## Diez Vistas Detalladas

### 1. Vista Dashboard (tecla `1`)

**Función**: Vista general de diagnóstico en tiempo real del proceso objetivo

**Características**:
- Resumen del número de hilos y sus estados
- Lista de hilos con marcadores de daemon y frame superior actual
- Vista general de memoria, GC e información de runtime
- Activity Log con eventos del cliente y del agente en tiempo real
- Soporte para refrescar la instantánea del proceso actual

**Modo de uso**:
1. Inicia TUI y adjúntate al proceso objetivo
2. Revisa el estado de hilos, memoria, GC y runtime
3. Usa Activity Log para confirmar la conexión y el estado de ejecución de comandos
4. Presiona el atajo de otra vista para cambiar a la vista dedicada

**Escenarios adecuados**: Comprobar rápidamente la salud del proceso, el estado de hilos y la conexión de la sesión de diagnóstico

#### Registro de actividad del panel (v0.1.13)

El componente Activity Log del Dashboard recibió las siguientes mejoras en v0.1.13:
- **Reproducción de historial en reconexión** (`15463f9`): El historial de actividad se reproduce al reconectar para restaurar el contexto
- **Ajuste de línea preservado** (`86aad2e`): La configuración de ajuste de línea del registro de actividad sobrevive a las reconexiones
- **Foco despejado antes de atajos de pestaña** (`7677e29`): El foco de entrada se borra antes de los atajos de pestaña para evitar interferencias
- **Inicialización diferida de pestañas ocultas** (`59dbe4d`): Las pestañas ocultas omiten operaciones de actualización, reduciendo el uso de CPU

#### Registro de actividad de attach (v0.1.16)

- El attach log del selector de procesos se puede copiar con `y`, útil para conservar diagnósticos de fallos
- Tras un attach correcto, Dashboard reproduce el resumen de actividad de attach para mantener visible el contexto de conexión

#### Metadatos de runtime (v0.1.17)

El panel de información de runtime del Dashboard ahora añade tres filas extra obtenidas del comando [`patch-status`]({% link commands/patch-status.md %}):

- `Gevent`: estado de gevent en el proceso objetivo (`patched` / `imported` / `none`, etc.)
- `Backend`: backend de trace activo (por ejemplo `profiler` o `wrapper_only`)
- `Downgraded`: indica si se activó una degradación de runtime (p. ej. por gevent); el motivo se añade entre paréntesis cuando está disponible

Estas filas se actualizan en cada refresco del Dashboard, permitiendo identificar - sin cambiar de vista - si el comportamiento de trace/watch está afectado por monkey patching.

---

### 2. Vista Watch (tecla `2`)

**Función**: Observa parámetros de entrada, valores de retorno, excepciones y tiempo consumido de llamadas a funciones

![Vista Watch de Peeka]({{ site.url }}/assets/images/screenshots/peeka-watch.png)

**Características**:
- Actualización en streaming en tiempo real (datos de observación se muestran continuamente)
- Soporta filtrado condicional
- Soporta control de puntos de observación (-b / -e / -s / -f)
- Salida en color (éxito=verde, excepción=rojo)
- Muestra conteo de llamadas y tiempo acumulado

**Operaciones interactivas**:
- Ingresa el patrón de función (como `module.Class.method`)
- Establece el número de observaciones (parámetro -n)
- Establece la expresión de condición (--condition)
- Presiona Enter para iniciar la observación
- Presiona Delete para detener la observación

**Formato de salida**:
- Vista de tabla: muestra campos clave (params, returnObj, cost, success)
- Vista detallada: formato JSON muestra datos completos

#### Banner de metadatos de runtime (v0.1.17)

Al abrir la vista Watch se invoca automáticamente [`patch-status`]({% link commands/patch-status.md %}) para obtener el estado de runtime del proceso objetivo. Cuando se detecta gevent, un backend no estándar o una degradación, se muestra un banner compacto debajo de los controles, por ejemplo:

```
Gevent: patched  Backend: wrapper_only (downgraded: gevent_patched_runtime)
```

El banner permanece oculto cuando no hay señal que mostrar (CPython por defecto, sin gevent). La información proviene del campo `runtime_meta` añadido al evento `watch_started`, lo que facilita detectar diferencias de comportamiento en entornos con gevent.

---

### 3. Vista Trace (tecla `3`)

**Función**: Rastrear la cadena de llamadas de funciones, mostrando la relación jerárquica y el tiempo consumido de las llamadas a métodos

![Vista Trace de Peeka]({{ site.url }}/assets/images/screenshots/peeka-trace.png)

**Características**:
- **Visualización en estructura de árbol**: Jerarquía de llamada visual
- **Codificación de colores por tiempo consumido**:
  - Verde: < 10ms
  - Amarillo: 10-100ms
  - Rojo: ≥ 100ms
- **Actualización en streaming en tiempo real**: Cada llamada se muestra inmediatamente
- **Soporta expandir/colapsar**: Nodos de árbol interactivos
- **Límite de profundidad**: Controla el número de capas de rastreo (parámetro -d)

**Operaciones interactivas**:
- Ingresa el patrón de función
- Establece la profundidad de rastreo (3 capas por defecto)
- Establece saltar funciones incorporadas (--skip-builtin)
- Presiona Enter para iniciar el rastreo
- Presiona Delete para detener el rastreo
- Clic en el nodo para expandir/colapsar (soporte de ratón)

**Ejemplo de salida**: La vista Trace muestra cadenas de llamadas y el tiempo de cada nodo como un árbol expandible.

#### Estado de backend y gevent (v0.1.17)

El panel de estadísticas de cada observación de trace ahora añade una línea `Backend: ... Gevent: ...` después de `Node Count` / `Function`:

- Cuando trace se ejecuta sobre el backend `profiler` completo, muestra `Backend: profiler (full)`.
- Cuando el objetivo tiene gevent activo y trace se degradó a `wrapper_only`, muestra `Backend: wrapper_only  Gevent: patched`, coincidiendo con la nota de compatibilidad gevent en la documentación del [comando trace]({% link commands/trace.md %}).

Esta información proviene del nuevo campo `runtime_meta` en las respuestas de trace, lo que ayuda a explicar por qué trace ya no reporta un árbol de llamadas recursivo completo en algunos runtimes.

---

### 4. Vista Stack (tecla `4`)

**Función**: Capturar la pila de llamadas completa cuando se llama a la función

**Características**:
- Muestra la cadena completa de llamadas (desde la entrada hasta la función actual)
- Soporta filtrado condicional
- Actualización en streaming en tiempo real
- Muestra ruta de archivo, número de línea y nombre de función

**Operaciones interactivas**:
- Ingresa el patrón de función
- Establece profundidad de pila (parámetro --depth)
- Establece expresión condicional
- Presiona Enter para iniciar la captura
- Presiona Delete para detener la captura

**Formato de salida**:
- Cada llamada muestra una instantánea de pila
- Ruta completa desde la entrada de llamada hasta la función objetivo

---

### 5. Vista Monitor (tecla `5`)

**Función**: Salida periódica de estadísticas de rendimiento de funciones (número de llamadas, tasa de éxito, tiempo de respuesta)

**Características**:
- Actualización periódica (intervalo configurable)
- Muestra estadísticas agregadas:
  - Número total de llamadas
  - Número de éxitos / número de fallos
  - Tiempo promedio / tiempo mínimo / tiempo máximo
- Soporta monitoreo simultáneo de múltiples funciones
- Visualización con gráficos en tiempo real (opcional)

**Operaciones interactivas**:
- Ingresa el patrón de función
- Establece el intervalo de actualización (parámetro --interval, unidad: segundos)
- Establece el número de ciclos (parámetro -c, -1 significa infinito)
- Presiona Enter para iniciar el monitoreo
- Presiona Delete para detener el monitoreo

**Ejemplo de salida**:
```
Function: module.func
  Total Calls: 1523
  Success: 1500 (98.5%)
  Failed: 23 (1.5%)
  Avg Time: 12.3ms
  Min Time: 0.5ms
  Max Time: 250.8ms
```

---

### 6. Vista Memory (tecla `6`)

**Función**: Analizar el uso de memoria y asignación de memoria del proceso

**Características**:
- Resumen de memoria (memoria total, RSS, tamaño de heap)
- Seguimiento Top N de asignaciones de memoria
- Agrupación por archivo / número de línea
- Soporta activación manual de GC
- Comparación de instantáneas de memoria

**Operaciones interactivas**:
- Ver resumen de memoria (operación overview)
- Iniciar seguimiento de memoria (operación start)
- Ver Top N asignaciones (operación top)
- Activar recolección de basura (operación gc)
- Detener seguimiento de memoria (operación stop)

**Atajos de operaciones comunes**:
- `r`: Actualizar datos
- `g`: Activar GC
- `t`: Alternar estado de seguimiento

---

### 7. Vista Logger (tecla `7`)

**Función**: Ajustar dinámicamente el nivel de registro de loggers de Python

**Características**:
- Enumera todos los loggers y su nivel actual
- Modificar el nivel de registro en tiempo de ejecución (DEBUG / INFO / WARNING / ERROR)
- No necesita reiniciar el proceso objetivo
- Soporta coincidencia de patrones (como `myapp.*`)

**Operaciones interactivas**:
- Enumera todos los loggers (operación list)
- Obtener nivel de logger específico (operación get)
- Establecer nivel de logger (operación set)
- Soporta coincidencia de patrones (--pattern)

**Ejemplo de salida**:
```
Logger: myapp.database
  Level: INFO
  Effective Level: INFO
  Handler: StreamHandler

Logger: myapp.api
  Level: DEBUG
  Effective Level: DEBUG
  Handler: FileHandler
```

---

### 8. Vista Inspect (tecla `8`)

**Función**: Inspección de objetos en tiempo de ejecución y evaluación de expresiones

**Características**:
- Ejecuta expresiones Python en un sandbox seguro
- Inspecciona atributos de objetos
- Muestra valores de variables
- Soporta inspección de `locals()` / `globals()`

**Operaciones interactivas**:
- Ingresa una expresión Python (por ejemplo, `len(my_list)`)
- Ingresa una ruta de objeto (por ejemplo, `module.MyClass.attr`)
- Presiona Enter para ejecutar
- Consulta resultados en formato JSON

**Restricciones de seguridad**:
- Deshabilita `eval` / `exec` / `__import__`
- Acceso de solo lectura (no modifica objetos)
- Sandbox seguro basado en `simpleeval`

---

### 9. Vista Threads (tecla `9`)

**Función**: Enumera todos los hilos, verifica estado de hilos y pila

**Características**:
- Actualización en tiempo real de la lista de hilos
- Muestra estado de hilos (RUNNABLE / WAITING / TIMED_WAITING)
- Muestra ID de hilo, nombre, bandera daemon
- Clic en el hilo para ver la pila completa
- Soporta filtrado por estado y ordenamiento por campo

**Operaciones interactivas**:
- Ver todos los hilos (modo list)
- Clic en el hilo para ver detalles (modo detail)
- Filtrar por estado (RUNNABLE / WAITING / TIMED_WAITING)
- Ordenar por campo (tid / name / state)

**Formato de salida**:
- Vista de tabla: lista de hilos (tid, name, state, stack_depth)
- Vista de detalles: tramas de pila completas (filename, lineno, funcname, locals_keys)

**Atajos**:
- `r`: Actualizar lista de hilos
- `Enter`: Ver detalles del hilo seleccionado
- `escape`: Volver a la vista de lista

---

### 10. Perfilador de Funciones (Perfilador de funciones) (tecla `0`)

**Función**: Analizador de rendimiento por muestreo a nivel de función (similar a `py-spy top`)

**Características**:
- Tabla de clasificación de rendimiento en tiempo real
- Muestra porcentaje de uso de CPU (own / total)
- Muestra tiempo consumido de función (own_time / total_time)
- Modo de muestreo, sobrecoste de rendimiento < 5%
- Soporta ordenamiento por diferentes campos
- Codificación de colores (rojo=CPU alto, amarillo=medio, verde=bajo)

**Operaciones interactivas**:
- Establece intervalo de muestreo (parámetro --interval, por defecto 10ms)
- Establece ciclo de visualización (parámetro --cycles)
- Selecciona campo de ordenamiento (own / total / own-time / total-time)
- Presiona Enter para iniciar el análisis
- Presiona Delete para detener el análisis
- Presiona `c` para vaciar estadísticas (reset)

**Ejemplo de salida**:
```
Function                     Own%   Total%  Own Time  Total Time
───────────────────────────────────────────────────────────────
compute_matrix              45.3%   58.7%   0.453s    0.587s
multiply                    12.8%   13.4%   0.128s    0.134s
log_result                   8.2%    8.9%   0.082s    0.089s
```

**Atajos**:
- `r`: Actualizar datos
- `s`: Cambiar campo de ordenamiento
- `Enter`: Iniciar análisis de rendimiento
- `Delete`: Detener análisis de rendimiento
- `c`: Vaciar datos estadísticos (reset)

---

## Características Exclusivas de TUI

Comparado con el modo CLI, TUI proporciona las siguientes características exclusivas:

### 1. Autocompletado

El cuadro de entrada de Dashboard soporta autocompletado de comandos y parámetros:

- Completado de nombre de comando: ingresa `wa` → presiona Tab → `watch`
- Completado de parámetros: ingresa `watch -` → presiona Tab → muestra todos los parámetros
- Completado de patrones: ingresa `watch mymod` → presiona Tab → muestra clases y métodos en el módulo

**Fuente de completado**:
- Obtiene dinámicamente información de módulos, clases y métodos desde el proceso objetivo
- Almacena en caché resultados de completado para mejorar la velocidad de respuesta

### 2. Datos en streaming en tiempo real

Todos los comandos de observación (watch / trace / stack / monitor / top) soportan actualización en streaming en tiempo real:

- Los datos se empujan en forma de fotogramas de observación (OBS frame)
- TUI analiza y actualiza la interfaz automáticamente
- Soporta pausa/reanudación de flujo (en algunas vistas)

### 3. Estructura de árbol interactivo

La vista Trace proporciona una estructura de árbol interactiva:

- Los nodos se pueden expandir/colapsar
- Soporte para clic de ratón
- Navegación con teclado (↑ / ↓ / Enter / ← / →)

### 4. Codificación de colores

TUI usa colores para mejorar la legibilidad de los datos:

- **Éxito/Fallo**: verde=éxito, rojo=fallo
- **Clasificación por tiempo**: verde=rápido (<10ms), amarillo=medio (10-100ms), rojo=lento (≥100ms)
- **Uso de CPU**: intensidad de color indica el grado de uso de CPU
- **Nivel de registro**: DEBUG=azul, INFO=verde, WARNING=amarillo, ERROR=rojo

### 5. Cliente dedicado

Cada vista usa un `StreamingAgentClient` independiente:

- Evita confusión de datos
- Soporta operación concurrente (múltiples vistas trabajan simultáneamente)
- Mecanismo automático de reconexión

### 6. Seguimiento automático

Las vistas Watch / Trace / Stack soportan desplazamiento automático:

- Los últimos datos siempre están visibles
- Puedes detener manualmente el seguimiento automático (desplazándote hacia arriba)
- Desplázate manualmente hasta el final para recuperar el seguimiento automático

---

## Comparación TUI vs CLI

| Característica | Modo TUI | Modo CLI |
|----------------|----------|----------|
| **Experiencia interactiva** | Tiempo real, visual, navegación con atajos | Entrada de línea de comandos, adecuado para scripting |
| **Visualización de datos** | Estructura de árbol, tablas, codificación de colores | Salida JSON pura |
| **Actualización en tiempo real** | Actualización automática, push en streaming | Necesita actualización manual o reejecutar el comando |
| **Autocompletado** | Soporta completado de comandos, parámetros y patrones | Necesita plugin de completado de shell |
| **Multitarea** | Múltiples vistas concurrentes, cambio rápido | Necesita múltiples ventanas de terminal |
| **Escenarios de aplicación** | Diagnóstico interactivo, monitoreo en tiempo real, depuración exploratoria | Scripting, automatización, integración CI/CD |
| **Formato de salida** | Tablas formateadas, estructura de árbol, resaltado de color | JSONL (fácil de procesar con jq / grep) |
| **Curva de aprendizaje** | Media (necesitas familiarizarte con los atajos) | Baja (comandos CLI estándar) |
| **Sobrecoste de rendimiento** | Ligeramente alto (renderizado UI) | Bajo (transmisión de datos pura) |

**Sugerencias de selección**:
- **Depuración en desarrollo**: Prioriza usar TUI (buena experiencia interactiva)
- **Scripts automatizados**: Usa CLI (formato de salida estandarizado)
- **Integración CI/CD**: Usa CLI (entorno sin TTY)
- **Análisis de rendimiento**: Ambos son adecuados (TUI visualización es más intuitiva, CLI facilita exportar datos)

---

## Consejos de Uso

### 1. Cambio rápido de vistas

Usa teclas numéricas para cambiar rápidamente entre vistas, no necesitas volver a Dashboard:

```
Presiona 2 → Vista Watch
Presiona 3 → Vista Trace
Presiona 5 → Vista Monitor
Presiona 0 → Vista Top
```

### 2. Usar múltiples vistas en combinación

Diferentes vistas pueden trabajar concurrentemente (cada vista usa un cliente independiente):

1. Inicia la observación en la vista Watch (presiona `2`, ingresa comando, presiona Enter)
2. Cambia a la vista Monitor para iniciar el monitoreo (presiona `5`, ingresa comando, presiona Enter)
3. Cambia a la vista Top para iniciar el análisis de rendimiento (presiona `0`, ingresa comando, presiona Enter)
4. Cambia entre vistas para ver datos en tiempo real

### 3. Usar historial de comandos

El cuadro de entrada de Dashboard soporta historial de comandos:

- `↑`: Comando anterior
- `↓`: Siguiente comando
- El historial se persiste durante la sesión

### 4. Copia rápida de salida

La salida de TUI se puede copiar directamente al portapapeles:

1. Selecciona el texto con el ratón
2. Ctrl+C para copiar (o el atajo por defecto del terminal)
3. Pega en otras herramientas (como editor, documentos)

### 5. Personalización de temas

Selecciona un tema adecuado según el fondo de tu terminal:

```bash
# Fondo claro
peeka --theme solarized-light

# Fondo oscuro
peeka --theme dracula

# Listar todos los temas
peeka --list-themes
```

---

## Resolución de Problemas

### 1. TUI falla al iniciar

**Error**: `ModuleNotFoundError: No module named 'textual'`

**Solución**:
```bash
pip install peeka[tui]
# o
pip install textual
```

### 2. Visualización anormal en el terminal

**Error**: Los caracteres no se muestran correctamente, se pierden colores

**Solución**:
- Asegúrate de que tu terminal soporte 256 colores o 24 bits de color verdadero
- Establece variables de entorno:
  ```bash
  export TERM=xterm-256color
  export COLORTERM=truecolor
  ```

### 3. Conflicto de atajos de teclado

**Problema**: Los atajos de teclado son interceptados por el terminal o shell

**Solución**:
- Verifica la configuración de atajos de tu terminal
- Usa la tecla `escape` para volver al nivel anterior (en lugar de `q`)
- Ingresa el comando completo en Dashboard (no uses atajos de teclado)

### 4. Problemas de rendimiento

**Problema**: La interfaz TUI se traba, alta latencia

**Solución**:
- Reduce la frecuencia de muestreo (el comando top usa un `--interval` más grande)
- Reduce el número de observaciones (el comando watch usa `-n` para limitar el número)
- Usa el modo CLI (el renderizado TUI tiene sobrecoste)

### 5. Pérdida de conexión

**Problema**: `Connection lost` o `Socket error`

**Solución**:
- Verifica si el proceso objetivo todavía está en ejecución
- Verifica si el archivo socket (`/tmp/peeka_<pid>.sock`) existe
- Vuelve a adjuntar al proceso objetivo

---

## Requisitos de Permisos

Los requisitos de permisos para el modo TUI son los mismos que para el modo CLI:

- **Adjuntar proceso**: Necesita `CAP_SYS_PTRACE` o mismo UID
- **ptrace_scope**: Necesita `ptrace_scope <= 1` (Linux)
- **Versión de Python**:
  - Python 3.14+: usa PEP 768 `sys.remote_exec()`
  - Python 3.8.1-3.13: Linux necesita GDB y python3-dbg; macOS necesita LLDB (Xcode Command Line Tools)

Para requisitos detallados de permisos, consulta [documentación del comando attach]({% link commands/attach.md %})

---

## Resumen

Peeka TUI proporciona una experiencia completa de diagnóstico interactivo, adecuada para escenarios que necesitan operaciones frecuentes, monitoreo en tiempo real y análisis visual. A través de 10 vistas dedicadas y atajos de teclado globales, los desarrolladores pueden diagnosticar eficientemente problemas en aplicaciones Python sin necesidad de memorizar parámetros de comandos complejos.

**Mejores prácticas**:
- Entorno de desarrollo: Prioriza usar TUI (buena experiencia interactiva)
- Entorno de producción: Elige TUI (monitoreo) o CLI (scripting) según necesidades
- Escenarios de automatización: Usa CLI (salida estandarizada, fácil de integrar)

Comienza a usar TUI:
```bash
peeka
```

Presiona `?` para ver la ayuda, presiona `1/2/3/4/5/6/7/8/9/0` para cambiar de vista, ¡comienza tu viaje de diagnóstico!

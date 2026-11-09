---
layout: default
title: trace 命令
parent: 命令参考
nav_order: 3
---

# trace 命令
{: .no_toc }

## 目录
{: .no_toc .text-delta }

1. TOC
{:toc}


## 简介

`trace` 命令用于追踪 Python 函数的**直接调用者**和执行耗时，以扁平列表展示目标函数下一层子调用的聚合信息。这是一个强大的性能分析工具，可以帮助开发者快速定位性能瓶颈和理解代码执行热点。

**v0.1.20 重要变更**：`trace` 不再支持 `-d/--depth` 选项，也不再输出递归嵌套的调用树。后端只会捕获并聚合目标函数的直接调用者（direct callees），输出更稳定、开销更可预测。


## TUI 使用

在 TUI 模式下，按 **`3`** 键切换到 **Trace 视图**，提供以下交互式功能：

- **模式输入**：支持函数名自动补全（从目标进程实时获取）
- **参数配置**：可视化配置最小耗时、次数、条件表达式、skip-builtin
- **活动追踪列表**：顶部显示当前追踪任务的状态和计数
- **调用树可视化**：以交互式树形结构展示每次观测的直接调用者，含聚合 callee 节点
- **详情统计面板**：展示选中观测或 callee 的耗时、次数、占比和异常信息
- **色彩编码耗时**：
  - 🟢 绿色：< 10ms（快速）
  - 🟡 黄色：10-100ms（中速）
  - 🔴 红色：>= 100ms（慢速）
- **快捷操作**：
  - 输入模式后按 Enter 开始追踪
  - 在活动追踪列表选中某行，查看该 pattern 的调用树
  - 在调用树选中 callee 节点，按 `t` 对该函数发起下钻追踪（Drill Trace）
  - 按 `c` 清空追踪记录并停止运行中的 trace
  - 按 Delete 停止所有 trace

![Peeka Trace 视图]({{ site.url }}/assets/images/screenshots/peeka-trace.png)

**CLI 等效命令**：下文所有示例使用 CLI 命令演示，TUI 提供了相同功能的图形化界面。

## 使用场景

- **性能瓶颈定位**：通过耗时数据快速找出目标函数下最慢的子调用
- **热点函数识别**：聚合统计直接调用者的总耗时、调用次数、最小/最大耗时
- **代码执行路径追踪**：观察不同条件下目标函数触发了哪些子调用
- **子函数耗时分析**：分析各个直接子调用的耗时占比
- **异常调用诊断**：追踪抛出异常的调用并查看异常类型

## 命令格式

```bash
peeka-cli attach <pid>    # 首先附加到目标进程
peeka-cli trace <pattern> [options]
```

### 参数说明

| 参数                    | 说明                    | 默认值     | 示例                                      |
|-----------------------|-----------------------|---------|-----------------------------------------|
| `pattern`             | 函数匹配模式                | -       | `module.Class.method`                   |
| `-n, --times`         | 观测次数（-1 表示无限）         | `-1`    | `-n 10`                                 |
| `--condition` | 条件表达式（支持 `cost` 变量）   | 无       | `--condition "cost > 50"`       |
| `--client`            | 使用已有客户端会话 ID；不提供时自动创建临时客户端 | 自动 | `--client client_123`                  |
| `--skip-builtin`      | 跳过内置函数和标准库函数          | `true`  | `--skip-builtin=false`                  |
| `--min-duration`      | 最小耗时过滤（毫秒），只记录大于等于该值的直接调用 | `0`     | `--min-duration 10`                     |

**注意**：
- `--skip-builtin` 默认启用，以减少输出噪音
- 条件表达式中的 `cost` 变量表示整个调用的总耗时（毫秒）
- v0.1.20 起不再支持 `-d/--depth`

### 函数匹配模式 (pattern)

支持以下格式：

```python
# 1. 模块级函数
"mymodule.my_function"

# 2. 类方法
"mymodule.MyClass.my_method"

# 3. 嵌套类方法
"mypackage.mymodule.OuterClass.InnerClass.method"

# 4. 模块路径
"package.subpackage.module.function"
```

**注意**：必须使用完整的模块路径（从导入根开始），当前版本不支持通配符匹配。

## 基本用法

### 1. 追踪函数调用

```bash
# 首先附加到目标进程
peeka-cli attach 12345

# 追踪 5 次调用
peeka-cli trace "calculator.Calculator.calculate" -n 5
```

**输出示例**：

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

**字段说明**：

| 字段                 | 说明           | 示例值                       |
|--------------------|--------------|---------------------------|
| `watch_id`         | 观测 ID        | `"trace_abc123"`          |
| `timestamp`        | 时间戳          | `1705586200.123`          |
| `func_name`        | 目标函数名        | `"calculator.Calculator.calculate"`  |
| `location`         | 观测位置         | `"AtExit"`                |
| `call_tree`        | 直接调用者列表（扁平聚合）    | `[...]`                   |
| `total_duration_ms`| 总执行耗时（毫秒）    | `125.3`                   |
| `self_time_ms`     | 目标函数自身耗时（毫秒） | `9.3`                     |
| `callee_count`     | 直接调用者种类数      | `3`                       |
| `node_count`       | 节点总数（目标函数 + 直接调用者） | `4`              |
| `thread_id`        | 线程 ID         | `140234567890`            |
| `thread_name`      | 线程名          | `"MainThread"`            |
| `exception`        | 异常信息（抛出时）    | `"ValueError: ..."`       |
| `runtime_meta`     | 运行时元数据（后端、gevent 等） | `{...}`        |

**`call_tree` 节点字段**：

| 字段            | 说明         | 示例值                    |
|---------------|------------|------------------------|
| `function`    | 函数完整名称     | `"module.Class.method"`|
| `filename`    | 文件路径       | `"/app/module.py"`     |
| `lineno`      | 行号         | `42`                   |
| `count`       | 该观测周期内调用次数 | `5`                    |
| `total_ms`    | 总执行耗时（毫秒）   | `125.3`                |
| `min_ms`      | 最小耗时（毫秒）    | `10.5`                 |
| `max_ms`      | 最大耗时（毫秒）    | `95.1`                 |

### 2. 可视化调用树（TUI）

在 TUI 模式下，Trace 视图采用上下布局：

![Peeka Trace 调用树]({{ site.url }}/assets/images/screenshots/peeka-trace.png)

**说明**：
- 顶部 Active Traces 显示当前追踪任务（Pattern / Status / Count）
- 下部左侧 Call Tree 展示选中 pattern 的观测节点、聚合 callee 节点和每个 callee 的耗时占比
- 下部右侧 Stats 面板展示当前选中观测或 callee 的详细统计
- 不同颜色用于突出不同耗时区间
- 选中 callee 后按 `t` 可快速发起新的下钻追踪

### 3. 按最小耗时过滤

```bash
# 只记录耗时 >= 10ms 的直接调用
peeka-cli trace "service.process" --min-duration 10
```

这可以减少高频短耗时辅助函数带来的噪音，聚焦于真正可能消耗资源的子调用。

### 4. 条件过滤

```bash
# 只追踪耗时超过 50ms 的调用
peeka-cli trace "api.handler" --condition "cost > 50"

# 组合参数和耗时条件
peeka-cli trace "service.query" --condition "cost > 100 and params[0] > 1000"
```

### 5. 跳过内置函数

```bash
# 默认行为：跳过内置函数（减少输出噪音）
peeka-cli trace "mymodule.func"

# 显示所有调用（包括内置函数）
peeka-cli trace "mymodule.func" --skip-builtin=false
```

**内置函数示例**：
- Python 内置函数：`len()`, `str()`, `isinstance()`, `print()`
- 标准库函数：`json.dumps()`, `os.path.join()`, `datetime.now()`

## 实现技术

### 实现原理

Peeka 的 `trace` 命令根据 Python 版本自动选择最优实现方案：

| Python 版本 | 实现方案            | 性能开销  | 说明                |
|-----------|-----------------|-------|-------------------|
| 3.12+     | sys.monitoring  | < 5%  | 官方 PEP 669 API，最优性能         |
| 3.8.1-3.11  | sys.settrace | < 20% | 兼容性好，自动启用         |

**直接调用者语义（v0.1.20+）**：所有后端只捕获目标函数的直接调用者，并对同一观测周期内相同 `(function, filename, lineno)` 的调用进行聚合，输出 `count / total_ms / min_ms / max_ms`。这样避免了递归/深层调用树带来的性能不确定性和数据膨胀。

**gevent 兼容性（v0.1.15+）**：当目标进程已启用 gevent monkey patch 或 active hub 时，Python 3.8–3.11 上的 `trace` 会退化为 `wrapper_only` 后端，避免 `sys.settrace` 破坏 frame stack 不变量。此模式仍报告目标函数观测结果，但不提供直接调用者列表。v0.1.21 起，Python 3.12+ 在 gevent 下继续使用 `sys.monitoring` 后端（按 tool 注册，不安装全局 tracer），保留完整的直接调用者统计。

**sys.monitoring 实现 (Python 3.12+)**:

- 基于 [PEP 669](https://peps.python.org/pep-0669/) 的官方监控 API
- 使用 `PY_START` 和 `PY_RETURN` 事件捕获调用
- 性能开销 < 5%，推荐生产环境使用
- 自动分配 tool_id，多个观测不冲突

**sys.settrace 实现 (Python 3.8.1-3.11)**:

- 使用 Python 内置的 `sys.settrace()` 机制
- 仅在目标函数执行期间启用（局部 trace）
- 性能开销 < 20%，完全可用于大多数场景

**skip-builtin 过滤机制**:

- 检查 `code.co_filename.startswith('<')` 过滤内置函数（如 `<built-in>`）
- 检查 Python 标准库路径，过滤标准库函数
- 默认启用，可减少 50% 以上的输出节点

## 性能影响

### 性能开销

| 场景                  | 开销    | 说明            |
|---------------------|-------|---------------|
| **简单函数**            | < 5%  | Python 3.12+  |
| **简单函数**            | < 20% | Python 3.8.1-3.11 |
| **高频子调用**           | 10-30%| 根据 Python 版本和 `--min-duration` 设置 |
| **高频调用（>1000 QPS）** | 20-50%| 建议限制观测次数      |

**说明**：
- Python 3.12+ 使用 `sys.monitoring`，性能开销显著降低
- v0.1.20 起只追踪一层直接调用者，开销更稳定、更可预测
- 建议生产环境使用条件过滤和次数限制

### 性能优化建议

1. **使用最小耗时过滤**
   ```bash
   # 只记录耗时 >= 10ms 的直接调用
   peeka-cli trace "func" --min-duration 10
   ```

2. **跳过内置函数**
   ```bash
   # 默认启用，减少 50% 以上的节点
   peeka-cli trace "func" --skip-builtin
   ```

3. **使用条件过滤**
   ```bash
   # 只追踪慢调用
   peeka-cli trace "func" --condition "cost > 100"
   ```

4. **限制观测次数**
   ```bash
   # 只观测 10 次
   peeka-cli trace "func" -n 10
   ```

## 使用示例

### 1. 定位性能瓶颈

```bash
# 追踪慢接口，找出耗时最长的子调用
peeka-cli trace "api.handler.process_request" --condition "cost > 100"
```

**输出**：
```
[1250ms] api.handler.process_request()
├── [10ms] api.validator.check_params() (count=1)
├── [1200ms] database.query.execute()  ← 瓶颈在这里！ (count=1)
└── [20ms] api.formatter.to_json() (count=1)
```

**结论**：数据库查询占用了 96% 的时间，需要优化 SQL 或添加索引。

### 2. 聚合高频调用

```bash
# 追踪循环内的函数，观察同一子调用被多次触发的情况
peeka-cli trace "algorithm.process_batch" -n 5
```

**输出示例**：
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

**结论**：`process_batch` 在单次执行中触发了 100 次数据库查询，考虑批量查询优化。

### 3. 理解代码执行路径

```bash
# 追踪条件分支的执行路径
peeka-cli trace "service.business_logic" -n 1
```

**场景 A（正常流程）**：
```
`---[50ms] service.business_logic()
    +---[5ms] service.validate_input()
    +---[30ms] service.process_data()
    `---[10ms] service.save_result()
```

**场景 B（异常流程）**：
```
`---[20ms] service.business_logic()
    +---[5ms] service.validate_input()
    +---[10ms] service.handle_invalid_input()
    `---[3ms] service.log_error()
```

### 4. 对比优化前后性能

```bash
# 优化前
peeka-cli trace "converter.parse_json" -n 10 > before.jsonl

# 优化后
peeka-cli trace "converter.parse_json" -n 10 > after.jsonl

# 分析耗时变化
jq '.total_duration_ms' before.jsonl | awk '{sum+=$1; count++} END {print "Before:", sum/count, "ms"}'
jq '.total_duration_ms' after.jsonl | awk '{sum+=$1; count++} END {print "After:", sum/count, "ms"}'
```

### 5. 集成到 CI/CD

```bash
# 性能回归测试
#!/bin/bash
THRESHOLD=100  # 最大允许耗时 100ms

peeka-cli attach $PID
RESULT=$(peeka-cli trace "critical.function" -n 50 | \
  jq -s 'map(select(.type == "observation")) | map(.total_duration_ms) | add / length')

if (( $(echo "$RESULT > $THRESHOLD" | bc -l) )); then
  echo "Performance regression detected: ${RESULT}ms > ${THRESHOLD}ms"
  exit 1
fi
```

## 数据处理与分析

### 使用 jq 处理 JSON

```bash
# 1. 提取直接调用者列表
peeka-cli trace "func" | jq '.call_tree'

# 2. 计算平均总耗时
peeka-cli trace "func" -n 100 | jq '.total_duration_ms' | \
  awk '{sum+=$1; count++} END {print "avg:", sum/count, "ms"}'

# 3. 找出最慢的子调用
peeka-cli trace "func" | jq '.call_tree | sort_by(.total_ms) | reverse | .[0]'

# 4. 统计调用频次（按 count 字段汇总）
peeka-cli trace "func" -n 100 | jq -s '[.[] | .call_tree[] | {function, count}] | group_by(.function) | map({function: .[0].function, total_count: map(.count) | add}) | sort_by(.total_count) | reverse'

# 5. 生成火焰图数据（使用 total_ms）
peeka-cli trace "func" -n 1000 | jq -r '.call_tree[] | "\(.function) \(.total_ms)"' > flamegraph.txt
```

### Python 数据分析

```python
import json
import sys
from collections import defaultdict

# 统计直接调用者的总耗时和次数
stats = defaultdict(lambda: {"count": 0, "total_ms": 0})

for line in sys.stdin:
    data = json.loads(line)
    if data["type"] == "observation":
        for callee in data.get("call_tree", []):
            func = callee.get("function")
            if func:
                stats[func]["count"] += callee.get("count", 1)
                stats[func]["total_ms"] += callee.get("total_ms", 0)

# 按总耗时排序
sorted_stats = sorted(stats.items(), key=lambda x: x[1]["total_ms"], reverse=True)

print("Top 10 Time-Consuming Direct Callees:")
print(f"{'Function':<60} {'Count':>10} {'Total (ms)':>15} {'Avg (ms)':>12}")
print("-" * 100)

for func, stat in sorted_stats[:10]:
    avg_ms = stat["total_ms"] / stat["count"] if stat["count"] else 0
    print(f"{func:<60} {stat['count']:>10} {stat['total_ms']:>15.2f} {avg_ms:>12.2f}")
```

**运行**：
```bash
peeka-cli trace "module.func" -n 100 | python analyze_trace.py
```

**输出**：
```
Top 10 Time-Consuming Functions:
Function                                                      Count     Total (ms)      Avg (ms)
----------------------------------------------------------------------------------------------------
database.query.execute                                          100       12500.00       125.00
api.handler.process_request                                     100       15000.00       150.00
json.dumps                                                      500        1000.00         2.00
...
```

## 常见问题

### 1. 为什么看不到更深层的调用？

**问题**：调用树只显示目标函数的直接调用者，不显示子调用的子调用

**原因**：v0.1.20 起 `trace` 只捕获并聚合直接调用者（direct-callee 语义）。这是为了提供更稳定的性能和更可预测的输出。

**解决方案**：

```bash
# 如果需要观察某个子调用内部的调用，可以对该子调用单独发起 trace
peeka-cli trace "module.sub_module.slow_func" -n 10
```

### 2. 输出数据过多

**问题**：包含大量内置函数调用，输出难以阅读

**解决方案**：

```bash
# 跳过内置函数（默认启用）
peeka-cli trace "module.func" --skip-builtin

# 只记录耗时 > 10ms 的调用
peeka-cli trace "module.func" --min-duration 10

# 使用条件过滤
peeka-cli trace "module.func" --condition "cost > 50"
```

### 3. 性能开销过大

**问题**：启用 trace 后应用响应变慢

**解决方案**：

```bash
# 1. 提高最小耗时阈值，减少记录节点
peeka-cli trace "module.func" --min-duration 10

# 2. 限制观测次数
peeka-cli trace "module.func" -n 10

# 3. 使用条件过滤，只追踪慢调用
peeka-cli trace "module.func" --condition "cost > 100"

# 4. 考虑升级到 Python 3.12+ 获得更好性能
```

### 4. 无法观测到数据

**可能原因**：
- 函数没有被调用
- 函数名拼写错误
- 条件表达式过于严格
- 已达到观测次数限制（-n 参数）

**排查步骤**：

```bash
# 1. 确认函数名是否正确
python3 -c "import mymodule; print(mymodule.MyClass.my_method)"

# 2. 去掉条件表达式，先观测一次
peeka-cli trace "mymodule.func" -n 1

# 3. 检查进程是否存在
ps aux | grep <pid>
```

## 高级技巧

### 1. 生成火焰图

```bash
# 收集追踪数据
peeka-cli trace "module.func" -n 1000 > trace.jsonl

# 转换为火焰图格式（直接调用者按 total_ms 折叠）
jq -r '.call_tree[] | "\(.function);\(.total_ms)"' trace.jsonl \
  > folded.txt

# 生成火焰图（需要安装 flamegraph.pl）
flamegraph.pl folded.txt > flamegraph.svg
```

### 2. 对比多个版本的性能

```bash
# 版本 A
git checkout v1.0
peeka-cli trace "module.func" -n 100 > trace_v1.jsonl

# 版本 B
git checkout v2.0
peeka-cli trace "module.func" -n 100 > trace_v2.jsonl

# 对比平均耗时
echo "v1.0: $(jq -s 'map(.total_duration_ms) | add / length' trace_v1.jsonl) ms"
echo "v2.0: $(jq -s 'map(.total_duration_ms) | add / length' trace_v2.jsonl) ms"
```

### 3. 自动化性能监控

```python
#!/usr/bin/env python3
"""性能回归监控脚本"""
import json
import subprocess
import time

THRESHOLD = 100  # 最大允许耗时 (ms)
CHECK_INTERVAL = 3600  # 检查间隔 (秒)

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
        send_alert(f"Performance regression: {avg_duration:.2f}ms > {THRESHOLD}ms")

    return avg_duration

def send_alert(message):
    # 发送告警（邮件、Slack、钉钉等）
    print(f"ALERT: {message}")

if __name__ == "__main__":
    pid = int(sys.argv[1])
    pattern = sys.argv[2]

    while True:
        duration = check_performance(pid, pattern)
        print(f"[{time.strftime('%Y-%m-%d %H:%M:%S')}] Avg duration: {duration:.2f}ms")
        time.sleep(CHECK_INTERVAL)
```

### 4. 集成到 Prometheus

```python
from prometheus_client import Histogram, start_http_server
import json
import subprocess

# 定义指标
trace_duration = Histogram('trace_duration_ms', 'Function trace duration', ['function'])

# 启动 Prometheus 服务器
start_http_server(8000)

# 收集追踪数据
proc = subprocess.Popen(
    ["peeka-cli", "trace", "module.func"],
    stdout=subprocess.PIPE,
    text=True
)

for line in proc.stdout:
    data = json.loads(line)
    if data["type"] == "observation":
        # 处理直接调用者列表
        for callee in data.get("call_tree", []):
            func = callee.get("function")
            total_ms = callee.get("total_ms", 0)
            if func:
                trace_duration.labels(function=func).observe(total_ms)
```

## 参考资料

- [PEP 669: Low Impact Monitoring for CPython](https://peps.python.org/pep-0669/)
- [Peeka 架构设计](../ARCHITECTURE.md)

## 更新日志

| 版本    | 日期      | 更新内容          |
|-------|---------|---------------|
| 0.2.0 | 2026-02 | 添加 trace 命令文档 |
| 0.1.0 | 2025-01 | 初始版本          |

## 更新历史

| 版本 | 发布日期 | 更新内容 |
|------|---------|---------|
| 0.1.21 | 2026-11-09 | Python 3.12+ 在 gevent patched/active hub 下使用 `sys.monitoring` 后端，不再退化为 `wrapper_only`；TUI Active Traces 表格新增运行时与性能列 |
| 0.1.20 | 2026-07-05 | 移除 `-d/--depth` 选项；trace 后端改为只捕获并聚合目标函数的直接调用者（direct callees），输出 `call_tree` 为扁平列表，新增 `self_time_ms`、`callee_count` 字段；TUI Trace 视图改为顶部 Active Traces 列表 + 下部调用树/统计分栏，新增 callee 下钻快捷键 `t` 和聚合 callee 节点 |
| 0.1.18 | 2026-06-24 | CLI `--times` 改为按活动 stream_id 计数，避免并发流的观测被误计入当前 trace 的限制；`run` 命令中的 trace 流现在会在达到限制后正确停止 |
| 0.1.17 | 2026-06-13 | trace 响应在被退化为 `wrapper_only` 后端时附带 `runtime_meta`（包含 `startup_backend`、`effective_backend`、`downgrade_reason`），TUI Trace 视图据此在统计面板显示 Backend / Gevent 状态 |
| 0.1.16 | 2026-06-07 | 支持 `--client` |
| 0.1.15 | 2026-05-27 | gevent patched/active hub 运行时退化为 `wrapper_only` trace 后端 |
| 0.1.12 | 2026-05-08 | TUI 面板系统统一，布局优化（commit 50c4af4） |
| 0.1.11 | 2026-05-07 | 客户端稳定来源标签化（commit 965ff22），活动诊断数据增强（commit b1b0412） |
| 0.1.10 | 2026-05-04 | TUI 按钮颜色规范化（commit fd6a0a1），活动日志折行可读性改进（commit 5f46ae8） |

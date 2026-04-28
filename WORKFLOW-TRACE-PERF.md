# Workflow trace 性能事故记录（供回顾）

## 现象

在 `packages/opencode/src/server/workflow-trace.ts` 增强 trace（记录 `user_input`、`model_stream_text`、`llm_stream` 等）之后，出现 **agent 长时间卡住、界面像全部失灵** 的情况。

## 根因（大坑）

流式输出时，模型会触发大量 **`text-delta`** 事件。若对**整段字符串**做反复拼接：

```text
s = s + delta   // 每次拷贝整段前缀，总长度 n、事件次数 m → 近似 O(n×m)，实际可视为 O(n²) 级别
```

两处都曾中招：

1. **`chat.model_stream_text`**：每个 delta 执行 `model_stream_text = model_stream_text + delta`。
2. **`llm_stream` 的批处理 buffer**：对 `_traceTextBuf` / `_traceReasonBuf` 使用 `buf += delta` 再 `slice`。

token 稍多就会把 **主线程/event loop 打满**，表现为全局卡死，而不是单个接口超时。

## 修复要点

1. **流式正文**：用 **`string[]` 分块 `push`**，写盘前 **`join` 一次**（线性时间）。
2. **`llm_stream` 的 delta**：用 **队列数组** 按 `TEXT_TRACE_BATCH` 批量写入事件，避免对大 buffer 反复 `+=`。
3. **写文件**：**不要**在 middleware 的同步路径里做 `materialize + JSON.stringify + fs.write`。使用 **`scheduleTracePersist`**：`setImmediate` 里再 `async` 执行 `materializeModelStreamText`、组 payload、`await writeTraceFile`，请求路径不等待。

## 相关代码

- `packages/opencode/src/server/workflow-trace.ts`：`appendModelStreamText`、`appendTraceDeltaParts`、`scheduleTracePersist`
- `packages/opencode/src/session/processor.ts`：每条流事件仍会调用 `traceRecordLlmStreamEvent`（需保持 O(1) 摊销/线性累计，避免热路径字符串拼接）

## 自检清单（以后改 trace）

- [ ] 热路径里是否对** growing string** 做 `+=`？
- [ ] 大对象 `JSON.stringify` 是否在请求返回前同步执行？
- [ ] 磁盘 I/O 是否已脱离 HTTP `finally` 的主线？

---

*记录日期：以本仓库当时修改为准；便于日后 code review 与 onboarding。*

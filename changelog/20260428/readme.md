# 2026-04-28 — Workflow trace 里程碑

## 概要

在 OpenCode 服务端 HTTP 工作流中引入 **workflow trace**：按请求生成结构化 JSON，便于排查会话、模型流与耗时。

## 主要能力

- **落盘位置**：与日志同级，例如 `~/.local/share/opencode/trace/`（随 `Global.Path.trace`）。
- **文件命名**：`trace_<request_id>_<iso-timestamp>.json`，**一请求一文件**（可配置/路径例外见下）。
- **内容**：
  - HTTP 元数据、`events` 时间线（`opencode.<business>.<method>=<ms>|k=v`）。
  - **`chat`**：`user_input`、`model_stream_text`（流式正文拼接）、`assistant_output`、`llm_stream`（工具/步骤等事件）、`model` 等。
- **Effect 贯通**：`WorkflowTraceSessionRef` 将 trace session 传入 `AppRuntime.runPromise`（ALS 无法穿透 Effect）。
- **`prompt_async`**：204 返回后 prompt 仍在后台执行，trace **推迟到 `runRequest.finally`** 再持久化，避免只有 `user_input`、无模型回复。
- **性能**：
  - 流式场景禁止对 growing string 反复 `+=`（曾导致近似 O(n²) 卡死）；改为分块数组 + 一次 `join`。
  - 持久化经 `setImmediate` 异步调度，避免阻塞 HTTP 收尾。
- **默认范围**：仅对 `POST /session/:id/message|prompt_async|command|shell` 写 trace；`OPENCODE_WORKFLOW_TRACE=all` 或请求头 `x-opencode-workflow-trace: 1` 可扩大范围。

## 相关代码（入口）

相对本文件 `changelog/20260428/readme.md` 的路径，可在 IDE / GitHub 中点击跳转。

- [`packages/opencode/src/server/workflow-trace.ts`](../../packages/opencode/src/server/workflow-trace.ts) — 中间件、`scheduleWorkflowTracePersist`、`traceRecordLlmStreamEvent` 等。
- [`packages/opencode/src/server/routes/instance/trace.ts`](../../packages/opencode/src/server/routes/instance/trace.ts) — `runRequest` + `WorkflowTraceSessionRef`。
- [`packages/opencode/src/session/prompt.ts`](../../packages/opencode/src/session/prompt.ts) — `traceChatInitFromUserMessage`。
- [`packages/opencode/src/session/processor.ts`](../../packages/opencode/src/session/processor.ts) — 流事件与 `traceChatFinalizeAssistant`。
- [`packages/core/src/global.ts`](../../packages/core/src/global.ts) — `Path.trace`。
- [`packages/opencode/src/server/routes/instance/session.ts`](../../packages/opencode/src/server/routes/instance/session.ts) — `prompt_async` 推迟 `scheduleWorkflowTracePersist`。

## 事故与备忘

根目录 [**`WORKFLOW-TRACE-PERF.md`**](../../WORKFLOW-TRACE-PERF.md) 记录了「流式 trace 字符串拼接导致全局卡顿」的根因与自检清单，评审时可对照。

## 环境变量（节选）

| 变量 | 含义 |
|------|------|
| `OPENCODE_WORKFLOW_TRACE` | `all` / `1`：对所有未跳过路由写 trace |
| `OPENCODE_TRACE_MAX_USER_CHARS` | `user_input` 上限 |
| `OPENCODE_TRACE_MAX_STREAM_CHARS` | `model_stream_text` 上限 |
| `OPENCODE_TRACE_MAX_ASSISTANT_CHARS` | `assistant_output` 上限 |
| `OPENCODE_TRACE_MAX_LLM_EVENTS` | `llm_stream` 最大条数 |

---

*本目录用于按日期归档重要变更；本条为 workflow trace 首次合入里程碑。*

---
title: telemetry — Claude Code 插件遥测
tags: [claude-code, plugin, telemetry, builtin, analytics, events, logging]
category: claude-code-mods
created: 2025-04-20
updated: 2026-10-20
source: claude-code/mods/telemetry
type: plugin
status: builtin
lifecycle: stable
---

# 💡 概述

`telemetry` 是 Claude Code 内建插件，提供 `$.telemetry` noun（`log`、`mark` 两个事件），供内建插件记录第一方分析数据。在引擎自身无 `$.telemetry` 时自动添加该 noun。

仅服务内建插件：个人安装或管理员列出的插件调用被拒绝。发送行为受 CLI 自身遥测开关控制。

---

# ⚡ 功能

- **`$.telemetry.log({ event, props })`**：排队一条事件为 `tengu_plugin_<event>` 第一方行。
- **`$.telemetry.mark({ feature, kind, reason?, props? })`**：标记一次功能使用，生成 `tengu_feature_<kind>` 事件，含 `feature_name` 和附加属性。
- 两种调用均异步（Promise resolve 在行排队后）。
- 行批量发送：首次排队后数秒、累积 100 行时、会话结束时各发送一次 POST 请求（使用 `$.session.authorize()` 的凭证）。
- 发送失败（服务器错误/超时/限流）重试一次；彻底失败则丢弃批次并记录 debug log。
- 每行携带：event id、设备 ID、账户 ID、会话 ID、model、client type、entrypoint、interactivity、环境信息（platform、architecture、terminal、shell、package managers、CI 检测、Git remote hash 等）。
- **隐私控制**：事件名和属性 key 限制为 snake_case token；值限制为有限数字、布尔或 Choice；自由文本不允许。
- **遥测开关**：`DISABLE_TELEMETRY`、`CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC`、`DO_NOT_TRACK` 任一设置时停止发送（每次批量前检查）。
- **第三方 provider**：非 Anthropic 直连（Bedrock、Vertex、Foundry 等）且非 host 管理的场景下不发送。
- **环境变量脱敏**：值为 secret 或包含人名/主机名的变量仅读取是否设置，不取具体值。

---

# 🏛️ 设计

## Hook 架构

| 事件 | 行为 |
| --- | --- |
| `telemetry.*` (gate) | 仅允许 builtin 层（或引擎）通过，其他层调用被拒绝 |
| `telemetry.log` | 校验 entry 格式、排队行、返回 `{ value }`；不合规 entry 返回 `{ deny }` + 原因 |
| `telemetry.mark` | 同上；`kind` 只能是 `ok`、`sad`、`bad`，`sad`/`bad` 必须带 `reason`，`ok` 拒绝 `reason` |
| `engine.create` | 构建 sender，通过 `await next(e)` 后以 spread 方式添加 noun：`{ ...{ telemetry }, ...beneath }`（下层有同名 noun 时保留下层） |
| `session.start` | 获取交互性状态 |
| `session.end` | 发送待发送队列 |

## Noun 契约

`types/index.d.ts` 是 `$.telemetry` 的唯一声明，导出 `Telemetry`、`TelemetryLogEntry`、`TelemetryMarkEntry` 等类型。调用 mod 和测试均读取同一文件。

## `$` 依赖

`session.authorize`、`session.id`、`session.model`、`session.surfaces`、`session.cwd`、`session.repo`、`settings.read`、`env.get`、`fs.read`、`fs.list`、`fs.exists`、`process.run`（`uname` 和 `command -v`）、`clock.after`、`clock.sleep`、`http.fetch`、`ui.log`。

---

# 📁 目录

```
telemetry/
├── hooks/
│   └── register.ts          # 插件入口模块
├── tests/
└── types/
    └── index.d.ts           # $.telemetry 类型声明（noun 契约）
```

---

# 🔗 资源

- 运行源码（不推荐，仅供开发）：
  ```sh
  claude --plugin-dir mods/telemetry
  ```
- 测试：`claude plugin test mods/telemetry`

> 注意：此插件由 CLI 自身加载，不应以 `--plugin-dir` 独立使用。该文件夹有 manifest 仅为遵循插件约定，不意味着可独立运行。内建插件在 telemetry 未加载时调用 `$.telemetry` 找到空 noun（无操作）或不存在该 noun——均为 "无分析" 状态。

---

# 🚀 快速开始

此插件为内建模块，CLI 自动加载（在自身遥测开启时）。不需要手动操作。

---

# ⚙️ 技术栈

- TypeScript
- Claude Code Plugin API (hooks: `telemetry.log`, `telemetry.mark`, `engine.create`, `session.start`, `session.end`)
- `http.fetch` 批量发送至事件日志 ingest
- `process.run` 环境探测（`uname`, `command -v`）
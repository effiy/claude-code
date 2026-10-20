---
title: agents-md — AGENTS.md as Claude Code Project Instructions
tags: [claude-code, plugin, builtin, agents-md, instruction-files, AGENTS.md, CLAUDE.md]
category: claude-code-mods
created: 2025-06-15
updated: 2026-10-20
source: claude-code/mods/agents-md
type: plugin
status: builtin
lifecycle: stable
---

# 💡 概述

`agents-md` 是 Claude Code 内建插件，将 `AGENTS.md` 文件作为项目 instruction file 加载，与引擎读取 `CLAUDE.md` 的方式一致。插件通过一个选项 `instructionFiles` 控制四种模式：

- `claude-md`：仅加载 `CLAUDE.md`，与当前行为一致。
- `claude-md-or-agents-md`（默认）：项目中无自有 instruction file（`CLAUDE.md`、`.claude/CLAUDE.md`、`CLAUDE.local.md`）时，加载 `AGENTS.md`，放置和框架方式与 `CLAUDE.md` 完全相同。
- `claude-md-and-agents-md`：`AGENTS.md` 与 `CLAUDE.md` 并列加载。
- `managed-only`：仅保留组织的托管 instruction file 和引擎 memory，丢弃项目及个人的 instruction file。

---

# ⚡ 功能

- 四种 instruction file 加载模式，通过 `/config` "Project instructions" 选项或 `pluginConfigs` JSON 配置。
- `AGENTS.md` 文件以 `prompt.context` hook 注入，引擎以 `claudeMd` 框架渲染，与 `CLAUDE.md` 地位完全对等。
- 嵌套 `AGENTS.md`：在 `Read` 工具读取子目录文件时自动附加该目录下的 `AGENTS.md`（在 `claude-md-or-agents-md` 和 `claude-md-and-agents-md` 模式下）。
- `agent.spawn` fork 继承父代理的 instruction file 集合。
- 旧键 `projectInstructions` 向下兼容，首次加载时在 transcript 中说明读取方式。

---

# 🏛️ 设计

## Hook 架构

| 事件 | 行为 |
| --- | --- |
| `session.start` | 记录模式行（usage row），首个 start 记录旧键 `projectInstructions` 的读取方式 |
| `prompt.context` | 在 `claude-md-or-agents-md` 和 `claude-md-and-agents-md` 下遍历 `$.fs.ancestors` 查找 `AGENTS.md` 并注入为 project instruction files；`managed-only` 下移除 project/local/user 类文件 |
| `agent.spawn` (fork: true) | fork 继承父代理的 nested file 集合 |
| `tool.call` (Read) | 在两种 AGENTS.md 模式下，为读取的文件附加路径上尚未交付的 `AGENTS.md` 文件 |

## `$` 依赖

`fs.ancestors`、`session.root`、`session.cwd`、`env.get`、`ui.log`、`telemetry.log`、`telemetry.mark`。

`$.telemetry` 是 [telemetry](../telemetry) 插件的 noun；telemetry 插件未加载时调用无操作。

## 遥测数据

| 事件 | 时机 | 属性 |
| --- | --- | --- |
| `agents_md_mode` | 每次新加载，`session.start` 时 | `mode`、`is_interactive` |
| `agents_md_load` | 首次 context 构建 | `mode`、`file_count`、`import_count`、`total_content_length`、`yielded`、`walk_failed`；附加 `$.telemetry.mark` 标记 `agents_md` 功能状态 |
| `agents_md_nested` | Read 附加嵌套文件时 | `mode`、`file_count` |

## 与 CLAUDE.md 的已知差异

以下 8 点仅适用于加载 `AGENTS.md` 的模式：

1. 嵌套文件仅在文本 `Read` 时附加（引擎还会在 `@` 提及、IDE 打开文件/选区、Notebook/图片/PDF 结果时附加 `CLAUDE.md`）。
2. 插件附加的嵌套文件不注册到 loop 的已读状态，compaction 后不恢复（下一次 Read 重新附加）。
3. `/cd` 变更目录时新目录的 `AGENTS.md` 在下一次请求中通过引擎的 instructions announcement 到达。
4. 路径比较按字面进行，引擎会先解析 symlink 工作目录。
5. `--add-dir` 目录不贡献 `AGENTS.md`。
6. `/memory` 和 `#` 不认识 `AGENTS.md` 文件。
7. `AGENTS.md` 中对工作区外的 `@` import 需等待引擎为 `CLAUDE.md` external import 批准的对话框后才生效。
8. 非 fork 子代理在首次 Read 时获取嵌套 `AGENTS.md`，即使父代理已获取过（fork 匹配引擎行为）。

---

# 📁 目录

```
agents-md/
├── hooks/
│   └── register.ts          # 插件入口模块
├── tests/
│   └── register.test.ts     # 测试覆盖默认模式
└── types/
    └── index.d.ts           # 类型声明
```

---

# 🔗 资源

- 配置示例：
  ```json
  {
    "pluginConfigs": {
      "agents-md@builtin": {
        "options": { "instructionFiles": "claude-md-and-agents-md" }
      }
    }
  }
  ```
- 测试命令：`claude plugin test mods/agents-md`
- 运行源码：`claude --plugin-dir mods/agents-md`

---

# 🚀 快速开始

```sh
claude --plugin-dir mods/agents-md
```

内建时已默认加载，可在 `/config` → "Project instructions" 中选择模式。

---

# ⚙️ 技术栈

- TypeScript
- Claude Code Plugin API (hooks: `session.start`, `prompt.context`, `agent.spawn`, `tool.call`)
- 依赖 telemetry 插件 noun（optional）
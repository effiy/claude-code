---
title: diff — Claude Code Diff Pane Plugin
tags: [claude-code, plugin, builtin, diff, git, pane, slash-command]
category: claude-code-mods
created: 2025-04-20
updated: 2026-10-20
source: claude-code/mods/diff
type: plugin
status: builtin
lifecycle: stable
---

# 💡 概述

`diff` 是 Claude Code 内建的 diff 面板插件。`/diff` 命令在 transcript 旁打开会话中未提交的变更（working tree vs HEAD），按文件列出 hunks，并在 Claude 编辑文件和运行 shell 命令时实时刷新。

---

# ⚡ 功能

- `/diff` 命令开关 diff 面板，每次 toggle 在 transcript 中留下 `Diff panel shown` / `Diff panel hidden`。
- Docked 布局：header + 文件列表（toggle）+ hunks 窗口；列表可滚 8 行，滚轮控制 hunks（每次 3 行）。
- Inline 布局（`CLAUDE_CODE_NO_FLICKER=0`）：以内置 dialog 样式展示，带 focus ring（`❯`）和键盘操作。
- 三种 diff 基准：会话开始时 HEAD（默认）、当前 HEAD、或与默认分支的 merge-base。
- 编辑历史选择器：可查看之前 turn 的编辑结果而非 working tree。
- 会话前已变更的文件和噪声文件（lockfiles、generated/test files）折叠展示。
- 文件 arming：可将某个文件的 hunks 作为下一次 prompt 的 context。
- 主循环首次成功编辑自动打开面板（需满足终端宽度、file checkpointing 开启等条件）。
- 面板每秒轮询 HEAD，外部 commit/checkout 也会反映；面板关闭时停止轮询。
- 非 git 仓库中 `/diff` 直接提示不在仓库中。

---

# 🏛️ 设计

## Hook 架构

| 事件 | 行为 |
| --- | --- |
| `session.start` | 绑定引擎，注册 `/diff`；已恢复会话若有编辑 turn 则自动打开面板 |
| `ui.render` (PromptHint) | 读取终端宽度和布局信息，决定首次编辑是否自动打开 |
| `ui.render` (Pane) | 绘制面板：docked 模式绘制 header/base/文件列表/hunks；inline 模式绘制 dialog |
| `command.run` (diff) | 固定仓库、打开/关闭面板 |
| `ui.close` (pane) | 在 dialog detail 视图时回退到列表而非关闭 |
| `ui.scroll` (pane) | Docked 模式下控制 hunks/列表滚动 |
| `ui.focus` (pane) | Dialog 列表中选择文件，重新居中 |
| `command.run` (clear, resume) | 关闭面板，清除会话状态和固定仓库 |
| `tool.call` (Edit, Write, NotebookEdit) | 编辑落地后刷新面板；主循环首次编辑自动打开 |
| `tool.call` (Bash, PowerShell) | 命令执行后刷新面板 |
| `prompt.submit` | 将 armed 文件的 hunks 加入 prompt context 并 disarm |

## `$` 依赖

`clock.after`、`clock.every`、`clock.now`、`command.register`、`env.get`、`fs.list`、`fs.read`、`fs.stat`、`process.run`（git，只读）、`session.id`、`session.messages`、`settings.read`、`store.get`、`store.set`、`telemetry.log`、`telemetry.mark`、`ui.close`、`ui.invalidate`、`ui.log`、`ui.open`、`ui.resolve`、`ui.status`。

`$.telemetry` 是 telemetry 插件的 noun；缺失时遥测行被丢弃。

## Git 调用策略

仅在需要时调用 git：`git rev-parse` 在首次 `/diff` 或首次条件满足的编辑时执行；结果缓存直到 `/clear` 或 `/resume`。working tree 仅在面板打开时获取（编辑落地后或 shell 命令后）。首次 fetch 额外执行一次 `git status` 以获取文件变更时间。

---

# 📁 目录

```
diff/
├── hooks/
│   ├── register.ts          # 插件入口模块
│   └── git/                 # Git 相关逻辑
├── tests/
│   └── register.test.ts     # 测试
└── types/
    └── index.d.ts           # 类型声明
```

---

# 🔗 资源

- 运行源码：`claude --plugin-dir /path/to/diff`
- 测试：`claude plugin test mods/diff`

---

# 🚀 快速开始

```sh
claude --plugin-dir mods/diff
```

然后在一个包含未提交修改的 git 仓库中输入 `/diff`。

---

# ⚙️ 技术栈

- TypeScript
- Claude Code Plugin API (hooks: `session.start`, `ui.render`, `command.run`, `tool.call`, `prompt.submit`)
- Git (`process.run`，只读)
- 依赖 telemetry 插件 noun（optional）
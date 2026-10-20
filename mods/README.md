---
title: Claude Code Mods — 内建插件集合
tags: [claude-code, plugin, mods, hooks, builtin, diff, telemetry, agents-md, sec-default]
category: claude-code-mods
created: 2025-04-01
updated: 2026-10-20
source: claude-code/mods
type: plugin-collection
status: builtin
lifecycle: early-access
---

# 💡 概述

Mod 是一种 Claude Code 插件，其行为完全在 hooks module 中定义：一个 `register(on, options)` 入口 hook 引擎事件为 `($, e, next)` 函数。本目录包含随 Claude Code 内建的 4 个 mod 的源代码，它们随二进制文件一起发布。

| Mod | 功能 | 加载位置 |
| --- | --- | --- |
| [`sec-default`](sec-default) | 保护组织经典 hooks、prompt 内容、managed settings、工具策略、权限规则和环境变量免受个人安装插件的影响 | 最外层（prepend tier），在存在 managed settings 或 Team/Enterprise 组织时 |
| [`diff`](diff) | `/diff`：在 transcript 旁显示会话未提交变更的面板 | 内建 |
| [`telemetry`](telemetry) | 挂载 `$.telemetry` noun（`log`、`mark`），供内建插件记录第一方分析数据 | 内建 |
| [`agents-md`](agents-md) | `AGENTS.md` 作为 project instructions，四种加载模式 | 内建 |

---

# ⚡ 功能

- 每个文件夹是一个完整插件：`.claude-plugin/plugin.json` + `hooks/hooks.json` + TypeScript hooks 模块。
- 类型化开发：hooks 模块以 `import type … from 'claude-code'` 引用 `/plugin-types` 生成的声明文件。
- 测试框架：`claude plugin test mods/<name>` 在每个 mod 的 `tests/` 目录下运行测试。
- 全局类型检查：`tsc -p mods/tsconfig.json`。

---

# 🏛️ 设计

## Noun 契约（Composing Mods）

在 `engine.create` fold 中向 `$` 添加 noun 的 mod 拥有该 noun 的类型。类型定义在 `types/index.d.ts` 中（无 import 的声明文件），并扩展 `EngineInterface`。

- **唯一声明**：noun 只在一个地方声明。mod 自身的 hooks 从 `../types` 导入，engine.create 返回的值与 `EngineInterface['nounName']` 类型校验。
- **调用方读取**：调用另一个 mod noun 的 mod 读取同一文件，从不复制。`mods/tsconfig.json` 通过 `*/types/**/*.d.ts` 包含所有类型。
- **测试 mock**：调用另一个 mod noun 的测试需要加载一个 provider（inline plugin），在 `engine.create` 中挂载 noun。

## 测试架构

```ts
import { describe, expect, mock, test, tier } from 'claude-code/testing'

tier('builtin')

describe('register', () => {
  test('...', async ($, on) => {
    mock.clock(on)
    on('session.start', ($, e) => ({ cwd: e.cwd }))
    // ...
    await $.session.start({ surface: 'terminal', isInteractive: true, cwd: '/work' })
    expect(text).toContain('...')
  })
})
```

Mock 工具：`mock.env(on, variables)`、`mock.store(on, entries)`、`mock.clock(on)`（`advance(ms)` 解决所有等待）、`$.ui.press({ plugin, key })`。

---

# 📁 目录

```
mods/
├── agents-md/       # AGENTS.md 加载插件
├── diff/            # Diff 面板插件
├── sec-default/     # 组织安全默认插件
├── telemetry/       # 遥测插件
├── types/           # 共享类型声明
└── tsconfig.json    # 全量类型检查配置
```

---

# 🔗 资源

- 运行单个 mod：`claude --plugin-dir mods/diff`
- 测试单个 mod：`claude plugin test mods/diff`
- 类型检查：`tsc -p mods/tsconfig.json`

> 注意：hooks modules 仅在 function hooks 启用时加载，API 可能在未来版本中变化（early access）。这些 mod 不在 marketplace 中列出——重要的是已经内建于你的 Claude Code 中的副本。

---

# 🚀 快速开始

```sh
# 运行某个 mod 的源码版本
claude --plugin-dir mods/diff

# 测试某个 mod
claude plugin test mods/agents-md
```

---

# ⚙️ 技术栈

- TypeScript
- Claude Code Plugin API (hooks system)
- 类型系统：自定义 `.d.ts` 声明扩展 `EngineInterface`
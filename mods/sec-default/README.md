---
title: sec-default — 组织安全默认插件
tags: [claude-code, plugin, security, organization, managed-settings, deny-rules, sec-default]
category: claude-code-mods
created: 2025-05-01
updated: 2026-10-20
source: claude-code/mods/sec-default
type: plugin
status: builtin
lifecycle: stable
---

# 💡 概述

`sec-default` 是面向组织的安全默认插件。Function hooks 赋予每个插件在事件链中的发言权，而个人安装的插件位于 user 层（低于组织的 prepend 层，高于 append 层）。组织的 classic hooks、managed CLAUDE.md、settings、MCP allowlist、deny rules 等在 function hooks 出现之前从未落入个人插件范围——本插件以最外层位置保护这些内容，不添加自身策略。

三种操作：`next.to(e, "append")`（跳过 user 层）、`{ deny }` / `{ refuse }`（拒绝 user 层调用者）、`next(e)`（放行）。

---

# ⚡ 功能

- **Classic hooks 保护**：组织的 settings hooks 直接接收引擎输入，答案不可被 user 层修改。
- **Prompt 内容保护**：managed CLAUDE.md、rules、policy skills 在 `prompt.section`、`prompt.context`、`prompt.attachment`、`skill.prompt`、`attribution.text` 事件上跳过 user 层。
- **会话行保护** (`session.append`)：引擎注入的 attachment、compaction 摘要、Skill tool 行不被 user 层改写。
- **Compaction 保护** (`session.compact`)：整个事件跳过 user 层。
- **System prompt 组成保护** (`prompt.compose`)：section 列表不被 user 层修改。
- **Settings 读取保护** (`settings.read`)：plugin policy read 在内的所有设置读取跳过 user 层。
- **工具/命令/Agent 描述保护** (`tool.describe`、`command.describe`、`agent.offer`、`agent.spawn`)：prepend/append 层提供的条目跳过 user 层。
- **工具注册控制** (`tool.register`)：user 层注册可被 managed settings 中的 `allowedMcpServers` 限制；防止伪装成组织 MCP 服务器的工具注册。
- **工具列表保护** (`tool.list`)：组织 MCP 服务器和托管插件的工具保持组织层排列。
- **Deny rules hold** (`tool.check`)：个人插件不能覆盖 deny rule、ask rule、classic hook ask、setting ask 或 host ask。
- **环境变量锁定**：managed `env` 中的变量、launcher 变量、9 个策略/sign-in 路径变量、10 个代码注入变量被锁定，不可被 user 层设置。
- **`allowManagedModsOnly` 选项**：仅允许组织部署的 mod 和内建 mod 加载。
- **`allowModsToOverrideDenyRules` 选项**：允许个人插件覆盖 deny rules（默认关闭）。

---

# 🏛️ 设计

## Hook 架构

| 事件 | 行为 |
| --- | --- |
| `classic.*` | `next.to(e, "append")` |
| `prompt.section`、`prompt.context`、`prompt.attachment`、`skill.prompt`、`attribution.text` | `next.to(e, "append")` |
| `session.append` | 对引擎注入行（attachment、compaction、skill body、组织插件 note）：`next.to(e, "append")` |
| `session.compact` | `next.to(e, "append")` |
| `prompt.compose` | `next.to(e, "append")` |
| `settings.read` | `next.to(e, "append")` |
| `tool.describe`、`command.describe`、`agent.offer`、`agent.spawn` | 当 subject tier 为 prepend/append 时：`next.to(e, "append")` |
| `tool.register` | user 层注册被拒绝（如有 MCP allowlist 或与组织服务器/插件名冲突） |
| `tool.list` | 组织层级工具保持原排列 |
| `tool.check` | 用户插件改写宽松判决时重新执行（跳过 user 层），deny 保持有效 |
| `ui.log` | prepend/append 层的日志行：`next.to(e, "append")` |
| `env.get` | prepend/append 层的读取：`next.to(e, "append")` |
| `env.set` | prepend/append 层的写入：`next.to(e, "append")`；被锁定变量的 user 层写入被拒绝 |
| `plugin.register` | `allowManagedModsOnly` 开启时拒绝 user 层 module |
| 其余事件 | `next(e)` |

## Deny Rules Hold 机制

1. 先执行完整事件链。若 user 层未改写判决（即 verdict 未被放宽），直接通过。
2. 若有改写，跳出事件的 trace 读取各层 tier 信息，再次执行但跳过 user 层 (`next.to(e, "append")`)。
3. 若两次结果不一致且链式答案不更严格，则 user 层改写了 verdict，原 deny/ask 结果被恢复。
4. 适用所有 deny rule、ask rule、classic hook ask、setting ask、host ask。
5. `allowModsToOverrideDenyRules: true` 可全局禁用此 hold。

## 锁定环境变量

**固定禁止 user 层写入的变量：**
- 9 个策略路径变量：`CLAUDE_CODE_MANAGED_SETTINGS_PATH`、`CLAUDE_CODE_REMOTE_SETTINGS_PATH`、`CLAUDE_CODE_MOCK_REMOTE_SETTINGS`、`CLAUDE_CODE_DISABLE_ADMIN_ENV_UNION`、`CLAUDE_CONFIG_DIR`、`CLAUDE_SECURESTORAGE_CONFIG_DIR`、`USE_LOCAL_OAUTH`、`USE_STAGING_OAUTH`、`CLAUDE_LOCAL_OAUTH_API_BASE`
- 10 个代码加载变量：`LD_PRELOAD`、`LD_LIBRARY_PATH`、`LD_AUDIT`、`DYLD_INSERT_LIBRARIES`、`DYLD_LIBRARY_PATH`、`DYLD_FRAMEWORK_PATH`、`BASH_ENV`、`ENV`、`ZDOTDIR`、`NODE_OPTIONS`
- Managed `env` 中设置的所有变量
- 有 `processWrapper` launcher 时的 launcher 变量

## 加载位置

CLI 在以下条件时将其加载为第一个 prepend 插件：
- 机器存在 managed settings 或 managed-mcp.json
- Team 或 Enterprise 组织

若 managed settings 定义了 `prependPlugins`，则需显式将 `sec-default@builtin` 列入。对 Cowork cloud session，即使 `prependPlugins` 未包含它也不会被卸载。

---

# 📁 目录

```
sec-default/
├── hooks/
│   ├── register.ts          # 插件入口模块
│   └── policy/              # Managed settings 读取
├── tests/
└── types/
    └── index.d.ts           # 类型声明
```

---

# 🔗 资源

- 管理员配置：
  ```json
  {
    "pluginConfigs": {
      "cc-plugin-sec-default@builtin": {
        "options": { "allowManagedModsOnly": true }
      }
    }
  }
  ```
- `allowModsToOverrideDenyRules` 配置（同上结构，值为 `true`）。
- 测试命令：`claude plugin test mods/sec-default`

---

# 🚀 快速开始

此插件为内建安全模块，由 CLI 自动加载。管理员通过 managed settings 配置 `pluginConfigs["cc-plugin-sec-default@builtin"].options`。

```json
{
  "pluginConfigs": {
    "cc-plugin-sec-default@builtin": {
      "options": {
        "allowManagedModsOnly": true,
        "allowModsToOverrideDenyRules": false
      }
    }
  }
}
```

---

# ⚙️ 技术栈

- TypeScript
- Claude Code Plugin API (hooks: 全事件覆盖，重点在 `tool.check`、`env.set`、`tool.register`、`plugin.register`)
- Managed settings (`$.settings.read({ source: "policy" })`)
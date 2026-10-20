---
title: Agent SDK Development Plugin
tags: [claude-code, plugin, agent-sdk, python, typescript, scaffolding, verification, verifier]
category: claude-code-plugins
created: 2025-06-01
updated: 2026-10-20
source: claude-code/plugins/agent-sdk-dev
type: plugin
status: stable
lifecycle: stable
author: Ashwin Bhat (ashwin@anthropic.com)
version: 1.0.0
---

# 💡 概述

Agent SDK Development Plugin 是一个综合插件，用于创建和验证 Claude Agent SDK 应用（Python 和 TypeScript）。覆盖从项目脚手架搭建到最佳实践验证的完整生命周期。

---

# ⚡ 功能

## 命令：`/new-sdk-app`

交互式命令，引导创建新的 Claude Agent SDK 应用。

- 询问项目细节：语言、名称、agent 类型、起点
- 检查并安装最新 SDK 版本
- 创建所有必要的项目文件和配置
- 设置 `.env.example`、`.gitignore` 等环境文件
- 提供定制化的工作示例
- 运行类型检查（TypeScript）或语法验证（Python）
- 自动使用对应语言的 verifier agent 验证设置

**使用：**
```bash
/new-sdk-app my-project-name
```

或直接 `/new-sdk-app`（交互式问答）。

**交互式问题：**
1. 语言选择（TypeScript / Python）
2. 项目名称（若未提供）
3. Agent 类型（coding / business / custom）
4. 起点（minimal / basic / 特定示例）
5. 工具偏好（npm/yarn/pnpm 或 pip/poetry）

## Agent：`agent-sdk-verifier-py`

验证 Python Agent SDK 应用的设置和最佳实践。

**检查项：**
- SDK 安装和版本
- Python 环境设置（requirements.txt、pyproject.toml）
- SDK 使用和模式正确性
- Agent 初始化和配置
- 环境和安全（.env、API keys）
- 错误处理和功能
- 文档完整性

**输出：** PASS / PASS WITH WARNINGS / FAIL 状态报告，含关键问题、警告、通过项、具体建议和 SDK 文档引用。

## Agent：`agent-sdk-verifier-ts`

验证 TypeScript Agent SDK 应用的设置和最佳实践。

**检查项：**
- SDK 安装和版本
- TypeScript 配置（tsconfig.json）
- SDK 使用和模式正确性
- 类型安全和 imports
- Agent 初始化和配置
- 环境和安全
- 错误处理和功能
- 文档完整性

**输出：** 同 Python verifier 的综合报告。

---

# 🏛️ 设计

## 工作流

1. 创建新项目：`/new-sdk-app code-reviewer-agent`
2. 回答交互式问题（语言、agent 类型、起点）
3. 自动验证：命令自动运行对应语言的 verifier agent
4. 开发：设置 API key、运行 agent
5. 修改后再次验证：`"Verify my SDK application"`

## 最佳实践

- 始终使用最新 SDK 版本（`/new-sdk-app` 自动检查）
- 部署前运行 verifier
- 保护 API keys：不提交 `.env` 或硬编码
- 遵循官方 SDK 文档（verifier 检查官方模式）
- TypeScript 项目定期运行 `npx tsc --noEmit`
- 为 agent 功能编写测试

---

# 📁 目录

```
agent-sdk-dev/
├── .claude-plugin/
│   └── plugin.json         # 插件清单
├── hooks/
│   └── hooks.json          # Hook 声明
├── agents/
│   ├── agent-sdk-verifier-py/   # Python 验证 Agent
│   └── agent-sdk-verifier-ts/   # TypeScript 验证 Agent
├── commands/
│   └── new-sdk-app.md      # /new-sdk-app 命令定义
└── README.md
```

---

# 🔗 资源

- [Agent SDK Overview](https://docs.claude.com/en/api/agent-sdk/overview)
- [TypeScript SDK Reference](https://docs.claude.com/en/api/agent-sdk/typescript)
- [Python SDK Reference](https://docs.claude.com/en/api/agent-sdk/python)
- [Agent SDK Examples](https://docs.claude.com/en/api/agent-sdk/examples)

---

# 🚀 快速开始

```bash
# 创建新项目
/new-sdk-app my-agent

# 设置 API key
echo "ANTHROPIC_API_KEY=your_key_here" > .env

# 运行 agent（TypeScript）
npm start

# 运行 agent（Python）
python main.py
```

---

# ⚙️ 技术栈

- Claude Code Plugin API
- Claude Agent SDK (TypeScript + Python)
- TypeScript / Python
- npm / yarn / pnpm（TS）
- pip / poetry（Python）
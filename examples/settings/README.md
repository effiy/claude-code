---
title: Settings Examples
tags:
  - claude-code
  - settings
  - configuration
  - enterprise
  - hipaa
  - security
  - sandbox
category: configuration
created: 2025-06-15
updated: 2026-10-20
source: claude-code/examples/settings
type: reference
status: stable
lifecycle: production
---

# Settings Examples

## 💡 概述

Example Claude Code settings files, primarily intended for organization-wide deployments. Use these as starting points — adjust them to fit your needs.

These may be applied at any level of the [settings hierarchy](https://code.claude.com/docs/en/settings#settings-files), though certain properties only take effect if specified in enterprise settings (e.g. `strictKnownMarketplaces`, `allowManagedHooksOnly`, `allowManagedPermissionRulesOnly`).

---

## ⚡ 功能

- Pre-built security posture profiles: lax, strict, and bash-sandbox
- HIPAA-compliant configuration pair (`settings-hipaa.json` + `managed-mcp-hipaa.json`)
- Permission bypass control, marketplace restrictions, and hook management
- Bash sandbox enforcement for isolated command execution
- Web fetch/search denial and mandatory Bash approval modes
- Network allowlist, credential protections, and local data retention (HIPAA)

---

## 📊 指标

「待补充」

---

## 🏛️ 设计

### Configuration Examples

> [!WARNING]
> These examples are community-maintained snippets which may be unsupported or incorrect. You are responsible for the correctness of your own settings configuration.

| Setting | [`settings-lax.json`](./settings-lax.json) | [`settings-strict.json`](./settings-strict.json) | [`settings-bash-sandbox.json`](./settings-bash-sandbox.json) |
|---------|:---:|:---:|:---:|
| Disable `--dangerously-skip-permissions` | ✅ | ✅ | |
| Block plugin marketplaces | ✅ | ✅ | |
| Block user and project-defined permission `allow` / `ask` / `deny` | | ✅ | ✅ |
| Block user and project-defined hooks | | ✅ | |
| Deny web fetch and search tools | | ✅ | |
| Bash tool requires approval | | ✅ | |
| Bash tool must run inside of sandbox | | | ✅ |

### Tips

- Consider merging snippets of the above examples to reach your desired configuration
- Settings files must be valid JSON
- Before deploying configuration files to your organization, test them locally by applying to `managed-settings.json`, `settings.json` or `settings.local.json`
- The `sandbox` property only applies to the `Bash` tool; it does not apply to other tools (like Read, Write, WebSearch, WebFetch, MCPs), hooks, or internal commands

### Deploying via MDM

To distribute these settings as enterprise-managed policy through Jamf, Iru (Kandji), Intune, or Group Policy, see the deployment templates in [`../mdm`](../mdm).

### HIPAA deployments

For an organization that has the HIPAA configuration applied, [`settings-hipaa.json`](./settings-hipaa.json) and [`managed-mcp-hipaa.json`](./managed-mcp-hipaa.json) are a fuller `managed-settings.json` and `managed-mcp.json` pair with sandboxing, a network allowlist, credential protections, and local data retention. [`README-hipaa.md`](./README-hipaa.md) explains each choice and what to edit before deploying.

---

## 📁 目录

| File | Purpose |
|------|---------|
| `settings-lax.json` | Relaxed security profile — disables permission bypass and blocks marketplaces |
| `settings-strict.json` | Strict security profile — full lockdown of permissions, hooks, web tools, and Bash |
| `settings-bash-sandbox.json` | Bash sandbox profile — enforces sandboxed Bash execution and blocks user/project permission rules |
| `settings-hipaa.json` | HIPAA-compliant managed settings with sandboxing, network allowlist, credential protections, and local data retention |
| `managed-mcp-hipaa.json` | HIPAA-compliant managed MCP configuration |
| `README-hipaa.md` | HIPAA deployment guide — explains each choice and what to edit before deploying |

---

## 🔗 资源

- **Full Documentation**: https://code.claude.com/docs/en/settings
- **Settings Hierarchy**: https://code.claude.com/docs/en/settings#settings-files
- **MDM Deployment Templates**: [`../mdm`](../mdm)
- **HIPAA Guide**: [`README-hipaa.md`](./README-hipaa.md)

---

## 🚀 快速开始

1. Review the configuration examples table to choose a starting profile (`lax`, `strict`, or `bash-sandbox`)
2. Merge snippets from multiple examples to reach your desired configuration
3. Test locally by applying to `managed-settings.json`, `settings.json`, or `settings.local.json`
4. Validate that settings files are valid JSON before deploying
5. For enterprise distribution, see the MDM deployment templates in [`../mdm`](../mdm)
6. For HIPAA environments, review [`README-hipaa.md`](./README-hipaa.md) and use the HIPAA-specific configuration pair

---

## ⚙️ 技术栈

- **Configuration Format**: JSON
- **Settings Levels**: Enterprise (managed), user (`settings.json`), local (`settings.local.json`), project
- **Security Features**: Permission bypass control, plugin marketplace blocking, hook management, Bash sandbox
- **Enterprise**: MDM-distributable via Jamf, Kandji, Intune, Group Policy
- **Compliance**: HIPAA-ready configuration with network allowlist and credential protections
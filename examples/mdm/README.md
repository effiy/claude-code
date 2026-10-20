---
title: MDM Deployment Examples
tags:
  - claude-code
  - mdm
  - jamf
  - kandji
  - intune
  - group-policy
  - managed-settings
  - enterprise
category: deployment
created: 2025-06-15
updated: 2026-10-20
source: claude-code/examples/mdm
type: reference
status: stable
lifecycle: production
---

# MDM Deployment Examples

## 💡 概述

Example templates for deploying Claude Code [managed settings](https://code.claude.com/docs/en/settings#settings-files) through Jamf, Iru (Kandji), Intune, or Group Policy. Use these as starting points — adjust them to fit your needs.

All templates encode the same minimal example (`permissions.disableBypassPermissionsMode`). See the [settings reference](https://code.claude.com/docs/en/settings#available-settings) for the full list of keys, and [`../settings`](../settings) for more complete example configurations.

---

## ⚡ 功能

- Cross-platform managed settings deployment via multiple MDM solutions
- macOS support: Jamf, Iru (Kandji) via plist or mobileconfig profiles
- Windows support: Intune via PowerShell scripts or ADMX/ADML Group Policy
- Universal JSON-based `managed-settings.json` for any platform
- Enterprise precedence: managed settings cannot be overridden by users

---

## 📊 指标

「待补充」

---

## 🏛️ 设计

### Templates

> [!WARNING]
> These examples are community-maintained templates which may be unsupported or incorrect. You are responsible for the correctness of your own deployment configuration.

| File | Use with |
| :--- | :--- |
| [`managed-settings.json`](./managed-settings.json) | Any platform. Deploy to the [system config directory](https://code.claude.com/docs/en/settings#settings-files). |
| [`macos/com.anthropic.claudecode.plist`](./macos/com.anthropic.claudecode.plist) | Jamf or Iru (Kandji) **Custom Settings** payload. Preference domain: `com.anthropic.claudecode`. |
| [`macos/com.anthropic.claudecode.mobileconfig`](./macos/com.anthropic.claudecode.mobileconfig) | Full configuration profile for local testing or MDMs that take a complete profile. |
| [`windows/Set-ClaudeCodePolicy.ps1`](./windows/Set-ClaudeCodePolicy.ps1) | Intune **Platform scripts**. Writes `managed-settings.json` to `C:\Program Files\ClaudeCode\`. |
| [`windows/ClaudeCode.admx`](./windows/ClaudeCode.admx) + [`en-US/ClaudeCode.adml`](./windows/en-US/ClaudeCode.adml) | Group Policy or Intune **Import ADMX**. Writes `HKLM\SOFTWARE\Policies\ClaudeCode\Settings` (REG_SZ, single-line JSON). |

### Tips

- Replace the placeholder `PayloadUUID` and `PayloadOrganization` values in the `.mobileconfig` with your own (`uuidgen`)
- Before deploying to your fleet, test on a single machine and confirm `/status` lists the source under **Setting sources** — e.g. `Enterprise managed settings (plist)` on macOS or `Enterprise managed settings (HKLM)` on Windows
- Settings deployed this way sit at the top of the precedence order and cannot be overridden by users

---

## 📁 目录

| File | Use with |
| :--- | :--- |
| [`managed-settings.json`](./managed-settings.json) | Any platform. Deploy to the [system config directory](https://code.claude.com/docs/en/settings#settings-files). |
| [`macos/com.anthropic.claudecode.plist`](./macos/com.anthropic.claudecode.plist) | Jamf or Iru (Kandji) **Custom Settings** payload. Preference domain: `com.anthropic.claudecode`. |
| [`macos/com.anthropic.claudecode.mobileconfig`](./macos/com.anthropic.claudecode.mobileconfig) | Full configuration profile for local testing or MDMs that take a complete profile. |
| [`windows/Set-ClaudeCodePolicy.ps1`](./windows/Set-ClaudeCodePolicy.ps1) | Intune **Platform scripts**. Writes `managed-settings.json` to `C:\Program Files\ClaudeCode\`. |
| [`windows/ClaudeCode.admx`](./windows/ClaudeCode.admx) + [`en-US/ClaudeCode.adml`](./windows/en-US/ClaudeCode.adml) | Group Policy or Intune **Import ADMX**. Writes `HKLM\SOFTWARE\Policies\ClaudeCode\Settings` (REG_SZ, single-line JSON). |

---

## 🔗 资源

- **Full Documentation**: https://code.claude.com/docs/en/settings#settings-files
- **Settings Reference**: https://code.claude.com/docs/en/settings#available-settings
- **Settings Examples**: [`../settings`](../settings)

---

## 🚀 快速开始

1. Choose the template matching your MDM platform from the table above
2. Replace placeholder values (e.g. `PayloadUUID`, `PayloadOrganization` in `.mobileconfig`)
3. Test on a single machine and confirm `/status` lists the source under **Setting sources**
4. Deploy to your fleet

---

## ⚙️ 技术栈

- **macOS MDM**: Jamf, Iru (Kandji) — plist and mobileconfig profiles
- **Windows MDM**: Intune — PowerShell platform scripts and ADMX/ADML Group Policy
- **Universal**: JSON-based `managed-settings.json` for any platform
- **Preference domain**: `com.anthropic.claudecode` (macOS), `HKLM\SOFTWARE\Policies\ClaudeCode\Settings` (Windows)
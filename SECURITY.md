# 安全策略 (Security Policy)

## 支持版本

| 版本 | 支持状态 |
|------|---------|
| v1.0.0 | ✅ 活跃支持 |

## 报告安全漏洞

本 Skill 本身不包含可执行代码，它是一个 AI Agent 指令集和知识库。但仍可能出现以下问题：

### 应报告的问题
- Human-in-the-Loop 安全红线存在可绕过的逻辑缺陷
- 命令示例中未脱敏的真实凭据/IP
- PoC 自检清单存在致命遗漏
- 可能导致 Agent 执行危险操作而未触发 HitL 确认的设计缺陷

### 报告渠道
- 在 GitHub 仓库提交 Issue（标记 `security` label）
- 或直接联系仓库维护者

### 期望
- 提供详细的复现步骤
- 说明潜在影响范围
- 在公开披露前给予合理修复时间（建议 30 天）

## Human-in-the-Loop 安全红线

本 Skill 内置的 **11 类 HitL 安全红线** 是深度防御第一层，**非代码级强制保证**：

| 类别 | 触发条件 | 拦截动作 |
|------|---------|---------|
| Destructive exploit | DROP/DELETE/UPDATE, file write/delete | 弹确认 |
| High-risk exploit | RCE, WebShell, config tamper | 弹确认 |
| Internal lateral | PsExec/WMI/scheduled task/cred spray | 弹确认 |
| Business interrupt | exploit inject, service restart | 弹确认 |
| Long-running | >256 IP scan | 弹确认 |
| Irreversible | data deletion, filesystem ops | 弹确认 |
| Trigger-hazardous | mass brute, dir bust | 弹确认 |
| High-traffic | mass extranet, DNS tunnel | 弹确认 |
| AD-specific | GPO push, Schema update, ACL change | 弹确认 |
| Cloud-specific | IAM mutation, 0.0.0.0/0 ingress, cluster-admin | 弹确认 |
| Mobile-specific | system app repackaging, Frida persistent | 弹确认 |

> ⚠️ **关键场景（生产环境、真实目标）务必配合外部 hook / 沙箱 / 权限隔离做二次校验**，不可仅依赖声明式红线。

## 脱敏规则

SKILL.md Section 7.2 的脱敏规则对所有输出具有强制约束力：

| 类型 | 替换 |
|------|------|
| Real IP | `[TARGET_IP]` |
| Password/Key | `[REDACTED]` |
| Token | `[TOKEN]` |
| API Key | `[API_KEY]` |

## 负责任使用

本 Skill 仅供**授权安全测试、CTF 竞赛、防御性安全研究和教育场景**使用。

- ✅ 获得书面授权的渗透测试
- ✅ CTF 竞赛
- ✅ 防御性安全研究
- ✅ 自身系统的安全自查
- ❌ 未授权攻击
- ❌ 恶意软件/后门开发

## 免责声明

使用者须自行确认测试目标授权。作者不对因使用本 Skill 造成的任何直接或间接损失承担责任。

攻击未授权目标可能触犯法律，包括但不限于《刑法》第 285/286 条（非法侵入/破坏计算机信息系统罪）。

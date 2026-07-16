> [点击查看英文文档](README_en.md)

# 全栈信息安全专家 (Full-Stack Cybersecurity Expert)

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![SKILL.md](https://img.shields.io/badge/Agent%20Skill-v1.0.0-blue)](SKILL.md)

> 一个实战驱动的 AI Agent 安全技能包。覆盖红蓝对抗、渗透测试、应急响应、代码审计、威胁情报与安全评估。
> **Claude Code / Codex CLI / ChatGPT** 等多平台兼容。
>
> **协议规范而非简单提示词** — 包含模式切换、状态管理(含断点续传)、Token 经济、防幻觉自检、持久化回滚等完整工程化设计。

---

## 概述

本技能包为 **Claude Code / Codex CLI / ChatGPT** 等 AI Agent 提供了一套标准化的网络安全作战框架。它不是一句简单的"你是安全专家"，而是一套可执行的**协议级规范**。

### 四大作战功能

| 模式 | 能力 | 典型场景 |
|------|------|---------|
| 🔴 **红队 (Red Team)** | 深层链路逆向、纯手工打点、现场锻造 PoC/EXP | 渗透测试、漏洞利用、全阶段攻击模拟 |
| 🔵 **蓝队 (Blue Team IR)** | 海量日志过滤研判、攻击链逆向溯源 | 应急响应、告警分析、入侵排查、样本分析 |
| 🟢 **审计 (Code Review)** | 跨文件污点追踪、全局防御层评估 | 代码审计、安全加固、漏洞验证 |
| ⚪ **咨询 (Advisory)** | 标准对照评估、量化合规评分 | 安全评估、威胁情报、等保、安全培训 |

### 核心设计理念

本技能的独特之处在于它解决了 AI Agent 在长时间安全任务中的几个痛点：

1. **上下文漂移** — 状态管理机制让 Agent 在长会话中记住进展，不会中途迷失
2. **Token 浪费** — Token 斩断机制强制 Agent 优化输入，省下资源用于分析更多数据
3. **幻觉控制** — CVE 验证、置信度标记、自检清单三层防幻觉
4. **操作安全** — 真人二次确认红线、持久化回滚盘点，防止忘记清理后门

---

## 文件结构

```
full-stack-cybersecurity-expert/
├── SKILL.md                     # 🎯 Agent 指令 — 中文
├── SKILL_en.md                  # 🎯 Agent 指令 — 英文
├── README.md                    # 👤 中文文档（GitHub 默认首页）
├── README_en.md                 # 👤 英文文档
├── LICENSE                      # MIT 许可证
├── SECURITY.md                  # 安全策略
├── CONTRIBUTING.md              # 贡献指南
├── .gitignore                   # Git 忽略规则
│
├── references/                  # 按需加载的深度材料
│   ├── mode-definitions.md      #   模式完整阶段流 + 漏洞提交格式
│   ├── state-persistence.md     #   状态持久化 JSON 格式 + 续传协议
│   ├── code-audit-deep-dive.md  #   代码审计完整规范（Source/Sink 表等）
│   ├── blocking-rules.md        #   生产级中间件阻断规则（Nginx/Apache/IIS）
│   ├── giant-log-protocol.md    #   巨型日志（>1GB）截流协议
│   ├── anti-hallucination.md    #   防幻觉完整自检清单
│   ├── ad-security.md           #   AD 域安全（Kerberos 攻击、ACL 滥用、AD CS 等）
│   ├── cloud-security.md        #   云安全（AWS/GCP/Azure、K8s 等）
│   ├── mobile-security.md       #   移动安全（Android/iOS 渗透、OWASP Mobile Top 10 2024）
│   ├── attack_framework.md      #   MITRE ATT&CK 技术映射
│   ├── owasp_top10.md           #   OWASP Top 10 (2021)
│   ├── commands_cheatsheet_offensive.md  #   渗透测试命令速查（信息收集/Web/内网/破解/代理）
│   ├── commands_cheatsheet_defensive.md  #   应急响应命令速查（Linux/Windows/Webshell/Event Log）
│   └── report_template.md       #   安全评估报告模板
│
├── examples/                    # 使用示例
│   ├── log-analysis-example.md  #   日志分析完整过程示例
│   └── pentest-workflow-example.md # 渗透测试工作流示例
│
└── scripts/                     # 动态武器加工厂（运行时 Agent 现场编写，不入库）
```

---

## 使用方式

### 在 Claude Code 等 Agent 中使用

```bash
# 将目录放入对应 Agent 的 skills/ 路径下即可
# Claude Code: ~/.claude/skills/full-stack-cybersecurity-expert/
# 通用规则:    $AGENT_HOME/skills/full-stack-cybersecurity-expert/
```

### 其他平台

```bash
# Codex CLI / ChatGPT 等：将 SKILL.md 内容作为 system prompt 注入即可
```

> **提示**：本项目的核心是 `SKILL.md`，任何支持 system prompt 注入的 AI 工具均可直接使用其内容。

### 支持的触发输入

| 你说 | Agent 会用的模式 |
|------|----------------|
| "做一次 Web 渗透测试" | 🔴 Red Team |
| "写一个 SSRF 的 POC" | 🔴 Red Team |
| "审计这个域环境" / "做内网渗透" | 🔴 Red Team — AD |
| "云上有个应用测一下安全" | 🔴 Red Team — Cloud |
| "测这个 App 的安全性" | 🔴 Red Team — Mobile |
| "分析这条 WAF 告警" | 🔵 Blue Team IR |
| "服务器被入侵了，做应急响应" | 🔵 Blue Team IR |
| "分析这个恶意样本" | 🔵 Blue Team IR |
| "审计这段 Java 代码" | 🟢 Code Review |
| "设计等保三级基线检查方案" | ⚪ Advisory |
| "这个 IP 是不是恶意的" | ⚪ Advisory |
| "讲一下 SQL 注入原理和防护" | ⚪ Advisory |

### 任务执行流程

每个任务遵循五步流程：

```
Step 1: 理解确认 → Step 2: 模式切换+规划 → Step 3: 逐步执行
→ Step 4: 结论修复 → Step 5: 报告+清理+回滚
```

---

## 设计哲学

### 为什么不是一句简单的"你是安全专家"？

安全工作的特殊性决定了简单的角色预设远远不够：

- **模式差异大** — 渗透测试需要攻击链发散思维，应急响应需要证据链收敛思维，强行用同一种推理逻辑效果很差
- **任务跨度长** — 一次完整的渗透测试可能需要数十轮交互，没有状态管理 Agent 会不断遗忘
- **风险高** — 不正确的操作可能触发 WAF 告警、破坏生产数据、留下后门被溯源
- **输出要严谨** — CVE 编号不能编、漏洞利用链必须闭合、修复代码必须可运行

本技能通过**协议化设计**而非"提示词工程"来解决这些问题。

### 关键机制

| 机制 | 解决的问题 |
|------|-----------|
| **模式切换** | 不同任务使用不同的推理逻辑，避免思维冲突 |
| **状态管理** | 长任务不下线，断点可续传 |
| **Token 斩断** | 优化上下文使用，省出资源给实战分析 |
| **防幻觉自检** | CVE 必须验证、PoC 必须自检，三思后输出 |
| **回滚盘点** | 渗透结束时强制清理后门，防止安全事件 |

---

## 许可

MIT License — 详见 [LICENSE](LICENSE)

## 免责声明

本技能包为安全研究与授权测试设计。所有技术信息仅供合法合规场景使用。使用者须自行确认拥有目标系统的书面授权。作者不对滥用行为承担责任。

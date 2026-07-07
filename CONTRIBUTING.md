# 贡献指南

欢迎为 Full-Stack Cybersecurity Expert Skill 做出贡献！本项目是一个 AI Agent 指令集和知识库，贡献方式和传统代码项目略有不同。

## 快速开始

### 理解项目

在动手之前，建议按顺序阅读：
1. [README.md](README.md) — 项目概述、设计哲学、使用方式
2. [SKILL.md](SKILL.md) — 核心 Agent 指令，理解架构全貌
3. [references/](references/) — 按需加载的深度材料

### 找事做

- **Good First Issue**: 修正命令语法错误、补充缺失的 Event ID、更新过时的工具名称
- **Help Wanted**: 补充领域知识（工控/车联网/IoT）、翻译英文版、优化 Token 效率
- **Discussion**: 新增 Mode、大改架构前先开 Issue 讨论

## 开发约定

### SKILL.md 约束

SKILL.md 是项目的核心，修改需遵守 **3 条硬约束**：

1. **行数 ≤ 250 行** — 这是 Agent 常驻上下文的容量上限。新内容要么压缩要么移到 references/
2. **引用走 On-Demand 路由** — 深度材料用 `(ref: references/xxx.md)` 引用，不内联到 SKILL.md
3. **安全红线只增不减** — 不得降低 Human-in-the-Loop 确认要求，不得移除脱敏规则

### references/ 修改

| 规则 | 说明 |
|------|------|
| 技术信息需附来源 | CVE 编号/Event ID/命令语法需标注参考来源（NVD/Microsoft Docs/官方文档） |
| 路径与 SKILL.md 一致 | 文件名和引用路径必须与 SKILL.md Section 10 的路由表对齐 |
| 保持按需加载粒度 | 单个 reference ≤ 300 行，超过则考虑拆分 |
| 占位符规范 | 使用 `[TARGET_IP]`/`[DOMAIN]`/`[USER]`/`[PASSWORD]`/`[REDACTED]` 统一格式 |

### examples/ 修改

- Phase 命名须与 `references/mode-definitions.md` 阶段流对齐
- State 更新须体现完整字段（Objective/Phase/Assets/Findings/Hypotheses/Next/Blockers/TempFiles）
- HitL 确认须展示 `⚠️ [action] [risk] [duration] Continue?` 格式

### 提交规范

- **分支命名**: `add/<feature>` 或 `fix/<issue>`（如 `add/ot-security`、`fix/event-id-4670`）
- **Commit Message**: 中文简洁描述改动（如 `fix: Golden Ticket 检测 Event ID 4670→4769`）
- **PR 描述**: 附上修改理由、影响范围、是否需更新 SKILL.md 引用

## 领域扩展清单

本项目当前覆盖 3 个垂直领域，欢迎扩展：

| 优先级 | 领域 | 状态 |
|--------|------|------|
| — | AD 域安全 | ✅ 已覆盖 (ad-security.md) |
| — | 云安全 (AWS/GCP/Azure/K8s) | ✅ 已覆盖 (cloud-security.md) |
| — | 移动安全 (Android/iOS) | ✅ 已覆盖 (mobile-security.md) |
| 高 | 工控安全 (ICS/SCADA/Modbus/DNP3) | ⏳ 待贡献 |
| 高 | LLM/AI 安全 (提示注入/模型提取/MLOps) | ⏳ 待贡献 |
| 中 | 车联网安全 (CAN Bus/UDS/DoIP) | ⏳ 待贡献 |
| 中 | IoT/嵌入式安全 (MQTT/CoAP/Firmware) | ⏳ 待贡献 |
| 低 | 区块链/Web3 安全 | ⏳ 待贡献 |

### 新增领域模板

添加新领域时需同时创建：

```
references/<domain>-security.md   # 领域参考文档（攻击手法/检测/防护/命令）
mode-definitions.md               # 补充 Red Team + Blue Team 专属阶段流
SKILL.md                          # 更新 Section 1 触发条件 + Section 2 Domain-Specific State + Section 4 HitL 红线 + Section 10 路由表
```

## 质量审核

PR 审核时检查：

- [ ] 技术准确性：CVE 版本号/Event ID/命令行开关是否正确
- [ ] 一致性：Phase 命名/State 字段/占位符格式与现有约定一致
- [ ] 安全性：未引入武器化代码，未降低 HitL 红线标准
- [ ] Token 效率：新增内容不超过对应文件的合理行数上限
- [ ] 引用完整性：SKILL.md 路由表是否包含新文件引用

感谢你的贡献！

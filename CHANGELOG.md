# Changelog

本项目版本号遵循 [Semantic Versioning](https://semver.org/)。

## [1.0.0] - 2026-07-06

### 首个正式发布版本

基于内部迭代版本（v1.0-v1.2 + pre-release v3.0-v3.7），经全面审计后发布为 v1.0.0。

### 核心功能

- 四大作战功能：🔴 红队 / 🔵 蓝队 IR / 🟢 代码审计 / ⚪ 咨询
- 动态模式切换协议 + 会话状态管理（含断点续传 + Fork/Converge）
- Token 斩断机制 + 防幻觉自检体系（CVE 验证、PoC Self-Check、置信度分级）
- Human-in-the-Loop 安全红线（11 类禁止操作 + 二次确认）
- 按需加载的 references/ 深度参考体系（13 个参考文件）
- 持久化回滚审计协议（Step 5 强制清理盘点）
- AD/Cloud/Mobile 三大领域专属阶段流 + 状态扩展 + 安全红线
- 完整的日志分析 + 渗透测试示例

### 开源前审计修复

经全面审核（P0-P3 共 22 项），修复内容如下：

**P0（阻断性）**
- 版本号三方不一致 → SKILL.md/README.md/CHANGELOG.md 统一为 v1.0.0

**P1（重要）**
- ad-security.md Golden Ticket 检测 Event ID 4670 → 4769/4724
- ad-security.md SID History 检测 Event ID 4670 → 4765/4766；防御清单同步补充
- mobile-security.md OWASP Mobile Top 10 对齐 2024 官方 Final Release
- 创建 .gitignore

**P2（一般）**
- ad-security.md AD CS 表补全 ESC5/6/7；ESC2/ESC8 攻击描述修正；ticketer.py 参数统一为 -domain-sid
- cloud-security.md "Azure Scout" → ScoutSuite；语法修正
- log-analysis-example.md Phase 命名对齐 mode-definitions.md
- 删除空 assets/ 目录

**P3（建议性）**
- mode-definitions.md few-shot 示例修正缩进 + 补充 [假设] 标注示范
- mode-definitions.md Blue Team State Phase 对齐阶段名（日志分析/攻击溯源）
- giant-log-protocol.md `grep -m` → `head` 管道（POSIX 兼容）
- commands_cheatsheet.md `grep '!\|\*'` → `grep -vE '!|\*'`（POSIX 兼容）
- ad-security.md `nxc` 统一为 `netexec`
- pentest-workflow-example.md Phase 命名对齐 + HitL 格式对齐
- README.md 贡献指南补充 PR 规范；文件结构更新

### Pre-release Development History

- **v1.2** (2026-06-14) — 补全 AD/Cloud/Mobile 协议层深度；新增 Domain-Specific State Extensions；HitL 新增 3 条领域专属红线；断点续传 Objective 一致性校验
- **v1.1** (2026-06-14) — 新增 AD/Cloud/Mobile 三大安全方向；references/ 从 10 扩展至 13；Red Team 增加 AD/Cloud/Mobile 触发条件
- **v1.0** (2026-06-14) — 首个内部版本：四大功能 + 模式切换 + 状态管理 + Token 斩断 + 防幻觉 + HitL 红线 + 10 个 references + 回滚审计 + 示例
- **v3.7** (2026-06-11) — 防幻觉框架、持久化回滚、代码审计防御层评估、MITRE ATT&CK 映射
- **v3.6** (2026-06-10) — 能力边界声明、回滚强化、HitL 红线加固、Token 斩断预过滤
- **v3.5** (2026-06-09) — 防幻觉体系建立、红队漏洞提交格式化、报告模板
- **v3.0** (2026-06-08) — 初始内部版本，四大功能基础框架建立

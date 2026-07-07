# 状态持久化与断点续传协议

> 被 SKILL.md "2. Session State" 引用。需要断点续传或序列化状态时加载。

## JSON 序列化格式

```json
{
  "version": "1.0",
  "mode": "Red Team",
  "objective": "...",
  "phase": "漏洞探测",
  "assets": ["192.168.1.1:443", "web01.internal"],
  "credentials": [],
  "findings": [{"id": 1, "title": "...", "confidence": "High"}],
  "hypotheses": ["Path-A: SSRF → internal API"],
  "next": "验证 SSRF 盲打",
  "blockers": [],
  "tempfiles": ["/tmp/scan_result.txt"],
  "persisted_changes": [],
  "last_updated": "2026-06-12T11:30:00+08:00"
}
```

## 断点续传复活逻辑

0. Step 1 初始化前检测 `.agent_session.json` 是否存在。不存在 → 空 State 初始化
1. 存在 → 读取反序列化，提取 `objective` 字段
2. 与当前任务声明的 `[OBJECTIVE]` 对比：
   - **一致** → 完整继承 Assets/Findings/Credentials/Hypotheses，声明：`[检测到历史中断会话，已成功载入状态机，当前进度：已确认 N 项 / 待验证 M 条假设]`
   - **不一致**（新任务 / 旧会话残留）→ 输出 `[检测到来自其他任务的残留会话文件，已忽略，本次为新任务]`，自动删除旧文件，初始化空 State
   - **`objective` 字段为空或缺失** → 输出 `[残留会话状态不完整，已忽略，本次从零开始]`，自动删除旧文件，初始化空 State
3. 用户要求"重新开始" → 删除文件后初始化空 State

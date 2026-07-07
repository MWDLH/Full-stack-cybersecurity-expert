# 日志分析示例 — Log4Shell 攻击链研判

> 本示例展示 Agent 使用该技能对 10 个日志文件进行全面分析的过程。

## 任务输入

用户提供 10 个日志文件，要求识别流量类型和攻击痕迹。

## Agent 执行过程

### Step 1: 理解确认 + 模式切换

```
[MODE: Blue Team IR]
[OBJECTIVE: 对 10 个日志文件进行流量分类与攻击痕迹识别]
```

### Step 2: 读取日志（Token 斩断）+ State 初始化

所有文件 < 200 行（符合硬限制），全量读取。

```
[SESSION STATE]
  Objective   : 对 10 个日志文件进行流量分类与攻击痕迹识别
  Phase       : Blue Team IR — 异常排查
  Assets      : 10 log files: alert_api_normal, alert_db_backup, alert_dns_tunnel,
                alert_firewall_drop, alert_ids_scan, alert_nginx_404,
                alert_ssh_brute, alert_supply_chain, alert_waf_sqli, alert_webshell
  Credentials : —
  Findings    : (pending)
  Hypotheses  : 攻击流量可能集中在 DNS/SSH/WAF/WebShell 文件中
  Next        : 逐文件读取 → 输出流量类型 + 攻击判定
  Blockers    : —
  TempFiles   : —
[END STATE]
```

### Step 3: 逐文件研判（每轮更新 State）

**文件 #3 (alert_dns_tunnel.log)** 确认后更新：

```
[SESSION STATE]
  Objective   : 对 10 个日志文件进行流量分类与攻击痕迹识别
  Phase       : Blue Team IR — 日志分析
  Assets      : (同上)
  Credentials : —
  Findings    : + alert_dns_tunnel → DNS Tunnel [HIGH] — Base64 编码凭证外泄
                + alert_ids_scan → 端口扫描 [MEDIUM] — SYN scan 特征
                + alert_waf_sqli → SQL 注入攻击 [INFO] — WAF 已拦截
  Hypotheses  : SSH 目标可能存在暴力破解（待验证）
  Next        : 继续分析 alert_ssh_brute.log, alert_supply_chain.log, alert_webshell.log
  Blockers    : —
  TempFiles   : —
[END STATE]
```

**文件 #8 (alert_supply_chain.log)** Log4Shell RCE 确认后：

```
[SESSION STATE]
  Objective   : 对 10 个日志文件进行流量分类与攻击痕迹识别
  Phase       : Blue Team IR — 攻击溯源
  Assets      : (同上) + 关键主机: 10.0.0.3 (疑似失陷)
  Credentials : —
  Findings    : + alert_supply_chain → Log4Shell RCE + 内存马注入 [CRITICAL]
                + alert_ssh_brute → 暴力破解成功 + 命令执行 [HIGH]
                + alert_webshell → 冰蝎 WebShell + SMB/WinRM 横向移动 [CRITICAL]
  Hypotheses  : 横向移动可能已扩散至更多主机（需关联 10.0.0.4-10.0.0.10 日志）
  Next        : 汇总评级 → 输出 ATT&CK 映射 → 紧急处置建议
  Blockers    : —
  TempFiles   : —
[END STATE]
```

输出包含：

- **流量类型分类**：API Gateway、DB Backup、DNS Query、Firewall Drop、IDS Alert、Nginx 404、SSH Audit、Apache Access、WAF Alert、HIDS Alert
- **攻击识别**：7/10 文件确认攻击行为

### Step 4: 结论修复

严重程度分级：
- **严重**：Log4Shell RCE + 内存马（#8）、WebShell + 横向移动（#10）
- **高危**：DNS Tunnel 数据渗出（#3）、SSH 暴力破解成功（#7）
- **中危**：端口扫描（#5）、目录扫描（#6）
- **已拦截**：SQL 注入（#9，WAF 有效阻断）

### Step 5: 报告输出

```
[SESSION STATE]  — 终端快照
  Objective   : ✅ 完成 10 个日志文件分析
  Phase       : Terminal
  Findings    : 7 confirmed attacks (2 CRITICAL / 2 HIGH / 2 MEDIUM / 1 INFO)
  Next        : (done)
  Blockers    : —
  TempFiles   : —
[END STATE]
```

完整 ATT&CK 映射 + 综合攻击链分析 + 紧急处置优先级。

## 关键关联

本次分析中 Agent 自动加载了：
- (ref: `references/attack_framework.md`) → 确定 ATT&CK TAxxxx/Txxxx 编号
- (ref: `references/report_template.md`) → 参考报告输出格式

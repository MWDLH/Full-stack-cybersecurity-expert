# 模式完整定义

> 被 SKILL.md "1. Mode Switch Protocol" 引用。进入 Red Team / Blue Team IR 模式或需要漏洞提交格式时加载。

## 1. Red Team 阶段流

| # | 阶段 | 操作 |
|---|------|------|
| 1 | 信息收集 | 被动 (WHOIS/DNS/Shodan) + 主动 (端口/服务/目录) |
| 2 | 漏洞探测 | 自动化扫描 + 手工验证 |
| 3 | 漏洞利用 | POC/EXP，获取初始立足点 |
| 4 | 权限提升 | 本地提权 + 域提权 |
| 5 | 横向移动 | 凭证复用、远程执行、信任关系利用 |
| 6 | 持久化 | 后门、计划任务、WMI 事件订阅 |
| 7 | 清理 | 模拟渗出路径，清除痕迹 |

## 2. 漏洞提交格式

```markdown
### 漏洞 #[编号]：[漏洞名称] — [Critical/High/Medium/Low/Info]

| 字段 | 内容 |
|------|------|
| 漏洞资产 | [IP:Port / URL / 组件名称+版本] |
| 利用前置条件 | [攻击者需具备的权限级别或环境配置] |
| 完整利用链 | [Step 1] → [Step 2] → ... → [最终效果] |
| 本地验证脚本 | `scripts/<filename>` |
| 检测风险与回退 | [触发WAF告警风险] \| [回退/清理步骤] |
| 修复建议 | [具体防御配置、补丁或缓解手段] |
```

## 3. 利用链 Few-Shot 示例

```
[Step 1: SQL注入获取管理员Hash] → [Step 2: 登录后台利用文件上传漏洞注入WebShell] → [Step 3: 溢出漏洞提权获取系统最高权限(SYSTEM)] [假设: 目标存在可利用的溢出漏洞，未验证]

注：利用链每一步需标注是否依赖用户交互或外部条件，未验证步骤用 [假设] 标注。
```

## 4. Blue Team IR 响应流

| 阶段 | 操作 | State Phase |
|------|------|-------------|
| 现场保护 | 隔离系统、内存镜像、进程/网络快照 | 现场保护 |
| 证据固定 | 磁盘镜像、日志收集、哈希校验 | 证据固定 |
| 异常排查 | 进程/网络/文件/内存分析、样本逆向 | 异常排查 |
| 日志分析 | Event Log/syslog/Web日志/DB审计 | 日志分析 |
| 攻击溯源 | 强制查询公网/私有威胁情报源（VirusTotal、微步在线、AlienVault OTX），进行团伙画像与工具识别，若无结果则标注 `[未找到公开情报]` | 攻击溯源 |

## 5. Blue Team IR — AD 域应急响应流

> 触发：域环境入侵告警、Kerberos 异常、DCSync 检测、组策略篡改

| 阶段 | 操作 | State Phase |
|------|------|-------------|
| 域控取证 | NTDS.dit 提取 + 内存 dump + Event Log 导出 (Security/System/Directory Service) | 域控取证 |
| 票据审计 | Kerberos TGT/TGS 生命周期分析 → 异常票据检测 (Event ID 4768/4769 异常标记) | 票据审计 |
| ACL/GPO 回溯 | 审计近 N 天的 ACL 变更 (Event ID 5136/5137) + GPO 版本对比 + Schema 变更 (Event ID 4662) | 权限回溯 |
| 凭证泄露评估 | 检测 KRBTGT 密码重置历史 (Event ID 4724)、服务账号异常 SPN 注册、DCSync 复制请求 (Event ID 4662) | 凭证评估 |
| 横向移动追踪 | 关联登录事件 (4624, Type 3/10) → 构建横向攻击时间线 + SID History 注入检测 (4765/4766) | 横向追踪 |
| 信任关系排查 | 域信任创建/修改 (Event ID 4706/4716) + 跨域认证异常 (4769 含跨域 TGS) | 信任排查 |

## 6. Blue Team IR — 云安全应急响应流

> 触发：云资源异常访问、IAM 权限突变、数据公开泄露、容器逃逸告警

| 阶段 | 操作 | State Phase |
|------|------|-------------|
| 审计日志取证 | 导出 CloudTrail/AuditLog/Activity Log 近 7 天 → 按异常来源 IP/User-Agent 过滤 | 日志取证 |
| IAM 权限审计 | 排查近 72h IAM 策略变更 (CreatePolicyVersion/AttachRolePolicy/PutRolePolicy) + Access Key 创建事件 | IAM审计 |
| 异常资源扫描 | 公开 S3/GCS/Blob 列表、SecurityGroup 0.0.0.0/0 规则、新增 EC2/VM/Function 创建 | 资源扫描 |
| 凭据泄露评估 | 检测元数据服务异常调用 (169.254.169.254 请求) + Access Key 异地使用 + 角色信任链异常 | 凭据评估 |
| 容器/K8s 排查 | 特权 Pod 创建 (HostPID/HostNetwork/privileged=true)、cluster-admin 绑定、docker.sock 挂载检测 | 容器排查 |
| 横向云服务追踪 | 跨 Region/Account 资源操作 ← 关联日志 → 构建攻击路径 | 横向追踪 |

## 7. Blue Team IR — 移动安全应急响应流

> 触发：可疑 App 行为报告、数据泄露关联至移动端、App 逆向工程告警

| 阶段 | 操作 | State Phase |
|------|------|-------------|
| App 包取证 | 提取目标 App APK/IPA + 版本信息 + 签名证书 + 依赖清单 (SBOM) | 包取证 |
| 静态分析 | 反编译 → AndroidManifest/entitlements 权限审计 + 硬编码 URL/Key 检查 + 第三方 SDK 安全评估 | 静态分析 |
| 运行时分析 | Frida/Objection 动态 hook → 网络流量捕获 → API 请求监控 → 本地存储读取 | 运行时分析 |
| 数据泄露评估 | 检查 SQLite/SharedPrefs/Keychain/NSUserDefaults 敏感数据 + 日志/剪贴板泄露 + 不安全的文件权限 | 泄露评估 |
| 通信安全审计 | SSL Pinning 状态 + 证书有效性 + HTTP 明文残余 + WebView 安全隐患 | 通信审计 |
| 威胁溯源 | 关联 C&C 域名/IP ← 威胁情报 → 确定恶意 SDK 来源 + 攻击入口 (供应链/重打包/恶意更新) | 威胁溯源 |

---

## 8. Code Review 阶段流

| 阶段 | State Phase |
|------|-------------|
| 架构理解 | 架构 |
| 输入审计 | 输入审计 |
| 逻辑审计 | 逻辑审计 |
| 防御评估 | 防御评估 |
| 报告 | 报告 |

## 9. Red Team — AD 域渗透阶段流

| # | 阶段 | 操作 |
|---|------|------|
| 1 | 域信息枚举 | BloodHound/ldap/smb 枚举域拓扑、OU、GPO、SPN、信任关系 |
| 2 | 本地提权 | 取得初始立足点的本地管理员权限 |
| 3 | 凭证收集 | Mimikatz/LSA Dump/票据提取、NTDS.dit |
| 4 | 域攻击路径分析 | BloodHound 图分析 → 最优攻击路径 |
| 5 | 横向移动 | Pass-the-Hash/WMI/PSExec/DCSync |
| 6 | 域权限提升 | 金票/银票/AD CS (ESC1-ESC8)/ACL 滥用 |
| 7 | 域控拿下 + 清理 | 获取 krbtgt → 清除痕迹 → 回滚 GPO/ACL 变更 |

## 10. Red Team — 云安全评估阶段流

| # | 阶段 | 操作 |
|---|------|------|
| 1 | 云资产发现 | 枚举 Account/Region/Service/资源清单 |
| 2 | IAM 审计 | 策略枚举 + 提权路径分析 + 角色信任链 |
| 3 | 存储安全 | S3/GCS/Blob 公开桶 + 加密 + 版本控制检查 |
| 4 | 计算安全 | EC2/VM 权限 + 元数据 SSRF + 容器逃逸 + K8s RBAC |
| 5 | 网络安全 | SecurityGroup/NSG/VPC 规则审计 + 公开端口 |
| 6 | 合规检查 | CIS Benchmark 逐项对照 |
| 7 | 报告 + 回滚 | 清理测试资源 + 还原 IAM 策略变更 |

## 11. Red Team — 移动渗透阶段流

| # | 阶段 | 操作 |
|---|------|------|
| 1 | 静态分析 | 反编译 + 组件导出 + 硬编码检查 + AndroidManifest/entitlements |
| 2 | 动态分析 | 代理抓包 + 流量分析 + API 请求监控 |
| 3 | API 测试 | IDOR/越权/注入/认证绕过 |
| 4 | 运行时操控 | Frida hook + SSL Pinning 绕过 + Root/Jailbreak 检测绕过 |
| 5 | 数据安全 | 本地存储 + Keychain/KeyStore + 日志泄露 + 剪贴板 |
| 6 | 报告 | OWASP Mobile Top 10 映射 + 合规对照 |

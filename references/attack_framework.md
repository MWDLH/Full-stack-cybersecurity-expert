> 被 SKILL.md "10. On-Demand References" 索引。需要 ATT&CK 攻击阶段映射或 Txxx 技术编号时加载。

# MITRE ATT&CK 框架常用技术映射参考

## 攻击阶段与常用技术

### TA0043: Reconnaissance（侦察）

| 技术 ID | 技术名称 | 说明 |
|---------|---------|------|
| T1595 | Active Scanning | 主动扫描目标（端口扫描、漏洞扫描） |
| T1590 | Gather Victim Network Information | 收集目标网络信息（域名、IP 段、DNS） |
| T1589 | Gather Victim Identity Information | 收集身份信息（员工邮箱、用户名） |
| T1593 | Search Open Websites/Domains | 搜索公开网站/域名信息 |
| T1596 | Search Open Technical Databases | 搜索公开技术数据库（Shodan、Censys） |

### TA0042: Resource Development（资源开发）

| 技术 ID | 技术名称 | 说明 |
|---------|---------|------|
| T1587 | Develop Capabilities | 开发恶意软件/漏洞利用 |
| T1588 | Obtain Capabilities | 获取攻击能力（购买/借用恶意软件） |
| T1583 | Acquire Infrastructure | 获取基础设施（C2 服务器、域名） |
| T1584 | Compromise Infrastructure | 攻陷基础设施作为跳板 |

### TA0001: Initial Access（初始访问）

| 技术 ID | 技术名称 | 说明 |
|---------|---------|------|
| T1190 | Exploit Public-Facing Application | 利用面向公众应用的漏洞 |
| T1566 | Phishing | 钓鱼攻击（含鱼叉式钓鱼、附件钓鱼） |
| T1078 | Valid Accounts | 使用有效账户登录 |
| T1133 | External Remote Services | 利用外部远程服务（VPN、Citrix） |
| T1189 | Drive-by Compromise | 水坑攻击/路过式下载 |
| T1195 | Supply Chain Compromise | 供应链攻击 |

### TA0002: Execution（执行）

| 技术 ID | 技术名称 | 说明 |
|---------|---------|------|
| T1059 | Command and Scripting Interpreter | 命令与脚本解释器（PowerShell、Bash、Python） |
| T1203 | Exploitation for Client Execution | 客户端漏洞利用 |
| T1204 | User Execution | 用户执行（诱骗打开恶意文件） |
| T1047 | Windows Management Instrumentation | WMI 执行 |
| T1053 | Scheduled Task/Job | 计划任务/作业 |

### TA0003: Persistence（持久化）

| 技术 ID | 技术名称 | 说明 |
|---------|---------|------|
| T1547 | Boot or Logon Autostart Execution | 启动/登录时自动执行（注册表 Run 键） |
| T1543 | Create or Modify System Process | 创建/修改系统服务 |
| T1546 | Event Triggered Execution | 事件触发执行（WMI 事件订阅） |
| T1136 | Create Account | 创建账户 |
| T1505 | Server Software Component | WebShell |
| T1053 | Scheduled Task/Job | 计划任务持久化 |

### TA0004: Privilege Escalation（权限提升）

| 技术 ID | 技术名称 | 说明 |
|---------|---------|------|
| T1068 | Exploitation for Privilege Escalation | 本地漏洞提权 |
| T1134 | Access Token Manipulation | Token 操作（Token Stealing） |
| T1548 | Abuse Elevation Control Mechanism | 滥用提权控制机制（UAC Bypass、Sudo） |
| T1055 | Process Injection | 进程注入 |
| T1078 | Valid Accounts | 使用高权限账户 |

**Linux 常用提权技术**：
- SUID/SGID 文件滥用（`find / -perm -4000 2>/dev/null`）
- Sudo 配置错误（`sudo -l`）
- Cron 任务劫持
- 内核漏洞（DirtyCow、DirtyPipe、OverlayFS）

**Windows 常用提权技术**：
- Token 窃取（Incognito）
- UAC Bypass
- 服务权限配置错误（PowerUp）
- 计划任务滥用
- 内核漏洞（MS16-032、MS17-010、PrintSpoofer）

### TA0005: Defense Evasion（防御规避）

| 技术 ID | 技术名称 | 说明 |
|---------|---------|------|
| T1027 | Obfuscated Files or Information | 文件/代码混淆 |
| T1055 | Process Injection | 进程注入 |
| T1562 | Impair Defenses | 破坏防御（关闭杀软、防火墙） |
| T1070 | Indicator Removal on Host | 清除主机痕迹 |
| T1218 | System Binary Proxy Execution | 系统二进制代理执行（LOLBins） |
| T1036 | Masquerading | 伪装（文件改名、模仿合法进程名） |

### TA0006: Credential Access（凭证访问）

| 技术 ID | 技术名称 | 说明 |
|---------|---------|------|
| T1003 | OS Credential Dumping | 操作系统凭证转储（Mimikatz、SAM dump） |
| T1558 | Steal or Forge Kerberos Tickets | Kerberos 票据攻击（Golden/Silver Ticket） |
| T1110 | Brute Force | 暴力破解密码 |
| T1555 | Credentials from Password Stores | 从密码存储获取凭证（浏览器、DPAPI） |
| T1040 | Network Sniffing | 网络嗅探 |

**Mimikatz 常用命令**：
```
sekurlsa::logonpasswords    # 内存中提取明文密码
sekurlsa::tickets /export    # 导出 Kerberos 票据
lsadump::dcsync /user:DOMAIN\krbtgt  # DCSync 攻击
```

### TA0007: Discovery（发现/侦察）

| 技术 ID | 技术名称 | 说明 |
|---------|---------|------|
| T1082 | System Information Discovery | 系统信息发现 |
| T1083 | File and Directory Discovery | 文件和目录枚举 |
| T1046 | Network Service Scanning | 网络服务扫描 |
| T1018 | Remote System Discovery | 远程系统发现（`net view`） |
| T1069 | Permission Groups Discovery | 权限组发现 |
| T1087 | Account Discovery | 账户发现 |
| T1016 | System Network Configuration Discovery | 网络配置发现 |

### TA0008: Lateral Movement（横向移动）

| 技术 ID | 技术名称 | 说明 |
|---------|---------|------|
| T1021 | Remote Services | 远程服务（RDP、SSH、SMB、WinRM） |
| T1550 | Use Alternate Authentication Material | 使用替代认证材料（Pass-the-Hash/Ticket） |
| T1563 | Remote Service Session Hijacking | 远程会话劫持 |
| T1210 | Exploitation of Remote Services | 利用远程服务漏洞 |
| T1570 | Lateral Tool Transfer | 横向工具传输 |

### TA0009: Collection（收集）

| 技术 ID | 技术名称 | 说明 |
|---------|---------|------|
| T1560 | Archive Collected Data | 压缩收集的数据 |
| T1005 | Data from Local System | 本地系统数据 |
| T1113 | Screen Capture | 屏幕截图 |
| T1056 | Input Capture | 输入捕获（键盘记录） |
| T1115 | Clipboard Data | 剪贴板数据 |

### TA0010: Exfiltration（渗出）

| 技术 ID | 技术名称 | 说明 |
|---------|---------|------|
| T1041 | Exfiltration Over C2 Channel | 通过 C2 通道渗出 |
| T1048 | Exfiltration Over Alternative Protocol | 通过替代协议渗出（DNS、ICMP 隧道） |
| T1567 | Exfiltration Over Web Service | 通过 Web 服务渗出（云存储、Pastebin） |

### TA0040: Impact（影响）

| 技术 ID | 技术名称 | 说明 |
|---------|---------|------|
| T1486 | Data Encrypted for Impact | 勒索软件加密 |
| T1485 | Data Destruction | 数据销毁 |
| T1490 | Inhibit System Recovery | 禁止系统恢复（删除卷影副本） |
| T1491 | Defacement | 网站篡改 |

## 应急报告 ATT&CK 映射示例

```
时间线                       | ATT&CK 技术               | 描述
----------------------------|---------------------------|------------------------------
2024-01-15 09:00:00        | T1190 Exploit Public-Facing App | 攻击者利用 Web RCE 漏洞
2024-01-15 09:02:15        | T1059.001 PowerShell       | 执行 PowerShell 下载 C2 载荷
2024-01-15 09:05:30        | T1055 Process Injection    | 注入 lsass.exe 进程
2024-01-15 09:10:00        | T1003.001 LSASS Memory     | 使用 Mimikatz 提取凭证
2024-01-15 09:25:00        | T1021.002 SMB/Windows Admin Shares | 使用窃取凭证横向移动
2024-01-15 10:00:00        | T1041 Exfiltration Over C2 | 通过 C2 通道外传数据
```

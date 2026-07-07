# 代码审计增强规范

> 被 SKILL.md 🟢 Code Review 引用。进行完整数据流审计时加载。

## 核心原则

**单点危险函数调用 ≠ 漏洞。** 必须通过 Source → Propagation → Sink 完整链路确认可达性与可利用性。

## 三步流程

### Step A — Source 识别

| 类别 | 典型 Source |
|------|------------|
| HTTP 参数 | `request.getParameter()`、`$_GET/$_POST`、`@RequestParam`、`req.body`、`req.query` |
| HTTP 头 | `User-Agent`、`Referer`、`X-Forwarded-For`、`Cookie` |
| 文件上传 | `$_FILES`、`MultipartFile` |
| 数据库读取 | 二次注入：DB 中用户可控数据 |
| 外部 API | 第三方返回值、Webhook |
| 序列化 | `ObjectInputStream.readObject()`、`pickle.loads()`、`unserialize()` |

### Step B — Propagation 追踪

逐层追踪，标注清洗/过滤/编码节点。跨层不跳过（Controller→Service→DAO）。
框架感知（MyBatis `#{}` vs `${}`）。动态调用标注"反射，静态不可全追踪"，降置信度。

```
Source: request.getParameter("keyword")
  → String keyword     (未过滤)
  → service.search()   (传递)
  → dao.query("SELECT..name='" + keyword + "'")  ← Sink: SQL拼接
```

### Step C — Sink 验证

| 漏洞类型 | 典型 Sink |
|---------|----------|
| SQL 注入 | `executeQuery()`、`Statement.execute()`、字符串拼接 SQL |
| 命令注入 | `Runtime.exec()`、`ProcessBuilder`、`system()`、`exec()`、`popen()` |
| XSS | `response.getWriter().write()`、`innerHTML`、`dangerouslySetInnerHTML` |
| SSRF | `HttpURLConnection`、`curl_exec()`、`requests.get()` (URL 可控) |
| 反序列化 | `ObjectInputStream.readObject()`、`pickle.loads()`、`unserialize()` |
| 路径遍历 | `FileInputStream`、`file_get_contents()` (路径可控) |
| 模板注入 | `render()`、`eval()`、模板引擎动态编译 |
| XXE | `DocumentBuilder.parse()` (未禁用外部实体) |

## 全局防御层评估

判定漏洞前**必须**评估以下四层防御。有效阻断→标注并降级。

```
□ 输入层: Filter/Interceptor | WAF | JSR-303 Validator | Unicode规范化
□ 业务层: 参数化查询/#{} | CSRF Token | Spring Security | 类型转换器
□ 输出层: Thymeleaf/Jinja2 autoescape | CSP | JSON序列化转义
□ 运行时: SecurityManager | Docker seccomp/AppArmor | SELinux | OS只读挂载
```

结论格式：
```
防御评估:
  □ 无防御 → 确认可利用
  □ [名称] 可被绕过 (方式) → 仍可利用
  □ [名称] 有效阻断 → 降级为 Info / 驳回
  □ [名称] 配置不当 (遗漏路径) → 标记配置缺陷
```

## 审计输出格式

```markdown
### #[编号] [类型] — [级别]

| 字段 | 内容 |
|------|------|
| Source | [文件:行号] [变量] [来源] |
| 传播 | Source → [节点1] → [节点2] → Sink |
| Sink | [文件:行号] [函数] [类型] |
| 利用条件 | [前置条件] |
| 防御评估 | [见结论格式] |
| CVSS | [分数] [向量] |
| 修复 | [Diff] |
```

## 误报规则

以下**不报告**（或降 Info）：
- Source 不可达 Sink（不可穿透清洗/转义）
- 全局防御有效阻断且无已知绕过
- Sink 参数硬编码/常量/枚举（攻击者不可控）
- ORM 自动参数化且无原生 SQL 拼接
- 接口仅内网可访问（需注明"内网场景可利用"）

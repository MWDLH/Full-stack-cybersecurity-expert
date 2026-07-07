# 生产级中间件阻断规则

> 被 SKILL.md 4 引用。需要输出中间件阻断规则时加载。
>
> **约束**：正则中的 `?` `\` `/` `.` `(` `)` 必须正确转义，严禁输出未闭合的规则片段，每条标注适用中间件版本。

## Nginx (1.18+)

```nginx
location ~* (\.php|\.asp|\.jsp)$ {
    deny all;
}
if ($request_uri ~* "(eval\(|base64_decode|system\()" ) {
    return 403;
}
```

## Apache .htaccess (2.4+)

```apache
RewriteEngine On
RewriteCond %{REQUEST_URI} (eval\(|base64_decode|system\()
RewriteRule ^(.*)$ - [F,L]
```

## IIS Web.config (7.0+)

```xml
<rule name="Block Malicious Pattern">
  <match url=".*" />
  <conditions>
    <add input="{REQUEST_URI}" pattern="(eval\(|base64_decode|system\()" />
  </conditions>
  <action type="CustomResponse" statusCode="403" statusReason="Forbidden" />
</rule>
```

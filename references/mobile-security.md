> 被 SKILL.md "10. On-Demand References" 索引。涉及移动 App 渗透测试、Android/iOS 安全评估、逆向分析时加载。

# 移动安全参考

## OWASP Mobile Top 10 (2024)

> 来源: https://owasp.org/www-project-mobile-top-10/2023-risks/

| 分类 | 典型场景 | 测试检查点 |
|------|---------|-----------|
| M1 — 不当凭证使用 (Improper Credential Usage) | 硬编码 API Key、Token 明文存储、密码缓存 | 检查代码/SharedPreferences/Keychain |
| M2 — 供应链安全不足 (Inadequate Supply Chain Security) | 第三方 SDK 漏洞、恶意依赖库、未验证的构建管线 | 检查依赖清单/SBOM/构建配置 |
| M3 — 不安全的认证/授权 (Insecure Authentication/Authorization) | 无 MFA、Token 永不过期、客户端直接调用 API 无服务端校验 | 暴力测试、Token replay、IDOR 测试 |
| M4 — 输入/输出验证不足 (Insufficient Input/Output Validation) | Deep Link 注入、Intent 劫持、WebView JS 注入 | Fuzzing 测试、Intent 劫持、参数注入 |
| M5 — 不安全通信 (Insecure Communication) | HTTP 明文、SSL 验证跳过、证书钉扎缺失 | 代理抓包验证、SSL Pinning bypass |
| M6 — 隐私控制不足 (Inadequate Privacy Controls) | 过度收集权限、未匿名化日志、剪贴板泄露 | 权限审计、ADB logcat / ASL 分析、运行时监控 |
| M7 — 二进制保护不足 (Insufficient Binary Protections) | 无混淆、无加固、重新打包、动态注入 | 反编译测试、签名检查、完整性验证 |
| M8 — 安全配置错误 (Security Misconfiguration) | 导出 Activity 绕过认证、WebView 远程代码执行、debuggable=true | 检查 AndroidManifest XML/entitlements、权限配置 |
| M9 — 不安全数据存储 (Insecure Data Storage) | SQLite 未加密、日志泄露凭据、Keychain 配置不当 | 文件系统枚举、ADB 读取 `/data/data/[PACKAGE]/` |
| M10 — 加密不足 (Insufficient Cryptography) | 弱算法 (DES/MD5)、硬编码密钥、随机数不安全 | 代码审计加密实现、密钥存储位置 |

## Android 安全

### 常见漏洞
| 类别 | 检查 | 命令 / 工具 |
|------|------|-------------|
| 导出组件 | `android:exported="true"` 未授权访问 | `apktool d app.apk` → 检查 AndroidManifest.xml |
| WebView | `setJavaScriptEnabled(true)` + `addJavascriptInterface` → RCE | Jadx 反编译搜索 |
| Intent 劫持 | 隐式 Intent 发送敏感数据 | `drozer` / Intent Fuzzing |
| Root 检测 | 仅依赖 `Build.TAGS.contains("test-keys")` | Frida bypass |
| SSL Pinning | 自定义 TrustManager 信任所有证书 | `Objection` bypass |
| 数据存储 | SQLite 明文 / SharedPreferences / Internal Storage | ADB 读取 `/data/data/[PACKAGE]/` |
| Deep Link | URL Scheme 注入 / 参数劫持 | `am start -d [SCHEME]://[HOST]/[PATH]` |

### 测试命令
```bash
# 提取安装包
adb shell pm list packages | grep [KEYWORD]
adb shell pm path [PACKAGE_NAME]
adb pull [APK_PATH]

# 反编译
jadx -d output/ app.apk
apktool d app.apk -o output/

# 组件测试
am start -n [PACKAGE]/[ACTIVITY]
am startservice -n [PACKAGE]/[SERVICE]

# Frida 注入
frida -U -l bypass.js -f [PACKAGE] --no-pause
frida-trace -U -j '*!*onClick*' [PACKAGE]

# Objection 运行时操作
objection -g [PACKAGE] explore
android sslpinning disable
android root disable
```

## iOS 安全

### 常见漏洞
| 类别 | 检查 | 工具 |
|------|------|------|
| Keychain | 可被越狱设备读取 | `objection ios keychain dump` |
| 数据保护 | NSFileProtectionNone | 文件系统分析 |
| 剪贴板泄露 | UIPasteboard 包含 Token | 运行时监控 |
| Cookie 存储 | NSHTTPCookieStorage 未加密 | 路径验证 |
| 本地存储 | NSUserDefaults / CoreData / Realm | 提取 .db / .plist |
| URL Scheme | 参数 URL 注入 | frida-trace |

### 测试命令
```bash
# 解压 IPA
unzip app.ipa -d Payload/ && cd Payload/*.app

# 分析二进制
otool -L [BINARY]    # 依赖库分析
nm [BINARY] | grep [KEYWORD]
strings [BINARY] | grep -i "password\|token\|api"

# Frida
frida -U -l hook.js [BINARY]
frida-trace -U -m "-[NSURLRequest initWithURL:]" [BINARY]

# Objection
objection -g [BUNDLE_ID] explore
ios info plist
ios cookies get
```

## 重打包与绕过

### 绕过检查
```bash
# 禁用 root/jailbreak 检测
frida -U --no-pause -l ssl_bypass.js -l root_bypass.js -f [PACKAGE]

# 修改 AndroidManifest 后重打包
apktool d app.apk   # 修改 debuggable=true
apktool b output/ -o mod.apk
keytool -genkey -alias key -keystore keystore.jks
jarsigner -keystore keystore.jks mod.apk key
adb install mod.apk
```

### 加固识别
| 加固方案 | 识别特征 | 绕过难度 |
|---------|---------|---------|
| 360 加固 | libjiagu.so | 高 |
| 腾讯加固 | libshell.so / libsec.so | 高 |
| 梆梆加固 | libDexHelper.so | 高 |
| 阿里加固 | libsgmain.so | 中-高 |
| Obfuscator-LLVM | 控制流平坦化 | 中 |
| ProGuard | 类/方法重命名 | 低 |

## 通用防护建议
1. 服务端强制执行所有安全检查（不要信任客户端输入）
2. 使用 Certificate Pinning + 证书透明度
3. 敏感数据用 Keychain/KeyStore + 生物识别保护
4. 实施 App Attest / SafetyNet / Play Integrity API
5. 代码混淆（ProGuard + R8 + Obfuscator-LLVM）
6. Deep Link 校验（验证 URL 参数签名）
7. 日志在 release 构建中完全移除
8. 定期进行自动化移动安全扫描（MobSF）

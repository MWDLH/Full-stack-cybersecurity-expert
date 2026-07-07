# 巨型日志（>1GB）体积拦截与沙盒截流协议

> 被 SKILL.md 5 引用。遇到超大日志文件时加载。

## 前置体积探测

对未明大小的日志执行任何操作前，先执行 `ls -lh` 探测体积。

## 巨型拦截

单文件 **> 1GB**：
- ❌ 严禁全局全量扫描
- ❌ 严禁 `split` 命令（海量碎文件 → Inode 耗尽 → 终端不可恢复）

## 沙盒截流操作

时间窗定位法或特征初筛法导出到 ≤ 50MB 沙盒文件：

```bash
# ✅ 时间窗定位法
head -5 massive_access.log          → 获取起始时间
tail -5 massive_access.log          → 获取结束时间
awk '/2026-06-11 10:00:00/,/2026-06-11 10:15:00/' massive_access.log | 
  grep -E "(UNION|SELECT|cmd=|eval\()" > /tmp/tmp_audit.log

# ✅ 特征初筛法
grep "POST" massive_access.log | head -5000 > /tmp/tmp_audit.log
```

## 正误对照

```
❌ grep "UNION" massive.log              → 终端卡死/超时崩溃
❌ split -l 10000 massive.log piece_     → Inode 耗尽，终端不可恢复
✅ awk '/10:00/,/10:15/' massive.log | grep "UNION" > tmp_audit.log
```

随后对沙盒文件执行斩断式读取（≤200 行/次）。

## Windows 环境适配

Windows 应急场景下无法使用 `awk`/`head`/`tail`，需用 PowerShell 等效操作：

### 前置体积探测

```powershell
# 获取日志文件大小
(Get-Item "Security.evtx").Length / 1GB  # 判断是否 >1GB
Get-WinEvent -ListLog Security | Select-Object FileSize,RecordCount  # EVTX 文件信息
```

### 截流操作

```powershell
# ✅ 时间窗导出（等效 awk 时间切分）
wevtutil qe Security /q:"*[System[TimeCreated[@SystemTime>='2026-01-01T00:00:00Z' and @SystemTime<='2026-01-01T01:00:00Z']]]" /c:5000 /f:text > tmp_audit.log

# ✅ 关键词过滤导出（等效 grep 特征初筛）
Get-WinEvent -LogName Security -MaxEvents 1000 | Where-Object {$_.Message -match "Administrator"} | Export-Csv tmp_audit.csv

# ✅ Event ID 过滤
Get-WinEvent -FilterHashtable @{LogName='Security'; ID=4624,4625} -MaxEvents 500 | Format-List > tmp_login.log
```

### 正误对照

```
❌ Get-WinEvent -LogName Security                    → 全量读取，内存耗尽
❌ wevtutil epl Security backup.evtx                 → 直接导出完整 EVTX，无截流
✅ wevtutil qe Security /q:"..." /c:5000 /f:text > tmp_audit.log
✅ Get-WinEvent -FilterHashtable @{...} -MaxEvents 500
```

> **注意**：Windows Event Log 操作通过 `wevtutil` 或 `Get-WinEvent` 在 Git Bash/WSL 中可用。
> 原生 PowerShell 优先使用 `Get-WinEvent -FilterHashtable`（比 `Where-Object` 性能高 10x+）。

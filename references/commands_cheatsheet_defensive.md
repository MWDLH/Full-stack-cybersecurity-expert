> 被 SKILL.md "10. On-Demand References" 索引。进入 Blue Team IR 模式或需要应急响应命令参考时加载。

# 应急响应常用命令速查

## Linux 应急响应

```bash
# 系统信息
uname -a && cat /etc/os-release && uptime

# 用户与登录
who && w && last -n 20 && lastb -n 20
cat /etc/passwd | grep -v nologin | grep -v false
cat /etc/shadow | grep -vE '!|\*'

# 进程分析
ps auxf --sort=-%cpu
ps aux --sort=-%mem
ls -la /proc/*/exe 2>/dev/null | grep deleted  # 检测隐藏进程

# 网络连接
netstat -tlnp
netstat -anop
ss -tlnp
lsof -i

# 计划任务
crontab -l
cat /etc/crontab
ls -la /etc/cron.*/
ls -la /var/spool/cron/

# 启动项
systemctl list-unit-files --type=service | grep enabled
ls -la /etc/systemd/system/
cat /etc/rc.local

# 文件变更
find / -mtime -3 -type f 2>/dev/null  # 近 3 天修改的文件
find / -name "*.php" -newer /etc/passwd  # 比 passwd 更新的 PHP 文件（可疑 Webshell）
find / -name "*.jsp" -mtime -7 2>/dev/null

# SSH 后门
cat /root/.ssh/authorized_keys
cat /home/*/.ssh/authorized_keys
```

## Windows 应急响应

```powershell
# 系统信息
systeminfo
Get-WmiObject Win32_OperatingSystem

# 用户与登录
net user
net localgroup administrators
qwinsta /server:localhost

# 进程分析
tasklist /svc
Get-Process | Select-Object Name, Id, Path
wmic process list full

# 服务
sc query state= all
Get-Service | Where-Object {$_.Status -eq "Running"}

# 网络连接
netstat -ano | findstr ESTABLISHED
netstat -ano | findstr LISTENING

# 计划任务
schtasks /query /fo LIST /v

# 启动项
reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
reg query HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
Get-CimInstance Win32_StartupCommand | Select-Object Name, command, Location, User

# 事件日志
Get-WinEvent -LogName Security -MaxEvents 100 | Where-Object {$_.Id -eq 4624 -or $_.Id -eq 4625}
wevtutil qe Security /c:100 /f:text /rd:true
```

## Webshell 排查

```bash
# 查找可疑 PHP 文件
find /var/www -name "*.php" -exec grep -lE "eval|base64_decode|system|exec|shell_exec|passthru|popen|proc_open|assert" {} \;

# 查找近 7 天创建的 Web 文件
find /var/www -name "*.php" -mtime -7 -type f

# 检查 .htaccess 篡改
find /var/www -name ".htaccess" -exec cat {} \;
```

## Windows Event Log 应急分析

```powershell
# 导出指定时间范围的安全日志
wevtutil qe Security /q:"*[System[TimeCreated[@SystemTime>='2026-01-01T00:00:00']]]" /c:5000 /f:text > tmp_security.log

# 高频事件 ID 快速统计
Get-WinEvent -LogName Security -MaxEvents 500 | Group-Object Id | Sort-Object Count -Descending

# 特定事件 ID 过滤（4624=成功登录, 4625=失败登录, 4672=特权分配, 4768=Kerberos TGT, 4776=凭据验证）
Get-WinEvent -FilterHashtable @{LogName='Security'; ID=4624,4625,4672} -MaxEvents 100 | Format-Table TimeCreated,Id,Message -AutoSize

# 服务创建事件 (7045)
Get-WinEvent -LogName System -MaxEvents 200 | Where-Object {$_.Id -eq 7045}

# 计划任务注册 (4698)
Get-WinEvent -LogName Security -MaxEvents 200 | Where-Object {$_.Id -eq 4698}
```

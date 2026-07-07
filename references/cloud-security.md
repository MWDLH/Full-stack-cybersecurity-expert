> 被 SKILL.md "10. On-Demand References" 索引。涉及云环境渗透、容器安全、云配置审计时加载。

# 云安全参考

## 云元数据服务（SSRF 目标）

| 平台 | 元数据端点 | 说明 |
|------|-----------|------|
| **AWS** | `http://169.254.169.254/latest/meta-data/` | IAM 凭证在 `iam/security-credentials/` |
| **GCP** | `http://metadata.google.internal/computeMetadata/v1/` | 需 Header `Metadata-Flavor: Google` |
| **Azure** | `http://169.254.169.254/metadata/instance?api-version=2021-02-01` | 需 Header `Metadata: true` |

- **利用**：SSRF 注入 → 读取元数据 → 提取云服务临时凭证
- **防护**：网络层阻断 169.254.169.254 访问；限制 IMDSv2 (AWS) 为会话方式；使用防火墙规则限制元数据服务

## AWS 安全

### IAM 权限提升
| 权限 | 攻击效果 |
|------|---------|
| `iam:PassRole` + `ec2:RunInstances` | 创建绑定高权限角色的 EC2 实例 |
| `iam:CreateAccessKey` | 为任意用户创建 Access Key |
| `iam:CreateLoginProfile` | 为任意用户创建登录密码 |
| `iam:UpdateAssumeRolePolicy` | 修改角色信任策略实现提权 |
| `lambda:CreateFunction` + `lambda:InvokeFunction` | 创建并执行带高权限的函数 |

### S3
- **常见问题**：公开读写桶、未启用加密、无版本控制
- **命令**：`aws s3 ls s3://[BUCKET] --no-sign-request`
- **检测**：CloudTrail `ListBuckets`/`GetObject` 异常来源 IP
- **防护**：默认阻止公开访问；启用 S3 Block Public Access；启用 S3 服务器访问日志

### Lambda / Serverless
- **注入**：事件数据注入 → 访问内部服务
- **提权**：执行角色权限过大 → 可操作其他服务
- **防护**：最小化执行角色；使用 Parameter Store/Secrets Manager 管理凭据

### EKS / ECS
- **容器逃逸**：特权容器被赋予 host network 访问
- **命令**：`aws eks update-kubeconfig --name [CLUSTER]`
- **防护**：限制 Pod 权限；使用 OIDC 认证；Pod Security Standards

## GCP 安全

- **IAM 探查**：`gcloud projects get-iam-policy [PROJECT]`
- **GCS 开放桶**：`gsutil ls gs://[BUCKET]`
- **GKE 攻击**：默认服务账号 `[PROJECT_NUMBER]-compute@developer.gserviceaccount.com` 通常权限过大
- **防护**：最小化服务账号权限；启用 VPC Service Controls；使用 Workload Identity

## Azure 安全

### Azure AD (Entra ID)
| 攻击 | 描述 | 检测 |
|------|------|------|
| 应用权限提升 | 高权限 OAuth 应用授予 | 异常 OAuth 许可事件 |
| 设备注册绕过 | 未受管理设备接入 | 条件访问策略检查 |
| PIM 滥用 | 特权角色激活频率异常 | 日志审计 |

### 存储账户
- **命令**：`az storage blob list --account-name [ACCT] --container [CTN]`
- **防护**：限制网络访问；启用软删除；使用托管标识替代连接字符串

### AKS
- **命令**：`az aks get-credentials --resource-group [RG] --name [CLUSTER]`
- **防护**：启用 Azure AD RBAC；限制 `cluster-admin`；使用 Pod Identity

## 容器/K8s 安全

### 容器逃逸
| 手法 | 条件 | 命令 |
|------|------|------|
| 宿主机挂载 | `/var/run/docker.sock` 挂载到容器 | `docker -H unix:///var/run/docker.sock run -it --privileged ubuntu` |
| `--privileged` 模式 | 特权容器 | `mount /dev/sda1 /mnt && chroot /mnt` |
| Capabilities | SYS_ADMIN, SYS_PTRACE | 可使用 `ptrace` 注入、`nsenter` 逃逸 |
| HostPID | 共享宿主机 PID 命名空间 | `nsenter --target 1 --mount --uts --ipc --pid -- sh` |

### K8s RBAC 滥用
| 权限 | 效果 |
|------|------|
| `create pods/exec` | 可在任意 Pod 中执行命令 |
| `list secrets` | 可读取所有 Secret |
| `create pods` + `delete pods` | 可部署恶意 Pod 并删除 |
| `cluster-admin` | 完全控制整个集群 |

### 检测命令
```bash
# 检查不安全的容器运行参数
kubectl get pods --all-namespaces -o json | jq '.items[] | select(.spec.containers[].securityContext.privileged==true)'
# 检查绑定到宿主机服务的 Pod
kubectl get pods --all-namespaces -o json | jq '.items[] | select(.spec.hostPID==true or .spec.hostNetwork==true)'
# 检查过多的 RBAC 权限
kubectl describe clusterrolebinding [NAME]
```

## 云配置审计（CIS Benchmarks）

| 层面 | 关键检查项 |
|------|-----------|
| IAM | 禁用根用户 Access Key；启用多因素认证；最小权限原则 |
| 日志 | 启用 CloudTrail / Audit Logs；日志保留 ≥365 天 |
| 网络 | 默认 VPC 无对等连接；安全组最小开放原则 |
| 存储 | 禁用公开读写；启用加密；启用版本控制 |
| 计算 | 限制公共 AMI；启用安全补丁自动更新 |

## 通用防护原则
1. 最小权限 IAM 策略
2. 多因素认证（MFA）全启用
3. 云日志集中到 SIEM
4. 基础设施即代码（IaC）安全扫描
5. 定期云配置审计（CIS Benchmark tools: Prowler/AWS, ScoutSuite/多云, Azucar/Azure）
6. 使用 Service Control Policies / Organization Policies 设置边界

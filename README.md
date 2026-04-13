# Prisma Access Skills for Claude Code

A set of Claude Code skills for managing Palo Alto Networks Prisma Access configurations via Strata Cloud Manager (SCM).

[English](#english) | [中文](#中文)

---

## English

### Skills Included

| Skill | Description |
|-------|-------------|
| `prisma-config` | Generate SCM-compatible configurations (security policies, NAT rules, decryption, URL filtering, GlobalProtect, etc.) |
| `prisma-audit` | Audit configurations against PAN-OS best practices, CIS benchmarks, and Zero Trust principles |
| `prisma-migrate` | Migrate configurations between Prisma Access tenants (TSGs) with real-world compatibility matrix |
| `prisma-troubleshoot` | Diagnose GlobalProtect connectivity, policy matching, tunnel, and API issues |
| `prisma-api` | Execute SCM API operations (authenticate, CRUD resources, push config) |
| `prisma-access` | All-in-one skill combining all of the above |

### Installation

#### Claude Code Plugin (recommended)

Add the marketplace and install the plugin:

```
/plugin marketplace add leesandao/prismaaccess-skill
/plugin install prisma-access@prisma-access-tools
```

Or test locally:

```bash
claude --plugin-dir /path/to/prismaaccess-skill
```

#### ClawHub

Install all-in-one:

```bash
clawhub install prisma-access
```

Or install individually:

```bash
clawhub install prisma-config
clawhub install prisma-audit
clawhub install prisma-migrate
clawhub install prisma-troubleshoot
clawhub install prisma-api
```

### Prerequisites

For skills that interact with the SCM API (`prisma-api`, `prisma-migrate`, `prisma-troubleshoot`), set these environment variables:

```bash
export SCM_CLIENT_ID="your-client-id"
export SCM_CLIENT_SECRET="your-client-secret"
export SCM_TSG_ID="your-tsg-id"
```

You also need `curl` and `jq` installed.

### Usage

#### Generate a security policy

```
/prisma-access:prisma-config security-policy for blocking social media apps
```

#### Audit existing configuration

```
/prisma-access:prisma-audit security-policies
```

#### Migrate between tenants

```
/prisma-access:prisma-migrate 1234567890 0987654321
```

#### Troubleshoot an issue

```
/prisma-access:prisma-troubleshoot GlobalProtect users cannot connect
```

#### Query SCM API

```
/prisma-access:prisma-api list security-rules
```

### Tenant Migration Compatibility (v1.1)

Based on real-world migration testing between Prisma Access tenants:

#### Directly migratable via API

Tags, Address Objects/Groups, Services/Groups, Application Filters/Groups, EDLs, HIP Objects/Profiles, File Blocking Profiles, Profile Groups, Security Rules (most), NAT Rules, Decryption Rules (most)

#### Requires manual handling

| Resource | Issue |
|----------|-------|
| URL Filtering / Data Filtering / AI Security Profiles | Service Account may lack API read permissions |
| Custom URL Categories | API may return Access denied |
| Profile Groups with inaccessible sub-profiles | Import with invalid refs stripped, fix manually later |
| Rules referencing missing objects | Create dependencies first, then retry |
| Cross-folder name conflicts | Skip — typically system presets already in target |

See the [prisma-migrate skill](skills/prisma-migrate/SKILL.md) for the full compatibility matrix and detailed workflow.

---

## 中文

### 包含的技能

| 技能 | 说明 |
|------|------|
| `prisma-config` | 生成 SCM 兼容配置（安全策略、NAT 规则、解密策略、URL 过滤、GlobalProtect 等） |
| `prisma-audit` | 根据 PAN-OS 最佳实践、CIS 基准和零信任原则审计配置 |
| `prisma-migrate` | 在 Prisma Access 租户 (TSG) 之间迁移配置，包含实测兼容性矩阵 |
| `prisma-troubleshoot` | 诊断 GlobalProtect 连接、策略匹配、隧道和 API 问题 |
| `prisma-api` | 执行 SCM API 操作（认证、增删改查资源、推送配置） |
| `prisma-access` | 以上所有功能的一站式合集 |

### 安装方式

#### Claude Code 插件（推荐）

添加 marketplace 并安装插件：

```
/plugin marketplace add leesandao/prismaaccess-skill
/plugin install prisma-access@prisma-access-tools
```

或本地测试：

```bash
claude --plugin-dir /path/to/prismaaccess-skill
```

#### ClawHub

一次安装全部：

```bash
clawhub install prisma-access
```

或按需单独安装：

```bash
clawhub install prisma-config
clawhub install prisma-audit
clawhub install prisma-migrate
clawhub install prisma-troubleshoot
clawhub install prisma-api
```

### 前置要求

使用与 SCM API 交互的技能（`prisma-api`、`prisma-migrate`、`prisma-troubleshoot`）前，需设置以下环境变量：

```bash
export SCM_CLIENT_ID="你的客户端ID"
export SCM_CLIENT_SECRET="你的客户端密钥"
export SCM_TSG_ID="你的租户服务组ID"
```

同时需要安装 `curl` 和 `jq`。

### 使用示例

#### 生成安全策略

```
/prisma-access:prisma-config security-policy for blocking social media apps
```

#### 审计现有配置

```
/prisma-access:prisma-audit security-policies
```

#### 租户间迁移

```
/prisma-access:prisma-migrate 1234567890 0987654321
```

#### 故障排查

```
/prisma-access:prisma-troubleshoot GlobalProtect users cannot connect
```

#### 查询 SCM API

```
/prisma-access:prisma-api list security-rules
```

### 租户迁移兼容性说明 (v1.1)

基于 Prisma Access 租户间的实际迁移测试结果：

#### 可直接通过 API 迁移

Tags、地址对象/组、服务/组、应用过滤器/组、EDL、HIP 对象/配置、文件拦截配置、配置组、安全规则（大部分）、NAT 规则、解密规则（大部分）

#### 需要手动处理

| 资源 | 问题 |
|------|------|
| URL 过滤 / 数据过滤 / AI 安全配置 | Service Account 可能缺少 API 读取权限 |
| 自定义 URL 类别 | API 可能返回 Access denied |
| 引用不可用子配置的配置组 | 剥离无效引用后导入，之后手动补齐 |
| 引用缺失对象的规则 | 先创建依赖对象，再重试导入规则 |
| 跨 folder 同名冲突 | 跳过 — 通常是目标租户中已有的系统预置对象 |

完整的兼容性矩阵和详细工作流程请查看 [prisma-migrate skill](skills/prisma-migrate/SKILL.md)。

---

## License

MIT-0

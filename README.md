# Prisma Access Skills for Claude Code

A set of Claude Code skills for managing Palo Alto Networks Prisma Access configurations via Strata Cloud Manager (SCM).

## Skills Included

| Skill | Description |
|-------|-------------|
| `prisma-config` | Generate SCM-compatible configurations (security policies, NAT rules, decryption, URL filtering, GlobalProtect, etc.) |
| `prisma-audit` | Audit configurations against PAN-OS best practices, CIS benchmarks, and Zero Trust principles |
| `prisma-migrate` | Migrate configurations between Prisma Access tenants (TSGs) |
| `prisma-troubleshoot` | Diagnose GlobalProtect connectivity, policy matching, tunnel, and API issues |
| `prisma-api` | Execute SCM API operations (authenticate, CRUD resources, push config) |

## Installation

### Claude Code Plugin (recommended)

Add the marketplace and install the plugin:

```
/plugin marketplace add rali/prismaaccess-skill
/plugin install prisma-access@prisma-access-tools
```

Or test locally:

```bash
claude --plugin-dir /path/to/prismaaccess-skill
```

### ClawHub

```bash
clawhub install prisma-config
clawhub install prisma-audit
clawhub install prisma-migrate
clawhub install prisma-troubleshoot
clawhub install prisma-api
```

## Prerequisites

For skills that interact with the SCM API (`prisma-api`, `prisma-migrate`, `prisma-troubleshoot`), set these environment variables:

```bash
export SCM_CLIENT_ID="your-client-id"
export SCM_CLIENT_SECRET="your-client-secret"
export SCM_TSG_ID="your-tsg-id"
```

You also need `curl` and `jq` installed.

## Usage

### Generate a security policy

```
/prisma-access:prisma-config security-policy for blocking social media apps
```

### Audit existing configuration

```
/prisma-access:prisma-audit security-policies
```

### Migrate between tenants

```
/prisma-access:prisma-migrate 1234567890 0987654321
```

### Troubleshoot an issue

```
/prisma-access:prisma-troubleshoot GlobalProtect users cannot connect
```

### Query SCM API

```
/prisma-access:prisma-api list security-rules
```

## License

MIT-0

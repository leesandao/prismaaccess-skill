---
name: prisma-migrate
description: Migrate Prisma Access configurations between different SCM tenants (TSGs). Use when moving security policies, NAT rules, address objects, and other configurations from one Prisma Access tenant to another.
argument-hint: "[source-tsg] [target-tsg]"
disable-model-invocation: true
version: 1.0.0
metadata:
  openclaw:
    requires:
      env:
        - SCM_CLIENT_ID
        - SCM_CLIENT_SECRET
      bins:
        - curl
    primaryEnv: SCM_CLIENT_ID
    emoji: "\U0001F500"
    homepage: https://github.com/rali/prismaaccess-skill
---

# Prisma Access Tenant-to-Tenant Configuration Migration

Migrate configurations between Prisma Access tenants (TSGs) via the Strata Cloud Manager API.

## Overview

This skill helps you export configurations from a source tenant and import them into a target tenant. It handles naming conflicts, reference resolution, and dependency ordering.

For detailed migration workflow steps, see [migration-workflow.md](reference/migration-workflow.md).

## Prerequisites

Set the following environment variables:

```bash
# Source tenant credentials
export SRC_SCM_CLIENT_ID="source-client-id"
export SRC_SCM_CLIENT_SECRET="source-client-secret"
export SRC_SCM_TSG_ID="source-tsg-id"

# Target tenant credentials
export DST_SCM_CLIENT_ID="target-client-id"
export DST_SCM_CLIENT_SECRET="target-client-secret"
export DST_SCM_TSG_ID="target-tsg-id"
```

## Migration Scope

The following object types can be migrated between tenants:

### Objects (migrate first — these are dependencies)
1. **Tags** — custom tags used by rules and objects
2. **Address objects** — IP netmask, FQDN, IP range, IP wildcard
3. **Address groups** — static and dynamic groups referencing address objects
4. **Service objects** — custom protocol/port definitions
5. **Service groups** — groups referencing service objects
6. **Application filters** — custom application filters
7. **Application groups** — groups of applications
8. **External dynamic lists (EDL)** — IP, URL, and domain EDLs
9. **Custom URL categories** — user-defined URL categories

### Profiles (migrate second)
10. **Security profiles** — antivirus, anti-spyware, vulnerability, URL filtering, file blocking, wildfire
11. **Security profile groups** — named groups of security profiles
12. **Decryption profiles** — SSL decryption settings
13. **Log forwarding profiles** — logging destinations
14. **HIP objects and HIP profiles** — host information profiles

### Policies (migrate last — depend on objects and profiles)
15. **Security policy rules** — pre-rules and post-rules
16. **NAT policy rules** — source and destination NAT
17. **Decryption policy rules** — SSL inspection rules
18. **Authentication policy rules** — MFA and auth enforcement

## Migration Workflow

### Step 1: Export from Source Tenant

Authenticate and export all configuration objects from the source tenant via SCM API:

```
GET https://api.sase.paloaltonetworks.com/sse/config/v1/{resource}?folder={folder}
```

Export objects in dependency order (tags → addresses → groups → profiles → policies).

### Step 2: Conflict Detection

Before importing, check the target tenant for:
- **Name conflicts**: objects with the same name but different definitions
- **Reference conflicts**: objects that reference other objects not present in the target
- **Zone mismatches**: rules referencing zones that don't exist in the target tenant

For each conflict, present the user with options:
- **Skip**: do not import the conflicting object
- **Overwrite**: replace the target object with the source object
- **Rename**: import with a prefix/suffix (e.g., `migrated-` prefix)

### Step 3: Transform and Import

For each object:
1. Remove source-tenant-specific fields (`id`, `created`, `last_modified`)
2. Update the `folder` parameter to match the target tenant structure
3. Resolve any renamed references from the conflict resolution step
4. POST to the target tenant API

```
POST https://api.sase.paloaltonetworks.com/sse/config/v1/{resource}?folder={folder}
```

### Step 4: Validation

After import:
1. List all imported objects and verify counts match
2. Check for broken references
3. Run a candidate config push to validate (without committing)

```
POST https://api.sase.paloaltonetworks.com/sse/config/v1/config-versions/candidate:push
```

### Step 5: Commit (User-Confirmed)

Only commit after user explicitly confirms:

```
POST https://api.sase.paloaltonetworks.com/sse/config/v1/config-versions/running:push
```

## Usage Examples

```
/prisma-access:prisma-migrate
```
Interactive mode: prompts for source and target tenant details.

```
/prisma-access:prisma-migrate 1234567890 0987654321
```
Migrate from TSG 1234567890 to TSG 0987654321.

## Safety Guardrails

- **Dry-run by default**: always show what would be imported before making changes
- **No auto-commit**: never commit configuration without explicit user confirmation
- **Rollback guidance**: provide instructions to undo changes if needed
- **Rate limiting**: respect SCM API rate limits (avoid bulk API flooding)

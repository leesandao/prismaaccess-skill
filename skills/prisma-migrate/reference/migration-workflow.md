# Tenant-to-Tenant Migration Workflow Reference

## SCM API Authentication

Each tenant requires its own OAuth2 token:

```bash
curl -s -X POST "https://auth.apps.paloaltonetworks.com/am/oauth2/access_token" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=client_credentials" \
  -d "client_id=${SCM_CLIENT_ID}" \
  -d "client_secret=${SCM_CLIENT_SECRET}" \
  -d "scope=tsg_id:${SCM_TSG_ID}"
```

Response provides a Bearer token valid for ~15 minutes.

## Export API Calls by Resource Type

All exports use:
```
GET https://api.sase.paloaltonetworks.com/sse/config/v1/{resource}?folder={folder}&limit=200
```

Handle pagination with `offset` parameter when total exceeds limit.

### Dependency-Ordered Export Sequence

| Order | Resource              | API Path                    |
|-------|-----------------------|-----------------------------|
| 1     | Tags                  | `/sse/config/v1/tags`       |
| 2     | Addresses             | `/sse/config/v1/addresses`  |
| 3     | Address Groups        | `/sse/config/v1/address-groups` |
| 4     | Services              | `/sse/config/v1/services`   |
| 5     | Service Groups        | `/sse/config/v1/service-groups` |
| 6     | Application Filters   | `/sse/config/v1/application-filters` |
| 7     | Application Groups    | `/sse/config/v1/application-groups` |
| 8     | External Dynamic Lists| `/sse/config/v1/external-dynamic-lists` |
| 9     | Custom URL Categories | `/sse/config/v1/custom-url-categories` |
| 10    | URL Filtering Profiles| `/sse/config/v1/url-filtering-profiles` |
| 11    | Antivirus Profiles    | `/sse/config/v1/anti-virus-profiles` |
| 12    | Anti-Spyware Profiles | `/sse/config/v1/anti-spyware-profiles` |
| 13    | Vulnerability Profiles| `/sse/config/v1/vulnerability-protection-profiles` |
| 14    | File Blocking Profiles| `/sse/config/v1/file-blocking-profiles` |
| 15    | Wildfire Profiles     | `/sse/config/v1/wildfire-anti-virus-profiles` |
| 16    | Profile Groups        | `/sse/config/v1/profile-groups` |
| 17    | Log Forwarding Profiles| `/sse/config/v1/log-forwarding-profiles` |
| 18    | Decryption Profiles   | `/sse/config/v1/decryption-profiles` |
| 19    | HIP Objects           | `/sse/config/v1/hip-objects` |
| 20    | HIP Profiles          | `/sse/config/v1/hip-profiles` |
| 21    | Security Rules (Pre)  | `/sse/config/v1/security-rules?position=pre` |
| 22    | Security Rules (Post) | `/sse/config/v1/security-rules?position=post` |
| 23    | NAT Rules             | `/sse/config/v1/nat-rules`  |
| 24    | Decryption Rules      | `/sse/config/v1/decryption-rules` |

### Folder Values

Common SCM folder values:
- `"Prisma Access"` — shared configuration
- `"Mobile Users"` — GlobalProtect mobile user settings
- `"Remote Networks"` — remote network site settings
- `"Service Connections"` — service connection settings
- `"Mobile Users Container"` — mobile user container
- `"Remote Networks Container"` — remote network container

## Import API Calls

All imports use:
```
POST https://api.sase.paloaltonetworks.com/sse/config/v1/{resource}?folder={folder}
Content-Type: application/json
Authorization: Bearer {token}

{object-json-body}
```

### Fields to Strip Before Import

Remove these server-generated fields from exported objects before importing:
- `id`
- `created`
- `last_modified`
- `snippet` (if present)

### Error Handling

| HTTP Code | Meaning | Action |
|-----------|---------|--------|
| 400       | Invalid object | Check field values and references |
| 409       | Object already exists | Offer skip/overwrite/rename options |
| 429       | Rate limited | Wait and retry with exponential backoff |
| 401/403   | Auth error | Refresh OAuth2 token |

## Candidate Config Validation

After importing, validate by pushing candidate config without committing:

```bash
curl -s -X POST "https://api.sase.paloaltonetworks.com/sse/config/v1/config-versions/candidate:push" \
  -H "Authorization: Bearer ${TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{"folders": ["Prisma Access"]}'
```

Check push job status:
```
GET https://api.sase.paloaltonetworks.com/sse/config/v1/jobs/{job-id}
```

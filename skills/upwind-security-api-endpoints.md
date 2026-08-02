# Upwind API Endpoints Reference

> **Full OpenAPI specs** (fetch via `WebFetch` for detailed schemas, parameters, and examples):
> - V1: https://docs.upwind.io/restapi/management/v1/openapi.yaml
> - V2: https://docs.upwind.io/restapi/management/v2/openapi.yaml

## V1 API Endpoints

Base path: `/v1/organizations/$UPWIND_ORG_ID/...`

Prefer V2 when available. V1 is in maintenance mode.

| Tag | Endpoint | Method | Summary | Key Query Parameters |
|-----|----------|--------|---------|----------------------|
| **threats** | `/threat-detections` | GET | List detections | `severity`, `type`, `category`, `upwind-asset-id`, `min-first-seen-time`, `max-first-seen-time` |
| | `/threat-detections/{id}` | GET | Get a detection | |
| | `/threat-detections/{id}` | PATCH | Update a detection | |
| | `/threat-events` | GET | List events | `cloud-account-id`, `category`, `severity` |
| | `/threat-policies` | GET | List policies | `managed-by` |
| | `/threat-policies/{id}` | PATCH | Update a policy | |
| **vulnerabilities** | `/vulnerability-findings` | GET | List findings | `severity`, `cloud-account-id`, `in-use`, `exploitable`, `fix-available`, `cve-id`, `image-name` |
| | `/vulnerability-findings/{id}` | GET | Get a finding | |
| **configurations** | `/configuration-findings` | GET | List findings | `status`, `severity`, `resource-name`, `check-id`, `framework-id` |
| | `/configuration-findings/{id}` | GET | Get a finding | `include-cloud-account-tags` |
| | `/configuration-frameworks` | GET | List frameworks | `framework-ids`, `cloud-providers`, `status` |
| | `/configuration-frameworks` | POST | Create a framework | |
| | `/configuration-frameworks/{id}` | PATCH | Update a framework | |
| | `/configuration-rules` | GET | List rules | `framework`, `name`, `has-findings` |
| | `/configuration-rules` | POST | Create a rule | |
| | `/configuration-rules/{id}` | PATCH | Update a rule | |
| | `/configuration-rules/{id}` | DELETE | Delete a rule | |
| | `/configuration-reevaluations` | POST | Trigger a reevaluation scan | *(not in published spec)* |
| | `/configuration-reevaluations/{id}` | GET | Get reevaluation status | *(not in published spec)* |
| **cloud-accounts** | `/cloud-accounts` | POST | Create cloud account | |
| | `/cloud-accounts/{id}` | PATCH | Update cloud account | |
| | `/cloud-accounts/{id}` | DELETE | Delete cloud account | |
| **workflows** | `/workflows` | GET | List workflows | |
| | `/workflows` | POST | Create a workflow | |
| | `/workflows/{id}` | GET | Get a workflow | |
| | `/workflows/{id}` | PATCH | Update a workflow | |
| | `/workflows/{id}` | DELETE | Delete a workflow | |
| **integrations** | `/integration-webhooks` | GET | List webhooks | `vendor` |
| | `/integration-webhooks` | POST | Create a webhook | |
| | `/integration-webhooks/{id}` | PATCH | Update a webhook | |
| | `/integration-webhooks/{id}` | DELETE | Delete a webhook | |
| **events** | `/events` | POST | Create an event | |
| | `/events/shift-left` | POST | Create a ShiftLeft event | *(not in published spec)* |
| | `/events/shift-left/search` | POST | Search ShiftLeft events | |
| **api-security** | `/apisecurity-endpoints` | GET | List API endpoints | `method`, `authentication-state`, `has-internet-ingress`, `has-vulnerability`, `has-sensitive-data`, `cloud-account-id`, `domain`, `cluster-id`, `namespace` |
| **packages** | `/sbom-packages` | GET | List SBOM packages | `cloud-account-id`, `framework`, `image-name`, `package-name`, `package-manager`, `package-license` |
| | `/sbom-packages/{name}/{version}` | GET | Get package details | |
| **shiftleft** | `/shiftleft/results/{fingerprint}` | GET | Get scan result | |

### V1 Pagination

Two patterns (prefer token-based):

**Token-based (preferred):**
- `per-page` (default: 100) - items per page
- `page-token` - opaque continuation token
- Response includes `Link` header with `rel="next"`, `rel="prev"`, etc.

**Page-based (deprecated):**
- `page` - page number
- `per-page` - items per page

### V1 Filtering

Endpoints accept query parameters for filtering (kebab-case names):
```
?severity=CRITICAL&min-first-seen-time=2024-01-01T00:00:00Z
```

## V2 API Endpoints

Base path: `/v2/organizations/$UPWIND_ORG_ID/...`

| Tag | Endpoint | Method | Summary | Key Query Parameters |
|-----|----------|--------|---------|----------------------|
| **inventory** | `/inventory/schema` | GET | List asset schemas | |
| | `/inventory/schema/{label}` | GET | Get asset schema | |
| | `/inventory/assets/search` | POST | Search inventory graph assets | *(not in published spec)* |
| | `/inventory/catalog/assets/search` | POST | Search catalog assets | |
| | `/inventory/catalog/assets/{id}` | GET | Get a catalog asset | |
| **threats** | `/threats/stories` | GET | List stories | `sort` |
| | `/threats/stories/search` | POST | Search stories | `sort` |
| | `/threats/stories/{id}` | GET | Get a story | |
| **events** | `/events/shift-left/search` | POST | Search ShiftLeft events | `sort` |
| **access-management** | `/access-management/scopes` | POST | Create a scope | |
| | `/access-management/scopes/{id}` | GET | Get a scope | |
| | `/access-management/scopes/{id}` | PATCH | Update a scope | |
| | `/access-management/scopes/{id}` | DELETE | Delete a scope | |
| | `/access-management/roles/{id}` | GET | Get a role | |
| | `/access-management/groups/{id}` | GET | Get a group | |
| | `/access-management/groups/{id}` | PATCH | Update a group | |

### V2 Pagination (Cursor-Based)

Query parameters:
- `limit` (default: 50, max: 100; catalog assets: default 20, max 200) - items per page
- `cursor` - opaque cursor string, omit for first page

Response includes `metadata`:
```json
{
  "items": [...],
  "metadata": {
    "limit": 50,
    "next_cursor": "eyJvZmZzZXQiOjIwfQ",
    "previous_cursor": null
  }
}
```

Loop until `next_cursor` is absent/null.

### V2 Sorting

Query parameter: `sort=field:direction`
- `sort=update_time:desc`
- Multiple: `sort=field1,field2:desc`
- Default direction: `asc`

### V2 Search / Filtering

POST to `*/search` endpoints with a JSON body:

```json
{
  "conditions": [
    {
      "field": "severity",
      "operator": "eq",
      "value": ["CRITICAL"]
    },
    {
      "field": "create_time",
      "operator": "gte",
      "value": ["2024-01-01T00:00:00Z"]
    }
  ]
}
```

**Operators:** `eq`, `in`, `gte`, `lte`, `gt`, `lt`, `exists`, `contains`, `like`

Available fields vary by endpoint. Use the inventory schema endpoint to discover searchable fields for inventory assets.

### V2 Response Wrappers

All V2 responses wrap data in `items` array:
```json
{ "items": [...] }                          // non-paginated
{ "items": [...], "metadata": { ... } }     // paginated
```

## Common Enum Values

| Field | Values |
|-------|--------|
| **severity** (V1 vuln) | `low`, `medium`, `high`, `critical`, `unclassified`, `other` |
| **severity** (V1 config) | `LOW`, `MEDIUM`, `HIGH`, `CRITICAL` |
| **configuration status** | `PASS`, `FAIL`, `ALL` |
| **framework status** | `ENABLED`, `DISABLED` |
| **authentication-state** | `AUTHENTICATED`, `UNAUTHENTICATED`, `UNKNOWN` |
| **webhook vendor** | `DATADOG`, `SPLUNK`, `MSTEAMS`, `TINES`, `CUSTOM` |

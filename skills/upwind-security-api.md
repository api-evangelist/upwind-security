---
name: upwind-api
description: Queries and manages Upwind cloud security data via REST API, including threats, vulnerabilities, configurations, inventory, and workflows. Use when user says "query Upwind", "fetch threats", "list vulnerabilities", "call Upwind API", "get detections", "search stories", or "authenticate with Upwind". Also use when making curl requests to api.upwind.io or api.eu.upwind.io. Do NOT use for internal Upwind service development (use project CLAUDE.md instead).
metadata:
  author: Upwind Security
  version: 1.0.0
---

# Upwind API

Upwind's public REST API for querying cloud security data programmatically: threats, vulnerabilities, configurations, inventory, workflows, and integrations.

## Important

- NEVER generate a new token for every API call. Tokens are valid for 24h - generate once, persist to file, reuse.
- Tokens are region-locked. A US token will NOT work with the EU endpoint.
- Prefer V2 endpoints when available. V1 is in maintenance mode.
- For the full endpoint catalog, consult `references/api-endpoints.md`.

## Instructions

### Step 1: Authenticate

Store credentials as environment variables, then generate a token ONCE and persist it:

```bash
# Generate token and save to file (US region example)
curl -s --request POST \
  --url "https://auth.upwind.io/oauth/token" \
  --data-urlencode "client_id=$UPWIND_CLIENT_ID" \
  --data-urlencode "client_secret=$UPWIND_CLIENT_SECRET" \
  --data-urlencode "audience=REGION_API_URL" \
  --data-urlencode "grant_type=client_credentials" | jq -r '.access_token' > /tmp/.upwind_access_token
```

CRITICAL: Persist to `/tmp/.upwind_access_token` because each Bash tool call runs in a fresh shell - env vars do not survive across invocations.

Only re-generate when:
- Token file does not exist
- You receive a `401 Unauthorized` response
- Token has expired (24h)

### Step 2: Select Region

Each region has its own API base URL AND requires a matching `audience` in the token request.

| Region | API Base URL | Token Audience |
|--------|-------------|----------------|
| **US** | `https://api.upwind.io` | `https://api.upwind.io` |
| **EU** | `https://api.eu.upwind.io` | `https://api.eu.upwind.io` |
| **ME** | `https://api.me.upwind.io` | `https://api.me.upwind.io` |

Auth endpoint is always: `https://auth.upwind.io/oauth/token` (all regions).

### Step 3: Make API Calls

Load the persisted token and call the appropriate endpoint:

```bash
ACCESS_TOKEN=$(cat /tmp/.upwind_access_token)

# V1 example - list critical threat detections
curl -s --request GET \
  --url "https://api.upwind.io/v1/organizations/$UPWIND_ORG_ID/threat-detections?severity=CRITICAL&per-page=10" \
  --header "Authorization: Bearer $ACCESS_TOKEN" | jq .

# V2 search example - search threat stories
curl -s --request POST \
  --url "https://api.upwind.io/v2/organizations/$UPWIND_ORG_ID/threats/stories/search?limit=10&sort=create_time:desc" \
  --header "Authorization: Bearer $ACCESS_TOKEN" \
  --header "Content-Type: application/json" \
  --data '{"conditions": [{"field": "severity", "operator": "eq", "value": ["CRITICAL"]}]}' | jq .
```

### Step 4: Handle Pagination

**V1 (token-based):** Use `per-page` and `page-token` from the `Link` response header.

**V2 (cursor-based):** Use `limit` and `cursor`. Loop until `next_cursor` is absent/null in `metadata`.

For full pagination details, see `references/api-endpoints.md`.

## Examples

### Example 1: List critical threats (V1)

User says: "Get me all critical threat detections from Upwind"

Actions:
1. Load token from `/tmp/.upwind_access_token` (generate if missing)
2. Call `GET /v1/organizations/$UPWIND_ORG_ID/threat-detections?severity=CRITICAL`
3. Paginate using `page-token` from `Link` header if more results exist

Result: JSON array of critical threat detections with details.

### Example 2: Search threat stories (V2)

User says: "Search for high severity stories in the last week"

Actions:
1. Load token from file
2. POST to `/v2/organizations/$UPWIND_ORG_ID/threats/stories/search` with conditions:
   - `severity` eq `HIGH`
   - `create_time` gte last week's date
3. Use cursor pagination if `next_cursor` is present

Result: Paginated list of threat stories matching criteria.

### Example 3: List vulnerability findings

User says: "Show me the latest vulnerability findings"

Actions:
1. Load token from file
2. Call `GET /v1/organizations/$UPWIND_ORG_ID/vulnerability-findings?per-page=20`

Result: List of vulnerability findings with CVE details.

## Troubleshooting

### Error: 401 Unauthorized
**Cause:** Token expired or audience mismatch.
**Solution:**
1. Check if token is older than 24h - regenerate if so
2. Verify the `audience` in token request matches the API base URL you're calling
3. Confirm `grant_type=client_credentials` was included in the token request

### Error: Empty or invalid token file
**Cause:** Token generation failed silently (e.g., bad credentials, missing `jq`).
**Solution:**
1. Run the curl token request without piping to `jq` to see the raw response
2. Verify `$UPWIND_CLIENT_ID` and `$UPWIND_CLIENT_SECRET` are set
3. Ensure `jq` is installed

### Error: No data returned
**Cause:** Wrong organization ID or region.
**Solution:**
1. Verify the org-id in the URL path matches your Upwind organization
2. Confirm you're calling the correct regional endpoint (US/EU/ME)
3. Check if the endpoint requires specific query parameters

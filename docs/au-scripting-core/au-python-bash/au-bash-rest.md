---
id: 'au-bash-rest'
title: 'Bash, REST APIs & Microsoft Graph'
level: 'Intermediate'
forest: 'Automation & Scripting'
tree: 'Scripting Languages'
branch: 'Python & Bash for Infrastructure'
---

# Bash, REST APIs & Microsoft Graph

**Level:** Intermediate
**Tree:** [Scripting Languages](../README.md)
**Branch:** [Python & Bash for Infrastructure](README.md)
**Forest:** [Automation & Scripting](../../../README.md)

## Explanation

# Bash and API-Driven Automation

**Bash** remains the fastest path to automation on Linux fleets and in CI runners. Production Bash starts with the safety header — set -euo pipefail — so failures stop the script, unset variables error out, and pipeline failures propagate. Quote every expansion, use local variables in functions, trap EXIT/ERR for cleanup and error context, and validate inputs with parameter expansion defaults. Anything exceeding a couple hundred lines or needing data structures should graduate to Python; knowing when Bash stops being appropriate is itself a senior skill.

API work centers on **curl + jq**: curl with --fail-with-body, --retry with backoff for transient codes, explicit timeouts, and headers from variables; jq to extract and transform JSON (filters, -r for raw strings, --arg for safe parameter injection). Understand REST mechanics: verbs and idempotency (PUT/DELETE idempotent, POST not), status classes (429 with Retry-After deserves special handling), pagination (Link headers, nextLink/continuation tokens), and **OAuth 2.0 client credentials flow** for daemon authentication — request a token from the identity provider, cache it until near expiry, send as a Bearer header.

**Microsoft Graph** is the single REST endpoint for Entra ID and Microsoft 365 (graph.microsoft.com/v1.0). Application permissions (app-only) versus delegated permissions is the key model: automation uses application permissions consented by an admin, ideally held by a managed identity. Graph specifics that bite: nearly everything is paginated via @odata.nextLink, $select/$filter reduce payloads dramatically, advanced queries need ConsistencyLevel: eventual with $count, and throttling (429) is routine at scale. The same skills transfer directly to any vendor API — vCenter, ServiceNow, GitHub — making REST fluency the most reusable automation competency of all.

## Architecture and flow

```mermaid
flowchart TD
  S[Bash: set -euo pipefail + trap] --> T[Get OAuth token - client credentials]
  T --> C[curl --fail-with-body --retry + Bearer header]
  C --> G[graph.microsoft.com/v1.0/users?$select=...]
  G --> J[jq extract fields]
  G -->|@odata.nextLink| C
  C -->|429 Retry-After| W[sleep + retry]
  J --> OUT[CSV / alert / next system]
```

## Commands

### Command 1

Robust Graph call with retries, timeout, and minimal payload

```text
curl -sS --fail-with-body --retry 4 --retry-delay 2 --max-time 30 -H "Authorization: Bearer $TOKEN" 'https://graph.microsoft.com/v1.0/users?$select=displayName,accountEnabled&$top=100'
```

### Command 2

Transform a Graph response page into CSV

```text
jq -r '.value[] | [.displayName, .accountEnabled] | @csv' users.json
```

### Command 3

Get a Graph token from the current Azure CLI login for testing

```text
az account get-access-token --resource-type ms-graph --query accessToken -o tsv
```

### Command 4

Emit the failing line number on any error in a Bash script

```text
trap 'echo "failed at line $LINENO" >&2' ERR
```

## Automation scripts

### graph_disabled_users.sh

```bash
#!/usr/bin/env bash
# Lists disabled Entra users via Graph with app-only auth and full pagination
set -euo pipefail
trap 'echo "ERROR at line \$LINENO" >&2' ERR

: "\${TENANT_ID:?TENANT_ID required}"
: "\${CLIENT_ID:?CLIENT_ID required}"
: "\${CLIENT_SECRET:?CLIENT_SECRET required}"  # inject from vault, never commit

TOKEN=\$(curl -sS --fail-with-body --max-time 20 \
  -d "client_id=\${CLIENT_ID}&scope=https%3A%2F%2Fgraph.microsoft.com%2F.default&client_secret=\${CLIENT_SECRET}&grant_type=client_credentials" \
  "https://login.microsoftonline.com/\${TENANT_ID}/oauth2/v2.0/token" | jq -r '.access_token')

if [[ -z "\$TOKEN" || "\$TOKEN" == "null" ]]; then
  echo 'Token acquisition failed' >&2
  exit 1
fi

URL='https://graph.microsoft.com/v1.0/users?\$filter=accountEnabled+eq+false&\$select=displayName,userPrincipalName&\$top=100'
COUNT=0
while [[ -n "\$URL" && "\$URL" != "null" ]]; do
  PAGE=\$(curl -sS --fail-with-body --retry 4 --retry-delay 3 --max-time 30 \
    -H "Authorization: Bearer \$TOKEN" -H 'ConsistencyLevel: eventual' "\$URL")
  echo "\$PAGE" | jq -r '.value[] | [.displayName, .userPrincipalName] | @csv'
  N=\$(echo "\$PAGE" | jq '.value | length')
  COUNT=\$(( COUNT + N ))
  URL=\$(echo "\$PAGE" | jq -r '."@odata.nextLink" // empty')
done
echo "Total disabled users: \$COUNT" >&2
```

## Lab

**Objective:** Automate an Entra ID report end-to-end: app registration, app-only Graph auth from Bash, and paginated extraction.

### Steps

1. Register an application, grant User.Read.All application permission, and record admin consent.
2. Store the client secret in a vault and export it into the shell only at runtime.
3. Acquire a token via client credentials with curl and decode its roles claim to confirm the permission.
4. Query users with $filter and $select, following @odata.nextLink until exhausted.
5. Emit CSV, then deliberately request an unauthorized resource and observe the 403.
6. Add 429 handling: honor Retry-After and re-request, verified against a burst of rapid calls.

### Validation

Token JWT roles claim contains User.Read.All.,Row count matches the Entra portal's disabled-user count.,Script exits non-zero with a clear message when the secret is wrong or permission missing.

## Operational automation

## From ad-hoc calls to managed integrations

- Replace client secrets with **managed identity** where the caller runs in Azure (Functions, Automation, Arc agents request Graph tokens locally).
- Wrap recurring reports as scheduled pipeline jobs emitting artifacts; alert on schema or count anomalies.
- Standardize a shared Bash library (token cache, paginated GET, 429 handler) sourced by all scripts.
- For complex flows, hand off to the Graph SDKs (PowerShell/Python) — same permissions, richer batching and retry built in.

## Troubleshooting

### Scenario 1: Graph returns only 100 items when thousands exist

**Likely cause:** Pagination ignored — results continue at @odata.nextLink

**Resolution:** Loop while nextLink is present; never assume a single page. Use $top=999 where supported to reduce round trips

### Scenario 2: 401 Unauthorized with a seemingly valid token

**Likely cause:** Token audience/scope wrong (for example an ARM token sent to Graph), or the permission granted is delegated while the script runs app-only

**Resolution:** Decode the JWT: check aud is graph.microsoft.com and roles (not scp) contains the needed permission; re-consent application permissions as admin

### Scenario 3: Script intermittently fails under cron but works interactively

**Likely cause:** Environment differences: PATH, missing env vars, no TTY, stricter noexec tmp

**Resolution:** Set explicit PATH and env in the unit/crontab, log stderr to a file, use absolute paths for binaries, and add the safety header so failures surface instead of half-running

## Interview questions

### 1. What does set -euo pipefail do and why is it standard?

-e exits on command failure, -u errors on unset variable expansion, and pipefail makes a pipeline's exit status reflect any failing stage rather than only the last. Together they convert silent partial failures — the classic Bash hazard — into immediate, visible stops. They have edge cases (conditionals, command substitution) that professionals learn deliberately.

### 2. Delegated versus application permissions in Microsoft Graph?

Delegated permissions act on behalf of a signed-in user and are bounded by both the app grant and the user's own rights — for interactive tools. Application permissions grant the app itself tenant-wide access with no user, require admin consent, and are what unattended automation uses; they appear in the token as roles rather than scp.

### 3. How should a script handle API throttling correctly?

Treat 429 as expected: read Retry-After and wait that long (falling back to exponential backoff with jitter), cap concurrency proactively, batch where the API allows, and reduce payloads with select/filter so fewer calls are needed. Log throttle events — sustained throttling means the design, not the retry, needs fixing.

### 4. When do you stop writing Bash and switch to Python?

When you need real data structures, JSON manipulation beyond simple jq pipes, unit tests, dependency management, concurrency, or the script passes roughly a few hundred lines. Bash excels at process orchestration and glue; the mistake is not choosing Bash, it is staying in it past complexity that demands testability.

## Certification alignment

- AZ-104 - Automate with CLI and REST-based tooling
- SC-300 - Implement app registrations and API permissions
- LFCS - Shell scripting fundamentals for Linux administration

## References

- Microsoft Graph documentation: auth, permissions, paging, throttling guidance
- OAuth 2.0 RFC 6749 - client credentials grant
- Google Shell Style Guide and jq manual

## Suggested video search

bash curl jq REST API automation Microsoft Graph client credentials

---

> Validate commands, versions, permissions, licensing, and rollback procedures in an isolated lab before production use.

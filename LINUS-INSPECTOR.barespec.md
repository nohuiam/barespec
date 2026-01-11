SERVER: linus-inspector
VERSION: 1.0.0
UPDATED: 2026-01-11
CATEGORY: Quality Gate
STATUS: Production
TESTS: 115 passed

---

## PURPOSE

Brutal quality gate for neurogenesis-generated servers. Runs comprehensive inspections on code before, during, and after build. Enforces vendor-specific best practices for SaaS integrations. Prevents bad code from entering the ecosystem.

Philosophy: "Physician, heal thyself" - includes self-inspection capability.

---

## PORTS

| Layer | Port | Purpose |
|-------|------|---------|
| MCP | stdio | Claude Desktop interface |
| UDP | 3037 | InterLock mesh |
| HTTP | 8037 | REST API |
| WS | 9037 | Real-time events |

---

## TOOLS (26)

### Pre-Build Inspection (7)
| Tool | Schema | Description |
|------|--------|-------------|
| inspect_saas_api | `{vendor}` | Get vendor config (rate limits, auth, webhooks) |
| inspect_prompt | `{prompt_content, prompt_id?, expected_output_format?, use_case?}` | Validate prompt safety/clarity/efficiency |
| inspect_skill | `{skill_path?, skill_content?, skill_name?}` | Validate skill.md against Anthropic guidelines |
| inspect_template | `{template_content, template_type?}` | Validate Handlebars/Mustache/EJS templates |
| inspect_config | `{config_content, expected_fields?}` | Validate JSON config syntax/structure |
| discover_api_schema | `{vendor}` | Get known API schema for vendor |
| check_port_availability | `{port}` | Check if port available in ecosystem |

### Build Inspection (8)
| Tool | Schema | Description |
|------|--------|-------------|
| inspect_build | `{server_path, server_name?, vendor?, regulation?, build_id?}` | Full inspection suite |
| inspect_code_quality | `{code, vendor?, regulation?}` | Static analysis |
| inspect_error_handling | `{code}` | Check error handlers |
| inspect_rate_limiting | `{code, vendor?}` | Verify rate limit implementation |
| inspect_security | `{code}` | Security audit (credentials, injection) |
| inspect_performance | `{code, vendor?}` | N+1 queries, batching, caching |
| inspect_edge_cases | `{code, vendor?}` | Empty responses, pagination, nulls |
| inspect_data_integrity | `{code, vendor?}` | Type preservation |

### Runtime Inspection (5)
| Tool | Schema | Description |
|------|--------|-------------|
| inspect_connection | `{vendor, test_mode?}` | Test live API connection |
| test_auth | `{vendor}` | Verify authentication |
| test_data_roundtrip | `{vendor, use_sandbox?}` | Create, read, delete test |
| test_rate_limit_backoff | `{vendor}` | Hit limit, verify backoff |
| test_webhook_delivery | `{vendor}` | Register and verify webhook |

### Ecosystem Inspection (5)
| Tool | Schema | Description |
|------|--------|-------------|
| inspect_integration | `{server_path, check_connectivity?, timeout_ms?}` | Verify InterLock mesh integration |
| inspect_mcp_protocol | `{server_path}` | Validate MCP server implementation |
| inspect_documentation | `{server_path}` | Check README, API docs |
| inspect_test_coverage | `{server_path}` | Verify tests exist |
| get_inspection_report | `{inspection_id?, build_id?}` | Retrieve full results |

### Self-Inspection (1)
| Tool | Schema | Description |
|------|--------|-------------|
| inspect_self | `{include_meta_rules?, emit_signal?, verbose?}` | Physician heal thyself - inspect own codebase |

---

## INSPECTION RULES

### Rate Limiting (35-40% of API failures)
- rl-001: No rate limit detection
- rl-002: No backoff implementation
- rl-003: No retry-after handling
- rl-004: No 429 handling
- rl-005: Linear retry (should be exponential)
- rl-006: Missing jitter
- Vendor-specific: Salesforce, HubSpot, Stripe, Snowflake, Zendesk, Slack, QuickBooks, Shopify, ServiceNow, Microsoft365

### OAuth/Auth (10-20% of failures)
- oauth-001: No token refresh
- oauth-002: No refresh buffer (5min)
- oauth-003: No state validation
- oauth-004: No multi-tenant isolation
- oauth-005: Token in URL
- oauth-006: Missing secure storage
- oauth-007: No PKCE
- oauth-008: Hardcoded credentials

### Error Handling (25-30% of failures)
- err-001: No 4xx handlers
- err-002: No 5xx handlers
- err-003: No Dead Letter Queue
- err-004: No max retry limit
- err-005: No timeout config
- err-006: Swallowed errors (empty catch)
- err-007: No error classification
- err-008: No monitoring/alerting
- err-009: Retrying non-retryable errors
- err-010: No graceful degradation
- err-011: No request ID tracking

### Webhooks (12-20% of failures)
- wh-001: No signature validation
- wh-002: No idempotency handling
- wh-003: No replay protection
- wh-004: No acknowledgment timing
- wh-005: No retry/DLQ
- wh-006: No payload validation

### Data Integrity (8-10% but highest cost)
- di-001: Type coercion
- di-002: ID truncation
- di-003: Pagination (HubSpot hasMore)
- di-004: Offset pagination
- di-005: Currency precision
- di-006: DateTime timezone
- di-007: Encoding issues
- di-008: Null handling
- di-009: Governor limits (Salesforce)

### Compliance
- HIPAA: Encryption, audit logs, access control, BAA, breach notification
- GDPR: Consent, data minimization, right to deletion, DPO, breach notification
- SOC2: Access control, encryption, change management, incident response, availability
- PCI-DSS: Encryption, access control, vulnerability management, network security, audit logs

### Meta-Rules (Self-Inspection)
- meta-001: Rate-limit inspector without rate limiting (CRITICAL)
- meta-002: Error inspector with poor error handling (HIGH)
- meta-003: Validator without input validation (HIGH)
- meta-004: Regex rules without multiline flag (MEDIUM)
- meta-005: HTTP endpoints without timeout (HIGH)
- meta-006: Compliance checker not self-compliant (HIGH)
- meta-007: Test inspector without tests (MEDIUM)
- meta-008: Security inspector with hardcoded secrets (CRITICAL)

---

## SUPPORTED VENDORS

salesforce, hubspot, stripe, snowflake, zendesk, slack, quickbooks, shopify, servicenow, microsoft365

---

## HTTP API

| Endpoint | Method | Description |
|----------|--------|-------------|
| /health | GET | Health check |
| /api/vendor/:vendor | GET | Get vendor config |
| /api/vendors | GET | List all vendors |
| /api/inspect/prompt | POST | Inspect prompt |
| /api/inspect/skill | POST | Validate skill |
| /api/inspect/build | POST | Full build inspection |
| /api/inspect/code | POST | Quick code inspection |
| /api/inspect/integration | POST | Ecosystem integration check |
| /api/compliance | GET | Get compliance rules |
| /api/compliance/:regulation | GET | Get rules by regulation |
| /api/inspections/:id | GET | Get inspection by ID |
| /api/builds/:buildId/inspections | GET | Get inspections by build |
| /api/status | GET | Server status |

Rate Limited: 100 requests/minute

---

## INTERLOCK SIGNALS

### Emits
| Code | Signal | Description |
|------|--------|-------------|
| 0xC4 | INSPECTION_PASSED | Build passed all inspections |
| 0xC5 | INSPECTION_FAILED | Build failed with issues |
| 0xC7 | LESSON_EXTRACTED | New lesson from failure |
| 0xD0 | PRE_INSPECTION_COMPLETE | Pre-build inspection done |
| 0xD1 | CONNECTION_TEST_RESULT | Live connection test completed |

### Receives
| Code | Signal | Description |
|------|--------|-------------|
| 0xC0 | BUILD_STARTED | Neurogenesis started build |
| 0xC3 | INSPECTION_REQUESTED | Request to inspect code |
| 0xD2 | PRE_INSPECTION_REQUESTED | Request to probe API |
| 0xD3 | SKILL_VALIDATION_REQUESTED | Request to validate skill |
| 0xD4 | PROMPT_VALIDATION_REQUESTED | Request to validate prompt |

---

## MESH PEERS

| Peer | Port | Role |
|------|------|------|
| neurogenesis-engine | 3010 | Primary client |
| experience-layer | 3031 | Lesson storage |
| skill-builder | 3029 | Skill validation |
| research-bus | 3019 | API documentation |
| context-guardian | 3001 | Safety validation |
| tenets-server | 3027 | Ethical checks |
| verifier-mcp | 3021 | Claim verification |
| toolee | 3004 | Tool registry |
| percolation-server | 3030 | Stress testing |
| consciousness-mcp | 3028 | Pattern detection |
| health-monitor | 3024 | Health metrics |
| consolidation-engine | 3032 | Documentation |

---

## DATABASE

SQLite: data/linus-inspector.db

### Tables
- inspections: Build inspection records
- inspection_issues: Individual issues found
- vendor_configs: Vendor-specific configurations
- compliance_rules: Compliance rule definitions
- auto_fixes: Applied auto-fix history

---

## VERDICTS

| Verdict | Criteria |
|---------|----------|
| PASSED | No critical or high issues |
| WARNING | Medium issues only |
| BLOCKED | Critical or high issues present |

---

## DEPS

| Dependency | Purpose |
|------------|---------|
| SQLite | Local database |
| better-sqlite3 | DB driver |
| express | HTTP server |
| zod | Schema validation |

---

## DIRECTORY STRUCTURE

```
linus-inspector/
├── config/
│   └── interlock.json
├── data/
│   └── linus-inspector.db
├── src/
│   ├── database/
│   │   ├── index.ts
│   │   └── schema.sql
│   ├── http/
│   │   ├── middleware.ts    # Rate limiting, timeout, error handling
│   │   └── server.ts
│   ├── inspectors/
│   │   ├── code-inspector.ts
│   │   ├── integration-checker.ts
│   │   ├── prompt-inspector.ts
│   │   └── skill-validator.ts
│   ├── interlock/
│   │   ├── handlers.ts
│   │   ├── protocol.ts
│   │   ├── socket.ts
│   │   └── tumbler.ts
│   ├── rules/
│   │   ├── compliance-rules.ts
│   │   ├── data-integrity-rules.ts
│   │   ├── error-rules.ts
│   │   ├── meta-rules.ts       # Self-inspection rules
│   │   ├── oauth-rules.ts
│   │   ├── rate-limit-rules.ts
│   │   ├── webhook-rules.ts
│   │   └── index.ts
│   ├── tools/
│   │   ├── inspect-self.ts     # Physician heal thyself
│   │   └── index.ts
│   └── index.ts
├── tests/
│   ├── database.test.ts
│   ├── http.test.ts
│   ├── inspectors.test.ts
│   ├── interlock.test.ts
│   ├── meta-rules.test.ts
│   └── rules.test.ts
├── package.json
└── tsconfig.json
```

---

## WORKFLOW

### Pre-Build (neurogenesis calls before coding)
```
1. inspect_saas_api(vendor) → get rate limits, auth, webhooks
2. inspect_prompt(system_prompt) → validate clarity/safety
3. inspect_skill(skill_file) → validate skill structure
```

### Build (neurogenesis calls after code generation)
```
1. inspect_build(server_path) → full inspection
2. If BLOCKED: return issues for fix
3. If PASSED: signal INSPECTION_PASSED
4. Store results in experience-layer
```

### Self-Inspection
```
1. inspect_self({include_meta_rules: true})
2. Runs all standard rules on own codebase
3. Runs meta-rules to detect ironic gaps
4. Returns physician_healed: true if no critical/high meta-violations
```

---

## EXAMPLE

```bash
# Quick code inspection
curl -X POST http://localhost:8037/api/inspect/code \
  -H "Content-Type: application/json" \
  -d '{
    "code": "async function fetchData() { const res = await fetch(url); return res.json(); }",
    "vendor": "hubspot"
  }'

# Response
{
  "verdict": "BLOCKED",
  "results": [
    {
      "category": "rate_limiting",
      "violations": [{
        "rule_id": "rl-001",
        "severity": "CRITICAL",
        "issue": "No rate limit detection",
        "remedy": "Add 429 handler with exponential backoff"
      }]
    }
  ],
  "summary": {
    "critical": 2,
    "high": 3,
    "medium": 1,
    "low": 0
  }
}
```

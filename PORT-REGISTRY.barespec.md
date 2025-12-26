REGISTRY: BOP Servers Port Allocation
VERSION: 2.0
UPDATED: 2025-12-26
AUDITED: Source code audit (not documentation)

---

## PORT ALLOCATION SCHEME

MCP: All servers use stdio transport (stdin/stdout JSON-RPC)
UDP_BASE: 3000 (InterLock mesh)
HTTP_BASE: 8000 (REST API)
WS_BASE: 9000 (WebSocket real-time)

PATTERN: Server N uses 300N, 800N, 900N

---

## SERVER PORT ASSIGNMENTS (Audited from Source Code)

| Server | UDP | HTTP | WS | Status | External Deps |
|--------|-----|------|----|--------|---------------|
| context-guardian | 3001 | 8001 | 9001 | Production | Redis 6379 |
| quartermaster | 3002 | 8002 | 9002 | Production | Redis 6379 |
| snapshot | 3003 | 8003 | 9003 | Production | - |
| (toolee - reserved) | 3004 | 8004 | 9004 | Not in Claude Desktop | - |
| catasorter | 3005 | 8005 | 9005 | Production | - |
| looker | - | 8006 | - | Production | - |
| smart-file-organizer | 3007 | 8007 | 9007 | Production | Redis 6379 |
| bonzai-bloat-buster | 3008 | 8008 | 9008 | Production | Qdrant 6333 |
| enterspect | 3009 | 8009 | 9009 | Production | - |
| neurogenesis-engine | 3010 | 8010 | 9010 | Production | - |
| chronos-synapse | - | 8011 | - | HTTP-only | - |
| trinity-coordinator | 3012 | 8012 | 9012 | Production | - |
| (reserved) | 3014 | 8014 | 9014 | - | - |
| niws-server | - | 8015 | - | HTTP-only | Notion API |
| project-context | 3016 | 8016 | 9016 | Production | - |
| knowledge-curator | 3017 | 8017 | 9017 | Production | - |
| pk-manager | 3018 | 8018 | 9018 | Production | - |
| research-bus | - | 8019 | - | HTTP-only | Perplexity API |
| intelligent-router | 3020 | 8020 | 9020 | Production | Claude API |
| verifier-mcp | 3021 | 8021 | 9021 | Production | Anthropic API, HuggingFace |
| filesystem | - | - | - | 3rd party | 3rd party npm |

---

## STATUS KEY

PRODUCTION: Connected to Claude Desktop, fully operational
PARTIAL: Some features implemented
DESIGNED: Specification complete, not yet built
NOT IN CLAUDE DESKTOP: Server exists but not registered

---

## PORT DETAILS (From Config Files)

### context-guardian (3001/8001/9001)
SOURCE: /repo/imminenceV2/context-guardian/config/interlock.json
UDP: 3001 - InterLock mesh
HTTP: 8001 - Stats API, rule management
WS: 9001 - Real-time validation events
DEPS: Redis 6379 (vault)

### quartermaster (3002/8002/9002)
SOURCE: /repo/Quartermaster/quartermaster/config/default.json
UDP: 3002 - InterLock mesh
HTTP: 8002 - Index stats, search API
WS: 9002 - Search streaming
DEPS: Redis 6379 (optional)

### snapshot (3003/8003/9003)
SOURCE: /repo/snapSHOT/config/interlock.json
UDP: 3003 - InterLock mesh
HTTP: 8003 - Snapshot listing API
WS: 9003 - Backup progress events

### catasorter (3005/8005/9005)
SOURCE: /repo/Catasorter/config/interlock.json
UDP: 3005 - InterLock mesh
HTTP: 8005 - Stats, Dewey counters
WS: 9005 - Batch progress events

### looker (8006)
SOURCE: /repo/looker-mcp/src/index.ts:667
UDP: None
HTTP: 8006 - Credibility API, search API
WS: None

### smart-file-organizer (3007/8007/9007)
SOURCE: /repo/smart_file_organizer/src/server.js
UDP: 3007 - InterLock mesh (graceful degradation)
HTTP: 8007 - Organization stats
WS: 9007 - Batch operation progress
DEPS: Redis 6379 (vault)

### bonzai-bloat-buster (3008/8008/9008)
SOURCE: /repo/bonzai-bloat-buster/config/interlock.json
UDP: 3008 - InterLock mesh
HTTP: 8008 - Duplicate detection stats
WS: 9008 - Analysis progress events
DEPS: Qdrant 6333 (vector DB)

### enterspect (3009/8009/9009)
SOURCE: /repo/EnterSpect/config/interlock.json
UDP: 3009 - InterLock mesh
HTTP: 8009 - Index stats API
WS: 9009 - Search result streaming

### neurogenesis-engine (3010/8010/9010)
SOURCE: /repo/neurogenesis-engine/src/constants.js
UDP: 3010 - InterLock mesh
HTTP: 8010 - Build queue, feedback stats
WS: 9010 - Generation progress

### chronos-synapse (8011)
SOURCE: /repo/Chronos_Synapse/dist/index.js:70
UDP: None
HTTP: 8011 - Event stats, Langfuse endpoint
WS: None (not implemented)

### trinity-coordinator (3012/8012/9012)
SOURCE: /repo/trinitycoordinator/dist/index.js:38-40
UDP: 3012 - InterLock mesh
HTTP: 8012 - Coordination status
WS: 9012 - Handoff notifications
ENV: TRINITY_HTTP_PORT, TRINITY_INTERLOCK_PORT, TRINITY_WEBSOCKET_PORT

### niws-server (8015)
SOURCE: /repo/niws-server/src/config.ts:74
UDP: None
HTTP: 8015 - Admin/production status
WS: None
DEPS: Notion API

### project-context (3016/8016/9016)
SOURCE: /repo/project-context/config/interlock.json
UDP: 3016 - InterLock mesh
HTTP: 8016 - Configuration API
WS: 9016 - Config change events

### knowledge-curator (3017/8017/9017)
SOURCE: /repo/knowledge-curator/config/interlock.json
UDP: 3017 - InterLock mesh
HTTP: 8017 - Compression stats API
WS: 9017 - Curation progress events

### pk-manager (3018/8018/9018)
SOURCE: /repo/pk-manager/config/interlock.json
UDP: 3018 - InterLock mesh
HTTP: 8018 - Rebuild status API
WS: 9018 - Rebuild progress events

### research-bus (8019)
SOURCE: /repo/research-bus/dist/index.js:27
UDP: None
HTTP: 8019 - Budget status, cache stats
WS: None
DEPS: Perplexity API

### intelligent-router (3020/8020/9020)
SOURCE: /repo/intelligentrouter/config/interlock.json
UDP: 3020 - InterLock mesh
HTTP: 8020 - Routing history API
WS: 9020 - Routing progress events
DEPS: Claude API

### verifier-mcp (3021/8021/9021)
SOURCE: /repo/verifier-mcp/config/interlock.json
UDP: 3021 - InterLock mesh
HTTP: 8021 - Claims extraction/verification API
WS: 9021 - Verification progress events
DEPS: Anthropic API, HuggingFace (MiniCheck)

### filesystem (stdio only)
SOURCE: npm @modelcontextprotocol/server-filesystem
UDP: None
HTTP: None
WS: None

---

## EXTERNAL SERVICES

| Service | Port | Purpose | Used By |
|---------|------|---------|---------|
| Redis | 6379 | Shared memory vault | context-guardian, quartermaster, smart-file-organizer |
| Qdrant | 6333 | Vector embeddings DB | bonzai-bloat-buster |
| Comet CDP | 9222 | Browser automation | research-bus (optional) |

---

## INTERLOCK MESH

PROTOCOL: UDP broadcast on 300N ports
PURPOSE: Server discovery, heartbeat, coordination
HEARTBEAT: Every 30 seconds
TIMEOUT: 90 seconds (3x heartbeat)

ACTIVE MESH PARTICIPANTS:
- context-guardian (3001)
- quartermaster (3002)
- snapshot (3003)
- catasorter (3005)
- smart-file-organizer (3007)
- bonzai-bloat-buster (3008)
- enterspect (3009)
- neurogenesis-engine (3010)
- trinity-coordinator (3012)
- project-context (3016)
- knowledge-curator (3017)
- pk-manager (3018)
- intelligent-router (3020)
- verifier-mcp (3021)

HTTP-ONLY SERVERS:
- looker (8006)
- chronos-synapse (8011)
- niws-server (8015)
- research-bus (8019)

3RD PARTY (Cannot modify):
- filesystem (npm package)

---

## NOTES

1. All servers use stdio transport for MCP (stdin/stdout JSON-RPC)
2. UDP/HTTP/WS ports are for InterLock mesh and REST APIs, not MCP
3. "Production" means connected to Claude Desktop config
4. Port assignments verified from source code, not documentation
5. Toolee (3004) exists but not registered in Claude Desktop

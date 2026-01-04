REGISTRY: BOP Servers Port Allocation
VERSION: 2.2
UPDATED: 2026-01-03
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
| tool-registry | 3004 | 8004 | 9004 | Production | - |
| catasorter | 3005 | 8005 | 9005 | Production | - |
| looker | - | 8006 | - | Production | - |
| smart-file-organizer | 3007 | 8007 | 9007 | Production | Redis 6379 |
| bonzai-bloat-buster | 3008 | 8008 | 9008 | Production | Qdrant 6333 |
| enterspect | 3009 | 8009 | 9009 | Production | - |
| neurogenesis-engine | 3010 | 8010 | 9010 | Production | - |
| chronos-synapse | - | 8011 | - | HTTP-only | - |
| trinity-coordinator | 3012 | 8012 | 9012 | Production | - |
| claude-code-bridge | 3013 | 8013 | 9013 | Production | SQLite |
| (reserved) | 3014 | 8014 | 9014 | - | - |
| niws-server | - | 8015 | - | HTTP-only | Notion API |
| project-context | 3016 | 8016 | 9016 | Production | - |
| knowledge-curator | 3017 | 8017 | 9017 | Production | - |
| pk-manager | 3018 | 8018 | 9018 | Production | - |
| research-bus | - | 8019 | - | HTTP-only | Perplexity API |
| intelligent-router | 3020 | 8020 | 9020 | Production | Claude API |
| verifier-mcp | 3021 | 8021 | 9021 | Production | Anthropic API, HuggingFace |
| safe-batch-processor | 3022 | 8022 | 9022 | Production | SQLite |
| intake-guardian | 3023 | 8023 | 9023 | Production | SQLite, BBB (8008) |
| health-monitor | 3024 | 8024 | 9024 | Production | SQLite |
| synapse-relay | 3025 | 8025 | 9025 | Production | SQLite |
| filesystem-guardian | 3026 | 8026 | 9026 | Production | macOS-native (xattr, mdfind) |
| tenets-server | 3027 | 8027 | 9027 | Production | SQLite |
| consciousness-mcp | 3028 | 8028 | 9028 | Production | SQLite |
| skill-builder | 3029 | 8029 | 9029 | Production | SQLite |
| percolation-server | 3030 | 8030 | 9030 | Production | SQLite |
| experience-layer | 3031 | 8031 | 9031 | Production | SQLite |
| consolidation-engine | 3032 | 8032 | 9032 | Production | SQLite |
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

### safe-batch-processor (3022/8022/9022)
SOURCE: /repo/safe-batch-processor/config/interlock.json
UDP: 3022 - InterLock mesh
HTTP: 8022 - Batch operations API
WS: 9022 - Batch progress events
DEPS: SQLite (local), consolidation-engine, project-context, snapshot, smart-file-organizer

### intake-guardian (3023/8023/9023)
SOURCE: /repo/intake-guardian/config/interlock.json
UDP: 3023 - InterLock mesh
HTTP: 8023 - Content admission API
WS: 9023 - Admission decision events
DEPS: SQLite (local), BBB (8008 for redundancy scores)

### health-monitor (3024/8024/9024)
SOURCE: /repo/health-monitor/config/interlock.json
UDP: 3024 - InterLock mesh
HTTP: 8024 - Health status API, alerts API
WS: 9024 - Real-time health alerts
DEPS: SQLite (local)
PURPOSE: Monitor health of all 22+ ecosystem servers

### synapse-relay (3025/8025/9025)
SOURCE: /repo/synapse-relay/config/interlock.json
UDP: 3025 - InterLock mesh
HTTP: 8025 - Relay rules API, buffer management
WS: 9025 - Relay events, buffer status
DEPS: SQLite (local)
PURPOSE: Neural packet routing, signal relay between servers

### filesystem-guardian (3026/8026/9026)
SOURCE: /repo/filesystem-guardian/config/interlock.json
UDP: 3026 - InterLock mesh
HTTP: 8026 - xattr API, Spotlight API, watch management
WS: 9026 - Filesystem events
DEPS: macOS-native (xattr CLI, mdfind, fs.watch)
PURPOSE: Extended attributes, Spotlight search, filesystem monitoring

### tenets-server (3027/8027/9027)
SOURCE: /repo/tenets-server/config/interlock.json
UDP: 3027 - InterLock mesh
HTTP: 8027 - Decision evaluation API, tenet queries
WS: 9027 - Evaluation events, violation alerts
DEPS: SQLite (local)
TESTS: 141 tests
CATEGORY: Cognitive
PURPOSE: Ethical decision evaluation against 25 Gospel tenets

### consolidation-engine (3032/8032/9032)
SOURCE: /repo/consolidation-engine/config/interlock.json
UDP: 3032 - InterLock mesh
HTTP: 8032 - Merge plans API, conflict resolution
WS: 9032 - Merge progress events
DEPS: SQLite (local)
TESTS: 217 (89% statement coverage, 76% branch coverage)
CI: GitHub Actions (Node 18/20/22)
REPO: https://github.com/nohuiam/consolidation-engine

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

ACTIVE MESH PARTICIPANTS (25):
- context-guardian (3001)
- quartermaster (3002)
- snapshot (3003)
- tool-registry (3004)
- catasorter (3005)
- smart-file-organizer (3007)
- bonzai-bloat-buster (3008)
- enterspect (3009)
- neurogenesis-engine (3010)
- trinity-coordinator (3012)
- claude-code-bridge (3013)
- project-context (3016)
- knowledge-curator (3017)
- pk-manager (3018)
- intelligent-router (3020)
- verifier-mcp (3021)
- safe-batch-processor (3022)
- intake-guardian (3023)
- health-monitor (3024)
- synapse-relay (3025)
- filesystem-guardian (3026)
- tenets-server (3027)
- consciousness-mcp (3028)
- skill-builder (3029)
- percolation-server (3030)
- experience-layer (3031)
- consolidation-engine (3032)

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

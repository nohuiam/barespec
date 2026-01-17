# BOP Gateway Barespec

**Server:** bop-gateway
**Version:** 1.0.0
**Updated:** 2026-01-17
**Port Assignment:** MCP stdio | HTTP 8038
**Purpose:** Single MCP entry point to 36-server BOP ecosystem. Token savings: ~150k to ~3k.

---

## Definition

BOP Gateway provides Claude Desktop access to the entire Box of Prompts server ecosystem through 5 meta-tools. Instead of connecting 36 servers with 450+ tools (~150k tokens), one gateway with 5 tools (~3k tokens) proxies HTTP calls to any server.

**Core Principle:** "Dumb proxy, smart routing"

---

## Architecture (2-Layer)

| Layer | Transport | Port | Purpose |
|-------|-----------|------|---------|
| MCP | stdio | stdin/stdout | Claude Desktop interface |
| HTTP | REST | 8038 | API access + web app integration |

**Not Used:** UDP (no mesh coordination), WebSocket (no real-time events)

**Rationale:** Gateway is an edge adapter, not an ecosystem participant. Stateless proxy design.

---

## MCP Tools (5)

### gateway_discover
Find servers by query, workflow, category, or name.

**Input:**
```typescript
{
  query?: string;      // Semantic search via Toolee
  workflow?: 'research' | 'playbooks' | 'implementation' | 'niws';
  server?: string;     // Specific server name
  category?: 'core' | 'workflow' | 'safety' | 'cognitive' | 'niws';
}
```

**Output:**
```typescript
{
  servers: Array<{
    name: string;
    http: string;       // e.g., "http://localhost:8027"
    category: string;
    workflows: string[];
  }>;
  count: number;
}
```

**Workflow Mappings:**
- `research`: research-bus, verifier-mcp, consciousness-mcp, knowledge-curator, pk-manager
- `playbooks`: skill-builder, experience-layer, tenets-server, toolee
- `implementation`: neurogenesis-engine, percolation-server, trinity-coordinator, safe-batch-processor
- `niws`: niws-intake, niws-analysis, niws-production, niws-delivery, tenets-server

---

### gateway_call
HTTP proxy to any server endpoint.

**Input:**
```typescript
{
  server: string;                    // Required: server name
  endpoint: string;                  // Required: API path (e.g., "/api/scripts")
  method?: 'GET' | 'POST' | 'PUT' | 'DELETE';  // Default: GET
  body?: object;                     // For POST/PUT
  params?: Record<string, string>;   // Query parameters
}
```

**Output:**
```typescript
{
  success: boolean;
  status?: number;
  server: string;
  endpoint: string;
  data?: unknown;
  error?: string;
}
```

**Security:**
- Server name validation: `/^[a-zA-Z0-9_-]+$/`
- Server must exist in registry (no external servers)
- Path traversal blocked: `..`, `%2e`, `%2E`
- Payload size limit: 10MB
- Request timeout: 30 seconds

---

### gateway_health
Check health of servers.

**Input:**
```typescript
{
  server?: string;     // Specific server, or omit for filtered set
  category?: string;   // Filter by category
  workflow?: string;   // Filter by workflow
}
```

**Output:**
```typescript
{
  servers: Array<{
    name: string;
    status: 'healthy' | 'unhealthy' | 'unknown';
    http: string;
    latency_ms?: number;
    error?: string;
  }>;
  healthy: number;
  unhealthy: number;
  total: number;
}
```

---

### gateway_tools
List MCP tools available on a server.

**Input:**
```typescript
{
  server: string;      // Required: server name
  tool?: string;       // Optional: specific tool for full schema
}
```

**Output:**
```typescript
{
  server: string;
  tools: Array<{
    name: string;
    description: string;
    inputSchema?: object;
  }>;
  count: number;
}
```

---

### gateway_mcp
Execute an MCP tool on a server via HTTP bridge.

**Input:**
```typescript
{
  server: string;      // Required
  tool: string;        // Required: tool name
  arguments?: object;  // Tool arguments
}
```

**Output:**
```typescript
{
  success: boolean;
  server: string;
  tool: string;
  result?: unknown;
  error?: string;
}
```

**Note:** Servers must expose MCP tools via HTTP POST to `/api/tools/:toolName`

---

## Server Integration Requirements

For `gateway_tools` and `gateway_mcp` to work, servers must implement:

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/api/tools` | GET | List all MCP tools with schemas |
| `/api/tools/:toolName` | POST | Execute tool via HTTP bridge |

**GET /api/tools response format:**
```typescript
{
  tools: Array<{
    name: string;
    description: string;
    inputSchema: object;
  }>;
  count: number;
}
```

**POST /api/tools/:toolName request format:**
```typescript
{
  arguments?: object;  // Tool arguments (also accepts flat body)
}
```

**POST /api/tools/:toolName response format:**
```typescript
{
  success: boolean;
  result?: unknown;
  error?: string;
}
```

**Servers with gateway integration (as of 2026-01-17):**
- tenets-server (8027)
- niws-intake (8033)
- niws-analysis (8034)
- niws-production (8035)
- niws-delivery (8036)
- linus-inspector (8037)

---

## Health Check Behavior

`gateway_health` checks both `/health` and `/api/health` endpoints per server to handle varying conventions.

---

## Server Registry

36 servers across ports 8001-8037 (with gaps).

### Core Infrastructure (8001-8013)
| Server | HTTP | Workflows |
|--------|------|-----------|
| context-guardian | 8001 | - |
| quartermaster | 8002 | - |
| snapshot | 8003 | - |
| toolee | 8004 | playbooks |
| catasorter | 8005 | - |
| looker-mcp | 8006 | - |
| smart-file-organizer | 8007 | - |
| bonzai-bloat-buster | 8008 | - |
| enterspect | 8009 | - |
| neurogenesis-engine | 8010 | implementation |
| chronos-synapse | 8011 | - |
| trinity-coordinator | 8012 | implementation |
| claude-code-bridge | 8013 | - |

### NIWS Legacy (8015)
| Server | HTTP | Status |
|--------|------|--------|
| niws-server | 8015 | DEPRECATED |

### Workflow & Coordination (8016-8020)
| Server | HTTP | Workflows |
|--------|------|-----------|
| project-context | 8016 | - |
| knowledge-curator | 8017 | research |
| pk-manager | 8018 | research |
| research-bus | 8019 | research |
| intelligent-router | 8020 | - |

### Safety & Quality (8021-8026, 8037)
| Server | HTTP | Workflows |
|--------|------|-----------|
| verifier-mcp | 8021 | research |
| safe-batch-processor | 8022 | implementation |
| intake-guardian | 8023 | - |
| health-monitor | 8024 | - |
| synapse-relay | 8025 | - |
| filesystem-guardian | 8026 | - |
| linus-inspector | 8037 | - |

### Cognitive Architecture (8027-8032)
| Server | HTTP | Workflows |
|--------|------|-----------|
| tenets-server | 8027 | playbooks, niws |
| consciousness-mcp | 8028 | research |
| skill-builder | 8029 | playbooks |
| percolation-server | 8030 | implementation |
| experience-layer | 8031 | playbooks |
| consolidation-engine | 8032 | - |

### NIWS Pipeline (8033-8036)
| Server | HTTP | Workflows |
|--------|------|-----------|
| niws-intake | 8033 | niws |
| niws-analysis | 8034 | niws |
| niws-production | 8035 | niws |
| niws-delivery | 8036 | niws |

---

## HTTP REST API (Port 8038)

For web app integration (boxofprompts.com):

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | /health | Gateway health check |
| GET | /api/servers | List all servers |
| GET | /api/servers/:name | Server details |
| GET | /api/workflows | List workflow mappings |
| GET | /api/discover | Discover servers (query params) |
| POST | /api/call | Proxy call to server |
| GET | /api/health | Check server health |
| GET | /api/tools/:server | List server tools |
| POST | /api/mcp | Execute MCP tool |

---

## File Structure

```
bop-gateway/
├── src/
│   ├── index.ts           # MCP entry point (stdio)
│   ├── types.ts           # TypeScript interfaces
│   ├── registry.ts        # 36 servers + workflow mappings
│   ├── tools/
│   │   ├── index.ts       # Tool exports
│   │   ├── discover.ts    # gateway_discover
│   │   ├── call.ts        # gateway_call (with security)
│   │   ├── health.ts      # gateway_health
│   │   ├── tools.ts       # gateway_tools
│   │   └── mcp.ts         # gateway_mcp
│   └── http/
│       └── server.ts      # REST API (port 8038)
├── dist/                  # Compiled JavaScript
├── package.json
├── tsconfig.json
└── README.md
```

---

## Dependencies

```json
{
  "@modelcontextprotocol/sdk": "^1.0.0",
  "express": "^4.18.2",
  "zod": "^3.22.4"
}
```

**Dev:**
- typescript: ^5.3.0
- @types/express, @types/node

**Not Used:** better-sqlite3 (no persistence), ws (no WebSocket)

---

## Security Features

| Feature | Implementation |
|---------|----------------|
| Server validation | Alphanumeric, hyphens, underscores only |
| Registry whitelist | Only registered servers allowed |
| Path traversal | Blocks `..`, encoded variants |
| Payload limit | 10MB maximum |
| Timeout | 30 seconds per request |

---

## Usage

**Build:**
```bash
cd /repo/bop-gateway
npm install
npm run build
```

**Start HTTP server:**
```bash
node dist/http/server.js
```

**Test MCP:**
```bash
echo '{"jsonrpc":"2.0","id":1,"method":"tools/list"}' | node dist/index.js
```

**Claude Desktop config:**
```json
{
  "mcpServers": {
    "bop-gateway": {
      "command": "node",
      "args": ["/repo/bop-gateway/dist/index.js"]
    }
  }
}
```

---

## Token Economics

| Configuration | Tools | Tokens (est.) |
|---------------|-------|---------------|
| All 36 servers | 450+ | ~150,000 |
| bop-gateway only | 5 | ~3,000 |
| **Savings** | - | **~98%** |

---

## Design Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Single vs multiple gateways | Single | All servers reachable; workflows are filters |
| 4-layer vs 2-layer | 2-layer | Edge adapter, not ecosystem server |
| Mesh participation | None | Dumb proxy; no InterLock overhead |
| Persistence | None | Stateless; no SQLite needed |
| Cognitive integration | Via proxy | Call cognitive servers through gateway_call |

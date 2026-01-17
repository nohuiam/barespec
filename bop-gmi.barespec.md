SERVER: bop-gmi
VERSION: 1.0.0
UPDATED: 2026-01-04
STATUS: Production
PORT: 3099 (Control API), 5173 (Vite Frontend)
MCP: none (web application)
DEPS: none
PURPOSE: AI-optimized visual dashboard for ecosystem monitoring and cognitive orchestration
CONFIG: /repo/bop-gmi/state.json

---

ARCHITECTURE

TYPE: Web Application (not MCP server)
FRONTEND: Vite + vanilla JS (port 5173)
BACKEND: Express.js Control API (port 3099)
STATE: JSON file (state.json)
ENCODING: GMVP (Graphical Machine Visual Protocol)

COGNITIVE LOOP:
GMI Screenshot → consciousness-mcp reflects → experience-layer records
       ↑                                              ↓
       └──────── lessons inform future actions ←──────┘

---

VIEWS (2)

VIEW: Grid
URL: http://localhost:5173/?view=grid
PURPOSE: Server cards with detailed health information
LAYOUT: Responsive grid, 3-4 cards per row
DATA: Full server metrics per card

VIEW: Matrix
URL: http://localhost:5173/?view=matrix
PURPOSE: All 31 servers in single screenshot (no scrolling)
LAYOUT: 8x4 matrix + 6 data panels
DATA: Compressed GMVP encoding per cell

---

GMVP ENCODING

STATUS GLYPHS:
● (U+25CF) = healthy
◐ (U+25D0) = degraded
◉ (U+25C9) = critical
○ (U+25CB) = offline
? (U+003F) = unknown

COLOR ENCODING (backgrounds):
#00FF00 (bright green)  = healthy
#CCFF00 (yellow-green)  = degraded
#FF9900 (orange)        = critical
#FF0000 (red)           = offline
#666666 (gray)          = unknown

CELL FORMAT (24 chars x 2 lines):
Line 1: ID status port tools uptime CPU% Mem%
Line 2: Response(ms) Errors Alerts Mesh Deps

SERVER ABBREVIATIONS (31):
CG=context-guardian, QM=quartermaster, SN=snapshot, TR=tool-registry
CT=catasorter, SFO=smart-file-organizer, BBB=bonzai-bloat-buster
ES=enterspect, NE=neurogenesis-engine, TC=trinity-coordinator
CCB=claude-code-bridge, PC=project-context, KC=knowledge-curator
PKM=pk-manager, IR=intelligent-router, VM=verifier-mcp
SBP=safe-batch-processor, IG=intake-guardian, HM=health-monitor
SR=synapse-relay, FG=filesystem-guardian, CE=consolidation-engine
TS=tenets-server, CM=consciousness-mcp, SB=skill-builder
PS=percolation-server, EL=experience-layer, LK=looker
XS=chronos-synapse, NW=niws-server, RB=research-bus

---

DATA PANELS (6)

PANEL: Ports
DATA: HTTP port listing (8001-8032)
FORMAT: Port numbers with status indicators

PANEL: External Deps
DATA: Redis (6379), Qdrant (6333), Comet (9222)
FORMAT: Service name + status glyph

PANEL: Category Summary
DATA: Server counts by category
CATEGORIES: Safety, Files, Backup, Class, Build, Coordinate, Discover, Knowledge, Route, Verify, Batch, Intake, Monitor, Relay, FS, Cognitive

PANEL: Mesh Connections
DATA: InterLock UDP peer counts
FORMAT: Connected/Total peers per server

PANEL: Response Times
DATA: HTTP health endpoint latency
FORMAT: Color-coded bars (green <100ms, yellow <500ms, red >500ms)

PANEL: Resource Gauges
DATA: CPU% and Memory% per server
FORMAT: Dual horizontal bars per server

---

HTTP REST API (Control API - port 3099)

GET  /api/state              → Full application state
POST /api/refresh/rate       → { ms: 5000|10000|30000|60000|"manual" }
POST /api/tool/:id/state     → { state: "on"|"off"|"standby" }
POST /api/tool/:id/intensity → { value: 0-100 }
POST /api/view/filter        → { status: string[], category: string[] }
POST /api/mesh/mode          → { mode: "text"|"visual"|"both" }
GET  /api/health/:serverId   → Proxy to server health endpoint

---

STATE SCHEMA

```json
{
  "refreshRate": 10000,
  "meshMode": "both",
  "viewFilter": { "status": [], "category": [] },
  "tools": {
    "server_tool_name": { "state": "on", "intensity": 75 }
  },
  "servers": {
    "server-name": { "enabled": true }
  }
}
```

---

SERVERS MONITORED (31)

| Category | Servers |
|----------|---------|
| Safety | context-guardian, intake-guardian |
| Files | quartermaster, smart-file-organizer, filesystem-guardian |
| Backup | snapshot |
| Classification | catasorter |
| Build | bonzai-bloat-buster, consolidation-engine |
| Coordination | trinity-coordinator, synapse-relay |
| Discovery | enterspect, looker |
| Generation | neurogenesis-engine |
| Knowledge | knowledge-curator, pk-manager, project-context |
| Routing | intelligent-router, research-bus |
| Verification | verifier-mcp, tenets-server |
| Batch | safe-batch-processor |
| Monitor | health-monitor |
| Cognitive | consciousness-mcp, skill-builder, percolation-server, experience-layer |
| Bridge | claude-code-bridge |
| Utility | chronos-synapse, niws-server, tool-registry |

---

KEY FILES

SOURCE: /repo/bop-gmi/
INDEX: index.html
ENTRY: src/main.js
STYLES: src/style.css
BACKEND: server.js
STATE: state.json
DATA: src/data/servers.js, src/data/abbreviations.js
UTILS: src/utils/api.js, src/utils/gmvp.js

DEPENDENCIES: express, vite

---

COGNITIVE INTEGRATION

EXPERIENCE RECORDING:
```bash
curl -X POST http://localhost:8031/api/experience -d '{
  "operation_type": "gmi_observation",
  "server": "gmi-matrix",
  "outcome": "success",
  "utility": 0.85
}'
```

CONSCIOUSNESS REFLECTION:
```bash
curl -X POST http://localhost:8028/api/reflect -d '{
  "input": "GMI matrix shows 28 healthy, 3 offline",
  "context": "ecosystem_monitoring"
}'
```

VERIFIED: Episode #131 recorded as gmi_observation (utility: 0.9)

---

GITHUB

REPO: github.com/nohuiam/bop-gmi
CREATED: 2026-01-04
BRANCH: main

---

# BARESPEC TEMPLATE v1.0

**Created:** 2025-12-26
**Purpose:** Standard template for all BOP/Imminence OS server documentation
**Based on:** context-guardian.barespec.md (validated 100% accurate)

---

## REQUIRED SECTIONS

Every barespec MUST include these sections in this order:

---

### HEADER (Required)

```
SERVER: {server-name}
VERSION: {semver}
UPDATED: {YYYY-MM-DD}
STATUS: {Development|Testing|Production}
PORT: {UDP} (UDP/InterLock), {HTTP} (HTTP), {WS} (WebSocket)
MCP: stdio transport (stdin/stdout JSON-RPC)
DEPS: {dependencies, comma-separated}
PURPOSE: {one-line description}
CONFIG: {path to interlock.json}
```

**Port Pattern:** Server N uses 300N (UDP), 800N (HTTP), 900N (WS)

---

### ARCHITECTURE (Required)

```
---

ARCHITECTURE

DATABASE: {database type}
WORKFLOW: {architecture description}
{Additional architecture notes as needed}

---
```

---

### TOOLS (Required)

```
---

TOOLS ({count})

TOOL: {server_prefix}_{tool_name}
INPUT: { param: type (required|optional), ... }
OUTPUT: { field: type, ... }
USE: {one-line description of when to use}
EXAMPLE: {tool_name}({ example_params })

{Repeat for each tool}

---
```

**Naming Convention:** All tools MUST be prefixed with `{server_name}_`

**Input Format:**
- Required params: `param: type (required)`
- Optional params: `param?: type` or `param: type (optional)`
- Enums: `param: "value1"|"value2"|"value3"`
- Objects: `param: { nested_field: type }`

**Output Format:**
- Simple: `{ field: type, ... }`
- Arrays: `{ items: [], total: number }`

---

### LAYERS (Required for InterLock servers)

```
---

LAYERS

1. MCP stdio ({tool_count} tools) - Primary interface
2. InterLock UDP mesh (port {UDP}) - Peer communication
3. HTTP REST API (port {HTTP}) - External integrations
4. WebSocket real-time (port {WS}) - Live updates

---
```

---

### SERVICES (Recommended)

```
---

SERVICES

- {ServiceName}: {description}
- {ServiceName}: {description}
...

---
```

---

### KEY FILES (Required)

```
---

KEY FILES

SOURCE: {repo path}
INDEX: {entry point file}
SERVICES: {services directory}
INTERLOCK: {interlock module path}
HTTP: {http server path}
WEBSOCKET: {websocket path}
DATABASE: {database files}
CONFIG: {config files}

DEPENDENCIES: {npm packages, comma-separated}

```

---

## TOOLS BARESPEC TEMPLATE

For `{server}-tools.barespec.md` files:

```
SERVER: {server-name}
VERSION: {semver}
UPDATED: {YYYY-MM-DD}
TOOLS: {count}

---

## {CATEGORY NAME} ({count})

### {tool_name}

**Purpose:** {description}

**Input Schema:**
```json
{
  "param1": "type (required)",
  "param2": "type (optional)"
}
```

**Output Schema:**
```json
{
  "field1": "type",
  "field2": "type"
}
```

**Example:**
```json
{tool_name}({
  "param1": "value"
})
```

---

{Repeat for each tool, grouped by category}
```

---

## VALIDATION CHECKLIST

Before a barespec is complete:

- [ ] SERVER name matches code
- [ ] VERSION matches package.json
- [ ] UPDATED is current date
- [ ] PORT assignments match config/interlock.json
- [ ] Tool count matches actual code implementation
- [ ] All tool names match code exactly
- [ ] Input schemas match Zod schemas in code
- [ ] Output schemas match actual responses
- [ ] All services listed exist in src/services/
- [ ] KEY FILES paths are accurate
- [ ] DEPENDENCIES match package.json

---

## EXAMPLE: Minimal Barespec

```
SERVER: example-server
VERSION: 1.0
UPDATED: 2025-12-26
STATUS: Production
PORT: 3099 (UDP/InterLock), 8099 (HTTP), 9099 (WebSocket)
MCP: stdio transport (stdin/stdout JSON-RPC)
DEPS: none
PURPOSE: Example server for demonstration
CONFIG: /repo/example-server/config/interlock.json

---

ARCHITECTURE

DATABASE: SQLite (better-sqlite3)
WORKFLOW: 4-layer architecture (MCP stdio, InterLock UDP, HTTP REST, WebSocket)

---

TOOLS (1)

TOOL: example_hello
INPUT: { name: string (required) }
OUTPUT: { greeting: string }
USE: Say hello to someone
EXAMPLE: example_hello({ name: "World" })

---

LAYERS

1. MCP stdio (1 tool) - Primary interface
2. InterLock UDP mesh (port 3099) - Peer communication
3. HTTP REST API (port 8099) - External integrations
4. WebSocket real-time (port 9099) - Live updates

---

KEY FILES

SOURCE: /repo/example-server/
INDEX: src/index.js
INTERLOCK: src/interlock/socket.js
HTTP: src/http/server.js
WEBSOCKET: src/websocket/server.js
CONFIG: config/interlock.json

DEPENDENCIES: better-sqlite3, @modelcontextprotocol/sdk, express, ws, zod

```

---

## HTTP-ONLY SERVERS

For servers without InterLock mesh (looker, chronos-synapse, niws-server, research-bus):

- Omit `PORT: {UDP}` from header (keep HTTP only)
- Omit LAYERS section or note "HTTP-only"
- Omit InterLock from KEY FILES

---

## NOTES

1. **Dense Format:** Barespec is optimized for Claude consumption - no prose, just structured data
2. **Machine-Readable:** Consistent formatting allows programmatic parsing
3. **Single Source of Truth:** Barespec should be authoritative - code changes require barespec updates
4. **Versioning:** Bump VERSION when tools are added/removed/modified

# percolation-server.barespec.md

## Server Identity
- **Name:** percolation-server
- **Version:** 1.1.0
- **Updated:** 2026-01-16
- **Port Pattern:** 3030 (UDP) / 8030 (HTTP) / 9030 (WebSocket)
- **Path:** `/repo/percolation-server/`
- **GitHub:** https://github.com/nohuiam/percolation-server

## Purpose

Blueprint optimization through stress testing and hole patching. The final stage before skill/pathway execution in the BOP/Imminence cognitive architecture.

**Core Principle:** "No pathway executes until it's been stress-tested and optimized."

**Core Functions:**
1. **Submit** - Receive blueprints for percolation
2. **Stress Test** - Run adversarial scenarios against blueprints
3. **Find Holes** - Identify weaknesses and gaps
4. **Patch** - Apply fixes to identified holes
5. **Optimize** - Research and apply improvements
6. **Report** - Return optimized blueprint with confidence score

## Research Foundation

From cognitive architecture vision: Once a blueprint is designed, it sits in the Percolator which:
1. Researches improvements via Research Bus MCP server
2. Stress tests with constraint and verification engines
3. Plays devil's advocate — shoots holes in the blueprint
4. Patches holes until the pathway is bulletproof

**Key insight:** Scalable resource management - percolation depth scales with available budget/tokens.

### Depth Levels

| Depth | Budget | Stress Tests | Research Queries |
|-------|--------|--------------|------------------|
| quick | 1000 | 3 basic | 1 |
| standard | 5000 | 10 varied | 3 |
| thorough | 20000 | 25 comprehensive | 10 |
| exhaustive | 100000 | 100+ adversarial | unlimited |

## 4-Layer Architecture

| Layer | Transport | Port | Purpose |
|-------|-----------|------|---------|
| MCP | stdio | stdin/stdout | Claude Desktop / skill-builder interface |
| InterLock | UDP | 3030 | Mesh signals |
| HTTP | REST | 8030 | API access |
| WebSocket | WS | 9030 | Real-time percolation progress |

## MCP Tools (8)

### 1. submit_blueprint
Submit a skill/pathway blueprint for percolation.

**Input:**
```typescript
{
  blueprint: string;           // Required: blueprint content
  depth?: 'quick' | 'standard' | 'thorough' | 'exhaustive';  // Default: 'standard'
  budget_tokens?: number;      // Custom budget override
  source?: string;             // Origin identifier
  metadata?: Record<string, unknown>;
}
```

**Output:**
```typescript
{
  blueprint_id: string;
  status: 'pending' | 'percolating';
  depth: string;
  budget_tokens: number;
  submitted_at: number;
}
```

### 2. get_blueprint_status
Check percolation status of a blueprint.

**Input:**
```typescript
{
  blueprint_id: string;        // Required
}
```

**Output:**
```typescript
{
  blueprint_id: string;
  status: 'pending' | 'percolating' | 'completed' | 'failed';
  depth: string;
  budget_remaining: number;
  holes_found: number;
  holes_patched: number;
  stress_tests_run: number;
  optimizations_applied: number;
  confidence_score?: number;
  submitted_at: number;
  completed_at?: number;
}
```

### 3. stress_test
Run adversarial tests against a blueprint.

**Input:**
```typescript
{
  blueprint_id: string;        // Required
  test_type?: 'edge_case' | 'error_condition' | 'resource_exhaustion' |
              'malformed_input' | 'concurrency' | 'security';
  intensity?: 'low' | 'medium' | 'high';  // Default: 'medium'
  custom_scenarios?: string[];
}
```

**Output:**
```typescript
{
  test_id: string;
  blueprint_id: string;
  test_type: string;
  intensity: string;
  passed: boolean;
  findings: string[];
  vulnerabilities: Array<{
    severity: 'low' | 'medium' | 'high' | 'critical';
    description: string;
    location?: string;
  }>;
  run_at: number;
}
```

### 4. find_holes
Identify weaknesses and gaps in a blueprint.

**Input:**
```typescript
{
  blueprint_id: string;        // Required
  analysis_depth?: 'surface' | 'deep' | 'exhaustive';  // Default: 'deep'
  focus_areas?: string[];      // Specific areas to analyze
}
```

**Output:**
```typescript
{
  blueprint_id: string;
  holes: Array<{
    hole_id: string;
    hole_type: 'logic_gap' | 'missing_step' | 'ambiguity' |
               'error_handling' | 'edge_case' | 'dependency';
    description: string;
    severity: 'low' | 'medium' | 'high' | 'critical';
    location?: string;
    suggested_fix?: string;
  }>;
  analysis_depth: string;
  analyzed_at: number;
}
```

### 5. patch_hole
Apply a fix to an identified hole.

**Input:**
```typescript
{
  blueprint_id: string;        // Required
  hole_id: string;             // Required
  patch: string;               // Required: fix content
  verification_notes?: string;
}
```

**Output:**
```typescript
{
  patched: boolean;
  blueprint_id: string;
  hole_id: string;
  patch_applied: string;
  new_content_hash: string;
  patched_at: number;
}
```

### 6. research_improvements
Query research-bus for optimization ideas.

**Input:**
```typescript
{
  blueprint_id: string;        // Required
  focus_areas?: string[];      // Areas to research
  max_queries?: number;        // Limit research queries
}
```

**Output:**
```typescript
{
  blueprint_id: string;
  improvements: Array<{
    optimization_id: string;
    source: string;
    description: string;
    estimated_improvement: number;  // 0-1 score
    applicability: number;          // 0-1 score
  }>;
  researched_at: number;
}
```

### 7. apply_optimization
Apply a researched improvement to a blueprint.

**Input:**
```typescript
{
  blueprint_id: string;        // Required
  optimization_id: string;     // Required
  custom_application?: string; // Override default application
}
```

**Output:**
```typescript
{
  applied: boolean;
  blueprint_id: string;
  optimization_id: string;
  improvement_score: number;
  new_content_hash: string;
  applied_at: number;
}
```

### 8. get_percolation_result
Get final optimized blueprint with confidence score.

**Input:**
```typescript
{
  blueprint_id: string;        // Required
  include_history?: boolean;   // Include full percolation history
}
```

**Output:**
```typescript
{
  blueprint_id: string;
  status: 'completed' | 'failed' | 'in_progress';
  original_content: string;
  optimized_content: string;
  confidence_score: number;    // 0-1 score
  summary: {
    holes_found: number;
    holes_patched: number;
    stress_tests_passed: number;
    stress_tests_failed: number;
    optimizations_applied: number;
    budget_used: number;
  };
  history?: PercolationLogEntry[];
  completed_at?: number;
}
```

## Database Schema (SQLite)

### blueprints
```sql
CREATE TABLE blueprints (
  id TEXT PRIMARY KEY,
  original_content TEXT NOT NULL,
  current_content TEXT NOT NULL,
  status TEXT DEFAULT 'pending',
  depth TEXT DEFAULT 'standard',
  budget_tokens INTEGER DEFAULT 5000,
  budget_remaining INTEGER,
  source TEXT,
  metadata TEXT,                    -- JSON
  confidence_score REAL,
  submitted_at INTEGER NOT NULL,
  completed_at INTEGER
);

CREATE INDEX idx_blueprints_status ON blueprints(status);
```

### holes
```sql
CREATE TABLE holes (
  id TEXT PRIMARY KEY,
  blueprint_id TEXT NOT NULL,
  hole_type TEXT NOT NULL,
  description TEXT NOT NULL,
  severity TEXT NOT NULL,
  location TEXT,
  suggested_fix TEXT,
  status TEXT DEFAULT 'open',
  patch TEXT,
  identified_at INTEGER NOT NULL,
  patched_at INTEGER,
  FOREIGN KEY (blueprint_id) REFERENCES blueprints(id)
);

CREATE INDEX idx_holes_blueprint ON holes(blueprint_id);
CREATE INDEX idx_holes_status ON holes(status);
```

### stress_tests
```sql
CREATE TABLE stress_tests (
  id TEXT PRIMARY KEY,
  blueprint_id TEXT NOT NULL,
  test_type TEXT NOT NULL,
  intensity TEXT DEFAULT 'medium',
  passed INTEGER NOT NULL,
  findings TEXT,                    -- JSON array
  vulnerabilities TEXT,             -- JSON array
  run_at INTEGER NOT NULL,
  FOREIGN KEY (blueprint_id) REFERENCES blueprints(id)
);

CREATE INDEX idx_stress_tests_blueprint ON stress_tests(blueprint_id);
```

### optimizations
```sql
CREATE TABLE optimizations (
  id TEXT PRIMARY KEY,
  blueprint_id TEXT NOT NULL,
  source TEXT NOT NULL,
  description TEXT NOT NULL,
  improvement_score REAL,
  applicability REAL,
  applied INTEGER DEFAULT 0,
  applied_at INTEGER,
  FOREIGN KEY (blueprint_id) REFERENCES blueprints(id)
);

CREATE INDEX idx_optimizations_blueprint ON optimizations(blueprint_id);
```

### percolation_logs
```sql
CREATE TABLE percolation_logs (
  id TEXT PRIMARY KEY,
  blueprint_id TEXT NOT NULL,
  action TEXT NOT NULL,
  details TEXT,                     -- JSON
  budget_used INTEGER DEFAULT 0,
  timestamp INTEGER NOT NULL,
  FOREIGN KEY (blueprint_id) REFERENCES blueprints(id)
);

CREATE INDEX idx_logs_blueprint ON percolation_logs(blueprint_id);
CREATE INDEX idx_logs_action ON percolation_logs(action);
```

## HTTP REST API (Port 8030)

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/health` | Server health with active percolations |
| GET | `/api/tools` | List all MCP tools (Gateway integration) |
| POST | `/api/tools/:toolName` | Execute MCP tool (Gateway integration) |
| GET | `/api/stats` | Percolation statistics |
| POST | `/api/blueprints` | Submit blueprint for percolation |
| GET | `/api/blueprints` | List all blueprints |
| GET | `/api/blueprints/:id` | Get blueprint details + status |
| GET | `/api/blueprints/:id/holes` | Get holes for blueprint |
| GET | `/api/blueprints/:id/tests` | Get stress test results |
| GET | `/api/blueprints/:id/optimizations` | Get applied optimizations |
| GET | `/api/blueprints/:id/logs` | Get percolation audit trail |
| POST | `/api/blueprints/:id/stress` | Trigger manual stress test |
| POST | `/api/blueprints/:id/analyze` | Trigger hole analysis |
| POST | `/api/blueprints/:id/patch` | Apply a patch |
| POST | `/api/blueprints/:id/research` | Research improvements |
| POST | `/api/blueprints/:id/optimize` | Apply optimization |
| GET | `/api/blueprints/:id/result` | Get final result |

## WebSocket Events (Port 9030)

| Direction | Event | Data |
|-----------|-------|------|
| S→C | `percolation_started` | `{ blueprint_id, depth, budget }` |
| S→C | `stress_test_running` | `{ blueprint_id, test_type, intensity }` |
| S→C | `stress_test_complete` | `{ blueprint_id, test_id, passed, findings }` |
| S→C | `hole_found` | `{ blueprint_id, hole_id, severity, description }` |
| S→C | `hole_patched` | `{ blueprint_id, hole_id }` |
| S→C | `research_complete` | `{ blueprint_id, improvements_count }` |
| S→C | `optimization_applied` | `{ blueprint_id, optimization_id, score }` |
| S→C | `percolation_complete` | `{ blueprint_id, confidence, summary }` |
| S→C | `percolation_failed` | `{ blueprint_id, reason }` |
| C↔S | `ping/pong` | `{ timestamp }` |

## InterLock Mesh (Port 3030)

### Listens To

| Signal | Code | From | Action |
|--------|------|------|--------|
| SKILL_CREATED | 0xA0 | skill-builder | Auto-submit for percolation |
| VERIFICATION_RESULT | 0xD0 | verifier-mcp | Update confidence scores |
| PATTERN_DETECTED | 0xE1 | consciousness-mcp | Incorporate into stress tests |
| PATTERN_EMERGED | 0xF1 | experience-layer | Learn from past failures |
| HEARTBEAT | 0x00 | * | Track server availability |

### Emits

| Signal | Code | To | When |
|--------|------|----|------|
| PERCOLATION_STARTED | 0xB0 | trinity, consciousness | Blueprint submitted |
| HOLE_FOUND | 0xB1 | consciousness, experience-layer | Weakness identified |
| HOLE_PATCHED | 0xB2 | consciousness | Fix applied |
| PERCOLATION_COMPLETE | 0xB3 | skill-builder, trinity | Optimization done |
| PERCOLATION_FAILED | 0xB4 | skill-builder, trinity | Could not optimize |

### Peer Configuration

```json
{
  "peers": [
    { "name": "skill-builder", "port": 3029 },
    { "name": "consciousness", "port": 3028 },
    { "name": "experience-layer", "port": 3031 },
    { "name": "verifier-mcp", "port": 3021 },
    { "name": "trinity-coordinator", "port": 3012 },
    { "name": "research-bus", "port": 8019 }
  ]
}
```

## Percolation Algorithm

```
1. RECEIVE blueprint + depth + budget
2. VALIDATE structure (schema check)
3. LOOP while budget > 0 AND (holes_found OR tests_failing):
   a. STRESS_TEST with adversarial scenarios
   b. FIND_HOLES in results
   c. IF holes found:
      - RESEARCH improvements via research-bus (HTTP)
      - PATCH holes with best improvement
      - Deduct budget
   d. LOG all actions
4. CALCULATE final confidence score:
   - Base: 0.5
   - +0.1 for each stress test passed
   - -0.1 for each unpatchable hole
   - +0.05 for each optimization applied
   - Cap at 0.95
5. EMIT optimized blueprint + report
```

## File Structure

```
percolation-server/
├── config/
│   └── interlock.json
├── src/
│   ├── index.ts                 # MCP entry point
│   ├── types.ts                 # Type definitions
│   ├── database/
│   │   └── schema.ts            # SQLite + CRUD
│   ├── percolator/
│   │   ├── engine.ts            # Main percolation loop
│   │   ├── stress-tester.ts     # Adversarial test runner
│   │   ├── hole-finder.ts       # Weakness analyzer
│   │   └── optimizer.ts         # Improvement applicator
│   ├── tools/
│   │   ├── index.ts
│   │   ├── submit-blueprint.ts
│   │   ├── get-blueprint-status.ts
│   │   ├── stress-test.ts
│   │   ├── find-holes.ts
│   │   ├── patch-hole.ts
│   │   ├── research-improvements.ts
│   │   ├── apply-optimization.ts
│   │   └── get-percolation-result.ts
│   ├── interlock/
│   │   ├── index.ts
│   │   ├── socket.ts
│   │   ├── protocol.ts
│   │   ├── handlers.ts
│   │   └── tumbler.ts
│   ├── http/
│   │   └── server.ts
│   └── websocket/
│       └── server.ts
├── tests/
│   ├── setup.ts
│   ├── database.test.ts
│   ├── tools.test.ts
│   └── percolator.test.ts
├── package.json
├── tsconfig.json
└── jest.config.js
```

## Dependencies

```json
{
  "@modelcontextprotocol/sdk": "^1.0.0",
  "better-sqlite3": "^11.7.0",
  "express": "^4.18.2",
  "ws": "^8.14.2",
  "zod": "^3.22.4",
  "uuid": "^9.0.0",
  "cors": "^2.8.5"
}
```

## Usage

### Start Server
```bash
cd /repo/percolation-server
npm start
```

### Build
```bash
npm run build
```

### Test
```bash
npm test
```

## Related Servers

| Server | Relationship |
|--------|--------------|
| skill-builder (3029) | Submits blueprints, receives optimized versions |
| consciousness-mcp (3028) | Receives hole/patch signals for pattern learning |
| experience-layer (3031) | Provides past failure patterns |
| verifier-mcp (3021) | Verification signals update confidence |
| research-bus (8019) | HTTP queries for improvement research |
| trinity-coordinator (3012) | Receives percolation completion signals |

## Test Coverage

- **81 tests** total
- Database operations: 25 tests
- Tool handlers: 32 tests
- Percolator engine: 24 tests

## Key Types

```typescript
interface Blueprint {
  id: string;
  original_content: string;
  current_content: string;
  status: 'pending' | 'percolating' | 'completed' | 'failed';
  depth: 'quick' | 'standard' | 'thorough' | 'exhaustive';
  budget_tokens: number;
  budget_remaining: number;
  source?: string;
  metadata?: Record<string, unknown>;
  confidence_score?: number;
  submitted_at: number;
  completed_at?: number;
}

interface Hole {
  id: string;
  blueprint_id: string;
  hole_type: HoleType;
  description: string;
  severity: Severity;
  location?: string;
  suggested_fix?: string;
  status: 'open' | 'patched' | 'wont_fix';
  patch?: string;
  identified_at: number;
  patched_at?: number;
}

interface StressTest {
  id: string;
  blueprint_id: string;
  test_type: TestType;
  intensity: 'low' | 'medium' | 'high';
  passed: boolean;
  findings: string[];
  vulnerabilities: Vulnerability[];
  run_at: number;
}

interface Optimization {
  id: string;
  blueprint_id: string;
  source: string;
  description: string;
  improvement_score: number;
  applicability: number;
  applied: boolean;
  applied_at?: number;
}

interface PercolationResult {
  blueprint_id: string;
  status: 'completed' | 'failed' | 'in_progress';
  original_content: string;
  optimized_content: string;
  confidence_score: number;
  summary: PercolationSummary;
  history?: PercolationLogEntry[];
  completed_at?: number;
}

type HoleType = 'logic_gap' | 'missing_step' | 'ambiguity' |
               'error_handling' | 'edge_case' | 'dependency';

type TestType = 'edge_case' | 'error_condition' | 'resource_exhaustion' |
               'malformed_input' | 'concurrency' | 'security';

type Severity = 'low' | 'medium' | 'high' | 'critical';
```

## Confidence Score Calculation

```typescript
function calculateConfidence(blueprint: Blueprint): number {
  let score = 0.5;  // Base score

  const tests = getStressTests(blueprint.id);
  const passed = tests.filter(t => t.passed).length;
  const failed = tests.filter(t => !t.passed).length;
  score += passed * 0.05;
  score -= failed * 0.1;

  const holes = getHoles(blueprint.id);
  const patched = holes.filter(h => h.status === 'patched').length;
  const open = holes.filter(h => h.status === 'open').length;
  score += patched * 0.03;
  score -= open * 0.15;

  const optimizations = getOptimizations(blueprint.id);
  const applied = optimizations.filter(o => o.applied).length;
  score += applied * 0.02;

  return Math.max(0, Math.min(0.95, score));  // Cap at 0-0.95
}
```

## Cognitive Architecture Role

```
┌─────────────────────────────────────────────────────────────────┐
│                    COGNITIVE FLOW                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  consciousness-mcp (3028)                                        │
│       │ patterns                                                 │
│       ▼                                                          │
│  skill-builder (3029) ──blueprint──► PERCOLATION-SERVER (3030)  │
│       ▲                                    │                     │
│       │                                    │ stress test         │
│       │                                    │ find holes          │
│       │                                    │ patch               │
│       │                                    │ optimize            │
│       │                                    ▼                     │
│       └────optimized blueprint────────────┘                      │
│                                                                  │
│  experience-layer (3031)                                         │
│       │ past failures inform hole detection                      │
│       ▼                                                          │
│  verifier-mcp (3021)                                             │
│       │ verification updates confidence                          │
│       ▼                                                          │
│  research-bus (8019)                                             │
│       │ HTTP queries for improvements                            │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

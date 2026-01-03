# experience-layer.barespec.md

## Server Identity
- **Name:** experience-layer
- **Version:** 1.0.0
- **Port Pattern:** 3031 (UDP) / 8031 (HTTP) / 9031 (WebSocket)
- **Path:** `/repo/experience-layer/`

## Purpose

Long-term experiential memory for the BOP/Imminence OS ecosystem. Records episodes with PSOM structure (Problem-Solution-Outcome-Metadata), detects patterns, extracts lessons, and tracks confidence with temporal decay.

**Core Functions:**
1. **Record** - Store episodes with full context and utility scoring
2. **Recall** - Find past experiences by type or outcome
3. **Learn** - Extract lessons from recurring patterns
4. **Apply** - Track lesson applications with Bayesian confidence updates

## Research Foundation

Based on 106 academic sources from Perplexity Deep Research on Case-Based Reasoning, Episodic Memory, Confidence Calibration, and Pattern Detection.

### Key Formulas

| Formula | Purpose |
|---------|---------|
| `CF(t) = CF₀ × e^(-kt)` | Temporal confidence decay |
| `U = 0.3×novelty + 0.5×effectiveness + 0.2×generalizability` | Utility-based retention |
| `w = success_rate × log(frequency+1) × e^(-k×days)` | Discrimination weighting |

### Constants

```typescript
PATTERN_CONFIG = {
  minEpisodes: 3,
  recencyWindowDays: 30,
  decayConstant: 0.01,
  minDiscriminationWeight: 0.3
}

CONFIDENCE_THRESHOLDS = {
  high: 0.7,
  medium: 0.4,
  deprecation: 0.1
}
```

## 4-Layer Architecture

| Layer | Transport | Port | Purpose |
|-------|-----------|------|---------|
| MCP | stdio | stdin/stdout | Claude Desktop interface |
| InterLock | UDP | 3031 | Mesh signals |
| HTTP | REST | 8031 | API access |
| WebSocket | WS | 9031 | Real-time events |

## MCP Tools (6)

### 1. record_experience
Store an episode with PSOM structure and utility scoring.

**Input:**
```typescript
{
  operation_type: string;        // Required: build, search, verify, etc.
  server_name?: string;          // Server that performed operation
  problem?: {                    // PSOM: Problem context
    query?: string;
    constraints?: Record<string, unknown>;
    context?: Record<string, unknown>;
  };
  solution?: {                   // PSOM: Solution applied
    tool?: string;
    params?: Record<string, unknown>;
    approach?: string;
  };
  outcome: 'success' | 'failure' | 'partial';  // Required
  metadata?: {                   // PSOM: Additional metadata
    environment?: string;
    dependencies?: string[];
    triggers?: string[];
  };
  quality_score?: number;        // 0-1
  duration_ms?: number;
  notes?: string;
}
```

**Output:**
```typescript
{
  episode_id: number;
  recorded: true;
  utility_score: number;
  patterns_triggered?: string[];
}
```

### 2. recall_by_type
Get past experiences for an operation type.

**Input:**
```typescript
{
  operation_type: string;                               // Required
  outcome_filter?: 'success' | 'failure' | 'partial';
  limit?: number;                                       // Default: 50
}
```

**Output:**
```typescript
{
  episodes: Episode[];
  count: number;
  patterns_detected?: Pattern[];
  avg_utility: number;
}
```

### 3. recall_by_outcome
Get experiences by outcome type.

**Input:**
```typescript
{
  outcome: 'success' | 'failure' | 'partial';  // Required
  operation_type?: string;
  limit?: number;
}
```

**Output:** Same as recall_by_type

### 4. get_lessons
Retrieve applicable lessons with current confidence.

**Input:**
```typescript
{
  context?: Record<string, unknown>;
  operation_type?: string;
  min_confidence?: number;    // Default: 0
}
```

**Output:**
```typescript
{
  lessons: Array<Lesson & { current_confidence: number }>;
  confidence_summary: {
    high: number;    // >= 0.7
    medium: number;  // 0.4 - 0.7
    low: number;     // < 0.4
  };
}
```

### 5. apply_lesson
Mark lesson as applied, update confidence with Bayesian update.

**Input:**
```typescript
{
  lesson_id: number;                           // Required
  outcome: 'success' | 'failure' | 'partial';  // Required
  notes?: string;
}
```

**Output:**
```typescript
{
  applied: true;
  lesson_id: number;
  previous_confidence: number;
  new_confidence: number;
  total_applications: number;
  success_rate: number;
}
```

### 6. learn_from_pattern
Extract a lesson from recurring pattern.

**Input:**
```typescript
{
  pattern_description: string;  // Required
  episode_ids: number[];        // Required, min 3
  lesson_statement: string;     // Required
}
```

**Output:**
```typescript
{
  lesson_id: number;
  created: true;
  initial_confidence: number;
  pattern_id?: number;
}
```

## Database Schema (SQLite)

### episodes
```sql
CREATE TABLE episodes (
  id INTEGER PRIMARY KEY,
  timestamp INTEGER NOT NULL,
  operation_type TEXT NOT NULL,
  server_name TEXT,
  problem TEXT,              -- JSON: PSOM Problem
  solution TEXT,             -- JSON: PSOM Solution
  outcome TEXT NOT NULL,     -- success|failure|partial
  metadata TEXT,             -- JSON: PSOM Metadata
  quality_score REAL,
  duration_ms INTEGER,
  notes TEXT,
  novelty_score REAL DEFAULT 0.5,
  effectiveness_score REAL DEFAULT 0.5,
  generalizability_score REAL DEFAULT 0.5,
  utility_score REAL DEFAULT 0.5
);
```

### patterns
```sql
CREATE TABLE patterns (
  id INTEGER PRIMARY KEY,
  pattern_type TEXT NOT NULL,      -- success|failure|correlation
  description TEXT NOT NULL,
  episode_ids TEXT,                -- JSON array
  frequency INTEGER DEFAULT 1,
  last_seen INTEGER,
  created_at INTEGER,
  initial_confidence REAL DEFAULT 0.5,
  decay_constant REAL DEFAULT 0.01,
  last_validated INTEGER,
  times_applied INTEGER DEFAULT 0,
  times_succeeded INTEGER DEFAULT 0,
  discrimination_weight REAL DEFAULT 0.5
);
```

### lessons
```sql
CREATE TABLE lessons (
  id INTEGER PRIMARY KEY,
  statement TEXT NOT NULL,
  pattern_id INTEGER,
  contexts TEXT,                   -- JSON array
  initial_confidence REAL DEFAULT 0.5,
  decay_constant REAL DEFAULT 0.01,
  last_validated INTEGER,
  times_applied INTEGER DEFAULT 0,
  times_succeeded INTEGER DEFAULT 0,
  created_at INTEGER,
  deprecated_at INTEGER,
  FOREIGN KEY (pattern_id) REFERENCES patterns(id)
);
```

## HTTP REST API (Port 8031)

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/health` | Server health with stats |
| GET | `/api/stats` | Episode/pattern/lesson counts |
| GET | `/api/episodes` | Recent episodes (paginated) |
| GET | `/api/episodes/type/:type` | Episodes by operation type |
| GET | `/api/episodes/outcome/:outcome` | Episodes by outcome |
| GET | `/api/episodes/:id` | Single episode |
| POST | `/api/experience` | Record new episode |
| GET | `/api/patterns` | All patterns |
| GET | `/api/patterns/type/:type` | Patterns by type |
| GET | `/api/patterns/:id` | Single pattern |
| GET | `/api/lessons` | Active lessons |
| GET | `/api/lessons/applicable` | Context-filtered lessons |
| POST | `/api/lessons/applicable` | POST with context body |
| GET | `/api/lessons/:id` | Single lesson |
| POST | `/api/lessons/:id/apply` | Apply lesson |
| POST | `/api/lessons/learn` | Create lesson from pattern |

## WebSocket Events (Port 9031)

| Direction | Event | Data |
|-----------|-------|------|
| S→C | `experience_recorded` | `{ episode_id, utility_score }` |
| S→C | `pattern_emerged` | `{ pattern_id, description }` |
| S→C | `lesson_extracted` | `{ lesson_id, statement, confidence }` |
| S→C | `lesson_validated` | `{ lesson_id, previous_confidence, new_confidence }` |
| S→C | `lesson_deprecated` | `{ lesson_id, reason }` |
| C↔S | `ping/pong` | `{ timestamp }` |

## InterLock Mesh (Port 3031)

### Listens To

| Signal | Code | From | Action |
|--------|------|------|--------|
| BUILD_COMPLETED | 0xB0 | neurogenesis | Record build episode |
| VERIFICATION_RESULT | 0xD0 | verifier | Record verification episode |
| VALIDATION_APPROVED | 0xC0 | context-guardian | Record success |
| VALIDATION_REJECTED | 0xC1 | context-guardian | Record failure |
| OPERATION_COMPLETE | 0xFF | * | Record operation episode |
| LESSON_LEARNED | 0xE5 | consciousness | Store lesson |
| CLAIM_VERIFIED | 0xD1 | verifier | Increase lesson confidence |
| CLAIM_REFUTED | 0xD2 | verifier | Decrease lesson confidence |
| HEARTBEAT | 0x00 | * | Log heartbeat |

### Emits

| Signal | Code | To | When |
|--------|------|----|------|
| EXPERIENCE_RECORDED | 0xF0 | consciousness | Episode stored |
| PATTERN_EMERGED | 0xF1 | consciousness, trinity | New pattern detected |
| LESSON_EXTRACTED | 0xF2 | * | New lesson created |
| LESSON_VALIDATED | 0xF3 | consciousness | Confidence updated |

### Peer Configuration

```json
{
  "peers": [
    { "name": "consciousness", "port": 3028 },
    { "name": "verifier", "port": 3021 },
    { "name": "context-guardian", "port": 3001 },
    { "name": "neurogenesis", "port": 3010 },
    { "name": "trinity", "port": 3012 }
  ]
}
```

## File Structure

```
experience-layer/
├── config/
│   └── interlock.json
├── src/
│   ├── index.ts              # MCP entry point
│   ├── types.ts              # Type definitions + formulas
│   ├── database/
│   │   └── schema.ts         # SQLite + CRUD
│   ├── tools/
│   │   ├── index.ts
│   │   ├── record-experience.ts
│   │   ├── recall-by-type.ts
│   │   ├── recall-by-outcome.ts
│   │   ├── get-lessons.ts
│   │   ├── apply-lesson.ts
│   │   └── learn-from-pattern.ts
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
│   ├── database.test.ts      # 21 tests
│   ├── tools.test.ts         # 29 tests
│   ├── interlock.test.ts     # 27 tests
│   └── integration.test.ts   # 15 tests
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
  "uuid": "^9.0.0"
}
```

## Usage

### Start Server
```bash
cd /repo/experience-layer
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
| consciousness-mcp (3028) | Sends LESSON_LEARNED, receives EXPERIENCE_RECORDED |
| verifier-mcp (3021) | Sends CLAIM_VERIFIED/REFUTED for confidence updates |
| context-guardian (3001) | Sends VALIDATION_APPROVED/REJECTED |
| neurogenesis-engine (3010) | Sends BUILD_COMPLETED |
| trinity-coordinator (3012) | Receives PATTERN_EMERGED |

## Test Coverage

- **92 tests** total
- Database operations: 21 tests
- Tool functions: 29 tests
- InterLock mesh: 27 tests
- Integration flows: 15 tests

## Research References

Source document: `/repo/GLECGUI_Imminence/perplexityreturns/CaseBasedReasoning1stResearch-FILTERED.md`

Key concepts incorporated:
1. CBR 4 REs Cycle: Retrieve → Reuse → Revise → Retain
2. PSOM Case Structure: Problem, Solution, Outcome, Metadata
3. Temporal Confidence Decay: CF(t) = CF₀ × e^(-kt)
4. Utility-Based Retention: U = α·novelty + β·effectiveness + γ·generalizability
5. Discrimination Weighting: success_rate × frequency_weight × recency_bonus

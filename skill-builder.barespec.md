# skill-builder.barespec.md

## Server Identity
- **Name:** skill-builder
- **Version:** 1.0.0
- **Port Pattern:** 3029 (UDP) / 8029 (HTTP) / 9029 (WebSocket)
- **Path:** `/repo/skill-builder/`

## Purpose

SKILL.md management for the BOP/Imminence OS ecosystem. Creates, validates, indexes, and matches skills following Anthropic's Claude Skills specification with progressive disclosure.

**Core Principle:** "MCP for access, Skills for expertise"

**Core Functions:**
1. **Create** - Generate SKILL.md files with proper structure
2. **Validate** - Check structure, frontmatter, and token counts
3. **Match** - Find skills by semantic task description matching
4. **Track** - Record skill usage and success rates

## Research Foundation

Based on Anthropic Claude Skills specification. Skills are YAML frontmatter + Markdown body files discovered automatically via semantic understanding.

### SKILL.md Structure

```yaml
---
name: skill-name
description: |
  When to use this skill, what it does,
  what triggers it, and boundaries/limitations
---

# Skill Name

## Overview
Brief explanation of what this skill accomplishes

## Prerequisites
- Required tools/access
- Required context

## Steps
1. First step
2. Second step
3. etc.

## Examples
Concrete, correct usage examples

## Error Handling
What to do when things go wrong

## Limitations
What this skill cannot or should not do
```

### Progressive Disclosure (3 Layers)

| Layer | Size | Purpose | When Loaded |
|-------|------|---------|-------------|
| 1 | ~100 tokens | Metadata (name, description) | Always - for matching |
| 2 | <5000 tokens | Full SKILL.md instructions | When skill triggered |
| 3 | Variable | Bundled files from references/ | As needed during execution |

### Constants

```typescript
SKILL_CONFIG = {
  LAYER1_MAX_TOKENS: 100,
  LAYER2_MAX_TOKENS: 5000,
  MIN_DESCRIPTION_LENGTH: 20,
  MAX_DESCRIPTION_LENGTH: 500,
  MIN_MATCH_CONFIDENCE: 0.3,
  HIGH_MATCH_CONFIDENCE: 0.7
}
```

## 4-Layer Architecture

| Layer | Transport | Port | Purpose |
|-------|-----------|------|---------|
| MCP | stdio | stdin/stdout | Claude Desktop interface |
| InterLock | UDP | 3029 | Mesh signals |
| HTTP | REST | 8029 | API access |
| WebSocket | WS | 9029 | Real-time events |

## MCP Tools (8)

### 1. create_skill
Generate a SKILL.md file following Anthropic's specification.

**Input:**
```typescript
{
  name: string;                    // Required: skill identifier
  description: string;             // Required: trigger description
  overview: string;                // Required: brief explanation
  steps: string[];                 // Required: ordered instructions
  examples?: string[];             // Example usage
  prerequisites?: string[];        // Required tools/context
  error_handling?: string;         // Error guidance
  limitations?: string;            // Scope boundaries
  tags?: string[];                 // Classification tags
}
```

**Output:**
```typescript
{
  skill_id: string;
  path: string;
  created: true;
  token_count: {
    layer1: number;
    layer2: number;
  };
}
```

### 2. validate_skill
Check SKILL.md structure, frontmatter, and token counts per layer.

**Input:**
```typescript
{
  skill_id?: string;               // ID of existing skill
  path?: string;                   // File path to validate
  content?: string;                // Raw content to validate
}
```

**Output:**
```typescript
{
  valid: boolean;
  errors: string[];
  warnings: string[];
  token_counts: {
    layer1: number;
    layer2: number;
  };
  progressive_disclosure_ok: boolean;
  description_analysis?: {
    clarity_score: number;
    suggestions: string[];
  };
}
```

### 3. analyze_description
Evaluate description field for semantic triggering effectiveness.

**Input:**
```typescript
{
  description: string;             // Required: description to analyze
  intended_triggers?: string[];    // Expected trigger words
}
```

**Output:**
```typescript
{
  clarity_score: number;           // 0-1 score
  trigger_keywords: string[];      // Extracted keywords
  suggestions: string[];           // Improvement suggestions
  too_broad: boolean;
  too_narrow: boolean;
}
```

### 4. list_skills
List all skills with Layer 1 metadata.

**Input:**
```typescript
{
  tags?: string[];                 // Filter by tags
  search?: string;                 // Search term
  include_deprecated?: boolean;    // Include deprecated skills
}
```

**Output:**
```typescript
{
  skills: SkillMetadata[];
  count: number;
}
```

### 5. get_skill
Retrieve full skill content (Layer 1 + Layer 2).

**Input:**
```typescript
{
  skill_id?: string;               // Get by ID
  name?: string;                   // Get by name
}
```

**Output:**
```typescript
{
  skill_id: string;
  name: string;
  description: string;
  full_content: string;
  token_counts: {
    layer1: number;
    layer2: number;
  };
  usage_count: number;
  success_rate: number;
  bundled_files: string[];
}
```

### 6. update_skill
Update an existing SKILL.md with versioning.

**Input:**
```typescript
{
  skill_id: string;                // Required: skill to update
  updates: Partial<{
    name: string;
    description: string;
    overview: string;
    steps: string[];
    examples: string[];
    prerequisites: string[];
    error_handling: string;
    limitations: string;
    tags: string[];
  }>;
}
```

**Output:**
```typescript
{
  updated: true;
  skill_id: string;
  previous_version: number;
  new_version: number;
}
```

### 7. match_skill
Find skills that match a task description (semantic matching).

**Input:**
```typescript
{
  task_description: string;        // Required: task to match
  context?: Record<string, unknown>;
  min_confidence?: number;         // Default: 0.3
}
```

**Output:**
```typescript
{
  matches: Array<{
    skill_id: string;
    name: string;
    description: string;
    confidence: number;
    triggered_by: string[];
  }>;
  best_match?: string;
}
```

### 8. record_skill_usage
Track skill usage outcome for learning.

**Input:**
```typescript
{
  skill_id: string;                // Required
  outcome: 'success' | 'failure' | 'partial';  // Required
  notes?: string;
  duration_ms?: number;
  context?: Record<string, unknown>;
}
```

**Output:**
```typescript
{
  recorded: true;
  skill_id: string;
  total_uses: number;
  success_rate: number;
}
```

## Database Schema (SQLite)

### skills
```sql
CREATE TABLE skills (
  id TEXT PRIMARY KEY,
  name TEXT UNIQUE NOT NULL,
  description TEXT NOT NULL,
  full_content TEXT NOT NULL,
  version INTEGER DEFAULT 1,
  token_count_layer1 INTEGER,
  token_count_layer2 INTEGER,
  file_path TEXT,
  tags TEXT,                        -- JSON array
  created_at INTEGER NOT NULL,
  updated_at INTEGER,
  deprecated_at INTEGER,
  usage_count INTEGER DEFAULT 0,
  success_count INTEGER DEFAULT 0
);

CREATE INDEX idx_skills_name ON skills(name);
CREATE INDEX idx_skills_tags ON skills(tags);
```

### skill_files
```sql
CREATE TABLE skill_files (
  id TEXT PRIMARY KEY,
  skill_id TEXT NOT NULL,
  file_name TEXT NOT NULL,
  file_path TEXT NOT NULL,
  file_type TEXT,
  token_count INTEGER,
  created_at INTEGER NOT NULL,
  FOREIGN KEY (skill_id) REFERENCES skills(id)
);

CREATE INDEX idx_skill_files_skill ON skill_files(skill_id);
```

### skill_usages
```sql
CREATE TABLE skill_usages (
  id TEXT PRIMARY KEY,
  skill_id TEXT NOT NULL,
  outcome TEXT NOT NULL,
  context TEXT,                     -- JSON
  notes TEXT,
  duration_ms INTEGER,
  created_at INTEGER NOT NULL,
  FOREIGN KEY (skill_id) REFERENCES skills(id)
);

CREATE INDEX idx_skill_usages_skill ON skill_usages(skill_id);
CREATE INDEX idx_skill_usages_outcome ON skill_usages(outcome);
```

### skill_versions
```sql
CREATE TABLE skill_versions (
  id TEXT PRIMARY KEY,
  skill_id TEXT NOT NULL,
  version INTEGER NOT NULL,
  content TEXT NOT NULL,
  created_at INTEGER NOT NULL,
  FOREIGN KEY (skill_id) REFERENCES skills(id)
);

CREATE INDEX idx_skill_versions_skill ON skill_versions(skill_id);
```

## HTTP REST API (Port 8029)

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/health` | Server health with stats |
| GET | `/api/stats` | Skills/usages summary |
| GET | `/api/skills` | List skills (Layer 1 metadata) |
| POST | `/api/skills` | Create skill |
| GET | `/api/skills/:id` | Get skill (Layer 1 + 2) |
| PUT | `/api/skills/:id` | Update skill |
| DELETE | `/api/skills/:id` | Deprecate skill |
| POST | `/api/validate` | Validate SKILL.md content |
| POST | `/api/analyze-description` | Analyze description effectiveness |
| POST | `/api/match` | Find matching skills for task |
| POST | `/api/skills/:id/usage` | Record skill usage |
| GET | `/api/skills/:id/usage` | Get skill usage history |
| GET | `/api/skills/:id/files` | List bundled files |
| POST | `/api/skills/:id/files` | Add bundled file |
| DELETE | `/api/skills/:id/files/:fileId` | Remove bundled file |

## WebSocket Events (Port 9029)

| Direction | Event | Data |
|-----------|-------|------|
| S→C | `skill_created` | `{ skill_id, name, token_counts }` |
| S→C | `skill_updated` | `{ skill_id, version }` |
| S→C | `skill_matched` | `{ skill_id, task, confidence }` |
| S→C | `skill_used` | `{ skill_id, outcome, success_rate }` |
| S→C | `validation_result` | `{ valid, errors, warnings }` |
| C↔S | `ping/pong` | `{ timestamp }` |

## InterLock Mesh (Port 3029)

### Listens To

| Signal | Code | From | Action |
|--------|------|------|--------|
| EXPERIENCE_RECORDED | 0xF0 | experience-layer | Learn from outcomes |
| PATTERN_EMERGED | 0xF1 | experience-layer | Consider new skill |
| LESSON_EXTRACTED | 0xF2 | experience-layer | Potential skill candidate |
| OPERATION_COMPLETE | 0xFF | * | Track skill usage if applicable |
| HEARTBEAT | 0x00 | * | Track server availability |

### Emits

| Signal | Code | To | When |
|--------|------|----|------|
| SKILL_CREATED | 0xA0 | consciousness, experience-layer | New skill added |
| SKILL_MATCHED | 0xA1 | consciousness | Skill matched to task |
| SKILL_USED | 0xA2 | experience-layer | Skill usage recorded |
| SKILL_DEPRECATED | 0xA3 | consciousness | Skill deprecated |
| SKILL_VALIDATION_FAILED | 0xA4 | consciousness | Validation errors |

### Peer Configuration

```json
{
  "peers": [
    { "name": "consciousness", "port": 3028 },
    { "name": "experience-layer", "port": 3031 },
    { "name": "context-guardian", "port": 3001 },
    { "name": "neurogenesis", "port": 3010 }
  ]
}
```

## File Structure

```
skill-builder/
├── config/
│   └── interlock.json
├── src/
│   ├── index.ts                 # MCP entry point
│   ├── types.ts                 # Type definitions
│   ├── database/
│   │   └── schema.ts            # SQLite + CRUD
│   ├── parser/
│   │   └── skill-parser.ts      # YAML frontmatter + Markdown parsing
│   ├── tools/
│   │   ├── index.ts
│   │   ├── create-skill.ts
│   │   ├── validate-skill.ts
│   │   ├── analyze-description.ts
│   │   ├── list-skills.ts
│   │   ├── get-skill.ts
│   │   ├── update-skill.ts
│   │   ├── match-skill.ts
│   │   └── record-skill-usage.ts
│   ├── services/
│   │   ├── token-counter.ts
│   │   └── skill-matcher.ts
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
│   ├── database.test.ts         # 25 tests
│   ├── parser.test.ts           # 17 tests
│   ├── services.test.ts         # 23 tests
│   ├── tools.test.ts            # 32 tests
│   ├── interlock.test.ts        # 27 tests
│   └── integration.test.ts      # 12 tests
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
  "yaml": "^2.3.4"
}
```

## Usage

### Start Server
```bash
cd /repo/skill-builder
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
| consciousness-mcp (3028) | Receives SKILL_CREATED, SKILL_MATCHED |
| experience-layer (3031) | Sends EXPERIENCE_RECORDED, receives SKILL_USED |
| context-guardian (3001) | Validates skill content |
| neurogenesis-engine (3010) | May generate skills from patterns |

## Skill Directories

```typescript
SKILL_DIRECTORIES = [
  '/repo/claude-skills/',    // 11 skills
  '/repo/bop/skills/'        // 2 skills
]
```

## Test Coverage

- **136 tests** total
- Database operations: 25 tests
- Parser functions: 17 tests
- Service functions: 23 tests
- Tool functions: 32 tests
- InterLock mesh: 27 tests
- Integration flows: 12 tests

## Key Types

```typescript
interface SkillMetadata {
  id: string;
  name: string;
  description: string;
  tags: string[];
  token_count_layer1: number;
  token_count_layer2: number;
  usage_count: number;
  success_rate: number;
  created_at: number;
  deprecated_at?: number;
}

interface Skill extends SkillMetadata {
  full_content: string;
  file_path?: string;
  version: number;
  bundled_files: BundledFile[];
}

interface ParsedSkill {
  frontmatter: {
    name: string;
    description: string;
    tags?: string[];
  };
  body: {
    overview?: string;
    prerequisites?: string[];
    steps?: string[];
    examples?: string[];
    error_handling?: string;
    limitations?: string;
    raw_body: string;
  };
  raw_content: string;
}

interface SkillMatch {
  skill_id: string;
  name: string;
  description: string;
  confidence: number;
  triggered_by: string[];
}
```

## Research References

Source documents:
- `/repo/barespec/claude-skills.barespec.md`
- `/repo/bop/recovered/skills-docs/claude-skills_humanspeak.md`

Key concepts incorporated:
1. Progressive Disclosure: Layer 1 → Layer 2 → Layer 3
2. Semantic Matching: Description-based skill discovery
3. Token Budgets: Layer 1 ~100, Layer 2 <5000
4. Usage Tracking: Success rate and confidence updates

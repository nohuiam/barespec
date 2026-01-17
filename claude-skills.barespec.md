TOPIC: Claude Skills
TYPE: Custom instructions for Claude
VERSION: 2025-12

---

DEFINITION

WHAT: Folders containing instructions, scripts, and resources Claude discovers and loads dynamically
FILE: SKILL.md (required)
FORMAT: YAML frontmatter + Markdown body
FUNCTION: Teach Claude how to handle specific scenarios, encode institutional knowledge, standardize outputs

---

PROGRESSIVE DISCLOSURE

LAYER_1: Metadata
TOKENS: ~100
CONTENT: Name + Description
WHEN: Always visible to Claude for matching

LAYER_2: Full Instructions
TOKENS: <5000
CONTENT: SKILL.md body
WHEN: Skill matches user request

LAYER_3: Bundled Files
TOKENS: Variable
CONTENT: References, scripts, assets
WHEN: Needed during execution

BENEFIT: Many skills available without overwhelming context window

---

SKILL.MD STRUCTURE

FRONTMATTER:
```yaml
name: skill-name
description: When to use, capabilities, triggers, boundaries (20-500 chars)
tags: [domain, operation-type]
cognitive_integration:           # REQUIRED for all BOP skills
  awareness: true                # Track with consciousness-mcp
  ethics_check: true|optional    # Tenets evaluation (true for destructive ops)
  verification: true|optional    # Verifier-mcp checking (true for factual outputs)
  experience_record: true        # Always true - record all outcomes
  learning: true|optional        # Pattern extraction
triggers:
  - "activation phrase 1"
  - "activation phrase 2"
signals:                         # InterLock signals used
  - EXPERIENCE_RECORDED
  - SKILL_USED
```

BODY:
- Overview
- Cognitive Hooks (Pre-Execution and Post-Execution tables)
- Prerequisites (including cognitive servers)
- Execution steps
- Phase 0: Track Awareness (curl command)
- Record Experience (curl command)
- Error handling (including cognitive failures)
- Limitations

NAMING: Lowercase with hyphens (e.g., pdf-editor, brand-guidelines)

---

COGNITIVE INTEGRATION (BOP/Imminence OS Requirement)

PURPOSE: All skills learn from execution, evaluate ethics, and verify outputs

7-STEP LOOP:
1. AWARENESS - consciousness-mcp POST /api/attention
2. PLANNING - skill-builder POST /api/match (when ambiguous)
3. ETHICS - tenets-server POST /api/evaluate (when destructive)
4. EXECUTION - Core skill operations
5. VERIFY - verifier-mcp POST /api/claims/verify (when factual)
6. RECORD - experience-layer POST /api/experience (always)
7. LEARN - experience-layer POST /api/lessons/learn (when patterns)

REQUIRED_SERVERS:
- consciousness-mcp (port 8028) - awareness tracking
- experience-layer (port 8031) - experience recording
- Optional: tenets-server (8027), verifier-mcp (8021), skill-builder (8029)

VALIDATION:
- skill-builder validates cognitive_integration on skill creation
- experience_record: true is MANDATORY
- Ethics check REQUIRED for destructive operations (DELETE, MERGE, MOVE)
- Verification RECOMMENDED for factual outputs

REFERENCE: /repo/claude-skills/cognitive-workflow-template/SKILL.md

---

DESCRIPTION FIELD (Critical)

PURPOSE: Determines when skill activates
PERSPECTIVE: Write from Claude's perspective
ELEMENTS: Specific capabilities, clear triggers, relevant context, boundaries

WEAK: "This skill helps with PDFs and documents."
STRONG: "Comprehensive PDF manipulation toolkit for extracting text and tables, creating new PDFs, merging/splitting documents, and handling forms. When Claude needs to fill in a PDF form or programmatically process, generate, or analyze PDF documents at scale. Use for document workflows and batch operations. Not for simple PDF viewing or basic conversions."

---

TRIGGERING

METHOD: Semantic understanding (not keyword matching)
EVALUATION: Claude evaluates descriptions against request
MULTIPLE: Multiple skills can activate simultaneously
ISSUES: Vague descriptions reduce accuracy, overly generic causes inappropriate activation

---

CREATION STEPS

STEP_1: Understand core requirements
- What problem does this skill solve?
- What triggers should activate it?
- What does success look like?
- What are edge cases?

STEP_2: Write the name
- Straightforward, descriptive
- Lowercase with hyphens
- Short and clear

STEP_3: Write the description
- Capabilities, triggers, use cases, boundaries
- Action verbs, concrete use cases, clear boundaries

STEP_4: Write main instructions
- Structured, scannable, actionable
- Markdown headers, bullet points, code blocks
- Include concrete examples
- Specify what skill cannot do

STEP_5: Upload skill
- Claude.ai: Settings → add custom skill
- Claude Code: skills/ directory with SKILL.md
- API: POST /v1/skills with beta headers

---

CLAUDE CODE STRUCTURE

```
my-project/
├── skills/
│   └── my-skill/
│       ├── SKILL.md
│       └── references/
│           ├── detailed-docs.md
│           └── schemas.md
```

DISCOVERY: Automatic when plugin installed
REFERENCES: Load only when needed during execution

---

TESTING

NORMAL_OPERATIONS: Typical requests skill should handle
EDGE_CASES: Incomplete inputs, unusual formats, ambiguous instructions
OUT_OF_SCOPE: Related but shouldn't trigger

TRIGGERING_TESTS:
- Does skill activate when expected?
- Does it stay inactive when irrelevant?

FUNCTIONAL_TESTS:
- Output consistency
- Usability
- Documentation accuracy

---

BEST PRACTICES

START_WITH_USE_CASES: Build when you have real, repeated tasks (done 5+ times, will do 10+ more)
DEFINE_SUCCESS: Include quality thresholds in instructions
USE_SKILL_CREATOR: Template at github.com/anthropics/skills/tree/main/skill-creator
MENU_APPROACH: Reference separate files for distinct processes, Claude reads only what's needed
NO_DUPLICATION: Info in SKILL.md OR references, not both
KEEP_SKILL_MD_LEAN: High-level instructions, details in references

---

FILE SIZE

AVOID: Bloating context with unnecessary content
STRATEGY: Break content into chunks, let Claude select based on task
SEPARATE_FILES: Don't need to be mutually exclusive, just reasonable chunks

---

SKILLS VS OTHER TOOLS

VS_PROMPTS:
- Prompts: Ephemeral, conversational, reactive, one-time
- Skills: Persistent, proactive, reusable across conversations
- USE_SKILL_WHEN: Same prompt repeated across conversations

VS_PROJECTS:
- Projects: Static context, background knowledge, "what you need to know"
- Skills: Dynamic capabilities, procedural knowledge, "how to do things"
- USE_SKILL_WHEN: Copying same instructions across multiple projects

VS_SUBAGENTS:
- Subagents: Independent task execution, specific tool permissions, context isolation
- Skills: Portable expertise any agent can use
- TOGETHER: Subagents can use Skills for specialized expertise

VS_MCP:
- MCP: Connectivity to external data/tools
- Skills: Procedural knowledge for using that data
- TOGETHER: MCP for access, Skills for how to use

VS_CLAUDE_MD:
- CLAUDE.md: Always loaded, project-specific, Claude Code only, single file
- Skills: Progressive disclosure, cross-platform, can include code/references

---

SKILL CANDIDATES

GOOD:
- Cross-repo relevance
- Multi-audience value
- Stable patterns (don't change every commit)
- Organizational workflows
- Domain expertise
- Personal preferences

EXAMPLES:
- Data warehouse queries
- Brand guidelines
- Document templates
- Compliance procedures
- Coding patterns

---

SHARING SKILLS

OPTIONS:
- Zip file (quick sharing)
- Internal versioned repository
- Git repository
- Plugin bundle (Claude Code)

STORAGE: Local on machine, full control over updates

---

API UPLOAD

```bash
curl -X POST "https://api.anthropic.com/v1/skills" \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "anthropic-beta: skills-2025-10-02" \
  -F "display_title=My Skill Name" \
  -F "files[]=@my-skill/SKILL.md;filename=my-skill/SKILL.md"
```

---

EXAMPLE: BRAND GUIDELINES

```markdown
name: brand-guidelines
description: Applies official brand colors and typography to artifacts. Use when brand colors, style guidelines, visual formatting, or company design standards apply.
---

# Brand Styling

## Colors
- Dark: #141413
- Light: #faf9f5
- Orange: #d97757
- Blue: #6a9bcc
- Green: #788c5d

## Typography
- Headings: Poppins (24pt+)
- Body: Lora
```

---

EXAMPLE: DATA WAREHOUSE

```markdown
name: sql-analysis
description: Use when analyzing business data: revenue, ARR, customer segments, product usage. Provides table schemas, metric definitions, required filters.
---

# SQL Analysis Skill

## Workflow
1. Clarify request (time period, segment, purpose)
2. Check existing dashboards
3. Identify data source
4. Execute with required filters

## Standard Filters
- Exclude test accounts: WHERE account != 'Test'
- Complete periods only: WHERE month <= DATE_TRUNC(CURRENT_DATE(), MONTH)

## References
- Revenue → references/finance.md
- Product → references/product.md
```

---

ITERATION

MONITOR: Real-world usage
REFINE: Descriptions if triggering inconsistent
CLARIFY: Instructions if outputs vary
EVOLVE: Through practical application

---

RESOURCES

SKILL_CREATOR: github.com/anthropics/skills/tree/main/skill-creator
SKILLS_LIBRARY: github.com/anthropics/skills
DOCUMENTATION: docs.anthropic.com
COOKBOOK: github.com/anthropics/claude-cookbooks/tree/main/skills
MARKETPLACE: skillsmp.com

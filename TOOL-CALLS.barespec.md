LIBRARY: Server Tool Calls
VERSION: 1.2
UPDATED: 2026-01-05
SERVERS: 32
TOOLS: 421 total
PURPOSE: Complete reference of all MCP server tools with inputs, outputs, and usage

---

## QUICK REFERENCE

| Server | Tools | Category | Primary Purpose |
|--------|-------|----------|-----------------|
| context-guardian | 14 | Safety | Validation, rules, mistake tracking |
| quartermaster | 23 | Files | Search, inventory, monitoring |
| snapshot | 6 | Backup | Capture, verify, restore |
| catasorter | 6 | Classification | GLEC classify, organize |
| looker | 5 | Research | Web search, credibility |
| smart-file-organizer | 3 | Files | Organize, rollback |
| bonzai-bloat-buster | 6 | Dedup | Compare, consolidate |
| enterspect | 8 | Search | Semantic, spatial, keyword |
| neurogenesis-engine | 26 | Meta | Server generation, workflows |
| chronos-synapse | 19 | Temporal | Events, patterns, predictions |
| trinity-coordinator | 14 | Orchestration | Routing, work orders |
| niws-server | 102 | News | RSS, bias, scripts, video |
| project-context | 5 | Config | Paths, limits, rate limits |
| knowledge-curator | 6 | Knowledge | Curation, compression |
| pk-manager | 4 | PK | Rebuild, rollback |
| research-bus | 12 | Research | Perplexity, citations |
| intelligent-router | 2 | Routing | Intent detection |
| tool-registry | 65 | Meta | Tool discovery, ecosystem coordination |
| claude-code-bridge | 8 | Bridge | Desktop↔Code work orders |
| filesystem-test | 12 | Files | CRUD operations |
| verifier-mcp | 3 | Verification | Claims, fact-check |
| consciousness-mcp | 10 | Cognitive | Meta-awareness, pattern learning |
| experience-layer | 6 | Cognitive | Experiential memory, lessons |
| skill-builder | 8 | Cognitive | SKILL.md management |
| percolation-server | 8 | Cognitive | Blueprint optimization |
| tenets-server | 8 | Cognitive | Ethical evaluation |
| consolidation-engine | 6 | Batch/Merge | Document merging |
| filesystem-guardian | 6 | Filesystem | macOS xattr, Spotlight |
| health-monitor | 7 | Health | Ecosystem monitoring |
| intake-guardian | 5 | Safety | Content admission |
| safe-batch-processor | 4 | Batch/Merge | Batch operations |
| synapse-relay | 4 | Health | Signal routing |

---

## CONTEXT-GUARDIAN (14 tools)

**Purpose:** Validation, mistake tracking, safety rules

| Tool | Purpose | Key Input |
|------|---------|-----------|
| `context_guardian_validate` | Validate operation | operation_type, context, severity |
| `context_guardian_validate_batch` | Validate batch ops | operation_type, batch_size, sample_items |
| `context_guardian_record_mistake` | Record mistake | what_happened, why_it_happened, impact |
| `context_guardian_get_stats` | Get statistics | time_period, include_trends |
| `context_guardian_get_mistakes` | Query mistakes | filter, limit, include_patterns |
| `context_guardian_session_state` | Track session | query_type, session_id, time_window |
| `context_guardian_knowledge_lookup` | Knowledge query | query_type, query, category |
| `context_guardian_add_rule` | Add validation rule | rule_type, severity, condition, action |
| `context_guardian_get_rules` | Query rules | filter, limit |
| `context_guardian_get_guidelines` | Get guidelines | operation_type, context |
| `context_guardian_search_solutions` | Search solutions | query, context, limit |
| `context_guardian_detect_drift` | Detect drift | target_type, target, baseline |
| `context_guardian_capture_snapshot` | Capture snapshot | snapshot_type, targets, label |
| `context_guardian_get_validation_history` | Validation history | filter, limit, include_stats |

---

## QUARTERMASTER (23 tools)

**Purpose:** File search, inventory, monitoring, alerts

| Tool | Purpose | Key Input |
|------|---------|-----------|
| `quartermaster_search` | FTS5 search | query, scope, path, file_types |
| `quartermaster_live_find` | Realtime search | query, path, max_depth |
| `quartermaster_hybrid_find` | Index + live | query, path, prefer |
| `quartermaster_recent` | Recent files | type, hours, limit |
| `quartermaster_inventory` | Directory analysis | path, options |
| `quartermaster_audit` | Health assessment | (none) |
| `quartermaster_get_metadata` | File metadata | filepath |
| `quartermaster_suggest_location` | Placement suggestion | filepath, content |
| `quartermaster_suggest_scope` | Scope suggestion | query |
| `quartermaster_inbox_drop` | Queue for classify | filepath |
| `quartermaster_inbox_status` | Inbox queue status | (none) |
| `quartermaster_watch_status` | Watcher health | (none) |
| `quartermaster_approval_state` | By approval status | state, limit |
| `quartermaster_health` | Health check | (none) |
| `quartermaster_duplicates` | Find duplicates | path, min_size |
| `quartermaster_alerts` | System alerts | severity, category |
| `quartermaster_refresh_index` | Refresh index | path, force_refresh |
| `quartermaster_snapshot` | Generate snapshot | (none) |
| `quartermaster_about` | Server info | (none) |
| `quartermaster_daily_digest` | Daily summary | date |
| `quartermaster_weekly_report` | Weekly trends | week_start |
| `reindex_files` | Legacy: refresh | path, force_refresh |
| `quartermaster_find` | Legacy: search | query, scope |

---

## SNAPSHOT (6 tools)

**Purpose:** Capture, verify, restore system state

| Tool | Purpose | Key Input |
|------|---------|-----------|
| `snapshot_capture_snapshot` | Capture state | type, label |
| `snapshot_verify_integrity` | Verify against snapshot | snapshot_id |
| `snapshot_list_snapshots` | List snapshots | limit, type, status |
| `snapshot_get_snapshot_details` | Snapshot details | snapshot_id, include_files |
| `snapshot_compare_snapshots` | Compare two | snapshot_id_1, snapshot_id_2 |
| `snapshot_restore_snapshot` | Restore (destructive) | snapshot_id, confirm |

---

## CATASORTER (6 tools)

**Purpose:** GLEC classification, document organization

| Tool | Purpose | Key Input |
|------|---------|-----------|
| `classify_document` | Classify file | file_path, extract_nuggets, user_hint |
| `get_classification` | Get existing | file_path |
| `get_dewey_stats` | Dewey counters | (none) |
| `get_workflow_metrics` | Workflow stats | include_details |
| `organize_document` | Classify + move | file_path, user_hint, dry_run |
| `batch_process` | Batch classify | max_files, dry_run, extract_nuggets |

---

## LOOKER (5 tools)

**Purpose:** Web search, credibility analysis

| Tool | Purpose | Key Input |
|------|---------|-----------|
| `enhanced_search` | Multi-source search | query, sources, credibilityThreshold |
| `validate_source` | Source credibility | url, contentType, verificationLevel |
| `search_history` | Manage history | action, timeRange, options |
| `content_analysis` | Content quality | content, analysisType, context |
| `cross_reference` | Find related | primaryUrl, searchDepth, options |

---

## SMART-FILE-ORGANIZER (3 tools)

**Purpose:** File organization with rollback

| Tool | Purpose | Key Input |
|------|---------|-----------|
| `organize_files` | Organize files | sourcePath, options |
| `get_organization_stats` | Organization stats | timeRange |
| `rollback_operation` | Rollback moves | movementId, batchId, confirmDangerous |

---

## BONZAI-BLOAT-BUSTER (6 tools)

**Purpose:** Duplicate detection, consolidation

| Tool | Purpose | Key Input |
|------|---------|-----------|
| `analyze_directory` | Scan for duplicates | directoryPath, generatePlan |
| `process_documents` | Add to vector DB | filePaths |
| `compare_documents` | Compare two files | filePathA, filePathB |
| `get_storage_stats` | Vector DB stats | (none) |
| `execute_consolidation_plan` | Execute plan | planPath, dryRun |
| `batch_compare_documents` | Bulk comparison | newFiles, existingFiles, threshold |

---

## ENTERSPECT (8 tools)

**Purpose:** Semantic search, spatial navigation

| Tool | Purpose | Key Input |
|------|---------|-----------|
| `semantic_search` | NLP search | query, scope, max_results |
| `keyword_search` | Keyword match | keywords, file_types, date_range |
| `filter_by_category` | Filter by GLEC | glec_type, subject, importance |
| `recent_activity` | Recent files | activity_type, limit, time_range_hours |
| `spatial_navigate` | 3D navigation | x, y, z, radius |
| `find_related` | Find similar | file_path, similarity_threshold |
| `search_index_status` | Index health | (none) |
| `update_search_index` | Refresh index | full_rebuild, path |

---

## NEUROGENESIS-ENGINE (26 tools)

**Purpose:** Server generation, workflows, diagnostics

### Core (7)
| Tool | Purpose | Key Input |
|------|---------|-----------|
| `create_synapse` | Create server | name, purpose, tools, ports |
| `fire_neurotransmitter` | Server-to-server | fromServer, toServer, signal |
| `create_workflow` | Define workflow | name, description, steps |
| `get_synaptic_network` | View network | includeHealth, includeConnections |
| `discover_emergent_capabilities` | Find synergies | servers, depth |
| `get_possible_pathways` | List routes | startServer, endServer |
| `visualize_pathway` | Visualize route | pathway, format |

### Meta-Memory (5)
| Tool | Purpose | Key Input |
|------|---------|-----------|
| `log_generation_error` | Record error | serverName, errorType, message |
| `get_errors_by_server` | Errors by server | serverName, limit |
| `get_errors_by_type` | Errors by type | errorType, limit |
| `get_recent_errors` | Recent errors | limit, severity |
| `get_error_stats` | Error trends | period, groupBy |

### Quality Control (4)
| Tool | Purpose | Key Input |
|------|---------|-----------|
| `validate_generated_code` | Full validation | buildId, checks |
| `check_syntax_only` | Syntax check | code, language |
| `check_mcp_compliance` | MCP compliance | buildId |
| `scan_security_vulnerabilities` | Security scan | buildId, depth |

### Diagnostic (4)
| Tool | Purpose | Key Input |
|------|---------|-----------|
| `check_server_health` | Health check | serverName, includeMetrics |
| `get_performance_metrics` | Performance | serverName, period |
| `get_health_snapshot` | Health snapshot | servers |
| `get_memory_usage` | Memory stats | serverName |

### Learning (5)
| Tool | Purpose | Key Input |
|------|---------|-----------|
| `submit_feedback` | Submit feedback | buildId, rating, category, comment |
| `get_feedback_by_server` | Feedback by server | serverName, limit |
| `get_feedback_stats` | Feedback stats | period |
| `get_open_feedback` | Unresolved items | priority, limit |
| `update_feedback_status` | Update status | feedbackId, status |

### Chronos (1)
| Tool | Purpose | Key Input |
|------|---------|-----------|
| `track_event` | Send event | entityType, entityId, eventType |

---

## CHRONOS-SYNAPSE (19 tools)

**Purpose:** Temporal analysis, patterns, predictions

### Event Tracking (4)
| Tool | Purpose | Key Input |
|------|---------|-----------|
| `track_event` | Record event | entityType, entityId, eventType |
| `track_snapshot` | Capture state | entityType, entityId, state |
| `batch_track_events` | Batch events | correlationId, events |
| `get_timeline` | Event history | entityType, entityId, eventTypes |

### Causality (4)
| Tool | Purpose | Key Input |
|------|---------|-----------|
| `analyze_causality` | Root cause | effectEventId, maxDepth |
| `detect_correlation` | Find patterns | eventId, timeWindowHours |
| `trace_impact` | Forward impact | causeEventId, maxDepth |
| `explain_change` | Why changed | entityType, entityId, change |

### Pattern Recognition (3)
| Tool | Purpose | Key Input |
|------|---------|-----------|
| `detect_cycles` | Find cycles | entityType, entityId, minOccurrences |
| `detect_trends` | Find trends | metric, entityType, entityId |
| `detect_anomalies` | Find outliers | entityType, entityId, metric |

### Prediction (4)
| Tool | Purpose | Key Input |
|------|---------|-----------|
| `predict_next_event` | When next | eventType, entityType, entityId |
| `predict_bottleneck` | Find limits | scope, horizon |
| `forecast_metric` | Forecast value | metric, entityType, forecastDays |
| `what_if_analysis` | Simulate change | scenario, affectedMetrics |

### Optimization (4)
| Tool | Purpose | Key Input |
|------|---------|-----------|
| `optimize_timing` | Best time | action, constraints, optimizeFor |
| `suggest_schedule` | Optimal schedule | task, frequency, priority |
| `optimize_sequence` | Order operations | operations, goal |
| `find_maintenance_window` | Best window | maintenanceDuration, criticalServices |

---

## TRINITY-COORDINATOR (14 tools)

**Purpose:** Multi-agent orchestration, work orders

| Tool | Purpose | Key Input |
|------|---------|-----------|
| `route_task` | Route to member | task, targetMember, priority |
| `execute_workflow` | Run workflow | workflowName, steps |
| `delegate_to_code` | Delegate to Code | task, filePaths, instructions |
| `delegate_to_perplexity` | Delegate research | query, domain, depth |
| `create_workflow` | Create template | name, description, steps |
| `get_routing_history` | Routing history | limit, targetMember |
| `watch_work_orders` | Monitor orders | status |
| `get_pending_work` | Pending orders | (none) |
| `get_work_status` | Order status | workOrderId |
| `get_trinity_status` | Overall status | (none) |
| `notify_desktop` | Send notification | message, workOrderId |
| `get_pending_handoffs` | Pending handoffs | member |
| `analyze_work_order_complexity` | Complexity analysis | workOrderId |
| `suggest_model_provider` | Model routing | workOrderId |

---

## NIWS-SERVER (102 tools)

**Purpose:** News processing, RSS, bias analysis, video production

### Core Research (4)
| Tool | Purpose |
|------|---------|
| `research_query` | Execute research |
| `get_research_stats` | Research stats |
| `get_budget_status` | Budget status |
| `get_cache_stats` | Cache stats |

### Outlets (10)
| Tool | Purpose |
|------|---------|
| `outlet_add` | Add outlet |
| `outlet_search` | Search outlets |
| `outlet_get` | Get outlet |
| `outlet_update` | Update outlet |
| `outlet_delete` | Delete outlet |
| `outlet_list_by_type` | List by type |
| `outlet_list_by_bias` | List by bias |
| `outlet_get_stats` | Outlet stats |
| `outlet_import` | Bulk import |
| `outlet_export` | Export all |

### RSS (13)
| Tool | Purpose |
|------|---------|
| `rss_add_feed` | Add feed |
| `rss_remove_feed` | Remove feed |
| `rss_get_feed` | Get feed |
| `rss_list_feeds` | List feeds |
| `rss_poll_feeds` | Poll all |
| `rss_poll_single` | Poll one |
| `rss_get_articles` | Get articles |
| `rss_set_poll_interval` | Set interval |
| `rss_get_poll_status` | Poll status |
| `rss_mark_article_read` | Mark read |
| `rss_get_unread` | Get unread |
| `rss_search_articles` | Search articles |
| `rss_get_stats` | RSS stats |

### Story Clustering (8)
| Tool | Purpose |
|------|---------|
| `story_cluster_articles` | Cluster articles |
| `story_get_cluster` | Get cluster |
| `story_list_clusters` | List clusters |
| `story_merge_clusters` | Merge clusters |
| `story_split_cluster` | Split cluster |
| `story_add_article` | Add to cluster |
| `story_remove_article` | Remove from cluster |
| `story_get_timeline` | Story timeline |

### Bias Analysis (11)
| Tool | Purpose |
|------|---------|
| `analysis_bias_check` | Check bias |
| `analysis_framing_detect` | Detect framing |
| `analysis_sentiment` | Analyze sentiment |
| `analysis_fact_check` | Fact check |
| `analysis_source_credibility` | Check source |
| `analysis_narrative_compare` | Compare narratives |
| `analysis_get_indicators` | Get indicators |
| `analysis_batch_process` | Batch analyze |
| `analysis_get_history` | Analysis history |
| `analysis_set_thresholds` | Set thresholds |
| `analysis_get_stats` | Analysis stats |

### Script Generation (20)
| Tool | Purpose |
|------|---------|
| `script_generate` | Generate script |
| `script_get` | Get script |
| `script_list` | List scripts |
| `script_update` | Update script |
| `script_delete` | Delete script |
| `script_set_template` | Create template |
| `script_get_templates` | Get templates |
| `script_apply_style` | Apply styling |
| `script_add_section` | Add section |
| `script_remove_section` | Remove section |
| `script_reorder` | Reorder sections |
| `script_calculate_timing` | Calc timing |
| `script_validate` | Validate script |
| `script_convert_format` | Convert format |
| `script_merge` | Merge scripts |
| `script_split` | Split script |
| `script_add_transition` | Add transition |
| `script_set_pacing` | Set pacing |
| `script_learn_from_edit` | Learn edits |
| `script_get_suggestions` | Get suggestions |

### Notion (5)
| Tool | Purpose |
|------|---------|
| `notion_sync_stories` | Sync to Notion |
| `notion_get_pending` | Pending approval |
| `notion_approve_story` | Approve story |
| `notion_reject_story` | Reject story |
| `notion_get_approved` | Get approved |

### Teleprompter (9)
| Tool | Purpose |
|------|---------|
| `teleprompter_export` | Export script |
| `teleprompter_format` | Format script |
| `teleprompter_set_speed` | Set speed |
| `teleprompter_preview` | Preview URL |
| `teleprompter_airdrop` | AirDrop to iPad |
| `teleprompter_queue_add` | Add to queue |
| `teleprompter_queue_list` | List queue |
| `teleprompter_get_device_status` | Device status |
| `teleprompter_sync` | Sync to device |

### Video Production (14)
| Tool | Purpose |
|------|---------|
| `video_generate_background` | Generate BG |
| `video_composite` | Composite layers |
| `video_add_overlay` | Add overlay |
| `video_add_lower_third` | Add lower third |
| `video_add_transition` | Add transition |
| `video_set_resolution` | Set resolution |
| `video_set_format` | Set format |
| `video_render` | Render video |
| `video_preview` | Preview URL |
| `video_export` | Export video |
| `video_get_templates` | Get templates |
| `video_apply_template` | Apply template |
| `video_batch_render` | Batch render |
| `video_get_render_status` | Render status |

### Orchestrator (8)
| Tool | Purpose |
|------|---------|
| `orchestrator_run_overnight` | Overnight run |
| `orchestrator_get_schedule` | Get schedule |
| `orchestrator_add_task` | Add task |
| `orchestrator_remove_task` | Remove task |
| `orchestrator_get_status` | Get status |
| `orchestrator_pause` | Pause |
| `orchestrator_resume` | Resume |
| `orchestrator_get_logs` | Get logs |

---

## PROJECT-CONTEXT (5 tools)

**Purpose:** Configuration, path validation, rate limits

| Tool | Purpose | Key Input |
|------|---------|-----------|
| `get_configuration` | Get config | section |
| `get_path_configuration` | Path permissions | path |
| `update_configuration` | Update config | section, settings |
| `validate_path` | Check path allowed | path, operation |
| `check_rate_limit` | Check/record op | operation, cost |

---

## KNOWLEDGE-CURATOR (6 tools)

**Purpose:** Document curation, compression, cross-refs

| Tool | Purpose | Key Input |
|------|---------|-----------|
| `curate_knowledge` | Transform to PK | filePath, classification, options |
| `analyze_nugget_patterns` | Pattern analysis | nuggetIds, analysisType |
| `update_search_index` | Update index | pkId, content, keywords |
| `build_cross_references` | Build refs | sourceId, targetIds, referenceType |
| `get_compression_stats` | Compression stats | timeRange |
| `query_knowledge_graph` | Query graph | startNode, depth, relationshipTypes |

---

## PK-MANAGER (4 tools)

**Purpose:** Project Knowledge rebuild, rollback

| Tool | Purpose | Key Input |
|------|---------|-----------|
| `pk_rebuild` | 7-phase rebuild | source_path, options |
| `get_rebuild_status` | Rebuild status | rebuildId |
| `list_rebuilds` | List rebuilds | limit, status |
| `rollback_rebuild` | Rollback rebuild | rebuildId, backupId |

---

## RESEARCH-BUS (12 tools)

**Purpose:** Perplexity research, citations, synthesis

### Core (4)
| Tool | Purpose | Key Input |
|------|---------|-----------|
| `research_query` | Execute research | query, backend, importance |
| `get_budget_status` | Budget status | (none) |
| `get_cache_stats` | Cache stats | (none) |
| `health_check` | Backend health | (none) |

### Citation Vault (5)
| Tool | Purpose | Key Input |
|------|---------|-----------|
| `get_citations` | Get citations | vault_id |
| `get_citation_summary` | Citation summary | vault_id |
| `get_vault_stats` | Vault stats | (none) |
| `list_vaults` | List vaults | include_expired, limit |
| `clear_expired_vaults` | Clear expired | (none) |

### Enhancement (3)
| Tool | Purpose | Key Input |
|------|---------|-----------|
| `optimize_query` | Improve query | query |
| `validate_source` | Source credibility | url, vault_id |
| `synthesize_results` | Combine results | query, research_ids, style |

---

## INTELLIGENT-ROUTER (2 tools)

**Purpose:** Intent detection, workflow routing

| Tool | Purpose | Key Input |
|------|---------|-----------|
| `route_request` | Route by intent | request, context, routing |
| `get_routing_history` | Routing history | limit, intent, status |

---

## TOOL-REGISTRY (65 tools)

**Purpose:** Tool discovery, ecosystem coordination, capability matching

### Core Registry (9)
| Tool | Purpose | Key Input |
|------|---------|-----------|
| `tool_registry_add` | Register new tool | toolName, serverName, description, riskLevel |
| `tool_registry_update` | Update tool metadata | toolName, description, enabled |
| `tool_registry_delete` | Delete tool | toolName |
| `tool_registry_get` | Get tool details | toolName |
| `tool_registry_list` | List with filters | enabled, serverName, category |
| `tool_registry_stats` | Registry statistics | (none) |
| `tool_registry_approve` | Approve pending op | requestId, reason |
| `tool_registry_deny` | Deny pending op | requestId, reason |
| `tool_registry_pending` | List pending requests | (none) |

### Capability Matching (1) - MOST IMPORTANT
| Tool | Purpose | Key Input |
|------|---------|-----------|
| `tool_registry_can_do` | "Can I do X?" query | query |

### Curation (4)
| Tool | Purpose | Key Input |
|------|---------|-----------|
| `tool_curator_flag` | Flag quality issue | toolName, flag, reason, curator |
| `tool_curator_verify` | Mark as verified | toolName, curator |
| `tool_curator_add_note` | Add curation note | toolName, note, curator |
| `tool_curator_get_flagged` | Get flagged tools | flag |

### Rating (3)
| Tool | Purpose | Key Input |
|------|---------|-----------|
| `tool_rating_rate` | Rate 1-5 stars | toolName, rating, userId |
| `tool_rating_review` | Write review | toolName, review, rating, userId |
| `tool_rating_get_top_rated` | Top rated tools | limit, minRatings |

### Analytics (4)
| Tool | Purpose | Key Input |
|------|---------|-----------|
| `tool_analytics_usage` | Usage analytics | toolName |
| `tool_analytics_trends` | Usage trends | toolName, days |
| `tool_analytics_popular` | Most used tools | limit |
| `tool_analytics_record_usage` | Record usage event | toolName, success, executionTime |

### Recommendations (3)
| Tool | Purpose | Key Input |
|------|---------|-----------|
| `tool_recommend_for_me` | Personalized recs | userId, limit |
| `tool_recommend_similar` | Similar tools | toolName, limit |
| `tool_recommend_trending` | Trending tools | limit, days |

### LLM Intelligence (4)
| Tool | Purpose | Key Input |
|------|---------|-----------|
| `tool_llm_suggest` | AI-powered LLM selection | taskDescription, requirements |
| `tool_llm_list` | List LLM services | provider, availability |
| `tool_llm_compare` | Compare LLMs | llmIds |
| `tool_intelligence_stats` | LLM stats | (none) |

### Search Intelligence (2)
| Tool | Purpose | Key Input |
|------|---------|-----------|
| `tool_search_suggest` | Best search tool | requirements |
| `tool_search_compare` | Compare search tools | (none) |

### Pattern Learning (3)
| Tool | Purpose | Key Input |
|------|---------|-----------|
| `tool_pattern_record_success` | Record success | taskType, llmUsed, executionTime, outcome |
| `tool_pattern_get_insights` | Get patterns | taskType |
| `tool_pattern_get_best_llm` | Best LLM for task | taskType, criteria |

### Deprecation (3)
| Tool | Purpose | Key Input |
|------|---------|-----------|
| `tool_deprecate` | Mark deprecated | toolIdentifier, reason, alternatives |
| `tool_deprecation_get_alternatives` | Get alternatives | toolIdentifier |
| `tool_deprecation_get_migration_guide` | Migration guide | fromTool, toTool |

### Compatibility (3)
| Tool | Purpose | Key Input |
|------|---------|-----------|
| `tool_compatibility_check` | Check compatible | tools |
| `tool_compatibility_register_version` | Register version | toolIdentifier, version |
| `tool_compatibility_report` | Compatibility report | (none) |

### Notifications (2)
| Tool | Purpose | Key Input |
|------|---------|-----------|
| `tool_notify` | Send notification | title, message, category, priority |
| `tool_notifications_get` | Get notifications | category, priority, unreadOnly |

### Learning System (3)
| Tool | Purpose | Key Input |
|------|---------|-----------|
| `tool_learning_record_interaction` | Record interaction | type, outcome, context |
| `tool_learning_get_recommendations` | Get learned recs | (none) |
| `tool_learning_get_insights` | Learning insights | (none) |

### Status & Health (3)
| Tool | Purpose | Key Input |
|------|---------|-----------|
| `tool_status_dashboard` | Status dashboard | (none) |
| `tool_status_summary` | Health summary | (none) |
| `tool_status_health_check` | Run health check | (none) |

### Ecosystem (5)
| Tool | Purpose | Key Input |
|------|---------|-----------|
| `tool_ecosystem_servers_list` | List servers | type, status |
| `tool_ecosystem_server_get` | Server details | serverId |
| `tool_ecosystem_map` | Ecosystem map | (none) |
| `tool_ecosystem_health` | Ecosystem health | (none) |
| `tool_ecosystem_statistics` | Ecosystem stats | (none) |

### Optimization (4)
| Tool | Purpose | Key Input |
|------|---------|-----------|
| `tool_ecosystem_coordinate` | Coordinate ops | operation, servers |
| `tool_optimizer_track_operation` | Track metrics | operationId, metrics |
| `tool_optimizer_suggest` | Get suggestions | targetMetric |
| `tool_optimizer_statistics` | Optimization stats | (none) |

### Integration Adapters (6)
| Tool | Purpose | Key Input |
|------|---------|-----------|
| `tool_guardian_validate` | Validate via guardian | operation, params |
| `tool_quartermaster_recommend` | QM recommendations | path, action |
| `tool_search_routing_suggest` | Search routing | query, context |
| `tool_glec_classify` | GLEC classify | content, contentType |
| `tool_glec_bulk_classify` | Bulk GLEC | tools |
| `discover_tools` | Discover all tools | force |

### Claude Integration (3)
| Tool | Purpose | Key Input |
|------|---------|-----------|
| `tool_claude_desktop_capabilities` | Desktop capabilities | capability |
| `tool_claude_desktop_updates` | Desktop updates | (none) |
| `tool_remind_claude_capabilities` | Remind capabilities | (none) |

---

## CLAUDE-CODE-BRIDGE (8 tools)

**Purpose:** Desktop-to-Code communication, work order management

| Tool | Purpose | Key Input |
|------|---------|-----------|
| `create_work_order` | Create task for Code | sessionId, operation, targetPath, fileCount |
| `get_work_order` | Get work order | id |
| `list_work_orders` | List work orders | status |
| `update_work_order` | Report progress | id, status, results |
| `cancel_work_order` | Cancel work order | id |
| `get_bridge_status` | Bridge health | (none) |
| `get_neural_metrics` | Performance metrics | timeWindow |
| `prune_old_work_orders` | Archive old orders | olderThanDays |

---

## FILESYSTEM-TEST (12 tools)

**Purpose:** File system CRUD operations

| Tool | Purpose | Key Input |
|------|---------|-----------|
| `read_file` | Read file | path |
| `read_multiple_files` | Read many files | paths |
| `write_file` | Write file | path, content |
| `edit_file` | Find/replace | path, edits, dry_run |
| `create_directory` | Create dir | path |
| `list_directory` | List dir | path |
| `directory_tree` | Tree view | path, depth |
| `move_file` | Move/rename | source, destination |
| `search_files` | Glob search | path, pattern |
| `get_file_info` | File metadata | path |
| `read_media_file` | Read binary | path |
| `list_directory_with_sizes` | List with sizes | path, sort_by |

---

## VERIFIER-MCP (3 tools)

**Purpose:** Claim extraction, fact verification

| Tool | Purpose | Key Input |
|------|---------|-----------|
| `extract_claims` | Extract claims | text, context, include_implicit |
| `verify_claims` | Verify claims | domain, claims, options |
| `list_sources` | Query sources | domain, query |

---

## CONSCIOUSNESS-MCP (10 tools)

**Purpose:** Meta-awareness, pattern learning, ecosystem reflection

| Tool | Purpose | Key Input |
|------|---------|-----------|
| `track_focus` | Log current focus to build attention history | event_type, target, server_name |
| `get_attention_patterns` | Analyze access patterns - hotspots, trends, anomalies | time_range, pattern_type |
| `identify_blind_spots` | Find what's NOT being attended to | scope, time_range |
| `reflect_on_operation` | Analyze operation - what worked, what could be better | operation_type, depth |
| `analyze_pattern` | Find recurring successes/failures across operations | pattern_query, depth |
| `audit_reasoning` | Extract assumptions, identify logical gaps | reasoning_text, verify_claims |
| `predict_outcome` | Predict success/failure from historical patterns | operation_description, context |
| `synthesize_context` | Integrate multiple sources into unified perspective | sources, question, depth |
| `suggest_next_action` | Recommend next steps based on patterns | current_context, goals |
| `get_ecosystem_awareness` | Get real-time state of entire ecosystem | include_history, history_hours |

---

## EXPERIENCE-LAYER (6 tools)

**Purpose:** Experiential memory, PSOM episodes, pattern detection, lessons

| Tool | Purpose | Key Input |
|------|---------|-----------|
| `record_experience` | Store episode with PSOM structure | operation_type, outcome, problem, solution |
| `recall_by_type` | Get past experiences by operation type | operation_type, outcome_filter, limit |
| `recall_by_outcome` | Get experiences by outcome | outcome, operation_type, limit |
| `get_lessons` | Retrieve applicable lessons with confidence | context, operation_type, min_confidence |
| `apply_lesson` | Mark lesson applied, update confidence | lesson_id, outcome, notes |
| `learn_from_pattern` | Extract lesson from recurring pattern | pattern_description, episode_ids, lesson_statement |

---

## SKILL-BUILDER (8 tools)

**Purpose:** SKILL.md creation, validation, matching, usage tracking

| Tool | Purpose | Key Input |
|------|---------|-----------|
| `create_skill` | Generate SKILL.md file | name, description, overview, steps |
| `validate_skill` | Check structure, frontmatter, token counts | skill_path or skill_content |
| `analyze_description` | Assess description quality | description |
| `list_skills` | List all indexed skills | status, sort_by, limit |
| `get_skill` | Get full skill content by ID | skill_id, include_layer3 |
| `update_skill` | Update existing skill | skill_id, updates |
| `match_skill` | Find skills matching task description | task_description, min_confidence, limit |
| `record_skill_usage` | Record skill usage and outcome | skill_id, outcome, context |

---

## PERCOLATION-SERVER (8 tools)

**Purpose:** Blueprint optimization, stress testing, hole detection

| Tool | Purpose | Key Input |
|------|---------|-----------|
| `submit_blueprint` | Submit blueprint for percolation | content, metadata, config |
| `get_blueprint_status` | Check percolation progress | blueprint_id |
| `stress_test` | Run stress test on blueprint | blueprint_id, test_type, parameters |
| `find_holes` | Detect gaps and weaknesses | blueprint_id, analysis_depth |
| `patch_hole` | Apply fix to detected hole | blueprint_id, hole_id, patch |
| `research_improvements` | Research optimization opportunities | blueprint_id, focus_areas |
| `apply_optimization` | Apply optimization to blueprint | blueprint_id, optimization_id |
| `get_percolation_result` | Get final percolation result | blueprint_id, include_history |

---

## TENETS-SERVER (8 tools)

**Purpose:** Ethical evaluation against 25 Gospel tenets

| Tool | Purpose | Key Input |
|------|---------|-----------|
| `evaluate_decision` | Assess decision against all tenets | decision_text, context, depth |
| `check_counterfeit` | Check for counterfeit patterns | action_description, claimed_tenet |
| `identify_blind_spots` | Find ethical gaps in plan | plan_description, stakeholders |
| `get_tenet` | Get specific tenet details | tenet_number |
| `list_tenets` | List all 25 tenets | category, include_counterfeits |
| `get_evaluation_history` | Get past evaluations | filter, limit |
| `suggest_alternatives` | Suggest ethical alternatives | rejected_action, context |
| `validate_consistency` | Check consistency across decisions | decision_ids |

---

## CONSOLIDATION-ENGINE (6 tools)

**Purpose:** Document merging, conflict detection, BBB integration

| Tool | Purpose | Key Input |
|------|---------|-----------|
| `create_merge_plan` | Create plan to merge documents | source_files, strategy |
| `detect_conflicts` | Detect conflicts before merge | file_paths |
| `merge_documents` | Execute merge with conflict resolution | plan_id, resolution_strategy |
| `resolve_conflicts` | Manually resolve detected conflicts | conflict_id, resolution |
| `get_merge_history` | Get history of merge operations | limit, status |
| `validate_plan` | Validate merge plan before execution | plan_id |

---

## FILESYSTEM-GUARDIAN (6 tools)

**Purpose:** macOS extended attributes, Spotlight integration

| Tool | Purpose | Key Input |
|------|---------|-----------|
| `get_xattr` | Get extended attribute from file | path, attribute_name |
| `set_xattr` | Set extended attribute on file | path, attribute_name, value |
| `list_xattr` | List all extended attributes | path |
| `spotlight_search` | Search using macOS Spotlight mdfind | query, scope, limit |
| `spotlight_reindex` | Force Spotlight to reindex path | path |
| `watch_volume` | Monitor volume for filesystem events | volume_path, event_types |

---

## HEALTH-MONITOR (7 tools)

**Purpose:** Ecosystem health monitoring, alerts, metrics

| Tool | Purpose | Key Input |
|------|---------|-----------|
| `check_health` | Check health of specific server | server_name |
| `get_aggregated_health` | Get aggregated health across servers | include_details |
| `get_metrics` | Get performance metrics | server_name, time_range |
| `get_server_status` | Get current status of server | server_name |
| `get_server_history` | Get historical health data | server_name, hours |
| `subscribe_to_alerts` | Subscribe to health alerts | alert_types, callback |
| `acknowledge_alert` | Acknowledge and dismiss alert | alert_id, notes |

---

## INTAKE-GUARDIAN (5 tools)

**Purpose:** Content admission, validation, gatekeeper

| Tool | Purpose | Key Input |
|------|---------|-----------|
| `check_content` | Check content against admission criteria | content, content_type |
| `check_file` | Check file for admission eligibility | file_path |
| `admit_content` | Admit content after validation | content_id, destination |
| `configure_thresholds` | Configure admission thresholds | threshold_type, value |
| `get_intake_history` | Get history of intake decisions | filter, limit |

---

## SAFE-BATCH-PROCESSOR (4 tools)

**Purpose:** Batch operations with rollback capability

| Tool | Purpose | Key Input |
|------|---------|-----------|
| `execute_batch` | Execute batch with rollback capability | operations, dry_run |
| `get_batch_status` | Get status of batch operation | batch_id |
| `rollback_batch` | Rollback completed batch | batch_id, confirm |
| `validate_batch` | Validate batch before execution | operations |

---

## SYNAPSE-RELAY (4 tools)

**Purpose:** InterLock signal routing, buffering

| Tool | Purpose | Key Input |
|------|---------|-----------|
| `relay_signal` | Relay signal to specified targets | signal, targets |
| `buffer_signals` | Buffer signals for batch delivery | signals, delay_ms |
| `get_relay_stats` | Get relay statistics and throughput | time_range |
| `configure_relay` | Configure relay routing rules | rules |

---

## TOOL COUNT BY CATEGORY

| Category | Servers | Tools |
|----------|---------|-------|
| Files/Search | 4 | 46 |
| Classification | 2 | 9 |
| Safety | 4 | 28 |
| Research | 3 | 20 |
| Temporal | 1 | 19 |
| Orchestration | 3 | 20 |
| Meta/Generation | 1 | 26 |
| Meta/Registry | 1 | 65 |
| Knowledge | 2 | 10 |
| News/Media | 1 | 102 |
| Verification | 1 | 3 |
| Routing | 1 | 2 |
| Bridge | 1 | 8 |
| Backup | 1 | 6 |
| Cognitive | 5 | 40 |
| Batch/Merge | 2 | 10 |
| Filesystem | 1 | 6 |
| Health | 2 | 11 |

---

## USAGE PATTERNS

### Common Tool Chains

```
CLASSIFY + ORGANIZE:
catasorter:classify_document → smart-file-organizer:organize_files

SEARCH + VALIDATE:
quartermaster:hybrid_find → context-guardian:context_guardian_validate

RESEARCH + VERIFY:
research-bus:research_query → verifier-mcp:verify_claims

BACKUP + RESTORE:
snapshot:snapshot_capture_snapshot → snapshot:snapshot_restore_snapshot

CLUSTER + SCRIPT:
niws-server:story_cluster_articles → niws-server:script_generate
```

### Server Collaboration

| From | To | Signal |
|------|-----|--------|
| intelligent-router | any | route_request workflow |
| trinity-coordinator | claude-code | delegate_to_code |
| neurogenesis-engine | all | fire_neurotransmitter |
| chronos-synapse | any | track_event logging |
| context-guardian | any | validate before action |


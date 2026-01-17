SERVER: niws-intake
VERSION: 1.1
UPDATED: 2026-01-17
STATUS: Production
PORTS: UDP 3033 | HTTP 8033 | WS 9033
MCP: stdio transport (stdin/stdout JSON-RPC)
INTERLOCK: Enabled (peer mesh communication)
DEPS: Congress.gov API (optional, for bill polling)
PURPOSE: News intake pipeline - RSS feeds, outlets, story clustering, and congressional bill tracking
CONFIG: ENV NIWS_INTAKE_PORT (default 8033), CONGRESS_API_KEY

---

ARCHITECTURE

FUNCTION: First stage of NIWS pipeline - ingestion and clustering
FLOW: RSS Feeds -> Articles -> Story Clusters -> Editorial Dashboard
STORAGE: SQLite (outlets.db, articles.db, bills.db)
DATABASES:
  - outletDatabase: News outlets, feeds, bias ratings
  - articleDatabase: RSS articles, fetch history, story assignments
  - billDatabase: Congressional bills, votes, text, article links
CLUSTERING: TF-IDF similarity with editorial track classification

---

TOOL CATEGORIES

OUTLETS (10 tools):
- add_outlet: Add news outlet with political lean classification
- update_outlet: Update outlet metadata
- delete_outlet: Delete outlet and all feeds
- get_outlet: Get outlet by ID with feeds and ratings
- list_outlets: List all outlets grouped by lean
- get_outlets_by_lean: Filter outlets by political lean
- add_feed: Add RSS feed to outlet
- remove_feed: Remove feed from outlet
- get_outlet_feeds: Get all feeds for outlet
- get_outlet_stats: Database statistics

RSS (14 tools):
- poll_all_feeds: Poll all active RSS feeds
- poll_outlet_feeds: Poll feeds for specific outlet
- poll_single_feed: Poll single feed
- get_article: Get article by ID
- get_articles: Query articles with filters
- get_articles_by_outlet: Articles from specific outlet
- get_articles_by_lean: Articles by outlet political lean
- search_articles: Keyword search in titles/descriptions
- get_recent_articles: Articles from last N hours
- mark_article_processed: Mark article as processed
- get_feed_status: Feed health and poll info
- get_poll_history: Poll logs for feed
- clear_old_articles: Delete articles older than N days
- validate_feed: Validate RSS feed URL before adding

STORIES (8 tools):
- cluster_recent_stories: Cluster articles into stories with track tagging
- get_story: Get story by ID
- get_stories: Query stories with filters
- get_stories_by_track: Filter by editorial track
- get_story_articles: Get articles in story cluster
- get_editorial_dashboard: Top stories across all tracks
- search_stories: Search stories by keyword
- compare_outlet_coverage: Compare how outlets covered same story

BILLS (15 tools):
- search_bills: Search by keyword, sponsor, policy area
- get_bill: Get detailed bill information
- get_bill_text: Get bill text (specific or latest version)
- search_bill_text: Full-text search across bill texts
- get_bill_actions: Bill action timeline
- get_bill_votes: Roll call votes for bill
- get_vote_details: Detailed vote with member positions
- get_member_voting_record: Member's voting history
- link_article_to_bill: Link news article to legislation
- get_articles_for_bill: Articles referencing bill
- get_bills_for_article: Bills referenced in article
- sync_congress: Full sync of congress from Congress.gov
- update_bill: Refresh specific bill
- fetch_bill_text_content: Fetch and store bill text
- get_bill_stats: Bill database statistics

TOTAL: 47 tools

---

EDITORIAL TRACKS

TRACK: trending
DESC: High coverage volume across outlets
CRITERIA: 3+ outlets, significant overlap

TRACK: christ_o_meter
DESC: Government actions for moral scoring
CRITERIA: Legislation, executive action, policy keywords

TRACK: environment_crisis
DESC: Environmental disasters and crises
CRITERIA: Climate, pollution, disaster keywords

TRACK: environment_hope
DESC: Environmental protection and restoration
CRITERIA: Conservation, renewable, cleanup keywords

TRACK: underreported
DESC: Significant stories with low coverage
CRITERIA: 1-2 outlets only, high engagement signals

---

KEY CONCEPTS

OUTLET: News source with political lean and bias rating
FEED: RSS feed attached to outlet with poll frequency
ARTICLE: Individual news item from feed
STORY_CLUSTER: Group of related articles about same topic
POLITICAL_LEAN: left | left-center | center | right-center | right
LEAN_COVERAGE: Distribution of coverage across political spectrum

---

DATABASE SCHEMA

TABLE: outlets
COLUMNS: id, name, domain, political_lean, bias_rating, factual_reporting, country, created_at

TABLE: feeds
COLUMNS: id, outlet_id, feed_url, feed_type, category, poll_frequency_minutes, last_polled_at, status

TABLE: articles
COLUMNS: id, feed_id, outlet_id, url, title, summary, content_hash, published_at, fetched_at, processed, story_id

TABLE: stories
COLUMNS: id, title, keywords, tracks, outlet_count, article_count, lean_coverage, first_published, created_at

TABLE: bills
COLUMNS: id, congress, bill_type, bill_number, title, short_title, sponsor_id, sponsor_name, sponsor_party, latest_action_text, latest_action_date, policy_area

TABLE: bill_texts
COLUMNS: id, bill_id, version_code, version_name, date, format, url, full_text, word_count

TABLE: bill_votes
COLUMNS: id, bill_id, chamber, roll_number, vote_date, vote_question, vote_result, yea_count, nay_count

TABLE: article_bill_links
COLUMNS: id, article_id, bill_id, confidence, link_type, extracted_quote, created_at

---

HTTP ENDPOINTS

GET /api/tools - List all MCP tools (Gateway integration)
POST /api/tools/:toolName - Execute MCP tool (Gateway integration)
GET /api/health - Health check
GET /api/status - Server status with stats
GET /api/outlets - List outlets
GET /api/outlets/:id - Get outlet
POST /api/outlets - Create outlet
GET /api/outlets/:id/feeds - Get outlet feeds
POST /api/feeds/poll - Trigger feed poll
GET /api/articles - Query articles
GET /api/articles/:id - Get article
GET /api/stories - List stories
GET /api/stories/:id - Get story
GET /api/stories/:id/articles - Get story articles
GET /api/dashboard - Editorial dashboard
GET /api/bills - Search bills
GET /api/bills/:id - Get bill

---

WEBSOCKET EVENTS

EVENT: article:new - New article fetched
EVENT: story:created - New story cluster formed
EVENT: story:updated - Story cluster updated
EVENT: poll:complete - Feed poll completed
EVENT: poll:error - Feed poll failed
EVENT: bill:updated - Bill information updated

---

INTERLOCK SIGNALS

SIGNAL: story_clustered - Broadcast new story cluster
SIGNAL: articles_ready - Notify articles available for analysis
SIGNAL: outlet_added - New outlet registered
SIGNAL: poll_complete - Feed polling batch complete
PEERS: niws-analysis (3034), niws-production (3035), niws-delivery (3036)

---

KEY FILES

SOURCE: /repo/niws-intake/
INDEX: src/index.ts
HTTP: src/http/server.ts
WS: src/websocket/server.ts
INTERLOCK: src/interlock/
TOOLS:
  - src/tools/outlet-tools.ts
  - src/tools/rss-tools.ts
  - src/tools/story-tools.ts
  - src/tools/bill-tools.ts
DATABASE:
  - src/database/outletDatabase.ts
  - src/database/articleDatabase.ts
  - src/database/billDatabase.ts
SERVICES:
  - src/services/feedPoller.ts
  - src/services/rssParser.ts
  - src/services/storyClustering.ts
  - src/services/billPoller.ts
  - src/services/congressApi.ts
CONFIG: config/interlock.json
DATA: data/outlets.db, data/articles.db, data/bills.db

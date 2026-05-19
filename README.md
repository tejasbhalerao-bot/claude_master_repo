# Tejas's Claude Code — Master Reference

One-stop reference for everything Tejas's Claude Code can do.
Agents · Plugins · MCPs · Skills · Routing Rules.

---

## What This Is

Claude Code configured as a PM + Data + Ops assistant for **Tejas Bhalerao**, Product Manager at **Truemeds** — an online pharmacy operating across Hyperlocal Forward/Reverse, Courier Forward/Reverse, B2B Forward/Reverse.

Master config: `~/Documents/claude-os/CLAUDE.md`

---

## Quick Routing Table

| Task | What to say |
|------|-------------|
| Create a PRD | `Create a PRD. Feature: X, Problem: Y, Success Metric: Z` |
| Review a PRD | `Review the PRD for [feature]` |
| Design an A/B experiment | `Design an experiment. Hypothesis: X, Target Metric: Y, Baseline: Z` |
| Review an experiment | `Review the experiment for [feature]` |
| Map stakeholder objections | `Map objections for [feature]` |
| Write an exec brief | `Write an exec brief. Feature: X, Audience: CEO` |
| Design test cases | `Design test cases for [feature]` + Google Drive PRD link |
| Run a Metabase query | `Query [what you want to know]` |
| Analyse a CSV / Excel | Upload file → `Analyse this. Question: X` |
| Weekly KPI digest | `Run weekly metrics monitor` |
| Post-release analysis | `Analyse [feature] post-release` |
| Find a meeting slot | `Find a slot for [attendees] on [day]` |
| Create a calendar invite | `Create an invite for [meeting name]` |
| Sprint board / blockers | `What's the sprint status?` |
| Create Jira epics from PRD | `Create Jira epics for [PRD]` |
| Stakeholder update email | `Draft a stakeholder update for sprint [X]` |
| Research a market/competitor | `Research [topic] last 30 days` |
| Build a GTM strategy | `Build GTM for [product]` |
| Slide deck | `Create a deck for [topic]` |
| Understand a codebase | `Understand [repo]` |

All PM workflows route through `~/pm-agent/workflows/supporting/recall-and-route.md` first.

---

## PM Agent — `~/pm-agent/`

GitHub-backed PM workflow system. Claude loads Truemeds org context from Google Drive → creates documents → reviews them → maps objections → auto-versions → pushes to GitHub. Everything is automated — zero manual file management.

### How to trigger

Paste a workflow prompt into Claude Code. Entry point is always:
```
Entry point: ~/pm-agent/workflows/supporting/recall-and-route.md
```

### Core Workflows

#### Create a PRD
```
Create a PRD.
Feature: [Feature name]
Problem: [What problem does it solve?]
Success Metric: [How will you measure success?]
Entry point: ~/pm-agent/workflows/supporting/recall-and-route.md
```

**What happens automatically:**
1. Loads Truemeds org context from Google Drive (cached same-day)
2. Asks for detailed problem statement + high-level solution if not provided
3. Fetches past PRDs from Drive to mirror writing style
4. Drafts the PRD with: RACI · Objective · Why Now · Use Cases · Metrics (with baseline, target, timeframe) · Rollout & Stage Gates
5. PRD Reviewer runs automatically (min 2 passes, P0/P1/P2 severity system)
6. Loops until no P0s remain, then offers sign-off
7. Saves to `~/pm-agent/archives/projects/[date]-[feature]-v1.md`
8. Offers to schedule a PRD review via Scheduler Agent
9. Objection Mapper runs automatically
10. Pushes to GitHub

Supports: **Executable PRD** · **Initiative Doc** (multiple chained PRDs) · **Vision Doc** (custom structure)

---

#### Design an Experiment
```
Design an experiment.
Hypothesis: [We believe X will cause Y to move by Z because ...]
Target Metric: [single north star metric]
Current Value: [baseline]
Target Value: [what improvement matters?]
Entry point: ~/pm-agent/workflows/supporting/recall-and-route.md
```

**What the XP Doc covers:**
- Formal hypothesis with magnitude estimate
- Primary metric · Secondary metrics · Guardrail metrics (with breach thresholds and actions)
- Full statistical design: MDE · α (default 0.05) · Power (default 0.80) · Sample size calculation with formula shown · Test type (z-test / t-test / Mann-Whitney / ANOVA / log-rank) · Duration estimate
- Traffic allocation table with mandatory holdout group (≥10%)
- Unit of randomization: `customer_id` preferred · hash-based deterministic assignment · bucket ranges defined
- Eligibility specification with mid-experiment change handling
- Interaction effects check against live experiments tracker in Drive
- Pre-registered decision criteria: Ship / Iterate / Kill / Inconclusive — defined before the experiment runs
- Pre-registered early stopping rules

Chains to **Metrics Designer** automatically to validate metric measurability in Metabase/Mixpanel.
Saves to: `~/pm-agent/archives/experiments/[date]-[feature]-v1.md`

---

#### Review a PRD
```
Review the PRD for [feature].
Entry point: ~/pm-agent/workflows/supporting/recall-and-route.md
```

**How the review works:**
- Minimum 2 passes, automatic loop until no P0s remain
- Reviews every section against 3 rubrics: Clarity · Metrics Quality · Use Case Coverage
- Severity system: P0 (blocks sign-off) · P1 (must fix) · P2 (polish)
- Visual findings widget rendered after each pass: colour-coded rows, pass count summary, resolved/persists/new tracking from pass 2 onward
- Sign-off only available after Pass 2 with no P0s

---

#### Map Stakeholder Objections
```
Map objections for [feature-name].
Entry point: ~/pm-agent/workflows/supporting/recall-and-route.md
```

**What it fetches before mapping:**
- Quarter Plan, AOP, In-flight Initiative Docs (Category A: strategic conflicts)
- All SOPs for touched verticals (Category B: process breaks)
- Existing PRDs in same domain (Category C: duplicated work, contradicted decisions, unacknowledged dependencies)
- Checks freshness of all fetched docs — flags stale context (>90 days old)

Output: objections grouped by stakeholder persona, each traced to a source doc. No suggested counters — just ammunition for your alignment meeting.
Saves to: `~/pm-agent/archives/objections/[date]-[feature]-v1-objections.md`

---

#### Write an Exec Brief
```
Write an executive brief.
Feature: [Feature name]
Audience: [CEO / CFO / CTO / COO / Board]
Entry point: ~/pm-agent/workflows/supporting/recall-and-route.md
```

**Output: 4-section brief, max 400 words**
- **The Ask** — precise decision or approval request (not a vague ask for "support")
- **Why Now** — at least one recent data point, operational constraint, or strategic window
- **Options Considered** — 2-3 row table including a "do nothing" option; each rejected option has a concrete reason
- **Recommendation** — path forward, quantified expected outcome, key risk and mitigation

Audience calibration is applied per section (CEO = growth/revenue framing, CFO = ROI/cost, CTO = technical risk, COO = operational impact, Board = governance).

Chains to **Objection Mapper** automatically after saving.
Saves to: `~/pm-agent/archives/briefs/[date]-[feature]-v1-brief.md`

---

#### Design Test Cases
```
Design test cases for [feature].
[Paste Google Drive PRD link]
Entry point: ~/pm-agent/workflows/supporting/recall-and-route.md
```

Fetches PRD from Drive · Analyses for happy path, alternate flows, edge cases, error scenarios, business logic (OTD%, promise calibration, etc.) · Generates 15-30 functional test cases in structured format with ID, preconditions, steps, expected result · Scope: functional only (no performance/security/accessibility).
Saves to: `~/pm-agent/archives/test-cases/[date]-[feature]-v1.md`

---

### Where Files Go

```
~/pm-agent/archives/projects/      → PRDs
~/pm-agent/archives/experiments/   → Experiment designs
~/pm-agent/archives/objections/    → Objection maps (linked to PRD versions)
~/pm-agent/archives/briefs/        → Exec briefs
~/pm-agent/archives/test-cases/    → Functional test suites
```

Versioning is automatic: v1 → v2 → v3. Claude detects existing versions and increments.

### Self-Learning System

`~/pm-agent/scripts/run-synthesis.js` (weekly) analyses recent git commits, extracts corrections and new patterns, appends to `~/pm-agent/changelogs/`. The agent improves over time based on actual usage — no manual changelog edits needed.

### GitHub Repo
`https://github.com/tejasbhalerao-bot/pm-agent`

---

## Data Agent — `~/data-agent/`

Skills for Truemeds Redshift data via Metabase, CSV/Excel analysis, and KPI monitoring.
Skill files live in `~/Documents/claude-os/data-agent/`.

**Hard rules for all data work:**
- Always prefix Redshift tables with `tmmumpsdb.`
- Always read the canonical schema reference doc — never infer from memory or prior queries
- Always present a plan and get sign-off before running any query or script
- Never dump raw data — always interpret through Truemeds business context

---

### Schema Reader + Query Runner

Triggered when: you ask about table structure, column names, data availability, or want to run any Metabase query.

**How it works:**
1. Checks session for existing schema manifest before re-fetching
2. Reads canonical schema reference doc from Google Drive
3. Understands the business question before touching any table
4. Selects only the tables needed (not full schema dump)
5. Applies mandatory business rules automatically:

| Table | Key Rule |
|---|---|
| `Order Details` | Always excludes unplaced orders: `orderstatus NOT IN (49, 312)` |
| `Order TAT Details` | Actual delivery = `Delivery Attempt Time` (latest attempt). Actual dispatch = `Pickup Time` |
| `Delivery Date Tracker` | Promise data overwritten until doctor confirms. Doctor hours: 7:30 AM–11:30 PM always |
| `Pincode Delivery TAT` | Use `Delivery Days in Mins` if non-NULL; fall back to `Delivery Days` only when NULL |
| `M Courier Partner Master` | `SVM ID` is the join key for all delivery partner references |
| `Order Status` | Always join with `M System Value Master` for human-readable status — never use raw integer |
| `Warehouse Details` | Always confirm with Tejas: is it still active? Is it FC or MFC? |

6. Emits schema manifest (tables in scope, columns used, join keys, business rules applied, assumptions)
7. Presents query plan and waits for explicit sign-off
8. Writes SQL, shows it before running
9. Executes via Metabase MCP
10. Interprets results with Truemeds context — leads with the direct answer, flags anomalies

---

### Raw Data Analyser

Triggered when: you upload a CSV or Excel file.

**How it works:**
1. Loads Truemeds org context first
2. Asks: what question are you trying to answer? Where did this file come from?
3. Reads header + first 20-30 rows of each file
4. Maps columns to Truemeds core metrics (fill rate, cancellation, SLA, GMV, etc.)
5. Presents analysis plan — waits for sign-off before writing any script
6. Generates a custom Python script scoped exactly to the confirmed plan
7. Outputs findings in priority order: data quality issues → anomalies in core metrics → patterns relevant to stated intent → cross-file discrepancies
8. Offers concrete drill-down questions based on findings
9. Saves report to `[Data Analysis] <title> <date>.md`

---

### Weekly Metrics Monitor

Triggered when: you ask for a weekly KPI digest.

Runs WoW KPI analysis across all fulfilment verticals. Generates custom SQL after plan sign-off. Output is interpreted — not a table dump.

---

### Post-Release Analyser

Two modes:

**Release Mode** — triggered when: you ask how a feature performed after launch.
Measures metric movement post-release vs baseline.

**Experiment Mode** — triggered when: you ask to monitor a live A/B experiment.
Covers: metric movement per variant, guardrail checks, SRM (Sample Ratio Mismatch) detection.

---

### Metrics Designer

Triggered when: you need to define, classify, or validate metrics for a feature or experiment.

Defines primary, secondary, and guardrail metrics. Validates measurability with data available in Metabase/Mixpanel. Always chained automatically by Experiment Designer.

---

## Scheduler Agent

| Workflow | What to say |
|---|---|
| Find a free slot | `Find a slot for [attendees] on [day] for [duration]` |
| Create an invite | `Create an invite for [meeting] with [attendees]` |

Slot-finder checks attendee calendar availability, respects timezone, preferred windows.
Invite-creator drafts agenda and attaches relevant docs automatically.

---

## Scrum Agent

| Workflow | What to say |
|---|---|
| Sprint status + blockers | `What's the sprint status?` |
| Standup / sprint summary | `Run standup` or `Sprint summary for sprint [X]` |
| Create Jira epics from PRD | `Create Jira epics for [PRD name]` |
| Stakeholder update email | `Draft a stakeholder update for [audience]` |

Jira Epic Creator generates epics and child stories from a signed-off PRD.
Stakeholder Update Drafter creates Gmail drafts calibrated to audience — does not send.

---

## MCP Integrations (Live Data Sources)

### Always Connected

| MCP | What It Provides |
|---|---|
| **Metabase (TM Chotu)** | Truemeds Redshift SQL execution, schema lookup — `query`, `execute_query`, `construct_query`, `get_table`, `search` |
| **Mixpanel** | Event queries, funnel analysis, retention, experiments (44 tools) — reports, dashboards, feature flags, lexicon |
| **Gmail** | Email search, thread view, draft creation, label management |
| **Google Calendar** | Event creation, availability check, invite management |
| **Google Drive** | Org docs, SOPs, PRD library, schema reference doc |
| **Lucid** | Lucidchart / Lucidspark diagram access |

### Auth-Required (connect when needed)

| MCP | Domain |
|---|---|
| Atlassian (Jira / Confluence) | Sprint tracking, epic creation, wiki |
| Slack | Channel read, message send, thread search |
| Notion | Notes, docs, databases |
| Linear | Engineering task tracking |
| ClickUp / Monday.com | Project management |
| MS365 | Office docs, Teams |
| Amplitude (EU + global) | Product analytics |
| Figma | Design file access |
| Miro | Board access, diagramming |
| Intercom | Customer support data |
| Pendo | In-app analytics |
| Fireflies | Meeting transcripts |
| Ahrefs | SEO and backlink data |
| BigQuery | Cloud data warehouse |
| Apollo / Close / Outreach | Sales intelligence and CRM |
| Klaviyo / Supermetrics | Marketing analytics |

---

## Installed Plugins — Full Catalog

### PM Skills (`pm-skills`)

**Product Strategy**
`/market-scan` · `/strategy` · `/product-strategy` · `/product-vision` · `/value-proposition` · `/business-model` · `/pricing` · `/pricing-strategy` · `/monetization-strategy` · `/lean-canvas` · `/swot-analysis` · `/startup-canvas` · `/ansoff-matrix` · `/porters-five-forces` · `/pestle-analysis`

**Market Research**
`/research-users` · `/analyze-feedback` · `/competitive-analysis` · `/competitor-analysis` · `/sentiment-analysis` · `/market-segments` · `/user-personas` · `/user-segmentation` · `/customer-journey-map` · `/market-sizing`

**Marketing & Growth**
`/market-product` · `/north-star` · `/north-star-metric` · `/positioning-ideas` · `/product-name` · `/marketing-ideas` · `/value-prop-statements`

**Go-To-Market**
`/plan-launch` · `/growth-strategy` · `/gtm-strategy` · `/gtm-motions` · `/battlecard` · `/competitive-battlecard` · `/growth-loops` · `/ideal-customer-profile` · `/beachhead-segment`

**Execution**
`/write-prd` · `/create-prd` · `/meeting-notes` · `/summarize-meeting` · `/write-stories` · `/user-stories` · `/job-stories` · `/transform-roadmap` · `/outcome-roadmap` · `/pre-mortem` · `/sprint` · `/sprint-plan` · `/plan-okrs` · `/brainstorm-okrs` · `/stakeholder-map` · `/test-scenarios` · `/release-notes` · `/prioritization-frameworks` · `/wwas` · `/retro` · `/generate-data` · `/dummy-dataset`

**Data Analytics**
`/analyze-cohorts` · `/cohort-analysis` · `/write-query` · `/sql-queries` · `/analyze-test` · `/ab-test-analysis`

**Product Discovery**
`/triage-requests` · `/analyze-feature-requests` · `/discover` · `/brainstorm` · `/interview` · `/interview-script` · `/summarize-interview` · `/setup-metrics` · `/metrics-dashboard` · `/opportunity-solution-tree` · `/identify-assumptions-new` · `/identify-assumptions-existing` · `/prioritize-assumptions` · `/brainstorm-ideas-new` · `/brainstorm-ideas-existing` · `/brainstorm-experiments-new` · `/brainstorm-experiments-existing` · `/prioritize-features`

**Toolkit**
`/tailor-resume` · `/review-resume` · `/proofread` · `/grammar-check` · `/draft-nda` · `/privacy-policy`

---

### Knowledge Work Plugins (`knowledge-work-plugins`)

**Product Management**
`/product-management:brainstorm` · `/metrics-review` · `/synthesize-research` · `/write-spec` · `/competitive-brief` · `/sprint-planning` · `/product-brainstorming` · `/stakeholder-update` · `/roadmap-update`

**Data**
`/data:analyze` · `/explore-data` · `/validate-data` · `/statistical-analysis` · `/data-visualization` · `/create-viz` · `/build-dashboard` · `/data:sql-queries` · `/write-query` · `/data-context-extractor`

**Marketing**
`/draft-content` · `/content-creation` · `/marketing:competitive-brief` · `/brand-review` · `/performance-report` · `/email-sequence` · `/campaign-plan` · `/seo-audit`

**Sales**
`/call-prep` · `/pipeline-review` · `/sales:competitive-intelligence` · `/call-summary` · `/account-research` · `/daily-briefing` · `/create-an-asset` · `/draft-outreach` · `/forecast`

**Finance**
`/financial-statements` · `/audit-support` · `/close-management` · `/variance-analysis` · `/journal-entry-prep` · `/sox-testing` · `/reconciliation` · `/journal-entry`

**Operations**
`/process-optimization` · `/change-request` · `/capacity-plan` · `/vendor-review` · `/risk-assessment` · `/process-doc` · `/compliance-tracking` · `/status-report` · `/runbook`

**Design**
`/design-system` · `/accessibility-review` · `/design-critique` · `/design-handoff` · `/ux-copy` · `/research-synthesis` · `/user-research`

**Enterprise Search**
`/enterprise-search:search` · `/source-management` · `/digest` · `/search-strategy` · `/knowledge-synthesis`

**Brand Voice**
`/brand-voice:generate-guidelines` · `/enforce-voice` · `/brand-voice:discover-brand` · `/guideline-generation` · `/brand-voice-enforcement`

**Productivity**
`/productivity:start` · `/task-management` · `/memory-management` · `/productivity:update`

**Tools**
`/pdf-viewer:open` · `/fill-form` · `/sign` · `/annotate` · `/view-pdf`
`/miro:miro-browse` · `/miro-diagram` · `/miro-table` · `/miro-doc` · `/miro-code-review` · `/miro-code-spec`
`/figma:figma-use` · `/figma-generate-design` · `/figma-generate-diagram` · `/figma-generate-library` · `/figma-create-new-file` · `/figma-use-slides` · `/figma-use-figjam` · `/figma-code-connect`

---

### Specialized Plugins

#### BMAD Method — Full agile product lifecycle

**Pro Skills (meta-tools)**
`/bmad-help` · `/bmad-brainstorming` · `/bmad-distillator` · `/bmad-party-mode` · `/bmad-shard-doc` · `/bmad-advanced-elicitation` · `/bmad-editorial-review-prose` · `/bmad-editorial-review-structure` · `/bmad-index-docs` · `/bmad-review-adversarial-general` · `/bmad-review-edge-case-hunter`

**Lifecycle (in order)**
`/bmad-product-brief` → `/bmad-domain-research` → `/bmad-market-research` → `/bmad-technical-research` → `/bmad-create-prd` → `/bmad-validate-prd` → `/bmad-create-ux-design` → `/bmad-create-architecture` → `/bmad-check-implementation-readiness` → `/bmad-create-epics-and-stories` → `/bmad-dev-story` → `/bmad-code-review` → `/bmad-qa-generate-e2e-tests` → `/bmad-sprint-planning` → `/bmad-sprint-status` → `/bmad-retrospective`

**Agent roles:** `/bmad-agent-analyst` · `/bmad-agent-pm` · `/bmad-agent-ux-designer` · `/bmad-agent-architect` · `/bmad-agent-dev` · `/bmad-agent-tech-writer`

#### Taskmaster
`taskmaster:task-orchestrator` · `taskmaster:task-executor` · `taskmaster:task-checker`
Full task dependency graph, parallel execution, QA checking.

#### Ralph
`/ralph-skills:prd` · `/ralph-skills:ralph` — Executes PRDs into development tasks.

#### GitNexus — Knowledge graphs for code
`/gitnexus-guide` · `/gitnexus-exploring` · `/gitnexus-debugging` · `/gitnexus-refactoring` · `/gitnexus-pr-review` · `/gitnexus-impact-analysis` · `/gitnexus-cli`

#### Understand-Anything — Deep codebase comprehension
`/understand` · `/understand-onboard` · `/understand-domain` · `/understand-explain` · `/understand-dashboard` · `/understand-diff` · `/understand-knowledge` · `/understand-chat`

#### PPT-Master
`/ppt-master` — Generates structured slide decks from any input.

#### Last 30 Days
`/last30days` — Research any topic across Reddit, HN, X, YouTube, Polymarket from last 30 days. Works out of the box.

#### Web Access
`/web-access` — General web browsing and research.

#### Hindsight Memory
`/hindsight-memory:create-agent` — Persistent agent memory across sessions.

#### Planning with Files
`/planning-with-files` (+ ES, DE, AR, ZH, ZHT variants) — Persistent planning saved to files, multilingual.

---

### AI Research Plugins

| Category | Skills |
|---|---|
| AutoResearch | `/autoresearch` |
| Ideation | `/brainstorming-research-ideas` · `/creative-thinking-for-research` |
| Evaluation | `/evaluating-llms-harness` · `/evaluating-code-models` · `/nemo-evaluator-sdk` |
| RAG | `/chroma` · `/faiss` · `/pinecone` · `/qdrant-vector-search` · `/sentence-transformers` |
| Agents | `/autogpt-agents` · `/crewai-multi-agent` · `/langchain` · `/llamaindex` |
| Prompt Engineering | `/dspy` · `/guidance` · `/instructor` · `/outlines` |
| Observability | `/langsmith-observability` · `/phoenix-observability` |
| ARA | `/ara-research-manager` · `/ara-compiler` · `/ara-rigor-reviewer` |
| Data Processing | `/nemo-curator` · `/ray-data` |
| Multimodal | `/whisper` · `/clip` · `/blip-2-vision-language` · `/llava` · `/stable-diffusion-image-generation` · `/audiocraft-audio-generation` · `/segment-anything-model` |

---

## TM Chotu — Truemeds Metabase Plugin

Custom plugin for Truemeds Metabase / Redshift access.

| Skill | What It Does |
|---|---|
| `/tm-chotu-onboard` | Initial setup and authentication |
| `/tm-chotu-ask` | Natural language → SQL query |
| `/tm-chotu-overview` | Business overview and context |
| `/tm-chotu-tables-enums` | Table and enum reference |
| `/tm-chotu-data-sources` | Data source catalog |
| `/tm-chotu-definitions` | Metric and term definitions |
| `/tm-chotu-business-flows` | Business process flows |
| `/tm-chotu-modules` | Module breakdown |
| `/tm-chotu-joins` | Common join patterns |
| `/tm-chotu-customer` | Customer data queries |
| `/tm-chotu-functions` | SQL function reference |
| `/tm-chotu-projects` | Project-specific context |
| `/tm-chotu-mood-router` | Routes query to right context |
| `/tm-chotu-query-rigor` | Query validation rules |
| `/tm-chotu-freshness` | Data freshness checks |
| `/using-tm-chotu` | Usage guide |

MCP tools: `query` · `execute_query` · `construct_query` · `get_table` · `get_table_field_values` · `get_metric` · `get_metric_field_values` · `search`

---

## Developer Skills

**Code Review**
`/code-review` · `/review` · `/security-review` · `/review-pr`
`/python-review` · `/go-review` · `/rust-review` · `/kotlin-review` · `/flutter-review` · `/cpp-review` · `/fastapi-review`

**Build**
`/build-fix` · `/go-build` · `/rust-build` · `/kotlin-build` · `/flutter-build` · `/cpp-build` · `/gradle-build`

**Test**
`/test-coverage` · `/go-test` · `/rust-test` · `/kotlin-test` · `/flutter-test` · `/cpp-test`

**Refactor**
`/refactor-clean` · `/prune` · `/simplify`

**Docs**
`/update-docs` · `/update-codemaps`

**PRP Workflow**
`/prp-prd` → `/prp-plan` → `/prp-implement` → `/prp-commit` → `/prp-pr`

**Multi-Agent**
`/multi-plan` · `/multi-workflow` · `/multi-execute` · `/multi-frontend` · `/multi-backend`

---

## Memory & Session Management

| Skill | What It Does |
|---|---|
| `/save-session` | Saves current session state |
| `/resume-session` | Resumes a prior session |
| `/sessions` | Lists saved sessions |
| `/checkpoint` | Mid-session checkpoint |
| `/aside` | Side task without losing main thread |
| `/claude-mem:mem-search` | Semantic search across all memory |
| `/claude-mem:smart-explore` | Explore memory graph |
| `/claude-mem:make-plan` | Creates plan from memory context |
| `/claude-mem:learn-codebase` | Indexes a codebase into memory |
| `/claude-mem:babysit` | Monitors long-running task |
| `/claude-mem:timeline-report` | Timeline of past work |
| `/claude-mem:knowledge-agent` | Deep knowledge retrieval |
| `/claude-mem:pathfinder` | Finds connections between memory nodes |
| `/claude-mem:wowerpoint` | Generates decks from memory content |

---

## Loop & Automation

| Skill | What It Does |
|---|---|
| `/loop` | Runs a prompt on a recurring interval |
| `/loop-start` · `/loop-status` | Start and monitor loops |
| `/schedule` | Schedule recurring remote agents (cron) |
| `/pm2` | Process management for long-running tasks |

---

## Caveman Mode

Active by default. Drops filler, keeps all technical substance.

| Skill | What It Does |
|---|---|
| `/caveman` | Toggle caveman writing style |
| `/caveman-help` | Explain caveman mode |
| `/caveman-stats` | Usage stats |
| `/caveman-commit` | Commit with caveman-style message |
| `/caveman-review` | Code review in caveman style |
| `/caveman-compress` | Compress verbose output |
| `/cavecrew` | Delegate to subagents (context-efficient) |

Levels: `full` (default) · `lite` · `ultra` — switch with `/caveman lite|full|ultra`
Off: type `stop caveman` or `normal mode`

---

## Hooks & Config

| Skill | What It Does |
|---|---|
| `/hookify` | Sets up Claude hooks from transcript analysis |
| `/hookify-list` · `/hookify-configure` · `/hookify-help` | Hook management |
| `/update-config` | Modifies `settings.json` — permissions, env vars, automated behaviors |
| `/keybindings-help` | Customize keyboard shortcuts |
| `/instinct-export` · `/instinct-import` · `/instinct-status` | Export/import Claude instincts |

---

## Behaviour Rules (Non-Negotiable)

1. **Never proceed on ambiguous input** — one clear question before acting
2. **Never produce output without a plan** — present plan, wait for sign-off
3. **Never dump raw data** — always interpret through Truemeds business context
4. **Always read skill changelog** before running any skill
5. **Always use schema reference doc** for Redshift queries — never infer from memory
6. **Always prefix Redshift tables with `tmmumpsdb.`**
7. **One deliverable at a time** — sign-off before moving to next
8. **State ship/pause/kill verdict at top** — not buried in the output

---

## Model Selection

| Model | Use When |
|---|---|
| **Haiku 4.5** | Lightweight agents, worker agents in multi-agent systems |
| **Sonnet 4.6** | Main PM/data work, complex tasks, orchestration (default) |
| **Opus 4.7** | Complex architecture, deep reasoning, research |

---

## File Locations

```
~/pm-agent/                          ← PM Agent (GitHub-backed)
~/pm-agent/workflows/core/           ← Core PM workflows
~/pm-agent/workflows/supporting/     ← Context load, recall-and-route
~/pm-agent/archives/                 ← All generated docs (auto-versioned)
~/pm-agent/changelogs/               ← Self-learning changelogs
~/pm-agent/scripts/                  ← commit-and-push.sh, weekly_synthesis.py

~/data-agent/                        ← Data Agent
~/Documents/claude-os/data-agent/    ← Data skill files (schema-reader, etc.)
~/Documents/claude-os/CLAUDE.md      ← Master config + routing table
~/.claude/CLAUDE.md                  ← Symlink to above
~/.claude/settings.json              ← Permissions, hooks, env
~/.claude/keybindings.json           ← Key bindings
```

---

*Last updated: May 2026 · tejas.bhalerao@truemeds.in*

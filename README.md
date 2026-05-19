# Tejas's Claude Code — Master Reference

One-stop reference for everything Tejas's Claude Code can do.
Agents · Plugins · MCPs · Skills · Routing Rules.

---

## What This Is

Claude Code configured as a PM + Data + Ops assistant for **Tejas Bhalerao**, Product Manager at **Truemeds** — an online pharmacy operating across Hyperlocal Forward/Reverse, Courier Forward/Reverse, B2B Forward/Reverse.

Master config: `~/.claude/CLAUDE.md`

> ⚠️ **LEGACY — DO NOT USE:** `~/Documents/claude-os/` is an old location that is out of date. Nothing in that directory should be referenced or relied on. All live skill files are in `~/pm-agent/` and `~/data-agent/`.

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

## Installed Plugins — What Each Does & When to Use It

> **For Truemeds-specific PM work** (PRDs, experiments, metrics, Redshift queries) — always use the PM Agent and Data Agent above. Plugins below are for generic/non-Truemeds-contextualized tasks, cross-functional work, and specialized workflows.

---

### PM Skills Plugin

**Product Strategy**

| Skill | What it does | Use when |
|---|---|---|
| `/market-scan` | Scans competitive landscape, market trends, and size for a product area | Starting research on a new domain or vertical |
| `/product-strategy` `/strategy` | Builds a structured product strategy document with goals, bets, and tradeoffs | Setting direction for a product line or initiative |
| `/product-vision` | Writes a long-term product vision statement | Aligning stakeholders on 2-3 year direction |
| `/value-proposition` | Defines what makes your product uniquely valuable to a customer segment | Sharpening positioning or preparing for GTM |
| `/business-model` | Maps how the product creates, delivers, and captures value | Evaluating a new product or business idea |
| `/pricing` `/pricing-strategy` | Analyzes and recommends a pricing approach | Launching a new tier, changing pricing, or evaluating competitors' pricing |
| `/monetization-strategy` | Designs a revenue model (subscription, transaction fee, freemium, etc.) | Building or rethinking how a product makes money |
| `/lean-canvas` | One-page business model canvas | New product ideas, pivots, or investor-facing summaries |
| `/swot-analysis` | Strengths, Weaknesses, Opportunities, Threats framework | Quarterly reviews or strategic planning sessions |
| `/ansoff-matrix` | Maps growth options: market penetration, development, product development, diversification | Choosing where to invest next |
| `/porters-five-forces` | Industry competitive dynamics (suppliers, buyers, substitutes, entrants, rivalry) | Market entry decisions or competitive strategy |
| `/pestle-analysis` | Political, Economic, Social, Technological, Legal, Environmental analysis | Risk assessment for new markets or regulatory environments |
| `/startup-canvas` | Startup-focused variant of lean canvas | Early-stage product or new business unit ideation |

---

**Market Research**

| Skill | What it does | Use when |
|---|---|---|
| `/research-users` | Synthesizes user research transcripts, notes, or feedback into structured insights | After user interviews or usability studies |
| `/analyze-feedback` | Analyzes a batch of customer feedback (NPS comments, support tickets, app reviews) into themes | After collecting feedback at scale |
| `/competitive-analysis` `/competitor-analysis` | Deep dive on a specific competitor: positioning, pricing, strengths, weaknesses | Pre-launch or quarterly competitive review |
| `/sentiment-analysis` | Runs sentiment analysis on a body of text | Analyzing social media, reviews, or support tickets |
| `/market-segments` | Identifies and describes distinct customer segments in a market | Market sizing or targeting decisions |
| `/user-personas` | Builds user personas from research data or stated assumptions | Before a new feature or product design sprint |
| `/user-segmentation` | Segments existing users by behavior, value, or demographics | Personalisation or prioritization decisions |
| `/customer-journey-map` | Maps the end-to-end customer experience with touchpoints, pain points, and emotions | Identifying drop-off points or redesigning a flow |
| `/market-sizing` | TAM / SAM / SOM calculation for a market | Business cases, investor decks, or new market entry |

---

**Marketing & Growth**

| Skill | What it does | Use when |
|---|---|---|
| `/north-star` `/north-star-metric` | Defines the single metric that best captures product value delivery | Aligning a team around a shared success measure |
| `/positioning-ideas` | Generates positioning statement options for a product or feature | Pre-launch messaging or rebranding |
| `/product-name` | Brainstorms and evaluates product or feature names | Naming a new feature or product |
| `/marketing-ideas` | Brainstorms marketing campaign and channel ideas | Planning a launch or growth sprint |
| `/value-prop-statements` | Generates multiple value proposition statement variants | A/B testing messaging or updating landing pages |
| `/market-product` | Product marketing: messaging framework, launch materials, positioning | Preparing a feature for external communication |

---

**Go-To-Market**

| Skill | What it does | Use when |
|---|---|---|
| `/plan-launch` | Builds a structured product launch plan with timeline, owners, and channels | 4-6 weeks before any significant launch |
| `/gtm-strategy` `/growth-strategy` | Full go-to-market strategy document covering segment, channel, motion, and pricing | New product or market entry |
| `/gtm-motions` | Helps choose between PLG, sales-led, channel-led, or hybrid GTM approaches | Deciding how to acquire and grow users |
| `/battlecard` `/competitive-battlecard` | Head-to-head competitor comparison for sales or positioning conversations | Arming sales or customer success with competitive context |
| `/growth-loops` | Designs self-reinforcing growth loops (viral, content, paid, community) | Growth strategy or identifying compounding acquisition mechanisms |
| `/ideal-customer-profile` | Defines ICP with firmographic/demographic criteria and intent signals | Targeting or sales qualification |
| `/beachhead-segment` | Identifies the first, most winnable customer segment to focus on | Early-stage GTM or market entry |

---

**Execution**

| Skill | What it does | Use when |
|---|---|---|
| `/create-prd` `/write-prd` | Generic PRD template (no Truemeds context) | Non-Truemeds projects; for Truemeds PRDs use the PM Agent |
| `/meeting-notes` `/summarize-meeting` | Structures or summarizes meeting notes into decisions, actions, and owners | After any meeting |
| `/user-stories` `/write-stories` | Writes stories in "As a... I want... So that..." format | Backlog grooming or sprint planning |
| `/job-stories` | Writes stories in JTBD format: "When... I want to... So I can..." | Capturing user motivations more than actions |
| `/outcome-roadmap` `/transform-roadmap` | Converts a feature list into an outcome-based roadmap | Quarterly roadmap planning or OKR alignment |
| `/pre-mortem` | Imagines a future failure and identifies what caused it | Before any high-stakes launch or initiative |
| `/sprint-plan` `/sprint` | Plans a sprint: goal, stories, capacity, and risks | Sprint kick-off |
| `/plan-okrs` `/brainstorm-okrs` | Defines or brainstorms OKRs for a team, product, or quarter | Quarterly planning |
| `/stakeholder-map` | Maps stakeholders by influence, interest, stance, and engagement strategy | Before alignment meetings or for new initiatives |
| `/test-scenarios` | Writes test scenarios for a feature | Handing off to QA or self-testing |
| `/release-notes` | Writes user-facing release notes | Post-launch communication |
| `/prioritization-frameworks` | Applies RICE, MoSCoW, ICE, or other frameworks to a backlog | When a backlog needs a structured prioritization pass |
| `/wwas` | Weekly wins and setbacks summary | End-of-week reflection or team updates |
| `/retro` | Sprint retrospective: what went well, what didn't, what to change | End of sprint |
| `/dummy-dataset` `/generate-data` | Generates synthetic data for testing or demo purposes | When you need realistic-looking data without using prod data |

---

**Data Analytics**

| Skill | What it does | Use when |
|---|---|---|
| `/cohort-analysis` `/analyze-cohorts` | Analyzes user retention or behavior by cohort (signup date, acquisition channel, etc.) | Understanding retention curves or LTV |
| `/sql-queries` `/write-query` | Writes generic SQL queries | Non-Truemeds databases; for Truemeds use the Data Agent |
| `/ab-test-analysis` `/analyze-test` | Analyzes A/B test results: statistical significance, effect size, guardrail checks | After an experiment concludes |

---

**Product Discovery**

| Skill | What it does | Use when |
|---|---|---|
| `/analyze-feature-requests` `/triage-requests` | Categorizes and prioritizes incoming feature requests by theme and impact | After collecting a batch of requests from users or sales |
| `/discover` | Runs a structured product discovery process | Starting work on a new problem area |
| `/brainstorm` | Open-ended idea generation for a problem or opportunity | Discovery sprints or early ideation |
| `/interview-script` | Writes a user interview script targeting a specific question | Before conducting user interviews |
| `/interview` | Structures and runs a user interview in conversation | Live or simulated interview session |
| `/summarize-interview` | Extracts key insights from an interview transcript | After user interviews |
| `/metrics-dashboard` `/setup-metrics` | Designs a metrics dashboard or KPI framework for a feature or team | Setting up tracking for a new area |
| `/opportunity-solution-tree` | Maps a goal to opportunities to solutions using OST framework | Structuring discovery work |
| `/identify-assumptions-new` `/identify-assumptions-existing` | Surfaces untested assumptions in a new or existing product/feature | Risk assessment before building |
| `/prioritize-assumptions` | Prioritizes which assumptions to test first by risk and testability | Deciding what to validate next |
| `/brainstorm-ideas-new` `/brainstorm-ideas-existing` | Generates solution ideas for a problem | Ideation phase of discovery |
| `/brainstorm-experiments-new` `/brainstorm-experiments-existing` | Brainstorms experiments to test specific assumptions | Experiment design and planning |
| `/prioritize-features` | Prioritizes a feature backlog using a chosen framework | Before roadmap planning |

---

**Toolkit**

| Skill | What it does | Use when |
|---|---|---|
| `/tailor-resume` | Rewrites resume to match a specific job description | Job applications |
| `/review-resume` | Reviews and improves a resume for clarity and impact | Resume polish |
| `/proofread` `/grammar-check` | Proofreads any writing for grammar, clarity, and flow | Before sending or publishing anything |
| `/draft-nda` | Drafts a non-disclosure agreement | Vendor or partnership conversations |
| `/privacy-policy` | Drafts a privacy policy | New product or feature launch |

---

### Knowledge Work Plugins

These are enterprise cross-functional tools. Most useful for non-PM functions or when collaborating across teams.

**Product Management** — `/metrics-review` (health-check a metrics set), `/synthesize-research` (synthesize research from multiple sources), `/write-spec` (write a technical spec), `/competitive-brief` (one-page competitive summary), `/sprint-planning` (plan a sprint), `/product-brainstorming` (structured brainstorm), `/stakeholder-update` (draft a stakeholder update), `/roadmap-update` (update an existing roadmap)

**Data** — Generic data work not tied to Truemeds: `/data:analyze` (analyze any dataset), `/explore-data` (EDA on uploaded data), `/validate-data` (check data quality), `/statistical-analysis` (run stats), `/data-visualization` `/create-viz` (build charts), `/build-dashboard` (design a metrics dashboard), `/data:sql-queries` (write SQL), `/data-context-extractor` (extract context and definitions from a data source)

**Marketing** — `/draft-content` (write marketing copy), `/content-creation` (long-form content), `/marketing:competitive-brief` (competitive positioning brief), `/brand-review` (check content against brand guidelines), `/performance-report` (marketing channel performance summary), `/email-sequence` (drip email sequence), `/campaign-plan` (full campaign plan), `/seo-audit` (SEO audit with recommendations)

**Sales** — `/call-prep` (prep for a sales or customer call), `/pipeline-review` (review deal pipeline health), `/sales:competitive-intelligence` (competitive intel for a deal), `/call-summary` (summarize a sales call), `/account-research` (research a specific account), `/daily-briefing` (daily sales briefing), `/draft-outreach` (write outreach email or message), `/forecast` (build a sales forecast)

**Finance** — `/financial-statements` (analyze financial statements), `/variance-analysis` (actual vs budget variance), `/reconciliation` (reconcile accounts), `/journal-entry` `/journal-entry-prep` (prepare journal entries), `/close-management` (month/quarter close tasks), `/sox-testing` (SOX compliance testing), `/audit-support` (audit documentation)

**Operations** — `/process-optimization` (identify and fix process inefficiencies), `/process-doc` (document a process as an SOP), `/runbook` (create an operational runbook), `/risk-assessment` (identify and rate operational risks), `/capacity-plan` (plan team or system capacity), `/vendor-review` (evaluate a vendor), `/change-request` (structure a change request), `/compliance-tracking` (track compliance requirements), `/status-report` (generate a status report)

**Design** — `/design-system` (define or review a design system), `/accessibility-review` (audit for accessibility), `/design-critique` (structured design review), `/design-handoff` (prepare design handoff notes for engineering), `/ux-copy` (write interface copy), `/research-synthesis` (synthesize design research), `/user-research` (plan and run user research)

**Enterprise Search** — `/enterprise-search:search` (search across connected knowledge sources), `/digest` (daily/weekly knowledge digest), `/search-strategy` (build a knowledge search strategy), `/knowledge-synthesis` (synthesize information from multiple sources), `/source-management` (manage connected knowledge sources)

**Brand Voice** — `/brand-voice:generate-guidelines` (generate brand voice guidelines from examples), `/brand-voice-enforcement` (check any content against brand guidelines), `/brand-voice:discover-brand` (discover brand materials across connected platforms), `/guideline-generation` (full brand guidelines document)

**Productivity** — `/productivity:start` (start a structured work session), `/task-management` (manage tasks in conversation), `/memory-management` (manage Claude's memory), `/productivity:update` (update session state)

**PDF Viewer** — `/pdf-viewer:open` `/view-pdf` (open and read a PDF), `/fill-form` (fill a PDF form), `/sign` (sign a PDF), `/annotate` (annotate a PDF)

**Miro** — `/miro:miro-browse` (browse existing Miro boards), `/miro-diagram` (create a diagram on Miro), `/miro-table` (create a table on Miro), `/miro-doc` (create documentation on Miro), `/miro-code-review` (run a code review on Miro), `/miro-code-spec` (create a code spec on Miro)

**Figma** — `/figma:figma-use` (interact with Figma files), `/figma-generate-design` (generate a design), `/figma-generate-diagram` (generate a diagram), `/figma-create-new-file` (create a new Figma file), `/figma-use-slides` (work with Figma slides), `/figma-use-figjam` (work with FigJam boards), `/figma-code-connect` (connect Figma designs to code), `/figma-generate-library` (generate a design library)

---

### Specialized Plugins

#### BMAD Method
**Use when:** Building a new product or feature end-to-end with a structured software development methodology. BMAD is a full lifecycle framework — from product brief to retrospective. Best for greenfield projects or when you want rigorous process across PM, design, architecture, and dev.

**Lifecycle order:** `/bmad-product-brief` → `/bmad-domain-research` → `/bmad-market-research` → `/bmad-technical-research` → `/bmad-create-prd` → `/bmad-validate-prd` → `/bmad-create-ux-design` → `/bmad-create-architecture` → `/bmad-check-implementation-readiness` → `/bmad-create-epics-and-stories` → `/bmad-dev-story` → `/bmad-code-review` → `/bmad-qa-generate-e2e-tests` → `/bmad-sprint-planning` → `/bmad-sprint-status` → `/bmad-retrospective`

**Agent roles (invoke a persona):** `/bmad-agent-analyst` · `/bmad-agent-pm` · `/bmad-agent-ux-designer` · `/bmad-agent-architect` · `/bmad-agent-dev` · `/bmad-agent-tech-writer`

**Pro tools (meta-level):** `/bmad-help` (guide), `/bmad-brainstorming` (structured ideation), `/bmad-distillator` (compress large docs), `/bmad-advanced-elicitation` (deep requirements extraction), `/bmad-editorial-review-prose` (review writing quality), `/bmad-editorial-review-structure` (review document structure), `/bmad-review-adversarial-general` (adversarial review), `/bmad-review-edge-case-hunter` (find edge cases)

---

#### Taskmaster
**Use when:** Managing a complex multi-step project within Claude — tasks have dependencies, some can run in parallel, and you want tracking and QA. Think of it as a task board inside Claude.

- `taskmaster:task-orchestrator` — Analyzes the task queue, identifies dependencies and parallelism, coordinates execution
- `taskmaster:task-executor` — Implements a specific task
- `taskmaster:task-checker` — QA check: verifies a completed task actually meets its requirements

---

#### Ralph
**Use when:** You have a signed-off PRD and want Claude to turn it into development-ready tasks, tickets, and implementation steps.

- `/ralph-skills:prd` — Takes a PRD as input and breaks it into implementation tasks
- `/ralph-skills:ralph` — Full Ralph workflow: PRD → actionable engineering work

---

#### GitNexus — Codebase Knowledge Graphs
**Use when:** Working with an unfamiliar codebase, doing impact analysis before a change, or trying to understand where a bug lives. Builds a searchable knowledge graph of a repository.

| Skill | Use when |
|---|---|
| `/gitnexus-guide` | Learning how to use GitNexus |
| `/gitnexus-exploring` | Exploring an unfamiliar codebase |
| `/gitnexus-debugging` | Tracing a bug through the codebase |
| `/gitnexus-refactoring` | Understanding what a refactor will affect |
| `/gitnexus-pr-review` | Deep PR review with full codebase context |
| `/gitnexus-impact-analysis` | What does changing X break or affect? |
| `/gitnexus-cli` | CLI-based GitNexus operations |

---

#### Understand-Anything — Deep Codebase Comprehension
**Use when:** Onboarding to a new repo, explaining architecture to stakeholders, or doing a thorough codebase audit. Goes deeper than GitNexus — builds full architecture layers and domain flow maps.

| Skill | Use when |
|---|---|
| `/understand` | Start here — indexes a codebase |
| `/understand-onboard` | Onboarding walkthrough for a new repo |
| `/understand-domain` | Maps business domain logic in the code |
| `/understand-explain` | Explains a specific part of the codebase |
| `/understand-dashboard` | Visual dashboard of codebase structure |
| `/understand-diff` | Explains what a diff or PR changed |
| `/understand-knowledge` | Queries the knowledge graph |
| `/understand-chat` | Chat interface to ask questions about the codebase |

---

#### PPT-Master
**Use when:** You need a PowerPoint or slide deck built quickly from any input — notes, PRD, strategy doc, or verbal description.

`/ppt-master` — Generates a structured slide deck. Provide your content; it handles slide structure, headers, and layout.

---

#### Last 30 Days
**Use when:** You need real-time market intelligence, competitor news, trending topics, or public discourse on any subject from the last 30 days. Searches Reddit, HN, X, YouTube, and Polymarket.

`/last30days` — Works out of the box. Say: `/last30days [topic]`

---

#### Web Access
**Use when:** General web research — fetching a page, reading documentation, researching a topic, checking a URL.

`/web-access` — General-purpose web browsing and research.

---

#### Hindsight Memory
**Use when:** You want Claude to remember project context, decisions, and patterns across sessions — like a persistent agent brain for a long-running project.

`/hindsight-memory:create-agent` — Creates a memory agent that persists learning across sessions.

---

#### Planning with Files
**Use when:** You're managing a long-running project and want plans saved to files and maintained across Claude sessions (not just in conversation context). Supports English, Spanish, German, Arabic, Chinese, Traditional Chinese.

`/planning-with-files` — Persistent planning saved to files. Say: `/planning-with-files [describe your project]`

---

### AI Research Plugins

For ML engineers and AI researchers building or evaluating AI systems.

| Category | Skill | Use when |
|---|---|---|
| **AutoResearch** | `/autoresearch` | Automated research paper search, analysis, and synthesis |
| **Ideation** | `/brainstorming-research-ideas` | Brainstorming new ML/AI research directions |
| **Ideation** | `/creative-thinking-for-research` | Creative approaches to research problems |
| **Evaluation** | `/evaluating-llms-harness` | Evaluating LLMs using standard harness frameworks |
| **Evaluation** | `/evaluating-code-models` | Benchmarking code generation models |
| **Evaluation** | `/nemo-evaluator-sdk` | Evaluation via NVIDIA NeMo SDK |
| **RAG** | `/chroma` | Setting up ChromaDB for vector search |
| **RAG** | `/faiss` | Setting up FAISS for similarity search |
| **RAG** | `/pinecone` | Setting up Pinecone managed vector database |
| **RAG** | `/qdrant-vector-search` | Setting up Qdrant vector search |
| **RAG** | `/sentence-transformers` | Embedding text with sentence transformers |
| **Agents** | `/autogpt-agents` | Building AutoGPT-style autonomous agents |
| **Agents** | `/crewai-multi-agent` | Multi-agent systems with CrewAI |
| **Agents** | `/langchain` | Building LLM apps with LangChain |
| **Agents** | `/llamaindex` | Building RAG and data apps with LlamaIndex |
| **Prompt Engineering** | `/dspy` | Structured prompting and optimization with DSPy |
| **Prompt Engineering** | `/guidance` | Constrained generation with Guidance |
| **Prompt Engineering** | `/instructor` | Structured outputs from LLMs with Instructor |
| **Prompt Engineering** | `/outlines` | Guaranteed structured generation with Outlines |
| **Observability** | `/langsmith-observability` | Tracing and monitoring LLM apps with LangSmith |
| **Observability** | `/phoenix-observability` | LLM observability with Arize Phoenix |
| **ARA** | `/ara-research-manager` | Managing agent-native research artifacts |
| **ARA** | `/ara-compiler` | Compiling research into structured artifacts |
| **ARA** | `/ara-rigor-reviewer` | Reviewing research artifacts for rigor |
| **Data Processing** | `/nemo-curator` | Curating ML training data with NeMo |
| **Data Processing** | `/ray-data` | Distributed data processing with Ray |
| **Multimodal** | `/whisper` | Speech-to-text transcription |
| **Multimodal** | `/clip` | Image-text embedding and similarity |
| **Multimodal** | `/blip-2-vision-language` | Vision-language model tasks |
| **Multimodal** | `/llava` | Visual question answering |
| **Multimodal** | `/stable-diffusion-image-generation` | Text-to-image generation |
| **Multimodal** | `/audiocraft-audio-generation` | Text-to-audio and music generation |
| **Multimodal** | `/segment-anything-model` | Image segmentation |

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
~/pm-agent/                          ← PM Agent (GitHub-backed, source of truth)
~/pm-agent/workflows/core/           ← Core PM workflows
~/pm-agent/workflows/supporting/     ← Context load, recall-and-route
~/pm-agent/archives/                 ← All generated docs (auto-versioned)
~/pm-agent/changelogs/               ← Self-learning changelogs
~/pm-agent/scripts/                  ← commit-and-push.sh, weekly_synthesis.py

~/data-agent/                        ← Data Agent (source of truth)

~/.claude/CLAUDE.md                  ← Master config + routing table
~/.claude/settings.json              ← Permissions, hooks, env
~/.claude/keybindings.json           ← Key bindings

⚠️  ~/Documents/claude-os/           ← LEGACY. Out of date. Do not use.
```

---

*Last updated: May 2026 · tejas.bhalerao@truemeds.in*

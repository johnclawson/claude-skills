---
name: security-qbr
description: Generate Quarterly Business Review (QBR) presentations for Information Security teams. Use this skill when creating QBR decks, quarterly security reviews, or executive briefings about security posture for CIO/CTO audiences. Covers content structure (risks, accomplishments, current work, KPIs), writing style, and slide organization. Triggers on requests for security QBRs, quarterly InfoSec reviews, CISO presentations, or security executive briefings.
---

# Security QBR Generator

Draft executive-level Quarterly Business Reviews for Information Security teams.

**Presenter:** VP of Information Security
**Audience:** CIO and CTO

---

## Agenda Structure (5 Sections)

| # | Section | Description | Data Source |
|---|---------|-------------|-------------|
| 1 | Risk Review | Top security risks facing the organization | `sources/` CSV |
| 2 | Roadmap | Past quarter accountability + current quarter preview | ClickUp + snapshots |
| 3 | Highlights | Key accomplishments from last quarter | ClickUp closed tasks |
| 4 | Notable Incidents | Security incidents worth executive attention | **TODO** - TBD |
| 5 | Metrics | KPIs and security posture measurements | **TODO** - TBD |

---

## Orchestration Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         MAIN ORCHESTRATOR                               │
│  1. Check sources/ and data/ directories                                │
│  2. Prompt user for quarter and any missing materials                   │
│  3. Write config.json with quarter info                                 │
│  4. Launch 5 section agents IN PARALLEL                                 │
│  5. Wait for all agents to complete                                     │
│  6. Launch Slide Builder Agent with all JSON outputs                    │
│  7. Report final output location                                        │
└─────────────────────────────────────────────────────────────────────────┘
                    │
    ┌───────────────┼───────────────┬───────────────┬───────────────┐
    ▼               ▼               ▼               ▼               ▼
┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐
│  Risk   │   │ Roadmap │   │Highlights│   │Incidents│   │ Metrics │
│  Agent  │   │  Agent  │   │  Agent  │   │  Agent  │   │  Agent  │
└─────────┘   └─────────┘   └─────────┘   └─────────┘   └─────────┘
    │               │               │               │               │
    ▼               ▼               ▼               ▼               ▼
risks.json    roadmap.json   highlights.json  incidents.json  metrics.json
                    │
                    ▼
         ┌─────────────────────┐
         │  Slide Builder      │
         │  Agent              │
         │  - Creates HTML     │
         │  - Runs build.js    │
         │  - Outputs PPTX     │
         └─────────────────────┘
```

---

## Getting Started Workflow

When the user requests a QBR:

### Step 1: Check the sources and data directories

Scan for available materials:

```
sources/
├── Top Risks for Quarterly Review*.csv    # Risk register export (REQUIRED)
├── risk-heat-map.*                        # Optional: Risk matrix image
└── [other materials]

data/roadmap-snapshots/
├── {PREV_QUARTER}-snapshot.json           # For accountability (e.g., FY25-Q4-snapshot.json)
└── {CURR_QUARTER}-snapshot.json           # For preview (e.g., FY26-Q1-snapshot.json)
```

### Step 2: Prompt user for configuration

```
To build your QBR deck, I need to confirm some details:

**Found in sources/:**
- [x] Risk register CSV
- [ ] Risk heat map image

**Found in data/roadmap-snapshots/:**
- [x] FY25 Q4 snapshot (for accountability)
- [ ] FY26 Q1 snapshot (for preview)

**Please confirm:**
1. Which quarter is this QBR for? (e.g., Q1 FY26)
2. Previous quarter being reviewed? (e.g., Q4 FY25)
```

### Step 3: Write config.json

Save configuration to `{scratchpad}/qbr-data/config.json`:

```json
{
  "qbr_quarter": "FY26 Q1",
  "review_quarter": "FY25 Q4",
  "presentation_date": "2026-01-28",
  "presenter": "VP of Information Security",
  "audience": "CIO, CTO"
}
```

### Step 4: Launch 5 section agents IN PARALLEL

Use the Task tool to launch all 5 data gathering agents simultaneously:

```javascript
// Launch ALL 5 agents in parallel (single message with 5 Task tool calls)
Task("Risk Review Agent", "general-purpose", riskAgentPrompt)
Task("Roadmap Agent", "general-purpose", roadmapAgentPrompt)
Task("Highlights Agent", "general-purpose", highlightsAgentPrompt)
Task("Incidents Agent", "general-purpose", incidentsAgentPrompt)
Task("Metrics Agent", "general-purpose", metricsAgentPrompt)
```

### Step 5: Launch Slide Builder Agent

After all data agents complete, launch the slide builder:

```javascript
Task("Slide Builder Agent", "general-purpose", slideBuilderPrompt)
```

### Step 6: Report output location

```
QBR presentation generated successfully!

Output files:
- output/html-slides/*.html (14 slides)

To present: Open output/html-slides/slide01-title.html in a browser.
Navigate with arrow keys or use the prev/next buttons.
```

---

## Section Agent Specifications

### 1. Risk Review Agent

**Input:** `sources/Top Risks for Quarterly Review*.csv`
**Output:** `{scratchpad}/qbr-data/risks.json`

**Agent Prompt:**
```
Parse the risk register CSV at sources/Top Risks for Quarterly Review*.csv

Tasks:
1. Read the CSV file
2. Filter out: closed risks (Status = "Closed"), incomplete entries (missing Residual score)
3. Sort by "Residual score" descending
4. Select top 3-4 risks (include 4th if 4+ are High-rated, score 12+)

For each risk, extract:
- name, internal_id, description, residual_risk, residual_score
- residual_likelihood, residual_impact, treatment_details

Output as JSON to: {scratchpad}/qbr-data/risks.json

Schema:
{
  "quarter": "FY26 Q1",
  "generated_at": "2026-01-28T10:00:00Z",
  "risks": [
    {
      "name": "Risk title",
      "id": "R0036",
      "likelihood": "Possible",
      "impact": "High",
      "score": 12,
      "description": "One-line summary of the risk",
      "mitigation": "Key treatment actions (1-2 sentences)"
    }
  ]
}
```

**Slides Generated:**
- `slide03-section1.html` - Section divider: "01. Risk Review"
- `slide04-risks.html` - Top Risks content slide

---

### 2. Roadmap Agent

**Input:**
- `data/roadmap-snapshots/{PREV_QUARTER}-snapshot.json` (for accountability)
- `data/roadmap-snapshots/{CURR_QUARTER}-snapshot.json` (for current preview)
- ClickUp API (for current task status)

**Output:** `{scratchpad}/qbr-data/roadmap.json`

**Agent Prompt:**
```
Generate roadmap data for QBR.

Read config from: {scratchpad}/qbr-data/config.json
- review_quarter: The quarter being reviewed (past)
- qbr_quarter: The upcoming quarter

PART 1 - Past Quarter Accountability:
1. Load snapshot: data/roadmap-snapshots/{PREV_QUARTER}-snapshot.json
2. For each task in snapshot, fetch current status via clickup_get_task
3. Categorize tasks:
   - Completed: In snapshot AND now status = done/closed/completed
   - Pushed: In snapshot AND still open
4. Calculate completion rate: Completed / Total in Snapshot × 100
5. Fetch tasks currently tagged for quarter but NOT in snapshot (Added tasks)

PART 2 - Current Quarter Preview:
1. Load snapshot: data/roadmap-snapshots/{CURR_QUARTER}-snapshot.json
   OR fetch from ClickUp if no snapshot exists
2. Group tasks by list (Security Engineering, GRC, Penetration Testing)
3. Include task name, status, assignees, due dates

Output as JSON to: {scratchpad}/qbr-data/roadmap.json

Schema:
{
  "quarter": "FY26 Q1",
  "generated_at": "2026-01-28T10:00:00Z",
  "past_quarter": {
    "quarter": "FY25 Q4",
    "has_snapshot": true,
    "completion_rate": 60,
    "total_planned": 5,
    "completed_count": 3,
    "tasks": [
      {
        "name": "Task name",
        "original_status": "Open",
        "current_status": "Closed",
        "result": "completed"  // or "pushed"
      }
    ],
    "added_tasks": [
      {
        "name": "Task added during quarter",
        "status": "Completed"
      }
    ]
  },
  "current_quarter": {
    "quarter": "FY26 Q1",
    "has_snapshot": true,
    "tasks_by_list": {
      "Security Engineering": [
        {"name": "Task", "status": "in progress", "assignees": ["Name"], "due_date": "2026-01-30"}
      ],
      "GRC": [],
      "Penetration Testing": []
    },
    "total_tasks": 10
  }
}
```

**Slides Generated:**
- `slide05-section2.html` - Section divider: "02. Roadmap"
- `slide06-roadmap-review.html` - Past quarter accountability
- `slide07-roadmap-preview.html` - Current quarter preview

---

### 3. Highlights Agent

**Input:** ClickUp API - closed tasks from previous quarter
**Output:** `{scratchpad}/qbr-data/highlights.json`

**Agent Prompt:**
```
Extract engineering highlights from ClickUp for the QBR.

Read config from: {scratchpad}/qbr-data/config.json
- review_quarter: The quarter to pull accomplishments from

Tasks:
1. Search ClickUp for completed tasks in the review quarter
   - Use clickup_search with filters:
     - task_statuses: ["done", "closed", "completed"]
     - location.projects: ["30075124"] (Infosec space)
   - Date range for the quarter
2. Filter to only: Security Engineering, Penetration Testing, GRC lists
3. For each task, fetch details with clickup_get_task
4. Categorize by theme:
   - Identity & Access Management (user access reviews, SSO, IAM)
   - Monitoring & Detection (SIEM, logging, alerting, detection rules)
   - GRC & Compliance (audits, assessments, policies, compliance programs)
   - Penetration Testing (pen tests, vuln assessments, remediation)
   - Security Operations (tools, processes, automation)
   - Infrastructure Security (cloud security, network security, hardening)
5. Select top 2-3 items per category (most impactful)

Output as JSON to: {scratchpad}/qbr-data/highlights.json

Schema:
{
  "quarter": "FY25 Q4",
  "generated_at": "2026-01-28T10:00:00Z",
  "categories": {
    "Identity & Access Management": [
      {"name": "Completed Quarterly User Access Reviews", "impact": "6 engineering teams audited"}
    ],
    "Monitoring & Detection": [
      {"name": "Integrated Workday logs into InsightIDR", "impact": "HR event visibility"}
    ],
    "GRC & Compliance": [],
    "Penetration Testing": [],
    "Security Operations": [],
    "Infrastructure Security": []
  },
  "total_highlights": 12
}
```

**Slides Generated:**
- `slide08-section3.html` - Section divider: "03. Highlights"
- `slide09-highlights.html` - Categorized accomplishments

---

### 4. Incidents Agent

**Status:** Ready

**Input:** ClickUp Document - Incident History
- Document ID: `26x8e-33627` (Information Security Home)
- Page ID: `26x8e-31105` (Incident History)
- Sub-pages contain individual incidents with date and title

**Output:** `{scratchpad}/qbr-data/incidents.json`

**Agent Prompt:**
```
Extract incident data from ClickUp for the QBR.

Read config from: {scratchpad}/qbr-data/config.json
- review_quarter: The quarter to pull incidents from (e.g., "FY25 Q4")

STEP 1: List incident pages
Use clickup_list_document_pages with document_id: "26x8e-33627"
Find the "Incident History" page (id: "26x8e-31105") and its sub-pages.

STEP 2: Filter incidents by quarter
Each sub-page name starts with a date (YYYY-MM-DD format).
Filter to incidents within the quarter date range:
- FY25 Q4: 2025-10-01 to 2025-12-31
- FY26 Q1: 2026-01-01 to 2026-03-31
- etc.

STEP 3: Fetch incident details
For each matching incident, use clickup_get_document_pages to fetch content.
Extract from the markdown table format:
- title: From page name (after date)
- date: From page name (YYYY-MM-DD prefix)
- classification: From "Classification:" row (Critical, High, Medium, Low, Protected)
- rationale: From "Rationale:" row
- found_by: From "Found By:" row
- location: From "Location:" row (if present)
- assessment: From "Incident Assessment" section
- resolution: From "Incident Response Plan" section

STEP 4: Map classification to severity
| Classification | Severity |
|----------------|----------|
| Critical | Critical |
| High | High |
| Protected | Medium |
| Medium | Medium |
| Low | Low |

Output as JSON to: {scratchpad}/qbr-data/incidents.json

Schema:
{
  "quarter": "FY25 Q4",
  "generated_at": "2026-01-28T10:00:00Z",
  "status": "configured",
  "source": {
    "type": "clickup_document",
    "document_id": "26x8e-33627",
    "page_id": "26x8e-31105"
  },
  "incidents": [
    {
      "id": "26x8e-167997",
      "title": "Employee machine compromised",
      "date": "2025-11-14",
      "severity": "Medium",
      "classification": "Protected",
      "rationale": "Brief description of what happened",
      "resolution": "Actions taken to resolve",
      "found_by": "Person who discovered it"
    }
  ],
  "summary": {
    "total_count": 2,
    "by_severity": {"Critical": 1, "Medium": 1}
  }
}
```

**ClickUp Document Structure:**

```
Information Security Home (26x8e-33627)
└── Incident Response / Incident History (26x8e-81447)
    ├── Security Incident Response Plan
    ├── Initiating a Security Incident
    ├── Security Incident Rating Classification
    ├── Incident Response Playbooks/
    ├── Security Incident Response Report Template
    └── Incident History (26x8e-31105)  ← Target page
        ├── 2022-02-25 insights IAM user key leak
        ├── 2022-06-27 muffin_man repo secrets leak
        ├── ...
        ├── 2025-11-14 Employee machine compromised
        └── 2025-12-13 Critical RCE in Next.js
```

**Incident Page Format:**

Each incident page contains markdown tables with:
- **Header table:** Incident name, Classification, Rationale, Found By, Date/Time, Location
- **Assessment table:** Description of investigation results
- **Incident Team table:** Roles and contacts
- **Response Plan table:** Actions taken
- **Additional Information:** Screenshots, evidence

**Classification Levels:**
| Level | Description |
|-------|-------------|
| Critical | Severe impact, immediate action required |
| High | Significant impact |
| Protected | Medium sensitivity, internal handling |
| Medium | Moderate impact |
| Low | Minor impact |

**Slides Generated:**
- `slide10-section4.html` - Section divider: "04. Notable Incidents"
- `slide11-incidents.html` - Incidents summary with severity breakdown

**Slide Content Format:**

```
Slide Title: "Notable Incidents - FY25 Q4"

Summary: 2 incidents (1 Critical, 1 Medium)

1. Critical RCE in Next.js (Dec 13)
   - CVE-2025-55182 exploited on blade.pattern.com
   - Resolution: Upgraded Next.js to patched version 15.5.9

2. Employee Machine Compromised (Nov 14)
   - Malicious LinkedIn link led to malware infection
   - Resolution: Workstation wiped, SentinelOne policy updated
```

---

### 5. Metrics Agent (TODO)

**Status:** Not yet implemented - data sources TBD

**Input:** TBD - possible sources:
- Wiz security scores
- InsightIDR metrics
- KnowBe4 training completion
- Manual KPI spreadsheet

**Output:** `{scratchpad}/qbr-data/metrics.json`

**Agent Prompt (Placeholder):**
```
Generate metrics data for QBR.

TODO: This agent is not yet implemented.

For now, create a placeholder JSON indicating no metrics sources are configured.

Output as JSON to: {scratchpad}/qbr-data/metrics.json

Schema:
{
  "quarter": "FY25 Q4",
  "generated_at": "2026-01-28T10:00:00Z",
  "status": "not_configured",
  "message": "KPI data sources not yet configured",
  "metrics": []
}
```

**Future Schema (when implemented):**
```json
{
  "quarter": "FY25 Q4",
  "generated_at": "2026-01-28T10:00:00Z",
  "status": "configured",
  "metrics": [
    {
      "name": "Mean Time to Detect",
      "category": "Detection",
      "target": "<24h",
      "actual": "18h",
      "status": "on_track",
      "trend": "improving",
      "notes": "Down from 22h last quarter"
    },
    {
      "name": "Patch Compliance",
      "category": "Vulnerability Management",
      "target": "95%",
      "actual": "92%",
      "status": "at_risk",
      "trend": "stable"
    }
  ]
}
```

**Open Questions:**
- Which KPIs matter most? (MTTD, MTTR, patch compliance, training completion?)
- Where is this data? (Wiz, InsightIDR, KnowBe4, manual?)
- Historical comparison? (vs last quarter, vs target?)

**Slides Generated:**
- `slide12-section5.html` - Section divider: "05. Metrics"
- `slide13-metrics.html` - KPI dashboard (or placeholder message)

---

### 6. Slide Builder Agent

**Input:** All JSON files from section agents in `{scratchpad}/qbr-data/`
**Output:** HTML slides in `output/html-slides/`

**Agent Prompt:**
```
Build QBR slides as 1080p HTML files with auto-scaling.

Read all data from {scratchpad}/qbr-data/:
- config.json (quarter, date, presenter info)
- risks.json
- roadmap.json
- highlights.json
- incidents.json
- metrics.json

Tasks:
1. Create 14 HTML slides following Pattern brand guidelines
2. Use 1920×1080 (1080p) slide dimensions
3. Include auto-scaling JavaScript in each slide
4. Add prev/next navigation links to each slide
5. Handle placeholder sections gracefully (Metrics if not configured)
6. Write slides directly to output/html-slides/

Slide sequence (14 slides):
- slide01-title.html (no prev link)
- slide02-agenda.html
- slide03-section1.html (Risk Review divider)
- slide04-risks.html
- slide05-section2.html (Roadmap divider)
- slide06-roadmap-review.html
- slide07-roadmap-preview.html
- slide08-section3.html (Highlights divider)
- slide09-highlights.html
- slide10-section4.html (Notable Incidents divider)
- slide11-incidents.html
- slide12-section5.html (Metrics divider)
- slide13-metrics.html
- slide14-closing.html (no next link)

Each slide must include:
- 1920×1080 fixed dimensions
- Auto-scale JavaScript (see base template)
- Navigation: prev/next links with slide counter (e.g., "4 / 14")
- Keyboard navigation support

Output:
- output/html-slides/*.html (14 presentation-ready slides)

See "HTML Slide Generation" section for base template and CSS patterns.
```

---

## Slide Deck Structure (14 Slides)

```
slide01-title.html           # Title: "Information Security QBR - Q1 FY26"
slide02-agenda.html          # 5-item agenda

slide03-section1.html        # "01. Risk Review"
slide04-risks.html           # Top 3-4 risks

slide05-section2.html        # "02. Roadmap"
slide06-roadmap-review.html  # Past quarter accountability
slide07-roadmap-preview.html # Current quarter preview

slide08-section3.html        # "03. Highlights"
slide09-highlights.html      # Categorized accomplishments

slide10-section4.html        # "04. Notable Incidents"
slide11-incidents.html       # Incidents summary (or placeholder)

slide12-section5.html        # "05. Metrics"
slide13-metrics.html         # KPI dashboard (or placeholder)

slide14-closing.html         # "Questions & Discussion"
```

---

## Writing Style

| Aspect | Guideline |
|--------|-----------|
| Reading level | 10th grade |
| Density | Minimal text—bullets, visuals, concise statements |
| Tone | Professional, confident, jargon-light |
| Structure | Story-driven, not data-dump |

---

## Risk Register Integration

Pull top risks from a CSV export of the risk register placed in the `sources/` directory.

**File location:** `sources/Top Risks for Quarterly Review*.csv`

### Processing the Risk Register

1. **Read the CSV file** from the sources directory
2. **Filter out** closed risks (Status = "Closed") and incomplete entries (missing residual scores)
3. **Sort by Residual Score** (descending) - this is the numeric risk score after controls
4. **Select top 3-4 risks:**
   - Always include at least the top 3
   - Include a 4th if there are 4 or more High-rated residual risks (score 12+)

### Key CSV Columns

| Column | Use |
|--------|-----|
| Name | Risk title for slide |
| Internal ID | Reference ID (e.g., R0036) |
| Description | Context for mitigation summary |
| Residual risk | Categorical rating (High/Medium/Low) |
| Residual score | Numeric score for sorting (higher = more risk) |
| Residual risk likelihood | For slide detail |
| Residual risk impact | For slide detail |
| Treatment details | Summarize for mitigation actions |
| Estimated Financial Impact | Include if significant ($500k+) |

### Risk Score Reference

| Residual Score | Rating |
|----------------|--------|
| 12+ | High |
| 6-9 | Medium |
| 1-4 | Low |

---

## Roadmap Snapshots

Snapshots capture the planned tasks for a quarter at a point in time.

**Snapshot Directory:** `data/roadmap-snapshots/`

**File naming:**
- "FY25 Q4" → `FY25-Q4-snapshot.json`
- "FY26 Q1" → `FY26-Q1-snapshot.json`

### Snapshot JSON Schema

```json
{
  "quarter": "FY25 Q4",
  "quarter_value": 4,
  "snapshot_date": "2025-10-01T00:00:00Z",
  "tasks": [
    {
      "id": "86dxqxa9r",
      "name": "2025 External Pen Test",
      "list": "Security Engineering",
      "list_id": "132251242",
      "status": "Open",
      "assignees": ["John Clawson"],
      "due_date": "2025-01-16"
    }
  ],
  "total_count": 5
}
```

### Quarter Timing Reference

**Fiscal Year = Calendar Year**

| Quarter | Date Range | Previous Quarter |
|---------|------------|------------------|
| FY26 Q1 | Jan 1 - Mar 31, 2026 | FY25 Q4 |
| FY26 Q2 | Apr 1 - Jun 30, 2026 | FY26 Q1 |
| FY26 Q3 | Jul 1 - Sep 30, 2026 | FY26 Q2 |
| FY26 Q4 | Oct 1 - Dec 31, 2026 | FY26 Q3 |

---

## ClickUp Integration

**Space:** Infosec (ID: `30075124`)

**Target Lists:**
| List | ID |
|------|-----|
| Security Engineering | `132251242` |
| Penetration Testing | `900202119210` |
| GRC | `901704270170` |

**Quarter Custom Field ID:** `df8ff1a4-7dd1-4129-b3f6-cf0b745a21c9`

**Quarter Values:**
| Quarter | Value |
|---------|-------|
| FY26 Q1 | 0 |
| FY26 Q2 | 1 |
| FY26 Q3 | 2 |
| FY26 Q4 | 3 |
| FY25 Q4 | 4 |
| FY25 Q3 | 5 |
| FY27 Q1 | 6 |

---

## HTML Slide Generation (1080p with Auto-Scale)

### Slide Dimensions
- **Width:** 1920px (Full HD)
- **Height:** 1080px — 16:9 aspect ratio
- **Auto-scale:** JavaScript scales slides to fit browser viewport

### Base HTML Template

Each slide is a standalone HTML file with:
- 1920×1080 fixed slide dimensions
- Auto-scaling JavaScript to fit any viewport
- Prev/Next navigation links
- Keyboard navigation (arrow keys, space)

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>Slide Title</title>
  <style>
    @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap');
    * { margin: 0; padding: 0; box-sizing: border-box; }
    html, body { width: 100%; height: 100%; overflow: hidden; background: #1a1a1a; }

    .slide-wrapper {
      width: 100%;
      height: 100%;
      display: flex;
      align-items: center;
      justify-content: center;
    }

    .slide {
      width: 1920px;
      height: 1080px;
      position: relative;
      background-color: #231F20;  /* or #FFFFFF for light slides */
      font-family: 'Inter', -apple-system, sans-serif;
      transform-origin: center center;
    }

    /* Navigation */
    .nav {
      position: fixed;
      bottom: 20px;
      left: 50%;
      transform: translateX(-50%);
      display: flex;
      gap: 20px;
      z-index: 1000;
    }
    .nav a {
      background: #333;
      color: #fff;
      padding: 10px 24px;
      border-radius: 6px;
      text-decoration: none;
      font-family: -apple-system, sans-serif;
      font-size: 14px;
    }
    .nav a:hover { background: #555; }
    .nav a.disabled { background: #222; color: #444; pointer-events: none; }
    .nav span { color: #666; font-size: 14px; padding: 10px 0; }
  </style>
</head>
<body>
  <div class="slide-wrapper">
    <div class="slide" id="slide">
      <!-- Slide content here -->
    </div>
  </div>

  <nav class="nav">
    <a href="prev-slide.html">← Prev</a>
    <span>X / 14</span>
    <a href="next-slide.html">Next →</a>
  </nav>

  <script>
    function scaleSlide() {
      const slide = document.getElementById('slide');
      const scaleX = window.innerWidth / 1920;
      const scaleY = window.innerHeight / 1080;
      const scale = Math.min(scaleX, scaleY, 1);
      slide.style.transform = `scale(${scale})`;
    }
    window.addEventListener('resize', scaleSlide);
    window.addEventListener('load', scaleSlide);
    scaleSlide();

    document.addEventListener('keydown', (e) => {
      if (e.key === 'ArrowRight' || e.key === ' ') {
        window.location.href = 'next-slide.html';
      } else if (e.key === 'ArrowLeft') {
        window.location.href = 'prev-slide.html';
      }
    });
  </script>
</body>
</html>
```

### Font Size Guide (1080p)

**Sizes scaled for 1920×1080 slides:**

| Element | Size (px) | Example Use |
|---------|-----------|-------------|
| Main title | 96-107px | Title slide heading |
| Section number | 128px | "01" on section dividers |
| Section title | 96px | "Risk Review" on dividers |
| Slide header | 59-75px | Content slide titles |
| Subheader | 37-48px | Subtitles, categories |
| Body text | 29-37px | Descriptions, details |
| List items | 24-29px | Bullet points |
| Small text | 21-24px | Captions, metadata |

**Dense content slides** (like roadmaps with 20+ items):
- Use 24px for list items
- Use two-column layout to fit more content
- Abbreviate status labels (e.g., "IP" instead of "IN PROGRESS")

### Pattern Brand Colors

| Name | Hex | Usage |
|------|-----|-------|
| Pattern Blue | `#009BFF` | Accent numbers, lines, subtitles |
| Dark Background | `#231F20` | Section dividers, title slide |
| Off-White | `#F2F2F2` | Titles on dark backgrounds |
| Light Gray | `#F0F0F0` | Section titles on dark |
| Warm Gray | `#E0DBD7` | Metadata, dates |
| White Background | `#FFFFFF` | Content slides |
| Body Text | `#231F20` or `#444444` | Text on light backgrounds |

### Slide Type CSS Patterns

All slides use the base template above. Here are the key CSS patterns for different slide types:

#### Dark Slide (Title, Section Dividers, Closing)
```css
.slide { background-color: #231F20; }
.blue-bar { position: absolute; top: 0; left: 0; width: 21px; height: 100%; background-color: #009BFF; }
.title { color: #F2F2F2; }
.subtitle { color: #009BFF; }
.section-number { color: #009BFF; font-size: 128px; font-weight: 700; }
.section-title { color: #F2F2F2; font-size: 96px; font-weight: 700; }
```

#### Light Slide (Content Slides)
```css
.slide { background-color: #FFFFFF; }
.blue-bar { position: absolute; top: 0; left: 0; width: 21px; height: 100%; background-color: #009BFF; }
.header { color: #231F20; font-size: 59px; font-weight: 700; }
.body-text { color: #444444; font-size: 29px; }
```

#### Severity Badges (Incidents)
```css
.severity { padding: 8px 27px; border-radius: 11px; font-weight: 600; font-size: 24px; }
.severity.critical { background: #DC3545; color: white; }
.severity.high { background: #FD7E14; color: white; }
.severity.medium { background: #FFC107; color: #231F20; }
.severity.low { background: #28A745; color: white; }
```

#### Status Labels (Roadmap)
```css
.done { color: #28A745; font-weight: 700; }
.progress { color: #009BFF; font-weight: 700; }
.next { color: #6F42C1; font-weight: 700; }
.open { color: #6C757D; font-weight: 700; }
.pending { color: #FD7E14; font-weight: 700; }
```

### Slide Builder Notes

The Slide Builder Agent creates individual HTML slides directly in `output/html-slides/`.

Each slide file is self-contained with:
- 1920×1080 slide content
- Auto-scaling JavaScript
- Prev/Next navigation links
- Keyboard navigation support

**No build script or combined viewer is needed** - the HTML slides are ready to present directly in any browser.

To start the presentation, open `output/html-slides/slide01-title.html` in a browser.

---

## Output

```
output/
├── {Quarter}-Security-QBR.pptx     # Final PowerPoint presentation (optional)
└── html-slides/                     # Individual HTML slides (primary output)
    ├── slide01-title.html
    ├── slide02-agenda.html
    ├── slide03-section1.html
    ├── slide04-risks.html
    ├── slide05-section2.html
    ├── slide06-roadmap-review.html
    ├── slide07-roadmap-preview.html
    ├── slide08-section3.html
    ├── slide09-highlights.html
    ├── slide10-section4.html
    ├── slide11-incidents.html
    ├── slide12-section5.html
    ├── slide13-metrics.html
    └── slide14-closing.html
```

### HTML Slide Features

Each HTML slide is a standalone presentation-ready file:
- **1080p resolution**: 1920×1080 native size
- **Auto-scaling**: Automatically scales to fit any browser viewport
- **Navigation**: Prev/Next links at bottom of each slide
- **Keyboard shortcuts**: `←` `→` arrows, `Space` for next
- **Dark background**: Slides are centered on dark (#1a1a1a) background

**To present**: Open `slide01-title.html` in any browser and navigate with arrow keys or click the nav buttons.

---

## Example Section Content

### Top Risks Example
```
Slide Title: "Top Security Risks - Q1 FY26"

1. DLP Visibility and Controls Risk (R0036)
   - Likelihood: Possible | Impact: High | Score: 12
   - Weak DLP controls could lead to undetected data leakage
   - Mitigation: Deploy comprehensive DLP; integrate with SIEM

2. Insufficient First-Party Code Security Testing (R0038)
   - Likelihood: Possible | Impact: High | Score: 12
   - No SAST tooling increases risk of vulnerabilities in production
   - Mitigation: Deploy SAST solution; integrate into SDLC

3. Lack of Network Traffic Filtering (R0041)
   - Likelihood: Possible | Impact: High | Score: 12
   - No visibility into employee web traffic or SaaS usage
   - Mitigation: Deploy ZTNA/SASE solution; establish governance
```

### Roadmap Accountability Example
```
Slide Title: "FY25 Q4 Roadmap Results"

**Completion Rate: 60%** (3 of 5 planned tasks completed)

Originally Planned:
| Task | Result |
|------|--------|
| 2025 External Pen Test | Completed |
| SASE Project | Pushed to Q1 FY26 |
| ISO 27001 Prep | Pushed to Q1 FY26 |
| SOC 2 Audit Prep | Completed |
| Wiz SAST Setup | Completed |

Added During Quarter:
- GRC Platform Eval (Completed)
- ROI Hunter Assessment (In Progress)
```

### Highlights Example
```
Slide Title: "FY25 Q4 Engineering Highlights"

**Identity & Access Management**
- Completed Quarterly User Access Reviews across 6 engineering teams
- Completed 3rd Party App SSO Inventory

**Monitoring & Detection**
- Integrated Workday logs into InsightIDR
- Continued Monthly Phishing Test program

**GRC & Compliance**
- Evaluated 5 GRC platforms (Delve, Auditboard, Vanta, Drata, Complyance)
- Built scoring matrix for platform selection
```

### Placeholder Examples

**Incidents (not configured):**
```
No incident tracking data source configured.
This section will be populated once incident data is available.
```

**Metrics (not configured):**
```
KPI data sources not yet defined.
This section will be populated once metrics integrations are configured.
```

---

## Validation

After building, verify the output:

1. **Check JSON outputs exist:**
   ```
   {scratchpad}/qbr-data/
   ├── config.json
   ├── risks.json
   ├── roadmap.json
   ├── highlights.json
   ├── incidents.json
   └── metrics.json
   ```

2. **Check slides generated:**
   ```bash
   ls output/html-slides/  # Should show 14 HTML files
   ```

3. **Test the presentation:**
   - Open `output/html-slides/slide01-title.html` in a browser
   - Verify slides scale correctly when resizing the window
   - Test navigation with arrow keys and prev/next buttons

4. **Confirm placeholder sections:**
   - Metrics slide shows placeholder message if KPI data not configured

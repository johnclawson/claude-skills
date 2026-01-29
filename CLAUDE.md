# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a **Claude Skills repository** - a collection of specialized skills that extend Claude's capabilities. Skills are defined through YAML frontmatter and markdown documentation, not traditional code.

Each skill follows a consistent structure:
- `SKILL.md` - Main skill definition with frontmatter and instructions
- `references/` - Detailed reference materials (design specs, inventories)
- `assets/` - Binary resources (templates, logos, images)

## Skills

| Skill | Purpose |
|-------|---------|
| `infosec-tasks` | Retrieve and filter security engineering tasks from ClickUp by fiscal quarter |
| `security-qbr` | Generate quarterly business review presentations for InfoSec leadership |
| `pattern-pptx` | Create PowerPoint presentations using Pattern Security brand guidelines |
| `clawson-family-travel` | Vacation planning optimized for credit card points redemptions |

## Architecture Patterns

### Multi-Agent Orchestration (security-qbr)

The QBR skill uses parallel agent execution:
```
Main Orchestrator
├── Risk Review Agent → risks.json
├── Roadmap Agent → roadmap.json
├── Highlights Agent → highlights.json
├── Incidents Agent → incidents.json
├── Metrics Agent → metrics.json
└── Slide Builder Agent (sequential) → HTML slides
```

### Data Snapshot Pattern (infosec-tasks)

Roadmap snapshots capture planned tasks at a point in time:
- Location: `data/roadmap-snapshots/{QUARTER}-snapshot.json`
- Enables accountability comparison: "planned vs. actual"
- Used by security-qbr for quarterly reporting

### 1080p HTML Slide Generation

Security QBR generates standalone HTML slides:
- Fixed 1920×1080 dimensions with auto-scaling JavaScript
- Keyboard navigation (arrow keys, space)
- Pattern brand colors embedded (#009BFF blue, #231F20 dark)

## ClickUp Integration

Key configuration values embedded in skill files:

| Resource | ID |
|----------|-----|
| Workspace | `2323726` |
| Infosec Space | `30075124` |
| Security Engineering List | `132251242` |
| Penetration Testing List | `900202119210` |
| GRC List | `901704270170` |
| Quarter Field | `df8ff1a4-7dd1-4129-b3f6-cf0b745a21c9` |
| Incident History Doc | `26x8e-33627` (page: `26x8e-31105`) |

Quarter field value mapping (order_index):
- FY26 Q1=0, Q2=1, Q3=2, Q4=3
- FY25 Q4=4, Q3=5
- FY27 Q1=6

## Pattern Brand Guidelines

| Element | Value |
|---------|-------|
| Primary Blue | `#009BFF` |
| Dark Text | `#231F20` |
| Off-White | `#F2F2F2` |
| Hero Titles | Ramaraja 148pt |
| Section Numbers | Oswald Medium 72pt |
| Body Text | Montserrat Light/Medium 18-30pt |

Template with 39 slide layouts: `pattern-pptx/assets/pattern-template.pptx`

## Common Pitfalls

1. **ClickUp status filters** - Always include `"completed"` status; ClickUp has separate status categories that may be missed
2. **Pagination** - ClickUp defaults to 200 results; always paginate through all results
3. **Quarter filtering** - Use the order_index mapping, not string matching
4. **Brand colors** - Use exact hex values; approximate colors break brand compliance
5. **Data dependencies** - QBR slide generation requires all parallel agents to complete first

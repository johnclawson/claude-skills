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
| `music-rating-system` | Rate music library for kid-safe content and sync to kids library |

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

## Music Rating System

Python scripts that scan a music library, fetch lyrics (Genius API), analyze content with Claude, and maintain a kid-safe subset of the library.

### Kid-Safe Criteria

- **No foul language** (mild "damn"/"hell" = caution, not blocked)
- **No LGBTQ+ themes**
- **No drug/alcohol themes** (literal use only; metaphorical "drunk on your love" = kid-safe)
- **Romance OK, no sexual content**
- **Figurative language is NOT flagged** - only literal references

### Rating Categories

| Rating | Description |
|--------|-------------|
| `kid-safe` | Appropriate for children |
| `caution` | Parental discretion advised (mild language, borderline content) |
| `not-safe` | Not appropriate for children |
| `unknown` | Could not fetch lyrics or analyze (obscure tracks) |

### Key Files

| File | Purpose |
|------|---------|
| `music-rating-system/rate_music.py` | Main rating script - scans library, fetches lyrics, rates songs |
| `music-rating-system/sync_kids_library.py` | Copies kid-safe music to `/mnt/media/Music-kids` via hardlinks |
| `music-rating-system/generate_playlist.py` | Generate M3U playlists and CSV reports from ratings |
| `music-rating-system/music_ratings.json` | Full ratings database (all 1,518 songs) |
| `music-rating-system/requirements.txt` | Python dependencies: anthropic, lyricsgenius, mutagen, tqdm |

### Library Paths

| Path | Description |
|------|-------------|
| `/mnt/media/Music` | Main music library (source of truth) |
| `/mnt/media/Music-kids` | Kid-safe subset (hardlinked, zero extra disk space) |

### Environment Variables Required

- `GENIUS_ACCESS_TOKEN` - Genius API token (https://genius.com/api-clients)
- `ANTHROPIC_API_KEY` - Anthropic API key
- Source these with `source ~/dotfiles/secrets` before running

### Common Workflows

**Rate new music after purchase:**
```bash
source ~/dotfiles/secrets
cd ~/claude-skills/music-rating-system
python rate_music.py /mnt/media/Music --rate-new --sync-kids /mnt/media/Music-kids
```

**Retry unknown songs** (after Genius rate limit resets):
```bash
python rate_music.py /mnt/media/Music --resume
```

**Sync kids library manually:**
```bash
python sync_kids_library.py music_ratings.json /mnt/media/Music-kids --clean
```

**Preview sync without changes:**
```bash
python sync_kids_library.py music_ratings.json /mnt/media/Music-kids --dry-run
```

### Architecture

```
rate_music.py
├── Scan library (mutagen metadata extraction, ffprobe fallback)
├── For each song:
│   ├── Check [Clean] tag → auto kid-safe
│   ├── Fetch lyrics from Genius API (1.5s rate limit between calls)
│   │   ├── If lyrics found → analyze_lyrics_with_claude()
│   │   └── If no lyrics → analyze_with_claude_knowledge() fallback
│   └── Save incrementally every 10 songs
├── Output: music_ratings.json + filtered kid_safe/not_safe JSONs
└── Optional: --sync-kids triggers sync_kids_library
```

### Key Design Decisions

- **Hardlinks** for kids library (same filesystem, zero extra space, instant sync)
- **Genius rate limit handling**: tracks `_genius_retry_after` globally, skips Genius while limited and falls back to Claude knowledge
- **[Clean] version trust**: if album/title contains `[Clean]`, auto-rated kid-safe without lyrics lookup
- **Incremental save + resume**: saves every 10 songs; `--resume` retries unknowns, `--rate-new` skips everything already rated
- **Claude Sonnet** (`claude-sonnet-4-20250514`) for content analysis - balances cost and accuracy
- **Figurative vs literal**: analysis prompt explicitly distinguishes metaphorical language from literal drug/alcohol use to avoid false positives

### Pitfalls

1. **Genius rate limiting** - API returns HTTP 429 after ~50-100 rapid requests. The 1.5s delay helps but long runs will still get limited. The Claude knowledge fallback handles this gracefully.
2. **Wrong lyrics from Genius** - Instrumental tracks or obscure songs may get matched to wrong lyrics (e.g., a Shostakovich violin adagio got explicit rap lyrics). The Claude knowledge fallback is better for classical/instrumental.
3. **Bare `except` clauses** - The original Genius library swallows errors silently. Our wrapper catches 429/1015 errors explicitly.
4. **`--resume` vs `--rate-new`** - `--resume` retries unknowns (for after rate limit resets). `--rate-new` skips everything already rated including unknowns (for new music additions).

## Common Pitfalls

1. **ClickUp status filters** - Always include `"completed"` status; ClickUp has separate status categories that may be missed
2. **Pagination** - ClickUp defaults to 200 results; always paginate through all results
3. **Quarter filtering** - Use the order_index mapping, not string matching
4. **Brand colors** - Use exact hex values; approximate colors break brand compliance
5. **Data dependencies** - QBR slide generation requires all parallel agents to complete first

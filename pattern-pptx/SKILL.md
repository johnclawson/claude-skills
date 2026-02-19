---
name: pattern-pptx
description: Create PowerPoint presentations using the Pattern Security brand template. Ensures pixel-perfect match to Pattern brand guidelines including colors (#009BFF blue, #231F20 dark), fonts (Ramaraja, Oswald Medium, Montserrat family), and layouts. Use this skill when creating any Pattern-branded slide decks, presentations, or pitch decks. Triggers on requests for Pattern presentations, Pattern-styled slides, or any presentation needing Pattern brand compliance.
---

# Pattern PowerPoint Template

Create presentations that match Pattern Security's brand template exactly.

## Brand Colors

| Name | Hex | Usage |
|------|-----|-------|
| **Pattern Blue** | `#009BFF` | Primary accent, numbers, lines, subtitles |
| **Dark Text** | `#231F20` | Body text, dark slide backgrounds |
| **Off-White** | `#F2F2F2` | Titles on dark slides |
| **Light Gray** | `#F0F0F0` | Section titles on dark slides |
| **Warm Gray** | `#E0DBD7` | Metadata (dates, labels) |
| **Medium Gray** | `#888888` | Footer, slide numbers |
| **White** | `#FFFFFF` | Light slide backgrounds |

## Typography

| Font | Size | Usage |
|------|------|-------|
| Ramaraja | 148pt | Hero titles (title slide only) |
| Oswald Medium | 72pt | Section numbers ("01.", "02.") |
| Oswald Medium | 76pt | Large subtitles |
| Montserrat Medium | 30pt | Slide titles |
| Montserrat Medium | 24pt | Subtitles |
| Montserrat Light | 18-24pt | Body text |
| Montserrat Light | 16pt | Captions |
| Calibri | varies | Tables and data |

## Slide Patterns

### Title Slide (Dark Background)
- Background: `#231F20`
- Logo: `assets/logo-light.png` (top-left)
- Title: Ramaraja 148pt, `#F2F2F2`
- Subtitle: Montserrat Medium 24pt, `#009BFF`
- Metadata: Montserrat Light 18pt, `#E0DBD7`

### Section Divider (Dark Background)
- Background: `#231F20`
- Number: Oswald Medium 72pt, `#009BFF`
- Title: Montserrat Light 28pt, `#F0F0F0`

### Content Slide (Light Background)
- Background: `#FFFFFF`
- Title: Montserrat Medium 30pt, dark text
- Accent line: `#009BFF`, 0.75pt thickness
- Body: Montserrat Light 18-24pt

## Logo Usage

| Background | Logo Asset |
|------------|------------|
| Light (`#FFFFFF`) | `assets/logo-dark.png` |
| Dark (`#231F20`) | `assets/logo-light.png` |

## Layout Reference

See [slide-layouts.md](references/slide-layouts.md) for the complete inventory of 39 available slide layouts.

See [design-spec.md](references/design-spec.md) for detailed positioning specs (EMUs, margins, placeholders).

## Quality Checklist

Before delivery, verify:
- [ ] Correct fonts used (no Arial/Helvetica substitutions)
- [ ] Pattern Blue `#009BFF` for all accents
- [ ] Dark slides use `#231F20` (not pure black)
- [ ] Light text uses `#F2F2F2` (not pure white)
- [ ] Logo variant matches background
- [ ] Section numbers in Oswald Medium + Pattern Blue
- [ ] Accent lines are 0.75pt in Pattern Blue

## Assets

| Asset | Path | Description |
|-------|------|-------------|
| Template | `assets/pattern-template.pptx` | Base template with all 39 layouts |
| Logo (dark) | `assets/logo-dark.png` | Dark wordmark for light backgrounds |
| Logo (light) | `assets/logo-light.png` | Light wordmark for dark backgrounds |

## Output

Use the pptx skill for file generation. Reference `assets/pattern-template.pptx` for master slides and layouts.

### QBR Compatibility

This template is used by the `security-qbr` skill to generate branded QBR decks. Key template slide indices used by the QBR builder:

| Template Slide | QBR Usage |
|----------------|-----------|
| 1 | Title slide |
| 2 | Agenda |
| 3, 20, 25, 32 | Section dividers |
| 4 | Title + bullets (risks, incidents) |
| 21-22 | Multi-column lists (roadmap, highlights) |
| 35 | Dashboard (metrics) |
| 36 | Closing slide |

When modifying these template slides, verify the QBR output still renders correctly.

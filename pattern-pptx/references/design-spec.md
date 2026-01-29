# Pattern Security Template - Design Specification

## Quick Reference

### Brand Colors
| Name | Hex | Usage |
|------|-----|-------|
| **Pattern Blue** | `#009BFF` | Primary accent, numbers, lines, subtitles |
| **Dark Text** | `#231F20` | Body text, dark backgrounds |
| **Off-White** | `#F2F2F2` | Titles on dark slides |
| **Light Gray** | `#F0F0F0` | Section titles on dark |
| **Warm Gray** | `#E0DBD7` | Metadata (dates, labels) |
| **Medium Gray** | `#888888` | Footer, slide numbers |
| **White** | `#FFFFFF` | Light backgrounds |
| **Black** | `#000000` | Default text |

### Font Hierarchy
| Font | Weight | Size | Usage |
|------|--------|------|-------|
| Ramaraja | Regular | 148pt | Hero titles (title slide only) |
| Oswald Medium | Medium | 72pt | Section numbers ("01.", "02.") |
| Oswald Medium | Medium | 76pt | Large subtitles |
| Montserrat Medium | Medium | 30pt | Slide titles |
| Montserrat Medium | Medium | 24pt | Subtitles |
| Montserrat Light | Light | 24pt | Body text (large) |
| Montserrat Light | Light | 18pt | Body text (standard) |
| Montserrat Light | Light | 16pt | Captions |
| Calibri | Regular | 18pt | Tables, data |

### Slide Dimensions
- **Size**: 16:9 widescreen (1920x1080 equivalent)
- **Width**: 18,288,000 EMUs (20.32")
- **Height**: 10,287,000 EMUs (11.43")

---

## Design Patterns

### Title Slide (Dark)
- Background: `#231F20`
- Logo: Light version (top-left, ~2" wide)
- Title: Ramaraja 148pt, `#F2F2F2`, 58% line spacing
- Subtitle: Montserrat Medium 24pt, `#009BFF`
- Metadata: Montserrat Light 18pt bold/regular, `#E0DBD7`

### Section Divider (Dark)
- Background: `#231F20`
- Number: Oswald Medium 72pt, `#009BFF`, centered
- Title: Montserrat Light 28pt, `#F0F0F0`, centered

### Content Slide (Light)
- Background: `#FFFFFF`
- Title: Montserrat Medium 30pt, `#000000` (dk1)
- Accent line: `#009BFF`, 0.75pt, below title
- Body: Montserrat Light 18-24pt

### Quote Slide
- Quote: Oswald Medium, uppercase
- Accent: Pattern Blue
- Attribution: Montserrat Light

---

## Positioning (EMUs)

### Standard Margins
- Left: 743,046 (~0.82")
- Title Y: 585,194 (~0.65")
- Accent line Y: 1,380,393

### Title Placeholder
- X: 623,400 | Y: 890,050
- W: 17,041,200 | H: 1,145,400

### Body Placeholder
- X: 623,400 | Y: 2,304,950
- W: 17,041,200 | H: 6,832,800

### Slide Number
- X: 16,944,916 | Y: 9,326,434
- W: 1,097,400 | H: 787,200
- Align: Right | Size: 20pt | Color: `#888888`

---

## Bullet Styles
| Level | Char | Size | Indent | Margin |
|-------|------|------|--------|--------|
| 1 | ● | 36pt | -457,200 | 457,200 |
| 2 | ○ | 28pt | -406,400 | 914,400 |
| 3 | ■ | 28pt | -406,400 | 1,371,600 |

---

## Shadow Effect (Images)
- Type: Outer shadow
- Blur: 241,300 EMUs
- Direction: 315° (bottom-left)
- Distance: 241,300 EMUs
- Color: `#000000` @ 10.98% opacity

---

## Logo Assets
- `logo-dark.png`: Blue icon + dark wordmark (for light backgrounds)
- `logo-light.png`: Blue icon + light wordmark (for dark backgrounds)
- Icon color: `#009BFF`

---

## Embedded Fonts
The template embeds these fonts for cross-platform consistency:
- Ramaraja (Regular)
- Oswald Medium (Regular, Bold)
- Montserrat (Regular, Bold, Italic, Bold Italic)
- Montserrat Medium (Regular, Bold, Italic, Bold Italic)
- Montserrat Light (Regular, Bold, Italic, Bold Italic)

---

## EMU Conversions
| EMUs | Inches | Points |
|------|--------|--------|
| 914,400 | 1" | 72pt |
| 457,200 | 0.5" | 36pt |
| 12,700 | ~0.014" | 1pt |

Formula: `inches = EMUs / 914,400`

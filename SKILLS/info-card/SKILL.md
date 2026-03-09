---
name: info-card
description: Generates magazine-quality HTML info cards from text or documents using Swiss editorial design. Applies big-typography layout for sparse content and multi-column newspaper grid for dense content. Use when user asks to "generate info card", "create info card", "make a knowledge card", or "信息卡". Don't use for AI image generation, interactive dashboards, data charts, slide decks, or social media post series.
---

# Info Card Generator

Transforms text or documents into styled HTML info cards with editorial magazine aesthetics. Uses Swiss International typography — rigorous grid structure with modern visual impact.

## Usage

```
# From a file
/info-card article.md

# Direct content input
/info-card
[paste content]

# With theme
/info-card article.md --theme dark

# With custom output
/info-card article.md --output my-card.html
```

## Options

| Option | Description |
|--------|-------------|
| `--theme cream\|white\|dark` | Color theme (default: `cream`) |
| `--output <path>` | Custom output HTML path |

## Output

Generates a self-contained HTML file (embedded CSS) rendering as a 900px-wide card. Open directly in any browser — no build step or server needed.

**File structure**:

```
info-card/{topic-slug}/
├── source-{slug}.{ext}    # Copy of source
└── card.html              # Generated info card
```

**Slug generation**: Extract 2-4 kebab-case words from the main topic.
**Conflict resolution**: If directory exists, append timestamp `{slug}-YYYYMMDD-HHMMSS`.

---

## Procedures

### Step 1: Load Preferences (EXTEND.md)

Check EXTEND.md existence using Bash (priority order):

```bash
test -f .skills/info-card/EXTEND.md && echo "project"
test -f "$HOME/.skills/info-card/EXTEND.md" && echo "user"
```

┌────────────────────────────────────────────┬───────────────────┐
│                    Path                    │     Location      │
├────────────────────────────────────────────┼───────────────────┤
│ .skills/info-card/EXTEND.md          │ Project directory │
├────────────────────────────────────────────┼───────────────────┤
│ $HOME/.skills/info-card/EXTEND.md    │ User home         │
└────────────────────────────────────────────┴───────────────────┘

┌───────────┬────────────────────────────────────────┐
│  Result   │               Action                   │
├───────────┼────────────────────────────────────────┤
│ Found     │ Read, parse, apply settings             │
├───────────┼────────────────────────────────────────┤
│ Not found │ Proceed with defaults (no setup needed) │
└───────────┴────────────────────────────────────────┘

**EXTEND.md Supports**: Default theme | Accent color override | Language preference

### Step 2: Analyze Content

1. Read the source file, or save pasted text to `source-content.md`.
2. Detect the content language (Chinese / English / mixed).
3. Assess information density using this table:

| Density | Signal | Layout |
|---------|--------|--------|
| **Low** | 1 main idea, quote, or stat | Big Typography |
| **Medium** | 1 topic with 3–5 supporting points | Standard |
| **High** | Multiple concepts, comparisons, or data tables | Multi-column Grid |

4. Extract: main title, key points, content tone/mood.
5. Infer accent color from mood (see `references/design-spec.md` → Accent Color by Mood).
6. Output one sentence: `Density: [Low/Medium/High] — [reason]`

### Step 3: Generate HTML Info Card

Read `assets/card-template.html` for the base structure and CSS variables. Produce a **complete, self-contained HTML file** — all CSS embedded in `<style>` tags, no external stylesheets.

Apply the layout matching the density from Step 2:

**Low density → Big Typography**
- Set `.hero-title` at 72–84px, weight 900, spanning full card width.
- Place the key stat or quote at 120px+, styled as a background overlay element.
- Use minimal body text and generous white space.

**Medium density → Standard**
- Set `.main-title` at 56px.
- Include 2–3 body sections at 18–20px.
- Add one or two `.callout` blocks with `rgba(0,0,0,0.03)` background.

**High density → Multi-column Grid**
- Set the title at 48–56px spanning full width.
- Split body into 2–3 `.column` elements with vertical dividers.
- Set each column's body at 18px; label each with a `.label` tag.
- Use compact gaps (20–24px) between components.

**Always include in every layout**:
- Noise texture overlay at 4% opacity (see template `::after` pseudo-element).
- `.accent-bar`: 4–6px solid rule in the chosen accent color.
- `.label` tags: `13px`, `font-weight: 700`, `letter-spacing: 0.15em`, `text-transform: uppercase`.
- Body text at ≥ 18px (ensures readability on mobile screens).

Full design rules: read `references/design-spec.md` when layout decisions require clarification.

### Step 4: Save and Report

1. Create output directory `info-card/{topic-slug}/`.
2. Copy source file to `info-card/{topic-slug}/source-{slug}.{ext}`.
   - If content was pasted: save as `source-content.md`.
3. Write the generated HTML to `info-card/{topic-slug}/card.html` (or `--output` path if provided).
4. Self-check: verify body text is ≥ 18px and the main title dominates visually.
5. Print the completion report:

```
Info Card Generated!

Topic:    [topic]
Density:  [Low / Medium / High]
Layout:   [Big Typography / Standard / Multi-column Grid]
Theme:    [cream / white / dark]
Accent:   [hex color] ([mood])
Output:   info-card/{topic-slug}/card.html

Open card.html in any browser to view.
```

## Card Modification

| Action | Steps |
|--------|-------|
| **Edit content** | Modify source → re-run Step 3–4 |
| **Change theme** | Re-run with `--theme <name>` |
| **Adjust accent** | Specify color or mood → regenerate |

## Error Handling

- **Source not found**: If the file path does not exist, halt and ask the user to confirm the path or paste the content directly.
- **Empty content**: If the source is blank or fewer than 10 words, halt and request more content before proceeding.
- **Unsupported theme**: If `--theme` is not `cream`, `white`, or `dark`, warn and fall back to `cream`.
- **Output path conflict**: If `card.html` already exists at the target path, rename the existing file to `card-backup-YYYYMMDD-HHMMSS.html` before writing.
- **Font load failure**: If Google Fonts is unavailable (offline environment), fall back to system fonts: `Georgia, serif` for headings and `system-ui, sans-serif` for body.

## Extension Support

Custom configurations via EXTEND.md. See **Step 1** for paths and supported options.

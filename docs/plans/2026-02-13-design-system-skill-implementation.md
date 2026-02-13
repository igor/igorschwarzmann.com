# Design System Skill — Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Convert the 2,100-line Obsidian design system into a Claude Code skill with progressive disclosure for igorschwarzmann.com.

**Architecture:** Skill at `~/.claude/skills/igorschwarzmann-design/` with SKILL.md (router + inline tokens) and 4 reference files + 2 HTML templates. Obsidian docs trimmed to rationale only.

**Tech Stack:** Claude Code skills (Markdown), HTML templates, Obsidian vault

## Source Files

These are the source documents to extract implementation specs from:

| Source | Path | Lines |
|--------|------|-------|
| Main Design System | `brain dead/2. Areas/igorschwarzmann.com/Design System igorschwarzmann.com.md` | ~1253 |
| Essay Mode | `brain dead/2. Areas/igorschwarzmann.com/Design System igorschwarzmann.com - Essay Mode.md` | ~416 |
| Personal Mode | `brain dead/2. Areas/igorschwarzmann.com/Design System igorschwarzmann.com - Personal Mode.md` | ~507 |
| Live essay `<head>` | `~/GitHub/igorschwarzmann.com/strategyasprotocol/strategy-as-protocol/index.html` | lines 1-50 |
| Live hub `<head>` | `~/GitHub/igorschwarzmann.com/strategyasprotocol/index.html` | lines 1-60 |

**Obsidian vault root:** `/Users/zeigor/Library/Mobile Documents/iCloud~md~obsidian/Documents/brain dead`

---

### Task 1: Create skill directory structure

**Files:**
- Create: `~/.claude/skills/igorschwarzmann-design/SKILL.md`
- Create: `~/.claude/skills/igorschwarzmann-design/reference/` (directory)
- Create: `~/.claude/skills/igorschwarzmann-design/templates/` (directory)

**Step 1: Create directories**

```bash
mkdir -p ~/.claude/skills/igorschwarzmann-design/reference
mkdir -p ~/.claude/skills/igorschwarzmann-design/templates
```

**Step 2: Verify structure**

```bash
ls -R ~/.claude/skills/igorschwarzmann-design/
```

Expected: Two empty subdirectories `reference/` and `templates/`

---

### Task 2: Write SKILL.md

**Files:**
- Create: `~/.claude/skills/igorschwarzmann-design/SKILL.md`

**Step 1: Write SKILL.md**

This is the always-loaded hub. Contains three parts: mode detection, inline design tokens, universal rules.

Content structure:

```markdown
# igorschwarzmann.com Design System

Design system for igorschwarzmann.com. Use when creating or modifying pages
on the personal website, implementing visual design, or checking design system
compliance. Covers Personal mode (landing pages), Essay mode (long-form articles),
and Presentation mode (slide decks).

## Mode Detection

Determine which mode applies based on the file being edited:

| File path | Mode | Load these references |
|-----------|------|----------------------|
| `index.html` (repo root) | Personal | `personal-mode.md` + `seo.md` |
| `strategyasprotocol/index.html` | Hub/Collection | `essay-mode.md` + `seo.md` |
| `strategyasprotocol/[slug]/index.html` | Essay | `essay-mode.md` + `seo.md` |
| `strategyasprotocol/designsystems/` | Presentation | `components.md` + `seo.md` |
| New page (any type) | Ask user which mode | mode-specific ref + `seo.md` |

Always load `seo.md` for any page creation or modification.

## Site Architecture

- **Repo:** `~/GitHub/igorschwarzmann.com/` (GitHub Pages, `igor/igorschwarzmann.com`)
- **Domain:** `igorschwarzmann.com`
- **Stack:** Single-file HTML with embedded CSS/JS. No build system.
- **Deploy:** Push to `main` → GitHub Pages auto-deploys

```
igorschwarzmann.com/
├── index.html                           ← Homepage (Personal mode)
├── robots.txt
├── sitemap.xml
├── llms.txt
└── strategyasprotocol/
    ├── index.html                       ← Hub page (Essay mode)
    ├── strategy-as-protocol/index.html  ← Essay
    ├── custodian-shift/index.html       ← Essay
    └── designsystems/index.html         ← Presentation (slide deck)
```

## Design Tokens

### Colors

| Name | Hex | Semantic meaning |
|------|-----|------------------|
| **Volt** | `#E2FF00` | Your territory — frameworks, voice, navigation, links, highlights |
| **Signal** | `#00FF00` | Emerging — AI, new thinking, nav name accent |
| **Shift** | `#FF00FF` | Transformation — paradigm shifts, used sparingly |
| **White** | `#FFFFFF` | Headings, bold text, light-mode backgrounds |
| **Black** | `#000000` | Body text on light backgrounds, dark-mode base |
| **Blush** | `#F5F0EB` | Warm neutral alternative background |

### Backgrounds

| Context | Background | Text color |
|---------|-----------|------------|
| Dark mode (essays, hub, homepage) | `#0A0A0A` | `rgba(255,255,255,0.87)` body, `#FFF` headings |
| Light mode (presentations) | `#FFFFFF` or `#F5F0EB` | `#000000` |

### Typography

| Role | Personal + Essay modes | Presentation mode |
|------|----------------------|-------------------|
| **Display** | Syne (600-800) | Darker Grotesque (600-700) |
| **Body** | IBM Plex Sans (300-600) | DM Sans (400-700) |
| **Accent/Tech** | IBM Plex Mono (400-500) | — |

### Spacing

| Token | Value |
|-------|-------|
| `--space-xs` | 8px |
| `--space-sm` | 16px |
| `--space-md` | 24px |
| `--space-lg` | 48px |
| `--space-xl` | 96px |

### Easing

| Token | Value | Use |
|-------|-------|-----|
| `--ease-out-expo` | `cubic-bezier(0.16, 1, 0.3, 1)` | Snappy entries |
| `--ease-out-quart` | `cubic-bezier(0.25, 1, 0.5, 1)` | Smooth general purpose |

## Universal Rules

These apply to ALL pages regardless of mode:

1. **URLs**: Always use trailing slashes (`/strategyasprotocol/` not `/strategyasprotocol`)
2. **Images**: Every `<img>` needs `alt` text and `loading="lazy"` (except above-fold hero images)
3. **SEO**: Every page needs the full `<head>` metadata block — see `reference/seo.md`
4. **Motion**: Respect `prefers-reduced-motion` — disable all animations when set
5. **Files**: Single-file HTML with embedded `<style>` and `<script>`. No external CSS/JS files.
6. **Fonts**: Load via Google Fonts with `rel="preconnect"` for both `fonts.googleapis.com` and `fonts.gstatic.com`
7. **Color scheme**: Add `<meta name="color-scheme" content="dark">` for dark-mode pages (essays, homepage)

## Reference Files

For detailed implementation specs, load the appropriate reference file:

- `reference/personal-mode.md` — Type hierarchy, hero layout, atmospheric effects, hover interactions, responsive rules
- `reference/essay-mode.md` — Two-zone layout, figure placement, blockquotes, typography sizing
- `reference/seo.md` — Meta tags, JSON-LD templates, new-page checklist
- `reference/components.md` — Nav bar, footer, progress bar, author bio

## Templates

Copy-pastable `<head>` blocks with `[REPLACE]` placeholders:

- `templates/essay-head.html` — For new Strategy-as-Protocol essays
- `templates/hub-head.html` — For new hub/collection pages
```

**Step 2: Verify file exists and is readable**

```bash
wc -l ~/.claude/skills/igorschwarzmann-design/SKILL.md
```

Expected: ~120-140 lines

**Step 3: Commit**

```bash
# No git repo for skills directory — just verify the file is correct
```

---

### Task 3: Write reference/personal-mode.md

**Files:**
- Create: `~/.claude/skills/igorschwarzmann-design/reference/personal-mode.md`
- Source: Obsidian `Design System igorschwarzmann.com - Personal Mode.md`

**Step 1: Write personal-mode.md**

Extract implementation specs from the Personal Mode doc. Include:

1. **Google Fonts link** — exact `<link>` tag to copy
2. **CSS custom properties** — the full `:root` block for Personal mode
3. **Type hierarchy table** — Element / Font / Size / Weight / Line-height / Usage
4. **Hero layout pattern** — HTML structure snippet (name top-left, contact top-right, statements center)
5. **Statement hover CSS** — The three `nth-child` hover rules with Volt/Signal/Shift
6. **Button hover CSS** — Contact button hover effect
7. **Atmospheric depth CSS** — Background gradients (3 radial-gradients at 1-3% opacity) + grain overlay
8. **Responsive breakpoints** — Desktop (1024+) / Tablet (768-1024) / Mobile (<768) with type sizes
9. **Reduced motion CSS** — The `@media (prefers-reduced-motion)` block
10. **Checklist** — Implementation checklist items

Format: Tables for specs, code blocks for CSS, no prose explanations.

Source sections to extract from (Personal Mode doc):
- Lines 68-74: Type scale CSS
- Lines 144-154: Typography hierarchy table
- Lines 160-165: Google Fonts link + CSS properties
- Lines 225-255: Hero layout HTML pattern
- Lines 278-298: Statement hover CSS
- Lines 302-318: Button hover CSS
- Lines 327-346: Atmospheric depth CSS
- Lines 357-374: Responsive breakpoints
- Lines 436-460: Reduced motion CSS
- Lines 486-498: Implementation checklist

**Step 2: Verify line count**

```bash
wc -l ~/.claude/skills/igorschwarzmann-design/reference/personal-mode.md
```

Expected: ~100-130 lines

---

### Task 4: Write reference/essay-mode.md

**Files:**
- Create: `~/.claude/skills/igorschwarzmann-design/reference/essay-mode.md`
- Source: Obsidian `Design System igorschwarzmann.com - Essay Mode.md`

**Step 1: Write essay-mode.md**

Extract implementation specs from the Essay Mode doc. Include:

1. **Google Fonts link** — Same as Personal mode (Syne + IBM Plex Sans + IBM Plex Mono)
2. **CSS custom properties** — Full `:root` block including essay layout tokens
3. **Layout architecture** — Two-zone proportions table (text 600px, gap 60px, margin 360px, container 1080px)
4. **Responsive collapse table** — 4 breakpoints with container/text/margin/behavior
5. **Type hierarchy table** — Element / Font / Size / Weight / Line-height / Max-width
6. **Figure placement rules** — Float rules, variants table (default/book/wide), CSS snippets
7. **Blockquote CSS** — Border-left accent style
8. **Link CSS** — Volt underline treatment
9. **Navigation spec** — Top bar: IBM Plex Mono, 11px, uppercase, Signal green name, tertiary section
10. **Essay header spec** — Metadata line, title, deck styling
11. **Progress bar CSS** — Fixed top, 2px Volt
12. **Reduced motion CSS** — Essay-specific reduced motion rules
13. **Checklist** — Essay creation checklist (including SEO steps)

Source sections to extract from (Essay Mode doc):
- Lines 35-43: Layout proportions table
- Lines 86-91: Responsive collapse table
- Lines 107-118: Type hierarchy table
- Lines 196-217: Blockquote CSS
- Lines 222-235: Link CSS
- Lines 239-249: Progress bar CSS
- Lines 259-266: Figure default CSS
- Lines 270-275: Figure variants table
- Lines 279-289: Book cover variant CSS
- Lines 291-296: Figure placement rules
- Lines 339-377: CSS design tokens block

**Step 2: Verify line count**

```bash
wc -l ~/.claude/skills/igorschwarzmann-design/reference/essay-mode.md
```

Expected: ~130-160 lines

---

### Task 5: Write reference/seo.md

**Files:**
- Create: `~/.claude/skills/igorschwarzmann-design/reference/seo.md`
- Source: Main Design System Section 10 (recently added) + live page implementations

**Step 1: Write seo.md**

This file was partially written during the SEO work earlier today. Extract and refine from the Section 10 we added to the main design system, plus the live implementations. Include:

1. **Required meta tags table** — Element / Format / Notes
2. **JSON-LD Article template** — Complete copy-pastable snippet with `[REPLACE]` markers
3. **JSON-LD CollectionPage template** — Complete copy-pastable snippet
4. **JSON-LD BreadcrumbList template** — 3-level breadcrumb
5. **Cross-reference rules** — @id conventions, trailing slashes, date formats
6. **Homepage entities** — Reference table of @id values defined on homepage (/#person, /#org, /#website, etc.)
7. **New page checklist** — sitemap.xml, llms.txt, internal linking, canonical, meta desc
8. **Site-wide files** — Brief reminder of robots.txt, sitemap.xml, llms.txt locations and what to update

**Step 2: Verify line count**

```bash
wc -l ~/.claude/skills/igorschwarzmann-design/reference/seo.md
```

Expected: ~80-110 lines

---

### Task 6: Write reference/components.md

**Files:**
- Create: `~/.claude/skills/igorschwarzmann-design/reference/components.md`
- Source: Essay Mode doc (nav, footer, bio) + live pages

**Step 1: Write components.md**

Shared components used across page types. Include:

1. **Site navigation (top bar)** — HTML structure, CSS specs (IBM Plex Mono 11px, name in Signal green, section in tertiary, uppercase, 0.12em letter-spacing)
2. **Essay header** — HTML structure (essay-meta with date/reading-time, h1 title, deck subtitle)
3. **Author bio footer** — HTML structure, CSS specs (IBM Plex Sans 16px italic, secondary color, CTA link in Volt)
4. **Site footer** — HTML structure (copyright left, email right, monospace, tertiary)
5. **Reading progress bar** — CSS + minimal JS snippet (fixed top, 2px Volt, width based on scroll %)

Source sections:
- Essay Mode doc lines 149-161: Navigation spec
- Essay Mode doc lines 162-178: Essay header spec
- Essay Mode doc lines 315-325: Essay footer / bio
- Live pages: Extract actual HTML patterns from implemented pages

**Step 2: Verify line count**

```bash
wc -l ~/.claude/skills/igorschwarzmann-design/reference/components.md
```

Expected: ~60-90 lines

---

### Task 7: Write templates/essay-head.html

**Files:**
- Create: `~/.claude/skills/igorschwarzmann-design/templates/essay-head.html`
- Source: Live `strategyasprotocol/strategy-as-protocol/index.html` lines 1-55

**Step 1: Write essay-head.html**

Copy the `<head>` block from a live essay page and replace specific values with `[REPLACE]` placeholders:

```html
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta name="color-scheme" content="dark">
    <title>[TITLE] — Igor Schwarzmann</title>
    <meta name="description" content="[DESCRIPTION — 150-160 characters]">

    <!-- Open Graph -->
    <meta property="og:title" content="[TITLE] — Igor Schwarzmann">
    <meta property="og:description" content="[OG_DESCRIPTION — shorter OK]">
    <meta property="og:type" content="article">
    <meta property="og:url" content="https://igorschwarzmann.com/strategyasprotocol/[SLUG]/">

    <!-- Twitter Card -->
    <meta name="twitter:card" content="summary">
    <meta name="twitter:title" content="[TITLE] — Igor Schwarzmann">
    <meta name="twitter:description" content="[TWITTER_DESCRIPTION]">

    <!-- Canonical -->
    <link rel="canonical" href="https://igorschwarzmann.com/strategyasprotocol/[SLUG]/">

    <!-- Structured Data -->
    <script type="application/ld+json">
    {
      "@context": "https://schema.org",
      "@graph": [
        {
          "@type": "Article",
          "@id": "https://igorschwarzmann.com/strategyasprotocol/[SLUG]/#article",
          "headline": "[TITLE]",
          "description": "[DESCRIPTION]",
          "author": { "@id": "https://igorschwarzmann.com/#person" },
          "publisher": { "@id": "https://igorschwarzmann.com/#org" },
          "datePublished": "[YYYY-MM-DD]",
          "dateModified": "[YYYY-MM-DD]",
          "isPartOf": { "@id": "https://igorschwarzmann.com/strategyasprotocol/#collection" },
          "mainEntityOfPage": "https://igorschwarzmann.com/strategyasprotocol/[SLUG]/"
        },
        {
          "@type": "BreadcrumbList",
          "itemListElement": [
            { "@type": "ListItem", "position": 1, "name": "Home", "item": "https://igorschwarzmann.com/" },
            { "@type": "ListItem", "position": 2, "name": "Strategy-as-Protocol", "item": "https://igorschwarzmann.com/strategyasprotocol/" },
            { "@type": "ListItem", "position": 3, "name": "[BREADCRUMB_NAME]", "item": "https://igorschwarzmann.com/strategyasprotocol/[SLUG]/" }
          ]
        }
      ]
    }
    </script>

    <!-- Fonts: Personal Mode stack -->
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Syne:wght@600;700;800&family=IBM+Plex+Sans:ital,wght@0,300;0,400;0,500;0,600;1,300;1,400;1,500&family=IBM+Plex+Mono:wght@400;500&display=swap" rel="stylesheet">
</head>
```

Add a comment block at the top explaining the placeholders:

```html
<!--
  Essay <head> template for igorschwarzmann.com
  Replace all [BRACKETED] values before use:
  - [TITLE]: Essay title (e.g., "Strategy-as-Protocol")
  - [SLUG]: URL slug (e.g., "strategy-as-protocol")
  - [DESCRIPTION]: 150-160 char meta description
  - [OG_DESCRIPTION]: Shorter version for social cards
  - [TWITTER_DESCRIPTION]: Can match OG description
  - [YYYY-MM-DD]: Publication and modification dates
  - [BREADCRUMB_NAME]: Short name for breadcrumb trail
-->
```

**Step 2: Verify template is valid HTML**

Read the file back and verify all placeholders are consistently named.

---

### Task 8: Write templates/hub-head.html

**Files:**
- Create: `~/.claude/skills/igorschwarzmann-design/templates/hub-head.html`
- Source: Live `strategyasprotocol/index.html` lines 1-60

**Step 1: Write hub-head.html**

Same pattern as essay-head.html but with CollectionPage schema instead of Article. Key differences:
- `og:type` is `"article"` (not `"website"` — we changed this in the SEO fix)
- JSON-LD uses `CollectionPage` with `hasPart` array listing child articles
- BreadcrumbList is 2 levels (Home → Hub) not 3

Include `[REPLACE]` placeholders and a comment block explaining them.

**Step 2: Verify template is valid HTML**

Read the file back and verify all placeholders are consistently named.

---

### Task 9: Trim Obsidian main design system doc

**Files:**
- Modify: `brain dead/2. Areas/igorschwarzmann.com/Design System igorschwarzmann.com.md`

**Step 1: Read current file to understand full content**

The main design system doc contains both rationale and implementation specs. We need to identify and remove:
- All CSS code blocks (move to skill)
- Type scale tables with exact values (move to skill)
- Layout dimension specs (move to skill)
- Diagram CSS implementations (move to skill)
- Application guidelines with CSS (move to skill)
- The SEO Section 10 we just added (move to skill)

**Step 2: Keep these sections (rationale):**
- Section 1: Brand Foundation (core concept, design principles, voice & tone)
- Section 2: Color System — keep semantic descriptions, remove CSS snippets
- Section 3: Typography — keep font pairing reasoning ("why Syne?"), remove exact size tables
- Section 4: Layout Patterns — keep pattern descriptions and ASCII diagrams, remove CSS
- Section 5: Diagram System — keep diagram type descriptions, remove CSS implementations
- Section 6: Transformation patterns — keep hierarchy rules, remove CSS
- Section 7: Animation Principles — keep "when to animate" guidelines, remove CSS
- Section 8: Image treatment — keep role descriptions and guidelines, remove CSS
- Section 9: Application Guidelines — keep web slides structure notes, remove all CSS

**Step 3: Add migration header**

Add after the YAML frontmatter:

```markdown
> **Note:** Implementation specs (CSS, exact values, checklists) have moved to the
> `igorschwarzmann-design` Claude Code skill. This document contains design rationale
> and decision history only.
```

**Step 4: Verify line count**

Target: Under 500 lines (currently ~1,253 lines)

**Step 5: Commit**

Not a git repo — changes are in Obsidian vault. No commit needed.

---

### Task 10: Trim Obsidian Essay Mode doc

**Files:**
- Modify: `brain dead/2. Areas/igorschwarzmann.com/Design System igorschwarzmann.com - Essay Mode.md`

**Step 1: Read current file**

**Step 2: Keep these sections (rationale):**
- Opening description ("A reading-optimized mode...")
- Layout Architecture — keep ASCII diagram and "How It Works" visualization, remove CSS
- The comparison table (Three Modes) at the bottom
- Related Documents links
- The "principle" statement ("spear vs carrier bag")

**Step 3: Remove these sections (moved to skill):**
- All CSS code blocks
- CSS Design Tokens block (lines 337-378)
- Type hierarchy table with exact values
- Component CSS specs (blockquote, link, progress bar)
- Figure CSS specs
- Template workflow (replaced by skill routing)

**Step 4: Add migration header**

Same note as main doc.

**Step 5: Verify line count**

Target: Under 150 lines (currently ~416 lines)

---

### Task 11: Trim Obsidian Personal Mode doc

**Files:**
- Modify: `brain dead/2. Areas/igorschwarzmann.com/Design System igorschwarzmann.com - Personal Mode.md`

**Step 1: Read current file**

**Step 2: Keep these sections (rationale):**
- Opening description ("A typography-focused fork...")
- "Why These Fonts?" section (rationale for Syne, IBM Plex Sans, IBM Plex Mono)
- "When to Use Each Mode" section
- Comparison table (Working Mode vs Personal Mode)
- The "3 seconds vs 30 minutes" rule
- Color System note (unchanged from main)

**Step 3: Remove these sections (moved to skill):**
- All CSS code blocks
- Type scale CSS variables
- Hero Statement HTML example
- Bold Statement Strip HTML example
- Micro-Interactions CSS
- Atmospheric Depth CSS
- Responsive Strategy exact breakpoints
- Animation CSS (marquee keyframes)
- Accessibility CSS (reduced-motion)
- Implementation Checklist
- Google Fonts implementation snippet

**Step 4: Add migration header**

Same note as main doc.

**Step 5: Verify line count**

Target: Under 150 lines (currently ~507 lines)

---

### Task 12: Update area CLAUDE.md

**Files:**
- Modify: `brain dead/2. Areas/igorschwarzmann.com/.claude/CLAUDE.md`

**Step 1: Read current file**

**Step 2: Update the Design System section**

Replace the current:
```markdown
## Design System

**Read before any design or styling work**: [Design System.md](../Design%20System.md)

Contains: color palette (Volt/Signal/Shift), typography scale, spacing system, semantic color usage, layout patterns, and design principles. All visual decisions should align with this system.
```

With:
```markdown
## Design System

Implementation specs are in the `igorschwarzmann-design` Claude Code skill (auto-triggers when working on site files).

For design rationale and decision history, see:
- [Design System.md](../Design%20System%20igorschwarzmann.com.md) — Brand foundation, principles, pattern descriptions
- [Essay Mode.md](../Design%20System%20igorschwarzmann.com%20-%20Essay%20Mode.md) — Essay mode rationale
- [Personal Mode.md](../Design%20System%20igorschwarzmann.com%20-%20Personal%20Mode.md) — Personal mode rationale
```

**Step 3: Verify the change**

Read the file back to confirm the update.

---

### Task 13: Verify skill triggers correctly

**Step 1: Test skill discovery**

In a new Claude Code conversation, work on an igorschwarzmann.com file and verify the skill is discovered and loaded.

**Step 2: Test progressive disclosure**

Ask Claude to "check the essay mode typography specs" and verify it loads `reference/essay-mode.md` (not the full Obsidian doc).

**Step 3: Test template usage**

Ask Claude to "create a new essay page" and verify it references `templates/essay-head.html`.

---

## Execution Order

Tasks 1-8 (skill creation) are independent of Tasks 9-12 (Obsidian migration). Build the skill first, then trim Obsidian docs.

```
Task 1 (dirs) → Task 2 (SKILL.md)
                    ↓
         Tasks 3-6 (reference files, parallel)
                    ↓
         Tasks 7-8 (templates, parallel)
                    ↓
         Tasks 9-11 (trim Obsidian docs, parallel)
                    ↓
         Task 12 (update CLAUDE.md)
                    ↓
         Task 13 (verification)
```

## Success Criteria

- [ ] `~/.claude/skills/igorschwarzmann-design/SKILL.md` exists and is <200 lines
- [ ] All 4 reference files exist and are <160 lines each
- [ ] Both templates have valid HTML with consistent `[REPLACE]` placeholders
- [ ] Obsidian docs are each under 500 lines (main) / 150 lines (modes)
- [ ] Area CLAUDE.md points to skill instead of Obsidian docs
- [ ] Skill triggers when editing igorschwarzmann.com files

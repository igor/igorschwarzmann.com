# Design: Convert Design System to Claude Code Skill

**Date:** 2026-02-13
**Status:** Approved
**Scope:** igorschwarzmann.com website only (presentations deferred)

## Problem

The design system documentation for igorschwarzmann.com spans 2,100+ lines across three Obsidian files. The primary consumer is Claude Code, but the docs are optimized for human browsing. Pain points:

1. **Hard to find info** — Things are duplicated or in unexpected places across three files
2. **Poor enforcement** — Pages ship without following the system because docs are too long to be actionable
3. **No progressive disclosure** — Claude loads everything or nothing

## Decision

Convert the design system into a Claude Code skill with atomized reference files. The Obsidian docs are trimmed to design rationale only.

### Alternatives Considered

| Approach | Verdict |
|----------|---------|
| **A: Full Skill** | **Selected** — auto-triggers, progressive disclosure, machine-optimized |
| B: Atomize in Obsidian | Rejected — no auto-triggering, no progressive disclosure |
| C: Thin Skill + Obsidian | Rejected — fragile paths, two things to maintain, no format optimization |

## Skill Architecture

```
~/.claude/skills/igorschwarzmann-design/
├── SKILL.md                  # Always loaded: routing, tokens, universal rules (~200 lines)
├── reference/
│   ├── personal-mode.md      # Landing pages: Syne/Plex stack, hero layout, hover, responsive (~120 lines)
│   ├── essay-mode.md         # Essays: two-zone layout, figures, blockquotes, progress bar (~150 lines)
│   ├── seo.md                # Meta tags, JSON-LD templates, new-page checklist (~100 lines)
│   └── components.md         # Nav, footer, bio, progress bar specs (~80 lines)
└── templates/
    ├── essay-head.html       # Copy-pastable <head> with [REPLACE] placeholders
    └── hub-head.html         # Copy-pastable <head> for collection pages
```

### SKILL.md Content (~200 lines)

**Part 1 — Mode Detection Table**

| If working on... | Mode | Load these references |
|---|---|---|
| `index.html` (root) | Personal | `personal-mode.md` + `seo.md` |
| `strategyasprotocol/index.html` | Hub | `essay-mode.md` + `seo.md` |
| `strategyasprotocol/*/index.html` | Essay | `essay-mode.md` + `seo.md` |
| `strategyasprotocol/designsystems/` | Presentation | `components.md` + `seo.md` |
| Any new page | Start here | mode-specific + `seo.md` |

**Part 2 — Design Tokens (Inline)**

All design tokens are embedded directly in SKILL.md (~20 lines). This means simple tasks (color tweaks, spacing adjustments) need zero reference file reads.

Contents:
- 6 colors with one-line semantic meanings
- 3 font families with roles
- 5 spacing values
- 2 background systems (dark #0A0A0A / light #FFFFFF, #F5F0EB)

**Part 3 — Universal Rules**
- All URLs use trailing slashes
- All images need alt text and `loading="lazy"`
- All pages need the SEO `<head>` block (see `seo.md`)
- `prefers-reduced-motion` must be respected
- Single-file HTML pages with embedded CSS/JS (no build system)

### Reference Files

Each file contains implementation specs only — tables, CSS snippets, checklists. No prose rationale.

**`personal-mode.md`** (~120 lines)
- Font stack: Syne (display) + IBM Plex Sans (body) + IBM Plex Mono (accent)
- Type hierarchy table (hero through caption, with sizes/weights/line-heights)
- Hero layout pattern (name top-left, contact top-right, statements center)
- Atmospheric depth (gradient + grain CSS snippets)
- Hover interactions (statement color shifts, button hover)
- Responsive breakpoints (desktop/tablet/mobile type scaling)
- Implementation checklist

**`essay-mode.md`** (~150 lines)
- Same font stack as Personal but different sizing/spacing
- Two-zone layout rules (600px text + 60px gap + 360px margin)
- Responsive collapse table (4 breakpoints with widths)
- Figure placement rules (float right, clear right, placement before reference paragraph)
- Figure variants (default, book cover, wide)
- Component specs: nav bar, essay header, intro block, blockquotes, links, progress bar, footer
- Implementation checklist

**`seo.md`** (~100 lines)
- Required meta tags table (canonical, OG, Twitter Card)
- JSON-LD Article template (copy-pastable)
- JSON-LD CollectionPage template (copy-pastable)
- BreadcrumbList template
- Cross-reference rules (@id entities, trailing slashes, dates)
- New page checklist (sitemap, llms.txt, internal linking)

**`components.md`** (~80 lines)
- Site nav (top bar): name left, section right, IBM Plex Mono 11px uppercase
- Site footer: copyright left, email right, monospace, tertiary
- Reading progress bar: 2px Volt, fixed top
- Author bio footer: IBM Plex Sans 16px italic, CTA link

### Templates

Two complete `<head>` blocks with `[REPLACE]` placeholder markers:
- `essay-head.html` — for new Strategy-as-Protocol essays
- `hub-head.html` — for new collection/hub pages

### Triggering

**Skill description:** "Design system for igorschwarzmann.com. Use when creating or modifying pages on the personal website, implementing visual design, or checking design system compliance. Covers three modes: Personal (landing pages), Essay (long-form articles), and Presentation (slide decks)."

## Migration Plan

### Obsidian Docs

After the skill is built:
1. Extract implementation specs from all three Obsidian docs into skill reference files
2. Trim Obsidian docs to design rationale + decision history only
3. Add header note to each: *"Implementation specs have moved to the `igorschwarzmann-design` Claude Code skill. This document contains design rationale and decision history."*
4. Update area CLAUDE.md to remove "read design system before any design work" directive

### What Stays in Obsidian

- Brand foundation / core concept ("THE NEW")
- Font pairing reasoning (why Syne, why Plex)
- Design principles (clarity through structure, color as meaning, etc.)
- Mode comparison tables (when to use which mode and why)
- Creative direction notes
- Session notes and development log

### What Moves to Skill

- All CSS code snippets
- Type hierarchy tables with exact values
- Layout dimension specs
- Color hex values and usage rules
- Component specs
- Responsive breakpoint rules
- Checklists
- SEO requirements

## Success Criteria

- Claude auto-loads the skill when working on igorschwarzmann.com files
- Simple modifications (color change, spacing tweak) require zero reference file reads
- New essay creation follows a clear path: SKILL.md routing → essay-mode.md + seo.md → templates
- No page ships without SEO metadata
- Obsidian docs are under 500 lines each (rationale only)

<![CDATA[<div align="center">

# pixel-perfect-audit

**Manual design audit skill for Claude Code — verify live web apps against a design system / brandbook**

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Agent Skill](https://img.shields.io/badge/Claude_Code-Agent_Skill-orange.svg)](https://agentskills.io)
[![Version](https://img.shields.io/badge/version-1.0.0-green.svg)](pixel-perfect/SKILL.md)
[![Lines](https://img.shields.io/badge/SKILL.md-540_lines-lightgrey.svg)](pixel-perfect/SKILL.md)

</div>

---

## What It Does

Systematically inspects every CSS property of a live web application via Chrome DevTools, compares values against a design system (brandbook), and produces a structured bug report. Catches what automated tools miss — wrong `font-weight` (600 vs 700), colors close but off-palette (#8B9DAD vs #8996A3), dropdown overflow bugs, typography outside the type scale.

## Quick Install

```bash
# Copy the skill to your Claude Code skills directory
cp -r pixel-perfect ~/.claude/skills/
```

Claude Code will auto-discover the skill and trigger it when you mention "pixel-perfect", "design audit", "UI audit", or "brandbook compliance".

## Requirements

| Requirement | Why |
|---|---|
| **Claude Code** | The skill runs as a Claude Code agent skill |
| **Chrome MCP extension** | All measurements use Chrome browser automation tools |
| **Design system / brandbook** | Source of truth — colors, typography, spacing, components |
| **Target URL** | Live web app or staging environment to audit |

## When to Use

| Use this skill | Don't use this skill |
|---|---|
| New build ready for design QA | Automated CI screenshot diffing (use visual regression) |
| Redesign needs brandbook verification | Functional / behavioral testing (use Playwright) |
| Checking implementation matches design specs | CSS performance / bundle audit |
| Finding cross-component inconsistencies | CSS linting or optimization |
| Pre-launch design sign-off | Just need a single screenshot |

## How It Works

The audit runs in **8 phases**:

```
Phase 0  →  Setup: get browser tab, navigate to target, set viewport 1440×900
Phase 1  →  Brandbook: extract colors, typography, spacing from design system
Phase 2  →  Navigation map: discover all pages/states to audit
Phase 3  →  Global audit: headers, footers, shared components
Phase 4  →  Page-by-page audit: measure every element with JS snippets
Phase 5  →  Cross-page consistency: verify shared components match everywhere
Phase 6  →  Report: generate structured Excel/Markdown with screenshots
Phase 7  →  Summary: present severity counts, verdict (SHIP / FIX)
```

Each element is measured using `getComputedStyle()` — font-size, font-weight, line-height, color, background, padding, margin, border-radius, gap, shadows, opacity. Deviations from the brandbook are logged with severity, screenshot, and reproduction path.

## What It Catches (Real Examples)

| Category | Bug | Current | Expected | Severity |
|---|---|---|---|---|
| Typography | Wrong font-weight | `600` | `700` (Bold) | High |
| Typography | Font-size outside type scale | `13px` | `14px` | High |
| Colors | Off-palette color | `#8B9DAD` | `#8996A3` | High |
| Colors | Opacity mismatch | `rgba(0,0,0,0.5)` | `rgba(0,0,0,0.64)` | Medium |
| UX-Bug | Dropdown items hidden on overflow | items cut off | scrollable / visible | Critical |
| Layout | Inconsistent padding | `16px 20px` | `16px 24px` | Medium |
| Consistency | Button style differs across pages | rounded on /home | square on /settings | High |

## Output Format

The default output is a **structured Excel file** (`.xlsx`) with columns:

`#` · `Page / Section` · `Element` · `Property` · `Current Value` · `Expected Value` · `Severity` · `Navigation Path` · `Screenshot` · `Notes`

Alternative formats available: Markdown table, CSV, or Notion page.

The report ends with a summary table:

| Severity | Count |
|---|---|
| Critical | 0 |
| High | 12 |
| Medium | 5 |
| Low | 2 |
| **Verdict** | **FIX → RE-AUDIT** |

## Configuration

The skill auto-configures based on your request. You can customize:

- **Output format** — ask for Excel (default), Markdown, CSV, or Notion
- **Language** — report outputs in your preferred language
- **Viewport** — default 1440×900, adjustable for responsive audits
- **Severity thresholds** — built-in: Critical (functional bugs), High (visible deviations), Medium (minor mismatches), Low (nitpicks)
- **Color tolerance** — per-channel RGB difference ≤3 is auto-dismissed

## Phases in Detail

### Phase 0 — Setup
Get browser tab via `tabs_context_mcp`, navigate to target URL, set viewport to 1440×900 (or custom). Confirm page loads correctly.

### Phase 1 — Brandbook Extraction
Open the design system URL/document. Extract the canonical palette (hex values), typography scale (font-family, sizes, weights, line-heights), spacing system, border-radius values, and shadow definitions.

### Phase 2 — Navigation Map
Discover all pages and interactive states. Build a checklist: main pages, modals, dropdowns, hover states, empty states, error states, loading states.

### Phase 3 — Global Audit
Measure shared components — header, footer, sidebar, navigation. These appear on every page, so deviations here multiply.

### Phase 4 — Page-by-Page Audit
For each page: screenshot → measure every visible element → compare against brandbook → log deviations. Uses batch JS snippets to measure multiple elements in one call.

### Phase 5 — Cross-Page Consistency
Compare the same component across different pages. A button on `/home` should match the button on `/settings`.

### Phase 6 — Report Generation
Compile all findings into the chosen format. Attach screenshots with annotations. Sort by severity.

### Phase 7 — Summary & Verdict
Present totals. Verdict logic:
- **SHIP AS-IS** — 0 Critical, 0 High
- **FIX → RE-AUDIT** — any Critical or High bugs remain

## FAQ

**Can I use this without a brandbook?**
No. The brandbook is the source of truth. Without it, there's nothing to compare against — every measurement would be subjective.

**How long does a full audit take?**
Depends on the number of pages and components. A typical 5-page app takes 15-30 minutes. Complex apps with many states can take longer.

**Does it work with dark mode?**
Yes. Audit each theme separately — switch themes via the app's toggle, then measure. The brandbook should define both light and dark palettes.

**What browsers does it support?**
Chrome only — the skill uses Chrome MCP browser extension for all measurements.

**Can I audit mobile layouts?**
Yes. Set the viewport to a mobile size (e.g., 375×812) in your request. The skill will measure at that viewport.

**What's the color tolerance?**
Per-channel RGB difference of ≤3 is auto-dismissed (e.g., `#8996A3` vs `#8996A4`). Larger deviations are flagged.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## License

[MIT](LICENSE) — use it, modify it, share it.
]]>
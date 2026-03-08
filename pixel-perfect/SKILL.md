---
name: pixel-perfect
description: >
  Manual pixel-perfect design audit — compare live web app CSS against a design system / brandbook.
  TRIGGER when: user mentions "pixel-perfect", "design audit", "UI audit", "brandbook compliance",
  "design QA", "check against design", "design implementation check",
  or wants to verify a live site matches the design system.
  DO NOT TRIGGER when: user wants automated screenshot comparison / visual regression testing,
  functional/behavioral testing, just wants a single screenshot,
  CSS performance/bundle audit, CSS linting, or CSS optimization (those are code-level tasks, not visual design QA).
license: MIT
metadata:
  category: technique
  author: anthropic-community
  version: "1.0.0"
  triggers: pixel-perfect, design audit, UI audit, brandbook compliance, design QA, design check, design implementation, check against design, check against brandbook
---

# Pixel-Perfect Design Audit

Systematic manual audit of a live web application against a design system (brandbook). Measure every CSS property via browser inspection, flag deviations, output a structured Excel report with screenshots and reproduction paths.

**When to use:** A new build, redesign, or feature is ready for design QA. You need to verify that the implementation matches the brandbook — colors, typography, spacing, component consistency. The output is a detailed bug report for the designer/developer.

**Core strength:** Catches what automated tools cannot — wrong font-weight (600 vs 700), colors close but not matching the palette (#8B9DAD vs #8996A3), dropdown overflow bugs, typography outside the type scale, cross-component inconsistencies. Requires a brandbook as source of truth.

> **Not this skill?** For automated CI screenshot comparison, use a visual regression testing skill. For browser interaction testing, use a Playwright-based skill. For a single screenshot, use a screenshots skill.

---

## Tool Quick Reference

Every action in this audit maps to a specific Chrome MCP tool. **This skill requires Chrome MCP browser extension.**

> **`javascript_tool` critical note:** Every call requires three parameters: `action: "javascript_exec"`, `text` (the JS code string), and `tabId`. Omitting `action` will cause the call to fail.

| Action | Tool | Notes |
|--------|------|-------|
| Get browser tab ID | `tabs_context_mcp` | **MUST call first** before any browser tool |
| Create new tab | `tabs_create_mcp` | For opening reference site alongside target |
| Navigate to URL | `navigate` | Requires `url` + `tabId`. Supports `"back"`/`"forward"` |
| Discover page structure | `read_page` | Accessibility tree with `ref` IDs. Params: `tabId` (required), `filter` (`"interactive"`), `depth`, `ref_id` (subtree), `max_chars` (default 50000) |
| Find elements by purpose | `find` | Natural language query + `tabId`. Returns max 20 matches |
| Measure CSS properties | `javascript_tool` | `action: "javascript_exec"` + `text` + `tabId` |
| Extract all page text | `get_page_text` | Quick text inventory for typography audit |
| Take viewport screenshot | `computer` | `action: "screenshot"` + `tabId` (no coordinate needed) |
| Click elements | `computer` | `action: "left_click"` + `coordinate: [x,y]` or `ref` + `tabId` |
| Hover elements | `computer` | `action: "hover"` + `coordinate: [x,y]` or `ref` + `tabId` |
| Scroll to element | `computer` | `action: "scroll_to"` + `ref` from read_page/find + `tabId` |
| Scroll viewport | `computer` | `action: "scroll"` + `coordinate` + `scroll_direction` + `tabId` |
| Wait for transitions | `computer` | `action: "wait"` + `duration` (seconds, max 30) + `tabId` |
| Resize viewport | `resize_window` | `width` + `height` + `tabId` (all required) |
| Read Figma brandbook | `get_design_context` / `get_screenshot` (Figma MCP) | Optional — extract tokens from Figma |
| Generate Excel report | openpyxl via Python script | See Phase 7 for complete template |

---

## Phase 0: Request Prerequisites

**Before any work, ask the user for:**

1. **Design system / Brandbook** — Figma link, PDF, Notion page, or explicit list of tokens:
   - Color palette (hex + names), typography scale (sizes, weights, line-heights, letter-spacing, font-family)
   - Button variants and states, spacing system, border-radii, shadows, component specs

2. **Target URL** — the site or build to audit
   - If localhost/dev server: warn that HMR may invalidate measurements — use a stable build or disable HMR

3. **Reference implementation** (optional) — production site or approved build for cross-validation

4. **Scope** — full audit or specific pages/components? For 7+ pages, suggest splitting into sessions

5. **Device targets** — desktop only? Mobile? Responsive breakpoints? Default: desktop 1920x1080

6. **Known exceptions** — any intentional deviations from brandbook?

7. **Authentication** — if login required: ask user to log in manually. Never enter credentials on behalf of the user

> **Critical:** Do NOT start without a brandbook. If the user cannot provide a formal one, ask for at minimum: brand colors (hex), font family, type scale (sizes + weights). If they cannot provide even this, **decline the audit** — otherwise every finding would be arbitrary opinion, not a verifiable bug.

---

## Phase 1: Browser Setup

### Step 1: Get tab context
Call `tabs_context_mcp` to get available tabs. If no MCP tab group exists, set `createIfEmpty: true`.

### Step 2: Open target site
Use `navigate` with the target URL. If a reference site is provided, use `tabs_create_mcp` for a second tab.

### Step 3: Handle cookie consent / overlays
Dismiss cookie banners before auditing — use `find(query="cookie banner")` or `find(query="accept cookies button")`. Click the reject/dismiss button (prefer privacy-preserving option).

### Step 4: Verify page loaded
Call `read_page` to confirm rendering. If output exceeds limit, reduce `depth` or use `ref_id` to drill into sections.

### Step 5: Handle authentication (if needed)
- Inform user, ask them to log in manually in the browser
- **HTTP Basic Auth** (browser-native popups): cannot be interacted with via MCP — ask user to enter directly
- Never enter credentials on behalf of the user

### SPA / Client-side routing
- Use `navigate` for full URL changes; `computer` (left_click) for internal links (preserves app state)
- After navigation: `read_page` to verify; `computer` (wait, duration: 2) for lazy content

---

## Phase 2: Design Token Extraction

Extract the full token set from the brandbook. Build a quick-reference lookup.

### From Figma
Use Figma MCP. Extract from URL `https://figma.com/design/{fileKey}/{fileName}?node-id={nodeId}` — replace dash with colon in nodeId; no node-id -> use "0:1"; branch URLs -> use branchKey as fileKey.
1. `get_design_context(fileKey, nodeId)` — reference code with tokens
2. `get_screenshot(fileKey, nodeId)` — visual reference
3. Extract hex colors, font sizes, weights, spacing

### From PDF
Use `Read` tool with `pages` parameter for large PDFs. URL PDFs: open in Chrome via `navigate` + `get_page_text`. Image-heavy: ask user for key values directly.

### From Notion
`notion-fetch` with URL/ID -> Markdown with tokens. For databases: `notion-search` -> `notion-fetch` each page.

### Manual specs
Build reference tables from whatever the user provides:

```
Colors:
Name            | Hex
Primary         | #XXXXXX
Text Primary    | #XXXXXX
Text Secondary  | #XXXXXX

Typography Scale:
Style  | Size | Weight | Line-Height | Letter-Spacing
H1     | XXpx | 700    | ...         | ...
Body   | XXpx | 400    | ...         | ...
```

Note buttons, inputs, border-radius, padding per variant. If CSS custom properties are defined (e.g., `--color-primary`), note them for checking: `getComputedStyle(document.documentElement).getPropertyValue('--color-primary')`.

> **Units:** `getComputedStyle()` returns **px** even if source uses rem/em/%. If brandbook uses rem: multiply by root font-size (default 16px).

**Keep these tables accessible throughout the entire audit.**

---

## Phase 3: Site Discovery & Navigation Map

Navigate the entire site and build an exhaustive map.

### Step 1: Map all pages
- Visit every route using `navigate` and internal link clicks
- `read_page` on each page (reduce `depth` if truncated); `filter="interactive"` for buttons/links/inputs
- Identify tabs, sub-views, nested navigation

### Step 2: Identify ALL interactive states

| Element Type | What to Open/Trigger |
|---|---|
| **Dropdowns** | Open each. Scroll to see ALL items (verify count via JS) |
| **Modals** | Open each. Test ALL internal dropdowns and forms |
| **Expandable sections** | Click "show more", accordions, collapsible cards |
| **Tooltips** | Hover over every info icon, truncated text |
| **Form states** | Empty, filled, error, success, disabled |
| **Hover/Focus/Active** | Buttons, links, cards, inputs — check visual changes |
| **Loading / Empty** | Skeleton and empty data views if accessible |
| **Themes / Dark mode** | Detect: `window.matchMedia('(prefers-color-scheme: dark)').matches` or UI toggle. Each theme = separate audit pass |

### Step 3: Create audit checklist
```
- Page A: default, card expanded, settings modal, tooltip
- Page B: default, period dropdown, export modal
```

For 5+ page sites, maintain a progress log. If context runs low, summarize completed pages before continuing.

**Session handoff:** When splitting across sessions, export: completed pages + bug counts, component registry, systemic issues, design token tables.

> **CRITICAL: Never assume visible items are all items.** Always verify via JS: `scrollHeight > clientHeight`, `querySelectorAll().length`.

---

## Phase 4: Component Grouping

Identify repeating UI patterns. Build a component registry:

```
Component  | Variant   | Expected Styles (from brandbook)
Button     | Primary   | bg: #XXXX, radius: 12px, weight: 700
Button     | Secondary | bg: transparent, border: 1px #XXXX
Card       | Default   | bg: ?, radius: ?, padding: ?
Text       | Heading   | size: 36px, weight: 700, color: #XXXX
Text       | Body      | size: 16px, weight: 400, color: #XXXX
Input      | Default   | bg: ?, border: ?, radius: ?
Dropdown   | Trigger   | weight may differ from options (standard UX)
Modal      | Container | bg: ?, radius: ?, padding: ?
```

For `?` values, measure one canonical instance during Phase 5 — use as expected value for cross-page consistency.

**Purpose:** Components within the same group MUST have identical styling. Any inconsistency = bug.

---

## Phase 5: Systematic Audit

Process one page at a time.

### 5.1 Element discovery
Use `read_page` for the accessibility tree. Use `find` to locate elements by purpose (max 20 per call). Use `computer` (scroll_to) to reach elements below the fold.

### 5.2 Take viewport screenshot
`computer` (action: screenshot) captures **visible viewport only**. Scroll and take multiple screenshots for long pages. Screenshots live in conversation context — cannot be saved to disk or embedded in Excel.

### 5.3 Typography audit (most common issue source)
For every text element, measure via `javascript_tool`:
- `fontSize` — must match type scale
- `fontWeight` — watch for 600 vs 700!
- `fontFamily` — strip quotes and fallbacks for comparison
- `lineHeight`, `letterSpacing`, `color`

**Font loading:** Verify before measuring: `document.fonts.check('1px "Inter"')` (any pixel size — checks availability, not size). Wait via `document.fonts.ready` if needed.

### 5.4 Color audit
- Convert rgb/rgba to hex (see Techniques section)
- Compare against palette; flag any hex not in brandbook
- Check alpha/opacity separately
- **Gradients:** check `backgroundImage` for `linear-gradient()` — won't appear in `backgroundColor`
- Check CSS custom properties vs hardcoded values

### 5.5 Spacing & layout
- **Padding/Margin:** use individual properties (`paddingTop`, etc.) — shorthand may return empty when sides differ
- **Gap:** `gap`, `rowGap`, `columnGap` for flex/grid
- **Alignment:** `textAlign`, `justifyContent`, `alignItems`
- **Width constraints:** `maxWidth`, `minWidth`

### 5.6 Interactive state audit
- **Open each** dropdown/modal/tooltip via `computer` (left_click)
- **Hover:** `computer` (hover, coordinate), then immediately screenshot/measure
- **Focus:** `computer` (left_click) on inputs — check outline, border changes
- **Wait for transitions:** `computer` (wait, duration: 1) before measuring if CSS transitions exist
- **Dropdowns in modals:** check if items overflow beyond modal boundary (see Techniques)
- **Scrollable containers:** always verify ALL items via JS count

### 5.7 Responsive audit (if specified in Phase 0)
- `resize_window(width=BREAKPOINT, height=HEIGHT, tabId=TAB_ID)` per breakpoint
- Re-audit key elements; check text overflow, element overlap
- Note: minimum window width ~500px on macOS; verify with `window.innerWidth`

### 5.8 Consistency cross-check
Compare against Component Registry from Phase 4: same component on different pages = same styles? Same button type = same padding, font, border-radius?

---

## Phase 6: Verification & Cleanup

### Remove false positives
| Type | Rule |
|------|------|
| **Imperceptible color** | Per-channel decimal difference <= 3 (each R,G,B in 0-255) -> dismiss |
| **Standard UX pattern** | Dropdown trigger bold, options regular -> by design |
| **Duplicate** | Same systemic issue -> remove, reference systemic |
| **Matches production** | Production has same value -> likely intentional |

**Color threshold detail:** Compare R, G, B independently as decimals (0-255). Example: `#8B9DAD` vs `#8996A3` — R: 139 vs 137 (2), G: 157 vs 150 (7), B: 173 vs 163 (10) — G and B exceed 3, this IS a bug. Also compare alpha separately.

### Group systemic issues
If a bug repeats across 3+ places, consolidate: "font-weight: 600 everywhere instead of 700" = 1 systemic issue. Note occurrences as instances, not separate bugs.

### Severity classification

| Severity | Criteria | Examples |
|----------|----------|---------|
| **Critical** | Functional impact, content hidden/inaccessible | Dropdown items hidden, text clipped, layout collapses |
| **High** | Clear spec violation, visible to users | Wrong font-weight on headings, wrong brand color |
| **Medium** | Minor deviation, noticeable on comparison | Secondary color off by >3/channel, padding 20px vs 24px |
| **Low** | Cosmetic, only visible with tools | Letter-spacing -0.5% vs -1%, border-radius 11px vs 12px |

Renumber sequentially after removals (1..N, no gaps).

---

## Phase 7: Documentation

Output in the user's preferred language. Column headers may stay English for developer compatibility.

### Excel report

Write Python to a temp file and execute. `openpyxl` is auto-installed if missing.

> **Output:** `/tmp/pixel-perfect-audit.xlsx`. Inform user of path and suggest moving to a permanent location after delivery. Alternative formats: **CSV** (`csv.writer`), **Markdown** (in conversation for <20 bugs).

```python
# Write to /tmp/generate_audit_report.py, then run: python3 /tmp/generate_audit_report.py
import subprocess, sys
from datetime import datetime
try:
    import openpyxl
except ImportError:
    subprocess.check_call([sys.executable, '-m', 'pip', 'install', '-q', 'openpyxl'])
    import openpyxl

from openpyxl.styles import PatternFill, Font, Alignment
from collections import Counter

wb = openpyxl.Workbook()
ws = wb.active
ws.title = 'Pixel-Perfect Audit'

headers = ['N', 'Page/Section', 'Element', 'Issue', 'Category',
           'Severity', 'Current Value', 'Expected Value', 'Screenshot', 'Route']
ws.append(headers)

for cell in ws[1]:
    cell.font = Font(bold=True)
    cell.alignment = Alignment(horizontal='center')

severity_fills = {
    'Critical': PatternFill(start_color='FF4444', end_color='FF4444', fill_type='solid'),
    'High': PatternFill(start_color='FF8800', end_color='FF8800', fill_type='solid'),
    'Medium': PatternFill(start_color='FFCC00', end_color='FFCC00', fill_type='solid'),
    'Low': PatternFill(start_color='88CC44', end_color='88CC44', fill_type='solid'),
}

# Bug data — fill from audit results
bugs = [
    # [1, 'Page/Section', 'Element', 'Issue', 'Category', 'Severity', 'Current', 'Expected', 'Screenshot desc', 'Route']
]

for bug in bugs:
    ws.append(bug)
    row = ws.max_row
    severity = bug[5]
    if severity in severity_fills:
        for cell in ws[row]:
            cell.fill = severity_fills[severity]

for col in ws.columns:
    max_len = max(len(str(cell.value or '')) for cell in col)
    ws.column_dimensions[col[0].column_letter].width = min(max_len + 2, 50)

# Summary sheet
ws2 = wb.create_sheet('Summary')
ws2.append(['Audit Date', datetime.now().strftime('%Y-%m-%d')])
ws2.append(['Target URL', 'https://...'])  # Replace with actual URL
ws2.append(['Reference URL', 'https://...'])  # Replace if applicable
ws2.append([])

ws2.append(['Severity', 'Count'])
sev_counts = Counter(bug[5] for bug in bugs)
for sev in ['Critical', 'High', 'Medium', 'Low']:
    ws2.append([sev, sev_counts.get(sev, 0)])
ws2.append(['Total', len(bugs)])
ws2.append([])

ws2.append(['Category', 'Count'])
cat_counts = Counter(bug[4] for bug in bugs)
for cat, count in cat_counts.most_common():
    ws2.append([cat, count])

output_path = '/tmp/pixel-perfect-audit.xlsx'
wb.save(output_path)
print(f'Report saved to {output_path}')
```

### Screenshots
Take `computer` (screenshot) for every visually evident bug. For hover states: hover first, then immediately screenshot. The Excel "Screenshot" column contains a text description (images live in conversation context only).

---

## CSS Measurement Techniques

### RGB/RGBA to Hex conversion

`getComputedStyle` returns colors as `rgb()`/`rgba()`. Convert for comparison:

```javascript
function rgbToHex(val) {
  if (!val || val === 'transparent') return 'transparent';
  if (!val.startsWith('rgb')) return val; // pass through hsl(), oklch(), color() as-is
  const match = val.match(/[\d.]+/g);
  if (!match || match.length < 3) return val;
  const [r, g, b] = match.map(Number);
  const hex = '#' + [r, g, b].map(x => Math.round(x).toString(16).padStart(2, '0')).join('').toUpperCase();
  const alpha = match.length >= 4 ? parseFloat(match[3]) : 1;
  return alpha < 1 ? hex + ' (alpha: ' + alpha + ')' : hex;
}
```

> Most browsers compute colors to `rgb()`/`rgba()` even if source uses `hsl()` or `oklch()`. The guard handles edge cases.

### Measure elements (primary technique)

Define `rgbToHex` once, then use `measure()` for any number of elements. **Use selectors from `read_page`/`find`** — CSS-in-JS hashed classes (`.sc-bdVTJa`) are unstable; prefer `[data-testid]`, `[role]`, semantic tags:

```javascript
const rgbToHex = (val) => {
  if (!val || val === 'transparent') return 'transparent';
  if (!val.startsWith('rgb')) return val;
  const m = val.match(/[\d.]+/g);
  if (!m || m.length < 3) return val;
  const hex = '#' + m.slice(0,3).map(x => Math.round(+x).toString(16).padStart(2,'0')).join('').toUpperCase();
  const a = m.length >= 4 ? parseFloat(m[3]) : 1;
  return a < 1 ? hex + ' (a:' + a + ')' : hex;
};
const measure = (sel, label) => {
  const el = document.querySelector(sel);
  if (!el) return { label, error: 'not found' };
  const s = getComputedStyle(el);
  const ff = s.fontFamily.split(',')[0];
  return {
    label, fontSize: s.fontSize, fontWeight: s.fontWeight,
    fontFamily: ff ? ff.trim().replace(/['"]/g, '') : s.fontFamily,
    lineHeight: s.lineHeight, letterSpacing: s.letterSpacing,
    color: rgbToHex(s.color), bg: rgbToHex(s.backgroundColor),
    padding: s.padding || [s.paddingTop, s.paddingRight, s.paddingBottom, s.paddingLeft].join(' '),
    margin: s.margin || [s.marginTop, s.marginRight, s.marginBottom, s.marginLeft].join(' '),
    gap: s.gap, borderRadius: s.borderRadius, border: s.border,
    boxShadow: s.boxShadow, opacity: s.opacity, backgroundImage: s.backgroundImage
  };
};
// Replace with ACTUAL selectors from read_page/find:
JSON.stringify([
  measure('[data-testid="page-title"]', 'Page title'),
  measure('h2.subtitle', 'Subtitle'),
  measure('button[type="submit"]', 'Primary button')
])
```

> **Notes:** `getComputedStyle()` returns px (even if source uses rem/em). Cache result in `const s` — never call multiple times on same element. `padding`/`margin` shorthand may return empty when sides differ — fallback reads individual sides. `backgroundImage` captures gradients that won't appear in `backgroundColor`.

### Specialized patterns

**Form/input values:**
```javascript
const el = document.querySelector('input[name="fieldname"]');
el ? JSON.stringify({ value: el.value, placeholder: el.placeholder, disabled: el.disabled, type: el.type })
   : JSON.stringify({ error: 'Input not found' })
```

**Pseudo-elements (::before/::after):**
```javascript
const el = document.querySelector('SELECTOR');
el ? JSON.stringify({
    beforeContent: getComputedStyle(el, '::before').content,
    beforeColor: getComputedStyle(el, '::before').color,
    afterContent: getComputedStyle(el, '::after').content
  }) : JSON.stringify({ error: 'not found' })
```

**Dropdown items (verify count):**
```javascript
const c = document.querySelector('DROPDOWN_SELECTOR');
c ? JSON.stringify({
    totalItems: c.querySelectorAll('li, [role="option"], [role="menuitem"]').length,
    texts: [...c.querySelectorAll('li, [role="option"], [role="menuitem"]')].slice(0, 20).map(i => i.textContent.trim()),
    scrollable: (() => { const s = c.querySelector('ul, [role="listbox"]'); return s ? s.scrollHeight > s.clientHeight : false; })()
  }) : JSON.stringify({ error: 'not found' })
```

**Modal overflow check:**
```javascript
const dd = document.querySelector('.dropdown-container');
const modal = document.querySelector('[role="dialog"]');
(dd && modal) ? JSON.stringify({
    overflows: dd.getBoundingClientRect().bottom > modal.getBoundingClientRect().bottom,
    hiddenPx: Math.max(0, Math.round(dd.getBoundingClientRect().bottom - modal.getBoundingClientRect().bottom))
  }) : JSON.stringify({ error: 'Elements not found' })
```

### Element discovery
Use `find` before writing selectors:
```
find(query="period selector dropdown", tabId=TAB_ID)
find(query="submit button", tabId=TAB_ID)
```
Then `read_page` with `ref_id` to drill into subtrees, or `javascript_tool` with a derived selector.

---

## Known Gotchas

### Typography & Fonts
1. **Font-weight 600 vs 700** — visually near-identical. Always check computed value; if brandbook says 700, then 600 is a bug.
2. **fontFamily comparison** — returns full stack with quotes. Strip: `.split(',')[0].trim().replace(/['"]/g, '')`.
3. **Font loading (FOUT)** — fonts load async. Check `document.fonts.check('1px "Inter"')` before measuring. Wait via `document.fonts.ready`.
4. **line-height: normal** — browsers compute as ~1.2x font-size. If brandbook specifies explicit values, `normal` is a deviation (usually low severity).
5. **Text overflow** — `text-overflow: ellipsis` + `overflow: hidden` clips text. Detect: `el.scrollWidth > el.clientWidth`.

### Colors & Visual
6. **Colors close but wrong** — dismiss only when ALL R,G,B channels differ by <= 3 (decimal 0-255). Compare alpha separately.
7. **SVG currentColor** — `fill="currentColor"` inherits parent's CSS `color`. Measure parent instead.
8. **CSS custom properties** — correct computed values may use wrong variable references. Check both when possible.

### Interactive Elements
9. **Dropdown trigger vs options** — trigger may be bold, options regular. Standard UX — don't flag unless brandbook specifies otherwise.
10. **Modal dropdown overflow** — `position: absolute` dropdown inside `overflow: hidden` modal -> items invisible. Always check `dropdown.bottom > modal.bottom`.
11. **Dropdown item count** — never trust visible count. Containers often have `maxHeight` + `overflow: auto`. Always count via JS.
12. **CSS transitions** — wait (`computer` wait, duration: 1) after hover/click before measuring to avoid intermediate values.

### DOM & Rendering
13. **SVG className** — returns `SVGAnimatedString`, not string. Use `el.getAttribute('class')` when iterating.
14. **Shadow DOM** — `querySelector` can't pierce shadow roots. Use `el.shadowRoot.querySelector('inner')` (only works for `mode: "open"`; `mode: "closed"` is inaccessible). Check: `el.shadowRoot !== null`.
15. **Canvas/WebGL** — elements inside `<canvas>` can't be measured via CSS. Note: "canvas-rendered — not auditable."
16. **CSS-in-JS hashed classes** — unstable across builds. Prefer: `[data-testid]`, `[role]`, semantic tags, `nth-child`.
17. **Iframes** — `javascript_tool` operates main frame only. Content inside iframes not measurable.

### Layout & Viewport
18. **Viewport screenshots** — `computer` (screenshot) captures visible viewport only. Scroll + take multiple for long pages.
19. **Sticky/fixed elements** — persist across scroll, may overlap content. Audit separately, check z-index.
20. **padding/margin shorthand** — may return empty when sides differ. Fall back to `paddingTop`/`Right`/`Bottom`/`Left`.
21. **Production cross-reference** — if production has same "bug", likely intentional. Lower severity or ask user.

---

## Critical Rules

### MUST DO
- Initialize browser via `tabs_context_mcp` BEFORE any inspection
- Pass `action: "javascript_exec"` in every `javascript_tool` call
- Cache `getComputedStyle(el)` — never call multiple times on same element
- Use `read_page` as PRIMARY element discovery tool on every page
- Measure EVERY value from live CSS — never assume
- Open EVERY dropdown, modal, tooltip, expandable on every page
- Verify dropdown item counts via JS — never trust visual count
- Cross-reference with production site when available
- Document navigation path to reproduce each bug
- Group systemic issues — don't report the same pattern 15 times
- Clean up false positives before delivering the report
- Ask the user for the brandbook BEFORE starting

### NEVER DO
- Never start without a design system reference
- Never flag color differences where ALL channels differ by <= 3 (decimal 0-255)
- Never flag standard UX patterns as bugs without brandbook evidence
- Never skip a page, dropdown, modal, or interactive state
- Never duplicate bugs — reference systemic issues instead
- Never deliver a report without a verification/cleanup pass
- Never enter passwords or credentials on behalf of the user

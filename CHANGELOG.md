# Changelog

All notable changes to this project are documented here. Format follows [Keep a Changelog](https://keepachangelog.com/).

## [1.3.0] - 2026-03-08

### Changed
- **BREAKING: Moved SKILL.md to repo root** — `git clone` now works directly without nested paths. Previous: `pixel-perfect/SKILL.md` → Now: `SKILL.md`
- Excel template aligned to 7-column format (removed N, Screenshot, Route columns)
- Fixed Phase 5.0 JS code: added inline `rgbToHex`, changed from statement to expression (was returning `undefined`)
- Fixed sample report bug #9 wording that contradicted dedup rules
- Fixed README summary table to match sample report data
- Softened FAQ accuracy claim (was absolute "zero errors", now "catches majority")

### Added
- Session handoff protocol in Phase 3 — save/resume audit across sessions via `/tmp/pixel-perfect-handoff.md`
- Batch measurement guidance in Phase 5.1 — reduce tool calls from ~200 to ~30
- Hex normalization step in Phase 2 — prevents case-mismatch false positives
- SPA lazy-load guidance in Phase 3 — scroll to trigger all lazy content before audit
- CHANGELOG.md (this file)
- Versioning scheme in CONTRIBUTING.md
- Testing instructions in CONTRIBUTING.md

### Fixed
- `rgbToHex` undefined in Phase 5.0 mandatory code snippet
- Excel template `bug[5]` index mismatch with 7-column format
- LICENSE year: 2025 → 2025-2026

## [1.2.0] - 2026-03-08

### Added
- **Anti-hallucination protocol** (Phase 5.0) — DOM existence verification + textContent capture before any bug is logged
- **Design intent filter** (Phase 6) — different categories/roles/states having different styles is NOT a bug
- **Context disambiguation** (Phase 5.3) — verify parent context when multiple elements share same text
- **Self-review expanded to 12 points** (Phase 6.5) — DOM existence proof, context verification, design intent check, navigation reproducibility
- Gotchas #13 (same text, different context) and #14 (intentional category differentiation)
- 6 new MUST DO rules (DOM existence, textContent capture, double-measure, design intent filter, 3-level nav, parent context)
- 4 new NEVER DO rules

### Changed
- Strengthened navigation format — minimum 3 levels with URL route and quoted element text
- Strengthened dedup rules — systemic+page overlap, same element different wording
- Updated severity classification with context-dependent examples

## [1.1.0] - 2026-03-08

### Added
- Initial public release
- 9-phase audit workflow (Phase 0–7 + Phase 6.5)
- Chrome MCP tool reference table
- CSS measurement techniques with reusable JS snippets
- Excel report generation via openpyxl
- 12 known gotchas
- Small font scan and color inventory utilities

## [1.0.0] - 2025

- Internal prototype

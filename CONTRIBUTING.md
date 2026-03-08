# Contributing

Thanks for your interest in improving claude-pixel-perfect-agent!

## How to Contribute

### Report a Bug
Open an [issue](../../issues) with:
- What you expected to happen
- What actually happened
- The brandbook / design system you were auditing against (if shareable)
- Your Claude Code version

### Suggest an Improvement
Open an [issue](../../issues) describing:
- The problem or limitation you encountered
- Your proposed solution
- Example use case

### Submit a Pull Request
1. Fork the repo
2. Create a branch (`git checkout -b improve-gotcha-list`)
3. Edit `SKILL.md` (the main skill file at repo root)
4. Test by cloning to `~/.claude/skills/pixel-perfect/` and running an audit
5. Submit a PR with a clear description of what changed and why

### Testing Your Changes
```bash
# Clone your fork into the skills directory
git clone https://github.com/YOUR_USERNAME/claude-pixel-perfect-agent.git ~/.claude/skills/pixel-perfect

# Start a new Claude Code conversation and trigger:
# "Run a pixel-perfect audit on [any site] against [any brandbook]"

# Verify the skill loads and follows your changes
```

### Skill Authoring Guidelines
- Keep SKILL.md concise — every line should earn its place
- Maintain YAML frontmatter with `name`, `description`, `license`, `metadata`
- Every JS snippet must work with Chrome MCP `javascript_tool` (test in browser console first)
- Test gotchas against real-world audits before adding
- Follow semantic versioning: MAJOR.MINOR.PATCH (bump in frontmatter `version` field)

## Versioning

This project follows [Semantic Versioning](https://semver.org/):
- **PATCH** (1.2.x): Bug fixes, typo corrections, minor wording improvements
- **MINOR** (1.x.0): New gotchas, new measurement techniques, new sections
- **MAJOR** (x.0.0): Breaking changes to audit workflow, report format, or phase structure

Update the `version` field in SKILL.md frontmatter and add a CHANGELOG.md entry.

## Code of Conduct

Be respectful, constructive, and helpful. We follow the [Contributor Covenant](https://www.contributor-covenant.org/) v2.1.

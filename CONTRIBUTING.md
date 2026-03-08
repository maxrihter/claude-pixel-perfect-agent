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
3. Edit `pixel-perfect/SKILL.md`
4. Test by copying to `~/.claude/skills/pixel-perfect/` and running an audit
5. Submit a PR with a clear description of what changed and why

### Skill Authoring Guidelines
- Keep SKILL.md concise — every line should earn its place
- Maintain YAML frontmatter with `name`, `description`, `license`, `metadata`
- Every JS snippet must work with Chrome MCP `javascript_tool`
- Test gotchas against real-world audits before adding

## Code of Conduct

Be respectful, constructive, and helpful. We follow the spirit of the [Contributor Covenant](https://www.contributor-covenant.org/).

# Teetsh Skills

Teetsh's own Claude Skills marketplace for specialized tasks.

## Installation

Add this marketplace to Claude Code:

```bash
# In Claude Code, add marketplace
/plugin marketplace add teetsh-org/claude-skills
```

Then install the plugin (skills are included):

```bash
/plugins install review-skills
/plugins install code-review
```

## Available Skills

### frontend-code-review

Teetsh frontend code review for React/TypeScript PRs. Reviews component architecture, React Query, twin.macro styling, i18n, accessibility, responsive design, and testing. Use for PR reviews to ensure adherence to Teetsh patterns.

**Keywords:** react, typescript, code-review, frontend, testing, accessibility

### golang-code-review

Comprehensive Go code review skill for PR reviews, architecture assessment, and test quality analysis. Ensures adherence to Go best practices, security standards, and project-specific patterns.

**Keywords:** go, golang, code-review, security, testing, architecture

## Repository Structure

```
.claude-plugin/
└── marketplace.json       # Marketplace definition
plugins/
└── review-skills/
    ├── .claude-plugin/
    │   └── plugin.json    # Plugin definition
    ├── skills/
    │   ├── frontend-code-review/
    │   │   ├── SKILL.md           # Skill definition
    │   │   └── references/        # Reference docs
    │   └── golang-code-review/
    │       ├── SKILL.md           # Skill definition
    │       └── references/        # Reference docs
    └── commands/          # Slash commands
```

## Contributing

Add new skills by creating PR with:

1. Create skill directory under `plugins/review-skills/skills/<skill-name>/`
2. Add `SKILL.md` with frontmatter (name, description)
3. Include any reference materials in `references/`
4. Update README to list the new skill
5. Skills are auto-discovered, no need to update marketplace.json or plugin.json

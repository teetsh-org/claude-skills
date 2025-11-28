# Teetsh Skills

Community-driven Claude Skills marketplace for specialized tasks.

## Installation

Add this marketplace to Claude Code:

```bash
# In Claude Code, add marketplace
/marketplace add https://github.com/teetsh-org/claude-skills
```

Then install skills:

```bash
/skills install golang-code-review
```

## Available Skills

### golang-code-review
Comprehensive Go code review skill for PR reviews, architecture assessment, and test quality analysis. Ensures adherence to Go best practices, security standards, and project-specific patterns.

**Keywords:** go, golang, code-review, security, testing, architecture

## Repository Structure

```
.claude-plugin/
└── marketplace.json       # Marketplace definition
skills/
├── golang-code-review/
│   ├── SKILL.md           # Skill definition
│   └── references/        # Reference docs
```

## Contributing

Add new skills by creating PR with:
1. Create skill directory under `skills/<skill-name>/`
2. Add `SKILL.md` with frontmatter (name, description)
3. Include any reference materials in `references/`
4. Update `.claude-plugin/marketplace.json` to register the skill

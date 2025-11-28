# Claude Skills Marketplace

Community-driven Claude Skills for specialized tasks.

## Available Skills

### [golang-code-review](./skills/golang-code-review)
Comprehensive Go code review skill for PR reviews, architecture assessment, and test quality analysis. Ensures adherence to Go best practices, security standards, and project-specific patterns.

## Structure

```
skills/
├── golang-code-review/
│   ├── SKILL.md           # Skill definition
│   └── references/        # Reference docs
```

## Installing Skills

Copy skill directory to your Claude Code config:
```bash
cp -r skills/<skill-name> ~/.claude/skills/
```

## Contributing

Add new skills by creating PR with:
- `skills/<skill-name>/SKILL.md` - skill definition with frontmatter
- `skills/<skill-name>/references/` - any reference materials

---
description: Review the current PR/branch diff against base branch for Teetsh frontend patterns
allowed-tools: bash, read, glob, grep
---

# Frontend Code Review for PR

Review the current branch's changes compared to the base branch (main/master).

## Instructions

1. First, determine the base branch and get the merge-base:

   ```bash
   git merge-base HEAD main || git merge-base HEAD master
   ```

2. Get the list of changed files in this PR:

   ```bash
   git diff --name-only $(git merge-base HEAD main) HEAD
   ```

3. Get the full diff for review:

   ```bash
   git diff $(git merge-base HEAD main) HEAD
   ```

4. Read the reference documentation to understand Teetsh patterns:

   - `../../review-skills/skills/frontend-code-review/references/common-mistakes.md` - Common antipatterns to catch
   - `../../review-skills/skills/frontend-code-review/references/react-patterns.md` - Component and hook patterns
   - `../../review-skills/skills/frontend-code-review/references/react-query.md` - Data fetching patterns
   - `../../review-skills/skills/frontend-code-review/references/styling.md` - twin.macro and styling conventions
   - `../../review-skills/skills/frontend-code-review/references/i18n.md` - Internationalization requirements
   - `../../review-skills/skills/frontend-code-review/references/testing.md` - Testing standards

5. For each changed file, review against the patterns. Focus on:
   - React component patterns (container vs component, compound components)
   - React Query usage (query keys, serializers, cache invalidation)
   - Styling (tw macro, theme constants, responsive design)
   - i18n (translations, date formatting)
   - Accessibility (semantic HTML, ARIA, keyboard navigation)
   - Testing (appropriate test types, coverage)

## Review Output Format

Start with a summary of what the PR changes, then provide structured feedback:

### Summary

Brief description of what this PR does based on the diff.

### Critical Issues

Issues that must be fixed before merge (security, broken functionality, accessibility blockers, data loss risks)

### Important Issues

Issues that should be fixed (missing error/loading states, incorrect patterns, i18n gaps, test gaps)

### Suggestions

Optional improvements (code clarity, performance, better patterns)

### Positive Feedback

What was done well in this PR

For each issue, include:

- **File**: Path to the file
- **Problem**: What's wrong and why it matters
- **Current code**: Show the problematic code from the diff
- **Suggested fix**: Show the corrected code
- **Reference**: Link to the relevant pattern documentation

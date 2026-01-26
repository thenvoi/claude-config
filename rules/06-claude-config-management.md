# Claude Code Configuration Management

## File Organization

When adding or updating Claude Code rules:

- **General standards** go in `claude-config` (`.claude/rules/`)
  - Coding standards, conventions, and best practices
  - Error handling patterns
  - Git workflow rules
  - Testing patterns
  - Tool usage guidelines
  - Any rule that applies across multiple Thenvoi repositories

- **Repository-specific rules** go in `CLAUDE.md` at the project root
  - Project description and architecture
  - Specific file paths and commands
  - Environment variables and configuration
  - Project-specific conventions that don't apply elsewhere
  - Build/run/test commands unique to that project

## Decision Guide

Ask yourself: "Would this rule be useful in other Thenvoi repositories?"

- **Yes** → Add to `.claude/rules/` in `claude-config`
- **No** → Add to the project's `CLAUDE.md`

## Updating Rules

- Shared rules: Submit a PR to `claude-config`, then update submodules in target repos
- Local rules: Edit `CLAUDE.md` directly in the target repository

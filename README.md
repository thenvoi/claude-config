# Claude Config

Shared Claude Code rules for Thenvoi repositories.

## Overview

This repository contains shared coding standards and conventions that are loaded by Claude Code across all Thenvoi repositories via git submodules.

## Structure

```
rules/
├── 00-coding-standards.md      # Type hints, logging, imports, Pydantic, async
├── 01-error-handling.md        # ValidationError handling, exception patterns
├── 02-git-workflow.md          # Branch naming, submodules, conventional commits
├── 03-code-quality.md          # Ruff, pyrefly, uv commands
├── 04-testing.md               # Pytest patterns, fixtures, markers
├── 05-optional-dependencies.md # Lazy imports, optional deps, skip markers
└── 06-claude-config-management.md # Where to put shared vs repo-specific rules
```

## Usage

### Adding to a Repository

Add this repository as a git submodule in your project:

```bash
git submodule add git@github.com:thenvoi/claude-config.git .claude/rules
git commit -m "chore: add shared claude rules"
```

### Cloning a Repository with Submodules

```bash
# Clone with submodules
git clone --recurse-submodules <repo-url>

# OR clone then init submodules
git clone <repo-url>
git submodule update --init --recursive
```

### Updating to Latest Rules

```bash
cd .claude/rules
git pull origin main
cd ../..
git add .claude/rules
git commit -m "chore: bump shared claude rules"
```

### Auto-Update Submodules on Pull

Enable automatic submodule updates (one-time setup):

```bash
git config --global submodule.recurse true
```

## Repository-Specific Rules

Each repository should have its own `CLAUDE.md` at the root for project-specific instructions:

- Project description and features
- Code structure overview
- Specific commands and paths
- Environment variables
- Project-specific conventions

## Content Strategy

| Location | Scope | Content |
|----------|-------|---------|
| `.claude/rules/*` | Shared | Coding standards, error handling, git workflow, testing |
| `CLAUDE.md` (root) | Local | Project structure, specific commands, env vars |

## Contributing

To update shared rules:

1. Clone this repository
2. Make changes to files in `rules/`
3. Submit a PR with your changes
4. After merge, update submodules in target repositories

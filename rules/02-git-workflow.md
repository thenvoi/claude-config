# Git Workflow

## Cloning Repositories

Always clone with submodules to get shared rules:

```bash
# Clone with submodules
git clone --recurse-submodules <repo-url>

# OR if already cloned, initialize submodules
git submodule update --init --recursive
```

After pulling, update submodules:

```bash
git pull
git submodule update --init --recursive
```

Enable auto-update (one-time setup):

```bash
git config --global submodule.recurse true
```

## Branch Naming

Branch names should match the Linear issue:

- Format: `<prefix>/<title>-<ISSUE-ID>`
- Example: `feat/add-user-auth-ENG-123`

Prefixes:

- `feat/` - New features
- `fix/` - Bug fixes
- `refactor/` - Code refactoring
- `docs/` - Documentation changes
- `chore/` - Maintenance tasks

## Commit Messages

Follow conventional commits format for all commits:

```
<type>: <description>

[optional body]

[optional footer]
```

Types:
- `feat:` - New feature
- `fix:` - Bug fix
- `docs:` - Documentation only
- `refactor:` - Code refactoring
- `test:` - Adding or updating tests
- `chore:` - Maintenance tasks

## Pull Request Titles

PR titles MUST use conventional commits format:

- `feat:` - New features
- `fix:` - Bug fixes
- `docs:` - Documentation changes

Examples:
- `feat: Add custom tools support to all adapters`
- `fix: Handle validation errors in execute_tool_call`
- `docs: Update README with new adapter examples`

## Pre-Commit Checklist

- Run tests before committing
- Run linting and formatting
- Ensure type checking passes
- Review changes with `git diff`

## Code Review

- Keep PRs focused and reasonably sized
- Respond to review comments promptly
- Squash commits when merging if history is messy

# GitHub PR Inline Comments

## Adding Inline Review Comments

To add inline comments at specific lines in a PR, use the GitHub Reviews API with `gh api`:

```bash
cat << 'EOF' | gh api repos/{owner}/{repo}/pulls/{pr_number}/reviews --method POST --input -
{
  "commit_id": "<commit_sha>",
  "event": "COMMENT",
  "body": "Review summary",
  "comments": [
    {
      "path": "src/path/to/file.py",
      "line": 42,
      "body": "Your comment here"
    }
  ]
}
EOF
```

## Getting the Correct Line Numbers

**Important:** Line numbers must be from the NEW version of the file, not diff line numbers.

1. Get the commit SHA:
   ```bash
   gh pr view {pr_number} --json headRefOid -q .headRefOid
   ```

2. Find correct line numbers in the actual file:
   ```bash
   # Get the file content at the PR's HEAD commit
   curl -s "https://raw.githubusercontent.com/{owner}/{repo}/{commit_sha}/path/to/file.py" | grep -n "pattern"
   ```

3. Alternatively, use the diff with grep:
   ```bash
   gh pr diff {pr_number} | grep -n "pattern_to_find"
   ```
   Note: These are diff line numbers, not file line numbers. Use the actual file method above for accuracy.

## Common Mistakes to Avoid

- **Don't use `gh pr review --comment`** - This adds a general comment, not inline comments
- **Don't use diff line numbers** - Use actual file line numbers from the new version
- **Don't use `-f` flag for JSON arrays** - Pass JSON via stdin with `--input -`
- **Don't guess line numbers** - Always verify by checking the actual file content

## Example: Full Workflow

```bash
# 1. Get commit SHA
COMMIT=$(gh pr view 83 --json headRefOid -q .headRefOid)

# 2. Find the line number for a specific pattern
curl -s "https://raw.githubusercontent.com/owner/repo/${COMMIT}/src/file.py" | grep -n "def my_function"

# 3. Add inline comment at that line
cat << 'EOF' | gh api repos/owner/repo/pulls/83/reviews --method POST --input -
{
  "commit_id": "abc123...",
  "event": "COMMENT",
  "body": "Code review",
  "comments": [
    {
      "path": "src/file.py",
      "line": 25,
      "body": "Consider renaming this function for clarity"
    }
  ]
}
EOF
```

## Multiple Comments

Add multiple inline comments in a single review:

```bash
cat << 'EOF' | gh api repos/owner/repo/pulls/83/reviews --method POST --input -
{
  "commit_id": "abc123...",
  "event": "COMMENT",
  "body": "Review with multiple comments",
  "comments": [
    {
      "path": "src/file.py",
      "line": 14,
      "body": "First comment"
    },
    {
      "path": "src/file.py",
      "line": 42,
      "body": "Second comment"
    },
    {
      "path": "src/other_file.py",
      "line": 10,
      "body": "Comment on different file"
    }
  ]
}
EOF
```

## Review Events

The `event` field can be:
- `"COMMENT"` - Submit general feedback without approval
- `"APPROVE"` - Approve the PR
- `"REQUEST_CHANGES"` - Request changes before merging

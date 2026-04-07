---
name: gh-cli
description: GitHub CLI (gh) comprehensive reference for repositories, issues, pull requests, Actions, projects, releases, and all GitHub operations. Use when working with GitHub from command line, managing repos, creating PRs/issues, reviewing code, running Actions, or automating GitHub workflows.
---

# GitHub CLI (gh)

Work seamlessly with GitHub from the command line.

**Version:** 2.85.0

## Preference

Clone repos via SSH (`git@github.com:...`), not HTTPS.

## Quick Start

```bash
# Authenticate
gh auth login

# Clone and work with repositories
gh repo clone owner/repo
cd repo

# Create and manage PRs
gh pr create --title "Fix bug" --body "Description"
gh pr list
gh pr checkout 123

# Work with issues
gh issue create --title "Bug report" --body "Details"
gh issue list
```

## Setup Sanity Check

Before any non-trivial `gh` command, especially when running as an AI agent, verify the environment is ready:

```bash
gh --version                                          # gh is installed
gh auth status                                        # logged in to the right host
gh repo view --json nameWithOwner -q .nameWithOwner   # cwd resolves to a gh-aware repo
```

If `gh auth status` fails, do not try to recover programmatically — see [Safety Rules](#safety-rules) below.

## CLI Structure

```
gh
├── auth          # Authentication
├── repo          # Repositories
├── pr            # Pull Requests
├── issue         # Issues
├── run           # Workflow runs
├── workflow      # Workflows
├── release       # Releases
├── project       # Projects
├── gist          # Gists
├── codespace     # Codespaces
├── search        # Search
├── api           # API requests
└── ...
```

## Module Reference

| Module | Command | Reference |
|--------|---------|-----------|
| Authentication | `gh auth` | [references/auth.md](references/auth.md) |
| Repositories | `gh repo` | [references/repos.md](references/repos.md) |
| Pull Requests | `gh pr` | [references/prs.md](references/prs.md) |
| Issues | `gh issue` | [references/issues.md](references/issues.md) |
| GitHub Actions | `gh run`, `gh workflow` | [references/actions.md](references/actions.md) |
| Releases | `gh release` | [references/releases.md](references/releases.md) |
| Projects | `gh project` | [references/projects.md](references/projects.md) |
| Gists & Codespaces | `gh gist`, `gh codespace` | [references/gists.md](references/gists.md) |
| Other commands | `gh label`, `gh search`, etc. | [references/misc.md](references/misc.md) |

Load the relevant reference file when working with specific commands.

## Common Workflows

### Create PR from Issue

```bash
# Create branch from issue
gh issue develop 123 --branch feature/issue-123

# Make changes, commit, push
git add .
git commit -m "Fix issue #123"
git push

# Create PR linking to issue
gh pr create --title "Fix #123" --body "Closes #123"
```

### Repository Setup

```bash
# Create repository with initial setup
gh repo create my-project --public \
  --description "My awesome project" \
  --clone --gitignore python --license mit

cd my-project

# Set up branch protection (via API)
gh api repos/owner/repo/branches/main/protection \
  --method PUT --input protection.json
```

### Bulk Operations

```bash
# Close multiple issues
gh issue list --search "label:stale" \
  --json number --jq '.[].number' | \
  xargs -I {} gh issue close {} --comment "Closing as stale"

# Add label to multiple PRs
gh pr list --search "review:required" \
  --json number --jq '.[].number' | \
  xargs -I {} gh pr edit {} --add-label needs-review
```

## Output Formatting

```bash
# JSON output for scripting
gh pr list --json number,title,author

# Filter with jq
gh pr list --json number,title | jq '.[] | select(.title | contains("fix"))'

# Template output
gh pr view 123 --template '{{.title}} by @{{.author.login}}'

# Quiet mode (for scripts)
gh pr checks --quiet
```

## Global Flags

```bash
# Specify repository
gh pr list --repo owner/repo

# Disable prompts
gh pr create --title "Fix" --body "Desc" --fill

# JSON output
gh issue list --json number,title

# Paginate all results
gh repo list --paginate

# Help for any command
gh pr create --help
```

## Environment Variables

```bash
export GH_HOST=github.com              # GitHub hostname
export GH_PROMPT_DISABLED=true         # Disable prompts
export GH_PAGER=cat                    # Disable pager
export GH_DEBUG=api                    # Debug API requests
```

## Best Practices

1. **Use `--web`** to open browser for complex operations
2. **Use `--json`** for scripting and automation
3. **Set default repo** with `gh repo set-default` to avoid `--repo` flag
4. **Create aliases** for frequently used commands: `gh alias set co "pr checkout"`
5. **Use `gh browse`** to quickly open repo, PR, or issue pages

## Safety Rules

These rules are load-bearing when this skill is used by an AI coding agent. Treat them as hard constraints, not suggestions.

1. **Never run `gh auth login` from the agent.** It is interactive and will hang. Ask the user to run it themselves (e.g. `! gh auth login` in their terminal). The same applies to `gh auth refresh`, `gh auth setup-git`, and any flow that opens a browser.
2. **Never force-push to the default branch** (`main` / `master`). Confirm with the user before any destructive operation: `gh pr merge --rebase`, `gh repo delete`, `gh release delete`, `gh issue delete`, `gh pr close --delete-branch` against shared branches.
3. **Prefer `gh pr create --draft`** when the change is not ready for review, so reviewers are not pinged prematurely.
4. **`--delete-branch` deletes the remote branch too.** Confirm with the user when merging if the branch may have other consumers (CI, deployments, downstream forks).
5. **For automation scripts, prefer `gh api` over scraping `gh` text output.** The text output format is not stable across versions; the JSON API is. Combine with `--jq` for server-side filtering.
6. **Verify `gh auth status` before any destructive command.** A misconfigured account (wrong host, expired token, wrong user) can apply changes to the wrong repository.

## Heredoc Bodies for Multi-line Text

Long PR/issue bodies are awkward to escape inline. Use a heredoc to preserve formatting and avoid quoting issues — this is the most reliable cross-shell pattern:

```bash
gh pr create --title "feat: add dark mode" --body "$(cat <<'EOF'
## Summary
- Adds a system-aware dark mode toggle
- Persists user preference in localStorage

## Test plan
- [ ] Toggle dark/light from the header
- [ ] Reload page, preference is retained
- [ ] System theme change updates the UI when no preference is set

## Screenshots
Before: ...
After: ...
EOF
)"
```

The single-quoted heredoc delimiter (`<<'EOF'`) prevents shell variable expansion inside the body, which matters for snippets containing `$variables` or backticks.

## PR Comments via the API

`gh pr view --comments` only shows the conversation thread. To inspect inline review comments (the ones attached to specific lines of a diff), or to script comment processing, hit the API directly:

```bash
gh api repos/OWNER/REPO/pulls/123/comments        # inline review comments
gh api repos/OWNER/REPO/issues/123/comments       # conversation comments (PRs are issues)
gh api repos/OWNER/REPO/pulls/123/reviews         # review summaries (approve/request-changes)
```

The two endpoints return different shapes — `pulls/{n}/comments` includes `path`, `line`, `commit_id`, while `issues/{n}/comments` is just markdown bodies. Combine with `--jq` for filtering:

```bash
gh api repos/OWNER/REPO/pulls/123/comments \
  --jq '.[] | {path, line, user: .user.login, body}'
```

## Getting Help

```bash
gh --help                    # General help
gh pr --help                 # Command help
gh pr create --help          # Subcommand help
gh reference                 # Full reference
```

## Official Resources

- [GitHub CLI Manual](https://cli.github.com/manual/)
- [GitHub CLI Repository](https://github.com/cli/cli)

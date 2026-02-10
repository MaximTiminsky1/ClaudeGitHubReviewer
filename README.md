# GitHub PR Reviewer

Automated Pull Request review on GitHub using Claude Code.

## Quick Start

### 1. Setup GitHub Token

Create `.env` file:

```bash
cp .env.example .env
```

Edit `.env` and add:
- `GITHUB_TOKEN` - Personal Access Token ([create here](https://github.com/settings/tokens))

**Note:** Claude API key is NOT needed - analysis is performed by Claude Code's built-in AI!

### 2. Usage in Claude Code

Run the command:

```
/review-pr
```

Enter the Pull Request URL:

```
https://github.com/owner/repo/pull/123
```

Claude will analyze the PR and create a review file in `docs/`.

### 3. Publishing Comments

Review the file:

```bash
code docs/review_pr123_*.md
```

Publish to GitHub:

```bash
python post_comments.py docs/review_pr123_*.md
```

Or with auto-confirmation:

```bash
python post_comments.py -y docs/review_pr123_*.md
```

## Features

- 🛡️ Security analysis (SQL injection, XSS, Path Traversal)
- ⚡ Performance checks (N+1 queries, memory leaks)
- 📝 Code style and best practices
- 🐛 Potential bug detection
- 🔴 Prioritization: critical / warnings / suggestions
- 🇷🇺 Comments in Russian language

## Project Structure

```
GitHubReviewer/
├── .claude/commands/
│   └── review-pr.md          # Slash command for Claude
├── docs/                     # Generated reviews
├── .env                      # Tokens (not committed)
├── .env.example             # Template
├── post_comments.py         # Comment publishing script
├── requirements.txt         # Python dependencies
└── README.md               # Documentation
```

## Requirements

- Python 3.8+
- Claude Code (built-in AI performs analysis)
- GitHub token with `repo` permissions

## Installation

The project uses only Python standard library - **NO additional packages required**!

Make sure you have installed:
- Python 3.8+ (already included on macOS)
- GitHub CLI: `brew install gh` ([instructions](https://cli.github.com/))
- Claude Code ([download](https://www.anthropic.com/claude-code))

## Comment Format

- 🔴 **Critical** - serious vulnerabilities, blockers
- ⚠️ **Warning** - performance issues, fragile code
- 💡 **Suggestion** - improvement recommendations

## License

MIT

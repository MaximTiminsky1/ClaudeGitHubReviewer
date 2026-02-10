# Generated Reviews

This folder contains Pull Request review files.

## File Format

Files are automatically created by the `/review-pr` command with the name:

```
review_pr{number}_{timestamp}.md
```

For example: `review_pr237_20260210_073305.md`

## Review File Structure

Each file contains:

1. **PR Information** - number, author, status, date
2. **Overview** - brief description of changes
3. **Code Comments** - detailed analysis including:
   - File and line number
   - Problem description
   - Solution
   - Severity: 🔴 critical / ⚠️ warning / 💡 suggestion
4. **Summary** - number of issues by category
5. **Recommendations** - prioritization of fixes
6. **Metadata** - HTML comment for automated publishing

## Publishing Comments

To publish comments on GitHub:

```bash
python post_comments.py docs/review_pr123_*.md
```

Or with auto-confirmation:

```bash
python post_comments.py -y docs/review_pr123_*.md
```

## Editing

You can edit any file before publishing:

1. Modify comment text
2. Remove unwanted comments from metadata
3. Add your own comments
4. Publish the edited version

**Important:** Do not modify the structure of the HTML comment with metadata at the bottom of the file - the publishing script uses it to send comments to GitHub.

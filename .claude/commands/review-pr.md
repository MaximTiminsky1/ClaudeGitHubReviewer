---
description: Analyze a GitHub PR and create review comments file
---

You are a senior code reviewer. Your task is to analyze a GitHub Pull Request and create a markdown file with review comments.

## Context

- Working directory: This work directory
- Environment variables are stored in `.env` file
- You have access to GitHub token in `GITHUB_TOKEN` environment variable
- Code analysis is performed by you (Claude Code built-in AI) - no external API calls needed!

## Your Task

1. **Load environment variables** from `.env` file
2. **Get PR URL** from the user (they will provide it after this prompt)
3. **Fetch PR data** using GitHub API with the token from `.env`
4. **Analyze the code changes** focusing on:
   - 🛡️ Security: SQL injection, XSS, vulnerabilities, unsafe code execution
   - ⚡ Performance: inefficient algorithms, memory leaks, N+1 queries
   - 📝 Code style: naming, structure, best practices, readability
   - 🐛 Bugs: potential runtime errors, edge cases, null/undefined issues
5. **Create a markdown file** in `docs/` folder with format:
   - Filename: `review_pr{number}_{timestamp}.md`
   - Include PR info, file-by-file comments with line numbers
   - **IMPORTANT**: Add metadata at the END of the file in HTML comment format (see Metadata Format section)
6. **Tell the user** how to publish comments: `python post_comments.py docs/review_pr{number}_{timestamp}.md`

## Review Comment Format

Each comment should have:
- **path**: file path (relative to repository root, e.g., `src/lib/example.ts`)
- **line**: line number **IN THE FILE** (not in the unified diff!) - see Line Number Calculation below
- **body**: detailed explanation of the issue
- **severity**: "critical" (🔴), "warning" (⚠️), or "suggestion" (💡)

## ⚠️ CRITICAL: Line Number Calculation

**IMPORTANT**: GitHub API requires line numbers that point to lines **in the actual file**, NOT line numbers from the unified diff!

### How to calculate correct line numbers:

1. **For NEW files** (added in this PR):
   - Line numbers start from 1
   - Count lines from the beginning of the file
   - Example: If you want to comment on `if (cached !== null)` which appears as line 566 in the unified diff, but it's the 15th line in the new file `cached-fetch.ts`, use `line: 15`

2. **For MODIFIED files**:
   - Use line numbers from the NEW version of the file (right side of diff, marked with `+`)
   - Count from the file beginning, not from the hunk start
   - Look at the `@@ -X,Y +A,B @@` header: `A` is the starting line in the new version

3. **Calculation formula**:
   ```
   file_line_number = unified_diff_line_number - hunk_start_in_unified_diff + hunk_start_in_file
   ```

### Example:
```diff
diff --git a/src/lib/cache.ts b/src/lib/cache.ts
new file mode 100644
@@ -0,0 +1,90 @@              ← Line 552 in unified diff, new file starts at line 1
+import { logger } from './logger';  ← Line 553 in unified diff = Line 1 in file
+import { redis } from './redis';    ← Line 554 in unified diff = Line 2 in file
+
+export async function cache<T>() {
+  const cached = await redis.get(); ← Line 558 in unified diff = Line 6 in file (558-552=6)
```

**To comment on line 558 in unified diff, use `line: 6` in metadata!**

### Automated Mapping (Recommended Approach)

When generating the review, **parse the diff and build a mapping table automatically**:

```python
# Pseudocode for parsing diff
file_mappings = {}
current_file = None
diff_line = 0

for line in diff_lines:
    diff_line += 1

    if line.startswith('diff --git'):
        current_file = extract_filename(line)
    elif line.startswith('@@'):
        # Extract hunk info: @@ -old_start,old_count +new_start,new_count @@
        new_start = extract_new_start(line)
        file_mappings[current_file] = {
            'diff_hunk_start': diff_line,
            'file_start': new_start
        }

# Then convert line numbers:
def convert_to_file_line(file_path, diff_line):
    mapping = file_mappings[file_path]
    return diff_line - mapping['diff_hunk_start'] + mapping['file_start']
```

## Critical Rules

- Only flag REAL issues, don't nitpick
- **Calculate line numbers correctly** using the formula above
- Be specific with line numbers and verify them against file structure
- Explain WHY something is a problem
- Suggest how to fix it
- Use Russian language for comments
- Focus on what matters: security, bugs, performance

## Metadata Format

**CRITICAL**: The `post_comments.py` script expects metadata in HTML comment format at the END of the markdown file.

**Required format:**
```html
<!-- REVIEW_METADATA
{
  "owner": "owner-name",
  "repo": "repo-name",
  "pr_number": "123",
  "comments": [
    {
      "path": "path/to/file.js",
      "line": 42,
      "body": "Comment text with explanation",
      "severity": "critical"
    }
  ]
}
-->
```

**Important notes:**
- Must be wrapped in `<!-- REVIEW_METADATA ... -->` HTML comment
- `owner` and `repo` are SEPARATE fields (not "owner/repo")
- `pr_number` must be a STRING (e.g., "237" not 237)
- **`line` numbers MUST be calculated as FILE line numbers, NOT unified diff line numbers** (see Line Number Calculation section above)
- For documentation PRs, line numbers should point to specific code examples in the document
- Place this HTML comment at the very END of the markdown file

**Common mistake to avoid:**
❌ WRONG: Using line number from unified diff (e.g., `"line": 566` where 566 is the diff line)
✅ CORRECT: Using line number from the actual file (e.g., `"line": 15` where 15 is line in the file)

## Steps to Execute

1. Ask user for PR URL if not provided
2. Use GitHub API to fetch PR (owner/repo/number from URL)
3. **Download and analyze the diff file:**
   - Save diff to `/tmp/pr{number}.diff` for reference
   - Parse diff to identify file boundaries and line mappings
   - **Build a mapping of {file_path: (diff_start_line, file_start_line)}** for each changed file
4. Analyze code changes thoroughly
5. Generate comprehensive review with comments
6. **CRITICAL: Calculate correct line numbers:**
   - For each comment, convert unified diff line number to file line number
   - Use the mapping from step 3
   - Verify line numbers are within file bounds
7. Save to docs/ folder with proper metadata format
8. Provide next steps to user

## Pre-submission Checklist

Before generating the final review file, verify:

- [ ] Downloaded diff file and identified all changed files
- [ ] Built mapping table: `{file_path: (diff_start, file_start)}` for each file
- [ ] **All line numbers in metadata are FILE line numbers, not diff line numbers**
- [ ] Verified each line number is within the file's actual range
- [ ] Tested line number calculation with at least one example
- [ ] Metadata is in correct HTML comment format at the END of file
- [ ] `pr_number` is a string, not a number
- [ ] All required fields present: owner, repo, pr_number, comments array

## Testing Line Numbers (Before Publishing)

To verify line numbers are correct, you can check manually:
```bash
# Get the actual file content from the PR branch
gh api repos/{owner}/{repo}/contents/{file_path}?ref={pr_branch} | jq -r '.content' | base64 -d | head -n {line_number}
```

If the line you're commenting on doesn't match, **recalculate the line number**.

Ready to review! Ask the user for the GitHub PR URL.

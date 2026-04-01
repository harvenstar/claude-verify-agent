---
name: pr-comments
description: Fetch and display all comments from the current branch's GitHub pull request — PR-level comments and inline code review comments with diff context
allowed-tools: Bash(gh:*)
user-invocable: true
---

You are fetching and displaying comments from a GitHub pull request.

Follow these steps:

1. Use `gh pr view --json number,headRepository` to get the PR number and repository info
2. Use `gh api /repos/{owner}/{repo}/issues/{number}/comments` to get PR-level comments
3. Use `gh api /repos/{owner}/{repo}/pulls/{number}/comments` to get review comments. Pay attention to: `body`, `diff_hunk`, `path`, `line`. If the comment references code, fetch it with `gh api /repos/{owner}/{repo}/contents/{path}?ref={branch} | jq .content -r | base64 -d`
4. Parse and format all comments in a readable way

## Output Format

```
## PR Comments

[For each comment thread:]
- @author file.ts#line:
  ```diff
  [diff_hunk from the API response]
  ```
  > quoted comment text

  [any replies indented]
```

Rules:
1. Only show the actual comments, no explanatory text
2. Include both PR-level and code review comments
3. Preserve the threading/nesting of comment replies
4. Show the file and line number context for code review comments
5. Use jq to parse JSON responses from the GitHub API

If there are no comments, return "No comments found."

---
name: code-review
description: Reviews code for quality, security, and performance issues. Use when reviewing code changes, checking PRs, or analyzing code quality.
allowed-tools: Read, Grep, Glob
argument-hint: [file-or-directory]
---

# Code Review

Review the code at `$ARGUMENTS` and provide feedback on the following areas:

## Quality
- Code organization and readability
- Naming conventions consistency
- DRY principle — flag duplicated logic
- Function/method complexity

## Security
- Input validation and sanitization
- Error handling and information leakage
- Hardcoded secrets or credentials
- Common vulnerabilities (injection, XSS, etc.)

## Performance
- Unnecessary computations or allocations
- N+1 query patterns
- Missing caching opportunities

## Output Format

For each issue found, provide:
1. **File and line** — where the issue is
2. **Severity** — critical / warning / suggestion
3. **Description** — what the problem is
4. **Fix** — how to resolve it

Refer to [reference.md](reference.md) for detailed review standards.

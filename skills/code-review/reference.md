# Code Review Standards

## Severity Levels

| Level | Description |
|-------|-------------|
| Critical | Must fix before merge — bugs, security vulnerabilities, data loss risks |
| Warning | Should fix — poor patterns, missing error handling, tech debt |
| Suggestion | Nice to have — readability improvements, minor optimizations |

## Security Checklist

- [ ] No hardcoded credentials or API keys
- [ ] User input is validated and sanitized
- [ ] SQL queries use parameterized statements
- [ ] Error messages do not expose internal details
- [ ] Authentication and authorization checks are in place

## Performance Checklist

- [ ] Database queries are optimized (no N+1)
- [ ] Large datasets use pagination
- [ ] Expensive operations are cached where appropriate
- [ ] No unnecessary synchronous blocking

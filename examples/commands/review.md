---
description: Quick code review with security focus
allowed-tools: [read_file, grep, glob]
argument-hint: [file-or-directory]
---

Please review {{$1 or "the recent changes"}} with focus on:

1. **Code Quality**: Readability, maintainability, naming conventions
2. **Potential Bugs**: Edge cases, error handling, null checks
3. **Security**: Input validation, injection risks, hardcoded secrets
4. **Performance**: Obvious inefficiencies, N+1 queries, memory leaks

Provide specific line references and actionable suggestions.

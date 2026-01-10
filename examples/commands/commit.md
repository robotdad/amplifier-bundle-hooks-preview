---
description: Create a well-formatted commit with context
allowed-tools: [bash, read_file]
requires-approval: true
approval-message: "This will create a git commit. Review the changes first?"
---

## Current Changes

!`git status --short`

## Diff Summary

!`git diff --stat`

Based on the changes above, create an appropriate commit message following conventional commits format (feat/fix/docs/refactor/test/chore).

Then execute: `git add -A && git commit -m "<your message>"`

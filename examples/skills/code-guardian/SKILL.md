---
name: code-guardian
version: 1.0.0
description: Skill with hooks that guard code quality

hooks:
  PreToolUse:
    - matcher: "write_file|edit_file"
      hooks:
        - type: command
          command: |
            FILE=$(cat | jq -r '.input.file_path // .input.path // ""')
            if [[ "$FILE" == *.py ]]; then
              echo "[CodeGuardian] Python file modification detected: $FILE" >> /tmp/code-guardian.log
            fi
    - matcher: "bash"
      hooks:
        - type: command
          command: |
            CMD=$(cat | jq -r '.input.command // ""')
            # Warn about potentially dangerous commands
            if echo "$CMD" | grep -qE 'rm -rf|sudo|chmod 777'; then
              echo "[CodeGuardian] WARNING: Potentially dangerous command: $CMD" >> /tmp/code-guardian.log
              echo '{"decision": "continue", "context": "WARNING: This command may be dangerous. Please confirm intent."}'
            fi
---

# Code Guardian Skill

This skill adds protective hooks around code modifications.

## What It Does

1. **Logs Python file modifications** - Tracks when .py files are written/edited
2. **Warns about dangerous bash commands** - Flags rm -rf, sudo, chmod 777

## Hook Behavior

The hooks run automatically when:
- Any `write_file` or `edit_file` targets a `.py` file
- Any `bash` command contains potentially dangerous patterns

## Log Location

Activity is logged to `/tmp/code-guardian.log`

## Important Notes

- Skill files must be named `SKILL.md` (uppercase)
- `name` and `description` must be at the top level of frontmatter (not nested under `skill:`)
- The skill directory name should match the `name` field

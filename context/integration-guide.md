# Hooks Integration Guide

This bundle demonstrates how hooks-shell, skills, and slash-commands work together.

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     Amplifier Session                        │
├─────────────────────────────────────────────────────────────┤
│  Events (tool:pre, session:start, etc.)                     │
│         │                                                    │
│         ▼                                                    │
│  ┌─────────────────┐                                        │
│  │  hooks-shell    │◄── .amplifier/hooks/*.json             │
│  │  (bridge)       │◄── skill hooks (via tool-skills)       │
│  └────────┬────────┘                                        │
│           │                                                  │
│           ▼                                                  │
│  ┌─────────────────┐                                        │
│  │ Shell Executor  │──► Runs hook commands                  │
│  └─────────────────┘                                        │
│                                                              │
│  ┌─────────────────┐                                        │
│  │ slash-command   │◄── .amplifier/commands/*.md            │
│  │ (tool)          │                                        │
│  └─────────────────┘                                        │
│                                                              │
│  ┌─────────────────┐                                        │
│  │ tool-skills     │◄── .amplifier/skills/*/skill.md        │
│  │ (tool + hook)   │──► Registers skill hooks dynamically   │
│  └─────────────────┘                                        │
└─────────────────────────────────────────────────────────────┘
```

## Component Interactions

### 1. hooks-shell ↔ Amplifier Events

The hooks-shell module subscribes to Amplifier events and translates them to Claude Code event names:

| Amplifier Event | Claude Code Event | When |
|-----------------|-------------------|------|
| `tool:pre` | PreToolUse | Before any tool executes |
| `tool:post` | PostToolUse | After any tool executes |
| `prompt:submit` | UserPromptSubmit | User submits a prompt |
| `session:start` | SessionStart | Session begins |
| `session:end` | SessionEnd | Session ends |
| `prompt:complete` | Stop | LLM finishes response |

### 2. tool-skills → hooks-shell

When a skill is loaded via `load_skill`, the tool-skills module:

1. Parses hooks from skill frontmatter
2. Registers them with hooks-shell bridge
3. Hooks are active while skill is loaded
4. On unload, hooks are removed

```yaml
# In skill.md frontmatter
hooks:
  PreToolUse:
    - matcher: "write_file"
      hooks:
        - type: command
          command: "echo 'Write detected'"
```

### 3. slash-command → approval hooks (future)

Commands with `requires-approval: true` will emit `approval:required` events:

```yaml
# In command.md frontmatter
requires-approval: true
approval-message: "This will modify production. Proceed?"
```

Hooks can intercept and approve/deny:

```json
{
  "hooks": {
    "PermissionRequest": [
      {
        "matcher": "command:deploy",
        "hooks": [
          {
            "type": "command",
            "command": "zenity --question --text='$APPROVAL_MESSAGE'"
          }
        ]
      }
    ]
  }
}
```

## Environment Variables

All hooks receive these environment variables:

| Variable | Description |
|----------|-------------|
| `AMPLIFIER_PROJECT_DIR` | Project root directory |
| `AMPLIFIER_SESSION_ID` | Current session ID |
| `AMPLIFIER_HOOKS_DIR` | Hooks configuration directory |
| `AMPLIFIER_ENV_FILE` | File for persisting env vars |
| `TOOL_NAME` | Name of tool being invoked |
| `TOOL_INPUT` | JSON input to the tool |

Plus Claude Code compatibility aliases (`CLAUDE_*`).

## Hook Response Protocol

Hooks communicate back via exit codes and stdout:

| Exit Code | Meaning |
|-----------|---------|
| 0 | Continue (approve) |
| 2 | Block/deny the operation |
| Other | Error (continue anyway) |

For richer responses, output JSON:

```json
{
  "decision": "block",
  "reason": "Unauthorized file modification"
}
```

Or inject context:

```json
{
  "decision": "continue",
  "context": "Remember: this file uses tabs, not spaces"
}
```

## Testing the Integration

### Quick Validation

```bash
# 1. Start Amplifier with this bundle
amplifier bundle use ./amplifier-bundle-hooks-preview

# 2. Check hooks are loaded
amplifier run --prompt "What hooks are active?"

# 3. Trigger a hook
amplifier run --prompt "Run: echo hello"
# Should see PreToolUse hook fire for bash tool
```

### Shadow Environment Testing

```bash
# Create isolated test environment
amplifier shadow create \
  --local-source hooks-shell:./amplifier-module-hooks-shell \
  --local-source tool-skills:./amplifier-module-tool-skills \
  --local-source slash-command:./amplifier-module-tool-slash-command

# Run tests in shadow
amplifier shadow exec "amplifier run --prompt 'test hooks'"
```

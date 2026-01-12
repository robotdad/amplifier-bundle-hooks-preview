# amplifier-bundle-hooks-preview

**Temporary preview bundle for acceptance testing** of the hooks ecosystem before upstream merge.

> **This bundle is for testing only.** It pulls from development forks and requires a modified CLI. Do not use for production work.

## Purpose

This bundle exists to validate the integrated hooks ecosystem:
- Human acceptance testing of hooks, commands, and skills working together
- AI-assisted testing in shadow environments
- Proving the architecture before submitting upstream PRs

Once validated and merged upstream, this bundle will be archived.

## Components

| Component | Repo | Branch | Description |
|-----------|------|--------|-------------|
| **hooks-shell** | robotdad/amplifier-module-hooks-shell | main | Claude Code compatible hooks bridge |
| **tool-skills** | robotdad/amplifier-module-tool-skills | feat/skill-scoped-hooks | Skills with hooks in frontmatter |
| **tool-slash-command** | robotdad/amplifier-module-tool-slash-command | main | Extensible custom commands |
| **app-cli** (required) | robotdad/amplifier-app-cli | feat/custom-slash-commands | Modified CLI with command support |

## Features

### Hooks-Shell Features

| Feature | Description |
|---------|-------------|
| **Command hooks** | Shell scripts triggered at lifecycle events |
| **Prompt hooks** | LLM evaluation for complex decisions |
| **Parallel execution** | Run multiple hooks concurrently |
| **Skill-scoped hooks** | Hooks embedded in SKILL.md frontmatter |
| **Pattern matching** | Regex matchers for selective execution |
| **Context injection** | Inject feedback into agent context |

### Slash Command Features

| Feature | Description |
|---------|-------------|
| **Markdown commands** | Define commands as `.md` files with YAML frontmatter |
| **Git URL sources** | Share commands from git repositories |
| **Granular permissions** | Fine-grained bash control: `Bash(git status:*)` |
| **Command composition** | Commands can invoke other commands |
| **Model override** | Per-command model selection |
| **LLM discovery** | AI can list and invoke commands programmatically |
| **File references** | `@path/to/file` includes file content |
| **Bash execution** | `` !`command` `` runs shell during template processing |

## Installation

### Recommended: Shadow Environment

The safest way to test is in an isolated shadow environment:

```bash
# From an existing Amplifier session, create a shadow with all components
amplifier shadow create \
  --local-source hooks-shell:~/path/to/amplifier-module-hooks-shell \
  --local-source tool-skills:~/path/to/amplifier-module-tool-skills \
  --local-source slash-command:~/path/to/amplifier-module-tool-slash-command \
  --local-source app-cli:~/path/to/amplifier-app-cli

# Run the modified CLI inside the shadow
amplifier shadow exec "amplifier run"
```

### Host Installation (Use With Caution)

Installing the modified CLI on your host **will replace your current Amplifier installation**.

```bash
# This REPLACES your current amplifier CLI
uv tool install git+https://github.com/robotdad/amplifier-app-cli@feat/custom-slash-commands

# Verify you have the modified version
amplifier --version
```

**To restore mainline:**
```bash
uv tool install git+https://github.com/microsoft/amplifier --force
```

## Testing Workflow

### 1. Set up test project

```bash
mkdir -p test-project/.amplifier/{hooks,commands,skills}
cd test-project
```

### 2. Add example hooks

```bash
cat > .amplifier/hooks/hooks.json << 'EOF'
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command", 
            "command": "echo \"[Hook] Bash command detected\" >> /tmp/hooks.log && exit 0"
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": ".*",
        "parallel": true,
        "hooks": [
          {"type": "command", "command": "echo \"[Parallel 1]\" >> /tmp/hooks.log"},
          {"type": "command", "command": "echo \"[Parallel 2]\" >> /tmp/hooks.log"}
        ]
      }
    ],
    "SessionStart": [
      {
        "matcher": ".*",
        "hooks": [
          {
            "type": "command",
            "command": "echo \"[Hook] Session started at $(date)\" >> /tmp/hooks.log"
          }
        ]
      }
    ]
  }
}
EOF
```

### 3. Add example commands

**Simple command:**
```bash
cat > .amplifier/commands/review.md << 'EOF'
---
description: Quick code review
allowed-tools: [read_file, grep, glob]
---

Review $ARGUMENTS for code quality and security issues.
EOF
```

**Command with granular bash permissions:**
```bash
cat > .amplifier/commands/git-status.md << 'EOF'
---
description: Safe git status check
allowed-tools:
  - Bash(git status:*)
  - Bash(git diff:*)
  - Bash(git log:*)
---

## Current Status
!`git status --short`

## Recent Commits  
!`git log --oneline -5`

## Changes
!`git diff --stat`
EOF
```

**Command with model override:**
```bash
cat > .amplifier/commands/quick.md << 'EOF'
---
description: Quick answer using faster model
model: claude-3-5-haiku-20241022
---

Give a brief answer to: $ARGUMENTS
EOF
```

### 4. Add shared commands from git (in bundle config)

Commands can be sourced from git repositories:

```yaml
tools:
  - module: tool-slash-command
    config:
      commands:
        - git+https://github.com/org/shared-commands@v1
        - git+https://github.com/team/review-tools@main:commands
```

**Command repo requirements:**
- Must contain a `.amplifier-commands` marker file
- Commands discovered recursively from marker location

### 5. Add example skill with hooks

```bash
mkdir -p .amplifier/skills/code-guardian

cat > .amplifier/skills/code-guardian/SKILL.md << 'EOF'
---
name: code-guardian
description: Guards code quality with hooks
version: 1.0.0

hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "echo \"[Skill Hook] Bash command\" >> /tmp/hooks.log"
---

# Code Guardian

This skill adds hooks that fire when loaded.
EOF
```

### 6. Run and verify

```bash
# Start session
amplifier run

# In another terminal, watch hook activity
tail -f /tmp/hooks.log
```

## Event Coverage

| Claude Code Event | Amplifier Event | Status |
|-------------------|-----------------|--------|
| PreToolUse | tool:pre | Supported |
| PostToolUse | tool:post | Supported |
| UserPromptSubmit | prompt:submit | Supported |
| SessionStart | session:start | Supported |
| SessionEnd | session:end | Supported |
| Stop | prompt:complete | Supported |
| Notification | user:notification | Supported |
| SubagentStart | subagent:start | Supported |
| SubagentStop | subagent:stop | Supported |

## Hook Types

| Type | Description | Use Case |
|------|-------------|----------|
| `command` | Execute shell command | Fast validation, logging, formatting |
| `prompt` | LLM evaluation | Complex decisions, security review |

**Prompt hook example:**
```json
{
  "type": "prompt",
  "prompt": "Review this command for security issues. Return JSON with decision (approve/block) and reason.",
  "timeout": 60
}
```

## Known Limitations

- Requires modified app-cli (not yet upstream)
- Prompt hooks use session's default provider (model override coming)

## Skill File Format Requirements

| Requirement | Details |
|-------------|---------|
| Filename | Must be `SKILL.md` (uppercase) |
| Frontmatter | `name` and `description` at top level (not nested) |
| Directory | Should match the `name` field |

**Correct format:**
```yaml
---
name: my-skill
description: What this skill does
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "echo 'hook fired'"
---
```

## After Testing

Once acceptance testing is complete:

1. **hooks-shell** → New repo microsoft/amplifier-module-hooks-shell
2. **tool-skills** → PR to microsoft/amplifier-module-tool-skills  
3. **tool-slash-command** → New repo microsoft/amplifier-module-tool-slash-command
4. **app-cli** → PR to microsoft/amplifier-app-cli

This bundle will then be archived or updated to pull from upstream.

## License

MIT

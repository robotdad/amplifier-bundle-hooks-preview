---
bundle:
  name: hooks-preview
  version: 0.1.0
  description: |
    Preview bundle integrating hooks-shell, enhanced skills, and slash commands.
    Module sources configured via .amplifier/settings.yaml (portable pattern).

includes:
  # Base foundation (from upstream)
  - bundle: git+https://github.com/microsoft/amplifier-foundation@main

# Hooks module - Claude Code compatible shell hooks
hooks:
  - name: shell-hooks
    module: hooks-shell
    config:
      enabled: true

# Tools from preview modules
tools:
  # Enhanced skills with skill-scoped hooks
  - name: load_skill
    module: tool-skills
    config:
      skill_dirs:
        - .amplifier/skills
        - ~/.amplifier/skills

  # Extensible slash commands
  - name: slash_command
    module: tool-slash-command
    config:
      # Local command directories (user can add their own)
      command_dirs:
        - .amplifier/commands
        - ~/.amplifier/commands
---

# Hooks Preview Bundle

This bundle integrates the hooks ecosystem components for testing before upstream merge.

## Components

| Component | Module Name | Features |
|-----------|-------------|----------|
| **hooks-shell** | `hooks-shell` | Claude Code compatible hooks bridge |
| **tool-skills** | `tool-skills` | Skill-scoped hooks in frontmatter |
| **tool-slash-command** | `tool-slash-command` | Extensible custom commands |

## Setup

### 1. Clone and use the bundle

```bash
git clone https://github.com/robotdad/amplifier-bundle-hooks-preview
cd amplifier-bundle-hooks-preview
amplifier bundle use .
```

The `.amplifier/settings.yaml` in this repo points to the preview module sources.

### 2. (Optional) Use local checkouts for development

Copy the example and point to your local repos:

```bash
cp .amplifier/settings.local.yaml.example .amplifier/settings.local.yaml
# Edit paths to point to your local checkouts
```

### 3. Create a test hook

```bash
mkdir -p .amplifier/hooks
cat > .amplifier/hooks/hooks.json << 'EOF'
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "bash",
        "hooks": [
          {
            "type": "command",
            "command": "echo '[HOOK] Bash command: '$(cat | jq -r '.input.command')"
          }
        ]
      }
    ],
    "SessionStart": [
      {
        "matcher": ".*",
        "hooks": [
          {
            "type": "command",
            "command": "echo '[HOOK] Session started at $(date)' >> /tmp/amplifier-hooks.log"
          }
        ]
      }
    ]
  }
}
EOF
```

### 4. Create a custom command

```bash
mkdir -p .amplifier/commands
cat > .amplifier/commands/review.md << 'EOF'
---
description: Quick code review
allowed-tools: [read_file, grep]
argument-hint: [file-path]
---

Review {{$1 or "the recent changes"}} for:
- Code quality
- Potential bugs
- Security issues
EOF
```

### 5. Create a skill with hooks

```bash
mkdir -p .amplifier/skills/my-workflow
cat > .amplifier/skills/my-workflow/SKILL.md << 'EOF'
---
name: my-workflow
version: 1.0.0
description: Custom workflow with hooks

hooks:
  PreToolUse:
    - matcher: "write_file"
      hooks:
        - type: command
          command: "echo '[SKILL HOOK] Write operation detected'"
---

# My Workflow

Custom workflow instructions here.
EOF
```

### 6. Run Amplifier

```bash
amplifier run
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
| PreCompact | context:pre_compact | Supported |
| PermissionRequest | approval:required | Supported |
| Notification | user:notification | Supported |

## Integration Points

### Hooks -> Skills

Skills can define hooks in their frontmatter. When a skill is loaded, its hooks become active. When unloaded, hooks are removed.

### Hooks -> Commands

Commands with `requires-approval: true` will (once integrated) emit `approval:required` events that hooks can intercept.

### Environment Persistence

Hooks can persist environment variables across executions:
```bash
echo "MY_VAR=value" >> "$AMPLIFIER_ENV_FILE"
```

## Module Source Configuration

This bundle uses the **portable module names** pattern. Actual sources are defined in `.amplifier/settings.yaml`:

- **settings.yaml** (committed) - Points to preview repos for shared testing
- **settings.local.yaml** (gitignored) - Override with local checkouts for development

See `settings.local.yaml.example` for the local development template.

## Testing

See the `examples/` directory for working examples of hooks, commands, and skills integration.

---

@hooks-preview:context/integration-guide.md

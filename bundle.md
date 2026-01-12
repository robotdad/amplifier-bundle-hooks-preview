---
bundle:
  name: hooks-preview
  version: 0.1.0
  description: |
    Preview bundle integrating hooks-shell, enhanced skills, and slash commands.
    Uses robotdad forks - for testing before upstream merge.

includes:
  # Foundation fork with bundle-level skills support
  - bundle: git+https://github.com/robotdad/amplifier-foundation@feat/bundle-skills-config

# Remote skill sources - loaded from git URLs
skills:
  - git+https://github.com/robotdad/skills@main
  - git+https://github.com/anthropics/skills@main

# Hooks module - Claude Code compatible shell hooks
hooks:
  - name: shell-hooks
    module: git+https://github.com/robotdad/amplifier-module-hooks-shell@main
    config:
      enabled: true

# Tools from forks with hooks integration
tools:
  # Enhanced skills with git URL sources support
  - name: load_skill
    module: git+https://github.com/robotdad/amplifier-module-tool-skills@feat/git-url-skill-sources
    config:
      skill_dirs:
        - .amplifier/skills
        - ~/.amplifier/skills

  # Extensible slash commands
  - name: slash_command
    module: git+https://github.com/robotdad/amplifier-module-tool-slash-command@main
    config:
      command_dirs:
        - .amplifier/commands
        - ~/.amplifier/commands
---

# Hooks Preview Bundle

This bundle integrates the hooks ecosystem components for testing before upstream merge.

## Components

| Component | Source | Features |
|-----------|--------|----------|
| **hooks-shell** | robotdad fork | Claude Code compatible hooks bridge |
| **tool-skills** | robotdad fork | Git URL skill sources + local dirs |
| **tool-slash-command** | robotdad fork | Extensible custom commands |
| **foundation** | robotdad fork | Bundle-level `skills:` config |

## Quick Start

### 1. Install the bundle

```bash
amplifier bundle use git+https://github.com/robotdad/amplifier-bundle-hooks-preview@main
```

### 2. Create a test hook

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

### 3. Create a custom command

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

### 4. Create a skill with hooks

```bash
mkdir -p .amplifier/skills/my-workflow
cat > .amplifier/skills/my-workflow/skill.md << 'EOF'
---
skill:
  name: my-workflow
  version: 1.0.0

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

### 5. Run Amplifier

```bash
amplifier run
```

## Event Coverage

| Claude Code Event | Amplifier Event | Status |
|-------------------|-----------------|--------|
| PreToolUse | tool:pre | ✅ |
| PostToolUse | tool:post | ✅ |
| UserPromptSubmit | prompt:submit | ✅ |
| SessionStart | session:start | ✅ |
| SessionEnd | session:end | ✅ |
| Stop | prompt:complete | ✅ |
| PreCompact | context:pre_compact | ✅ |
| PermissionRequest | approval:required | ✅ |
| Notification | user:notification | ✅ |

## Integration Points

### Hooks → Skills

Skills can define hooks in their frontmatter. When a skill is loaded, its hooks become active. When unloaded, hooks are removed.

### Hooks → Commands

Commands with `requires-approval: true` will (once integrated) emit `approval:required` events that hooks can intercept.

### Environment Persistence

Hooks can persist environment variables across executions:
```bash
echo "MY_VAR=value" >> "$AMPLIFIER_ENV_FILE"
```

## Testing

See the `examples/` directory for working examples of hooks, commands, and skills integration.

---

@hooks-preview:context/integration-guide.md

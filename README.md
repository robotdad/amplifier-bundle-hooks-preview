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

## Installation

### Recommended: Shadow Environment

The safest way to test is in an isolated shadow environment. This avoids any impact to your host Amplifier installation.

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

#### To Install for Testing

```bash
# This REPLACES your current amplifier CLI
uv tool install git+https://github.com/robotdad/amplifier-app-cli@feat/custom-slash-commands

# Verify you have the modified version
amplifier --version
```

#### To Restore Mainline Amplifier

```bash
# Reinstall from upstream to restore normal operation
uv tool install git+https://github.com/microsoft/amplifier --force

# Verify restoration
amplifier --version
```

#### Alternative: Isolated venv

If you want to keep both versions available:

```bash
# Create isolated environment for preview testing
python -m venv ~/.amplifier-preview
~/.amplifier-preview/bin/pip install git+https://github.com/robotdad/amplifier-app-cli@feat/custom-slash-commands

# Run preview version explicitly
~/.amplifier-preview/bin/amplifier run

# Your normal 'amplifier' command remains unchanged
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
        "matcher": "bash",
        "hooks": [
          {
            "type": "command", 
            "command": "echo \"[Hook] Bash: $(cat | jq -r '.input.command')\" >> /tmp/hooks.log"
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
            "command": "echo \"[Hook] Session started\" >> /tmp/hooks.log"
          }
        ]
      }
    ]
  }
}
EOF
```

### 3. Add example command

```bash
cat > .amplifier/commands/review.md << 'EOF'
---
description: Quick code review
allowed-tools: [read_file, grep, glob]
---

Review $ARGUMENTS for code quality and security issues.
EOF
```

### 4. Run and verify

```bash
# Start session
amplifier run

# In another terminal, watch hook activity
tail -f /tmp/hooks.log
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

## Known Limitations

- Requires modified app-cli (not yet upstream)
- Prompt-based hooks (Phase 2.5) not yet implemented
- Command approval integration pending

## After Testing

Once acceptance testing is complete:

1. **hooks-shell** → New repo microsoft/amplifier-module-hooks-shell
2. **tool-skills** → PR to microsoft/amplifier-module-tool-skills  
3. **tool-slash-command** → New repo microsoft/amplifier-module-tool-slash-command
4. **app-cli** → PR to microsoft/amplifier-app-cli

This bundle will then be archived or updated to pull from upstream.

## License

MIT

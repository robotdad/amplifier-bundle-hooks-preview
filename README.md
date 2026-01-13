# amplifier-bundle-hooks-preview

**Temporary preview bundle for acceptance testing** of the hooks ecosystem before upstream merge.

> **This bundle is for testing only.** It pulls from development forks and requires a modified CLI. Do not use for production work.

## Quick Start

### 1. Install the modified CLI

```bash
# This REPLACES your current amplifier CLI
uv tool install git+https://github.com/robotdad/amplifier-app-cli@feat/custom-slash-commands
```

**To verify:** Run with the bundle and try `/hello`:
```bash
amplifier run --bundle git+https://github.com/robotdad/amplifier-bundle-hooks-preview@main
# Type /hello - if it responds, you have the modified CLI
```

### 2. Run with the bundle

```bash
amplifier run --bundle git+https://github.com/robotdad/amplifier-bundle-hooks-preview@main
```

That's it! The bundle automatically loads:
- **Example commands** (`/review`, `/commit`) - via git URL, no copying needed
- **hooks-shell module** - ready for your hooks
- **enhanced skills module** - supports skill-scoped hooks

### 3. Add hooks to your project

Hooks must be local (git URL not yet supported). Copy the examples:

```bash
# Clone the bundle repo
git clone https://github.com/robotdad/amplifier-bundle-hooks-preview.git /tmp/hooks-preview

# Copy hooks to your project
mkdir -p .amplifier/hooks
cp /tmp/hooks-preview/examples/hooks/hooks.json .amplifier/hooks/
```

### 4. (Optional) Add skills with hooks

Skills must also be local. Copy the example:

```bash
cp -r /tmp/hooks-preview/examples/skills/code-guardian .amplifier/skills/
```

### 5. Verify hooks are working

```bash
# Start a session
amplifier run --bundle git+https://github.com/robotdad/amplifier-bundle-hooks-preview@main

# In another terminal, watch the hook log
tail -f /tmp/hooks-preview.log
```

## What's Included

### Commands (auto-loaded via git URL)

| Command | Description |
|---------|-------------|
| `/hello` | Simple test to verify CLI works |
| `/review` | Code review with security focus |
| `/commit` | Create commits with context (requires approval) |

### Example Hooks (`examples/hooks/hooks.json`)

Logs activity for these events:
- SessionStart, SessionEnd
- PreToolUse (bash, file operations)
- PostToolUse
- UserPromptSubmit
- Stop

Output: `/tmp/hooks-preview.log`

### Example Skill (`examples/skills/code-guardian/`)

A skill with embedded hooks that:
- Logs Python file modifications
- Warns about dangerous bash commands

Output: `/tmp/code-guardian.log`

## Features

### Hooks-Shell

| Feature | Description |
|---------|-------------|
| Command hooks | Shell scripts triggered at lifecycle events |
| Prompt hooks | LLM evaluation for complex decisions |
| Parallel execution | Run multiple hooks concurrently |
| Skill-scoped hooks | Hooks embedded in SKILL.md frontmatter |
| Pattern matching | Regex matchers for selective execution |
| Context injection | Inject feedback into agent context |

### Slash Commands

| Feature | Description |
|---------|-------------|
| Markdown commands | Define commands as `.md` files with YAML frontmatter |
| Git URL sources | Share commands from git repositories (auto-loaded!) |
| Granular permissions | Fine-grained bash control: `Bash(git status:*)` |
| Model override | Per-command model selection |
| Bash execution | `` !`command` `` runs shell during template processing |

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

## Components

| Component | Repo | Branch |
|-----------|------|--------|
| hooks-shell | robotdad/amplifier-module-hooks-shell | main |
| tool-skills | robotdad/amplifier-module-tool-skills | feat/skill-scoped-hooks |
| tool-slash-command | robotdad/amplifier-module-tool-slash-command | main |
| app-cli | robotdad/amplifier-app-cli | feat/custom-slash-commands |

## Known Limitations

- Requires modified app-cli (not yet upstream)
- Hooks and skills must be local (git URL support not yet implemented for these)
- Prompt hooks use session's default provider (model override coming)

## Restoring Mainline Amplifier

```bash
uv tool install git+https://github.com/microsoft/amplifier --force
```

## After Testing

Once acceptance testing is complete:

1. **hooks-shell** → New repo microsoft/amplifier-module-hooks-shell
2. **tool-skills** → PR to microsoft/amplifier-module-tool-skills
3. **tool-slash-command** → New repo microsoft/amplifier-module-tool-slash-command
4. **app-cli** → PR to microsoft/amplifier-app-cli

This bundle will then be archived.

## License

MIT

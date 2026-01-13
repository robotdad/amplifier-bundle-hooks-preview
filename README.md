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

### Host Installation

Installing the modified CLI on your host **will replace your current Amplifier installation**.

```bash
# This REPLACES your current amplifier CLI
uv tool install git+https://github.com/robotdad/amplifier-app-cli@feat/custom-slash-commands

# Verify you have the modified version
amplifier --version
# Should show: amplifier, version YYYY.MM.DD-<commit>
# The commit hash should match the feat/custom-slash-commands branch
```

**To verify custom commands work:**
```bash
mkdir -p /tmp/test-cmd/.amplifier/commands
echo -e "---\ndescription: test\n---\nHello" > /tmp/test-cmd/.amplifier/commands/hello.md
cd /tmp/test-cmd && amplifier run
# Type /hello - if it works, you have the modified CLI
```

**To restore mainline:**
```bash
uv tool install git+https://github.com/microsoft/amplifier --force
```

## Quick Start

### 1. Clone this bundle

```bash
git clone https://github.com/robotdad/amplifier-bundle-hooks-preview.git
cd amplifier-bundle-hooks-preview
```

### 2. Copy examples to your project

```bash
# Create a test project
mkdir -p ~/test-project
cd ~/test-project

# Copy the example hooks, commands, and skills
cp -r /path/to/amplifier-bundle-hooks-preview/examples/hooks .amplifier/
cp -r /path/to/amplifier-bundle-hooks-preview/examples/commands .amplifier/
cp -r /path/to/amplifier-bundle-hooks-preview/examples/skills .amplifier/
```

Or if you cloned to a predictable location:
```bash
cd ~/test-project
cp -r ~/amplifier-bundle-hooks-preview/examples/* .amplifier/
```

### 3. Run and verify

```bash
# Start session
amplifier run

# In another terminal, watch hook activity
tail -f /tmp/hooks-preview.log
```

## Included Examples

### `examples/hooks/hooks.json`

Comprehensive hook configuration demonstrating:
- **SessionStart** - Logs session start, detects resume
- **PreToolUse** - Logs bash commands and file operations
- **PostToolUse** - Logs tool completion
- **UserPromptSubmit** - Logs new prompts
- **Stop** - Logs response completion
- **SessionEnd** - Logs session end

All output goes to `/tmp/hooks-preview.log`.

### `examples/commands/`

| Command | Description |
|---------|-------------|
| `review.md` | Code review with security focus |
| `commit.md` | Create commits with context (uses bash, requires approval) |

### `examples/skills/code-guardian/`

A skill with embedded hooks that:
- Logs Python file modifications
- Warns about dangerous bash commands (rm -rf, sudo, chmod 777)

Output goes to `/tmp/code-guardian.log`.

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

## Git URL Command Sources

Share commands across projects by referencing git repositories:

```yaml
tools:
  - module: tool-slash-command
    config:
      commands:
        - git+https://github.com/org/shared-commands@v1
        - git+https://github.com/team/review-tools@main:commands
```

**Requirements:**
- Repo must contain a `.amplifier-commands` marker file
- Commands discovered recursively from marker location

## Known Limitations

- Requires modified app-cli (not yet upstream)
- Prompt hooks use session's default provider (model override coming)

## Skill File Format

| Requirement | Details |
|-------------|---------|
| Filename | Must be `SKILL.md` (uppercase) |
| Frontmatter | `name` and `description` at top level (not nested) |
| Directory | Should match the `name` field |

## After Testing

Once acceptance testing is complete:

1. **hooks-shell** → New repo microsoft/amplifier-module-hooks-shell
2. **tool-skills** → PR to microsoft/amplifier-module-tool-skills  
3. **tool-slash-command** → New repo microsoft/amplifier-module-tool-slash-command
4. **app-cli** → PR to microsoft/amplifier-app-cli

This bundle will then be archived or updated to pull from upstream.

## License

MIT

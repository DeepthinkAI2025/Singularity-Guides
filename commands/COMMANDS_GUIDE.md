# Singularity Commands Guide

## Complete Command Reference

### Table of Contents

1. [Core Commands](#core-commands)
2. [Server Commands](#server-commands)
3. [Management Commands](#management-commands)
4. [Session Commands](#session-commands)
5. [Integration Commands](#integration-commands)
6. [Utility Commands](#utility-commands)

---

## Core Commands

### `singularity` (Default Command)

Start the interactive TUI.

```bash
singularity [project_path] [options]
```

**Options:**

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `project_path` | string | current directory | Project directory to work in |
| `-m, --model` | string | default provider | Model to use (format: `provider/model`) |
| `-c, --continue` | boolean | false | Continue the last session |
| `-s, --session` | string | - | Continue a specific session by ID |
| `--prompt` | string | - | Initial prompt to send |
| `--agent` | string | default | Agent to use |

**Examples:**

```bash
# Start TUI in current directory
singularity

# Start TUI in specific project
singularity ~/projects/my-app

# Continue last session
singularity --continue

# Use specific model
singularity -m opencode/big-pickle "Build a REST API"

# Start with specific agent
singularity --agent coder "Fix the authentication bug"
```

---

### `singularity run`

Run a single prompt without entering the TUI.

```bash
singularity run [message..] [options]
```

**Options:**

| Option | Type | Description |
|--------|------|-------------|
| `-m, --model` | string | Model to use |
| `-a, --agent` | string | Agent to use |
| `--no-tui` | boolean | Disable TUI output |

**Examples:**

```bash
singularity run "Create a TypeScript interface for User"

singularity run "Fix the bug in auth.ts" --model opencode/big-pickle

singularity run "Write unit tests for calculateTotal" --agent tester
```

---

### `singularity attach`

Attach to a running Singularity server.

```bash
singularity attach <url>
```

**Examples:**

```bash
singularity attach http://localhost:4096

singularity attach http://singularity.local:4096
```

---

## Server Commands

### `singularity serve`

Start a headless server (no TUI).

```bash
singularity serve [options]
```

**Options:**

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `--port` | number | 0 (random) | Port to listen on |
| `--hostname` | string | 127.0.0.1 | Hostname to bind to |
| `--mdns` | boolean | false | Enable mDNS discovery |
| `--cors` | array | [] | CORS allowed domains |

**Examples:**

```bash
# Start server on default port
singularity serve

# Specific port
singularity serve --port 3000

# Enable mDNS
singularity serve --mdns --hostname 0.0.0.0

# Allow CORS
singularity serve --cors http://localhost:3000
```

**Environment Variables:**

```bash
SINGULARITY_PORT=3000 singularity serve
SINGULARITY_HOSTNAME=0.0.0.0 singularity serve
```

---

### `singularity web`

Start server with web interface.

```bash
singularity web [options]
```

**Options:** Same as `serve`

**Examples:**

```bash
singularity web --port 8080

singularity web --mdns
```

---

## Management Commands

### `singularity auth`

Manage authentication credentials.

```bash
singularity auth <subcommand> [options]
```

**Subcommands:**

#### `singularity auth login`

```bash
singularity auth login <provider>
```

**Providers:** `opencode`, `anthropic`, `openai`, `google`, `groq`, etc.

**Examples:**

```bash
singularity auth login opencode

singularity auth login anthropic
```

#### `singularity auth logout`

```bash
singularity auth logout [provider]
```

**Examples:**

```bash
# Logout from all providers
singularity auth logout

# Logout from specific provider
singularity auth logout opencode
```

#### `singularity auth providers`

List configured providers.

```bash
singularity auth providers
```

**Output:**

```
Provider: opencode
  Status: authenticated
  Models: big-pickle, haiku-4-5, opus-4-5

Provider: anthropic
  Status: not configured
```

#### `singularity auth tokens`

Manage API tokens.

```bash
singularity auth tokens list
singularity auth tokens add <name> <token>
singularity auth tokens remove <name>
```

---

### `singularity agent`

Manage agents.

```bash
singularity agent <subcommand> [options]
```

**Subcommands:**

#### `singularity agent list`

List all configured agents.

```bash
singularity agent list
```

**Output:**

```
Agents:
  default     - General purpose coding agent
  coder       - Specialized for code generation
  tester      - Specialized for writing tests
  reviewer    - Code review specialist
  debug       - Debugging and fixing issues
```

#### `singularity agent add`

Add a new agent.

```bash
singularity agent add <name> [options]
```

**Options:**

| Option | Description |
|--------|-------------|
| `--system-prompt` | System prompt for the agent |
| `--model` | Default model for the agent |
| `--tools` | Comma-separated list of tools |

**Examples:**

```bash
singularity agent add frontend-dev \
  --system-prompt "You are a frontend developer specializing in React and TypeScript" \
  --model opencode/big-pickle

singularity agent add api-expert \
  --system-prompt "You are an API design expert" \
  --model anthropic/claude-sonnet-4
```

#### `singularity agent remove`

Remove an agent.

```bash
singularity agent remove <name>
```

#### `singularity agent edit`

Edit an existing agent.

```bash
singularity agent edit <name> [options]
```

---

### `singularity mcp`

Manage MCP (Model Context Protocol) servers.

```bash
singularity mcp <subcommand> [options]
```

**Subcommands:**

#### `singularity mcp list`

List configured MCP servers.

```bash
singularity mcp list
```

**Output:**

```
MCP Servers:
  filesystem  - @modelcontextprotocol/server-filesystem
  github      - @modelcontextprotocol/server-github
  postgres    - @modelcontextprotocol/server-postgres
```

#### `singularity mcp add`

Add an MCP server.

```bash
singularity mcp add <name> <command> [args]
```

**Examples:**

```bash
# Filesystem server
singularity mcp add filesystem npx -y @modelcontextprotocol/server-filesystem /path/to/dir

# GitHub server
singularity mcp add github npx -y @modelcontextprotocol/server-github

# Custom server
singularity mcp add myserver python /path/to/server.py
```

**Environment Variables:**

```bash
singularity mcp add myserver --env GITHUB_TOKEN=$GITHUB_TOKEN
```

#### `singularity mcp remove`

Remove an MCP server.

```bash
singularity mcp remove <name>
```

#### `singularity mcp start`

Start an MCP server.

```bash
singularity mcp start <name>
```

#### `singularity mcp stop`

Stop an MCP server.

```bash
singularity mcp stop <name>
```

---

### `singularity models`

List available models.

```bash
singularity models [provider]
```

**Examples:**

```bash
# List all models
singularity models

# List specific provider models
singularity models opencode
singularity models anthropic
singularity models google
```

**Output:**

```
opencode:
  big-pickle    - Most capable model
  haiku-4-5     - Fast, efficient model
  opus-4-5      - Balanced performance

anthropic:
  claude-sonnet-4-20250514
  claude-haiku-4-20250514
  claude-opus-4-20250514
```

---

### `singularity upgrade`

Upgrade Singularity to latest version.

```bash
singularity upgrade [target]
```

**Examples:**

```bash
# Upgrade to latest stable
singularity upgrade

# Upgrade to specific version
singularity upgrade 0.0.2

# Upgrade to latest preview
singularity upgrade preview
```

---

### `singularity uninstall`

Uninstall Singularity.

```bash
singularity uninstall
```

**This will remove:**
- Singularity binary
- Configuration files
- Cache data
- Session data

**Options:**

| Option | Description |
|--------|-------------|
| `--keep-config` | Keep configuration |
| `--keep-sessions` | Keep session data |
| `--keep-cache` | Keep cache |

---

## Session Commands

### `singularity session`

Manage sessions.

```bash
singularity session <subcommand> [options]
```

#### `singularity session list`

```bash
singularity session list
```

#### `singularity session export`

```bash
singularity session export [session_id]
```

#### `singularity session import`

```bash
singularity session import <url_or_file>
```

#### `singularity session share`

```bash
singularity session share [session_id]
```

---

### `singularity export`

Export session data.

```bash
singularity export [session_id] [options]
```

**Options:**

| Option | Description |
|--------|-------------|
| `--format` | json, markdown |
| `--include-messages` | Include messages |
| `--include-files` | Include file changes |

**Examples:**

```bash
singularity export abc123

singularity export abc123 --format markdown

singularity export --include-files
```

---

### `singularity import`

Import session data.

```bash
singularity import <file_or_url>
```

**Examples:**

```bash
singularity import session.json

singularity import https://opencode.ai/share/abc123
```

---

## Integration Commands

### `singularity github`

Manage GitHub integration.

```bash
singularity github <subcommand> [options]
```

#### `singularity github login`

```bash
singularity github login
```

#### `singularity github search`

Search for code.

```bash
singularity github search <query>
```

#### `singularity github issues`

Manage issues.

```bash
singularity github issues list
singularity github issues create <title> <body>
```

---

### `singularity pr`

Manage pull requests.

```bash
singularity pr <number>
```

**Examples:**

```bash
singularity pr 123

# Fetch, checkout, and start coding
singularity pr 456 --checkout
```

---

## Utility Commands

### `singularity completion`

Generate shell completion script.

```bash
singularity completion [shell]
```

**Supported shells:** bash, zsh, fish, powershell

**Examples:**

```bash
# Bash
singularity completion bash >> ~/.bashrc

# Zsh
singularity completion zsh >> ~/.zshrc

# Fish
singularity completion fish > ~/.config/fish/completions/singularity.fish
```

---

### `singularity debug`

Debugging tools.

```bash
singularity debug <subcommand>
```

**Subcommands:**

- `logs` - View recent logs
- `config` - Show current configuration
- `health` - Check system health
- `reset` - Reset debug state

---

### `singularity stats`

Show token usage and cost statistics.

```bash
singularity stats [period]
```

**Examples:**

```bash
singularity stats           # Current month
singularity stats week      # Last week
singularity stats day       # Today
singularity stats 2025-01   # Specific month
```

---

## Global Options

Available for all commands:

| Option | Description |
|--------|-------------|
| `-h, --help` | Show help |
| `-v, --version` | Show version |
| `--print-logs` | Print logs to stderr |
| `--log-level` | DEBUG, INFO, WARN, ERROR |
| `--no-color` | Disable colors |
| `--json` | JSON output (where supported) |

---

## Exit Codes

| Code | Description |
|------|-------------|
| 0 | Success |
| 1 | General error |
| 2 | Invalid arguments |
| 3 | Authentication failed |
| 4 | Provider error |
| 5 | Session not found |
| 130 | Interrupted (Ctrl+C) |

---

## Examples

### Complete Workflow

```bash
# 1. Start a coding session
singularity "Build a REST API for user management"

# 2. Save session
/save user-api

# 3. Continue later
singularity --continue

# 4. Share with team
/share user-api

# 5. Export for documentation
/export user-api --format markdown --include-files
```

### CI/CD Usage

```bash
# Run specific task
singularity run "Fix all TypeScript errors" --log-level WARN

# Check health
singularity debug health

# View stats
singularity stats yesterday --json > stats.json
```

---

**Last Updated**: 2026-01-16

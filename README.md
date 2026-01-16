# Singularity Code - Complete Developer Guide

## Table of Contents

1. [Overview](#overview)
2. [Quick Start](#quick-start)
3. [Architecture](#architecture)
4. [Directory Structure](#directory-structure)
5. [Commands Guide](#commands-guide)
6. [Configuration](#configuration)
7. [Rules System](#rules-system)
8. [Custom Commands](#custom-commands)
9. [Plugins](#plugins)
10. [Best Practices](#best-practices)
11. [Troubleshooting](#troubleshooting)
12. [Contributing](#contributing)

---

## Overview

**Singularity Code** is a powerful AI-powered development CLI based on [OpenCode](https://github.com/anomalyco/opencode). It provides an interactive terminal-based coding experience with support for multiple AI providers, MCP servers, agents, and more.

### Key Features

- **Interactive TUI**: Beautiful terminal UI built with SolidJS + OpenTUI
- **Multi-Provider Support**: OpenAI, Anthropic, Google, Groq, and many more
- **MCP Integration**: Model Context Protocol server support
- **Agent System**: Built-in agents for various tasks
- **Session Management**: Save, share, and resume coding sessions
- **Plugin System**: Extensible architecture for custom functionality

---

## Quick Start

### Installation

```bash
# Clone the repository
git clone https://github.com/DeepthinkAI2025/singularity-code.git
cd singularity-code

# Install dependencies
bun install

# Run in development mode
bun dev

# Build for production
cd packages/opencode && bun run build --single

# Install globally
cp dist/singularity-darwin-arm64/bin/singularity ~/.local/bin/
export PATH="$HOME/.local/bin:$PATH"
```

### Basic Usage

```bash
# Start interactive TUI
singularity

# Run a single command
singularity "Build a React component for user login"

# Continue previous session
singularity --continue

# Use specific model
singularity -m anthropic/claude-sonnet-4 "Fix the bug in auth.ts"
```

---

## Architecture

Singularity is built on a modular architecture inspired by OpenCode:

```
singularity-code/
├── packages/
│   ├── opencode/           # Core CLI application
│   │   ├── src/
│   │   │   ├── cli/        # CLI commands and TUI
│   │   │   ├── server/     # HTTP server
│   │   │   ├── session/    # Session management
│   │   │   ├── provider/   # AI provider integration
│   │   │   ├── plugin/     # Plugin system
│   │   │   └── ...
│   │   └── bin/            # Binary entry point
│   ├── sdk/                # TypeScript SDK
│   ├── plugin/             # Plugin API
│   ├── script/             # Build scripts
│   ├── ui/                 # Shared UI components
│   ├── app/                # App wrapper
│   └── ...
├── .singularity/           # Singularity configuration
├── singularity.json        # User configuration
└── CLAUDE.md              # AI assistant context
```

### Core Components

1. **CLI Layer** (`packages/opencode/src/cli/`)
   - Entry point for all commands
   - TUI rendering and interaction
   - Command parsing and execution

2. **Server Layer** (`packages/opencode/src/server/`)
   - HTTP API for TUI communication
   - Session state management
   - Provider orchestration

3. **Session Layer** (`packages/opencode/src/session/`)
   - Message handling
   - Retry logic
   - Prompt management

4. **Provider Layer** (`packages/opencode/src/provider/`)
   - AI provider abstraction
   - Authentication
   - Model selection

---

## Directory Structure

### Project Root

```
singularity-code/
├── .claude/                # Claude Code configuration
│   ├── EXECUTORS/          # Automation executors
│   ├── CONFIGS/            # Configuration files
│   └── CLAUDE.md           # Master configuration
├── .github/                # GitHub workflows
├── packages/               # Monorepo packages
├── themes/                 # UI themes
├── nix/                    # Nix configuration
└── bun.lock                # Dependency lock file
```

### Core Package (`packages/opencode/`)

```
packages/opencode/
├── src/
│   ├── index.ts           # CLI entry point
│   ├── acp/               # Agent Client Protocol
│   ├── bun/               # Bun-specific utilities
│   ├── cli/               # CLI commands
│   │   ├── cmd/
│   │   │   ├── tui/       # TUI components
│   │   │   ├── run.ts     # Run command
│   │   │   ├── auth.ts    # Auth command
│   │   │   ├── agent.ts   # Agent command
│   │   │   ├── mcp.ts     # MCP command
│   │   │   └── ...
│   │   └── ui.ts          # CLI UI utilities
│   ├── installation/      # Installation management
│   ├── lsp/               # LSP integration
│   ├── mcp/               # MCP client
│   ├── plugin/            # Plugin system
│   ├── project/           # Project management
│   ├── provider/          # AI providers
│   ├── server/            # HTTP server
│   ├── session/           # Session management
│   ├── share/             # Sharing functionality
│   ├── storage/           # Storage utilities
│   ├── tool/              # Tool registry
│   └── util/              # Utilities
├── script/                # Build scripts
├── bin/                   # Binary scripts
├── AGENTS.md             # Agent documentation
└── package.json
```

### Configuration Locations

```
~/.config/singularity/     # Global config (Linux)
~/Library/Application Support/singularity/  # macOS
%APPDATA%/singularity/     # Windows

.singularity/              # Project-local config
  ├── plans/              # Execution plans
  ├── session/            # Session data
  └── singularity.json    # Project config
```

---

## Commands Guide

### Core Commands

```bash
# Start interactive TUI
singularity [project_path]

# Run a single prompt
singularity run "Your prompt here"

# Attach to running server
singularity attach <url>

# Manage authentication
singularity auth login
singularity auth logout
singularity auth providers

# Manage agents
singularity agent list
singularity agent add <name>
singularity agent remove <name>

# MCP server management
singularity mcp list
singularity mcp add <name> <command>
singularity mcp remove <name>

# List available models
singularity models
singularity models opencode

# Server commands
singularity serve           # Headless server
singularity web           # Server + web interface

# Session management
singularity session list
singularity session export <id>
singularity session import <url>

# Import/Export
singularity import <file_or_url>
singularity export [session_id]
```

### Global Options

```bash
-h, --help           Show help
-v, --version        Show version
--print-logs         Print logs to stderr
--log-level          DEBUG, INFO, WARN, ERROR
-m, --model          Provider/model (e.g., opencode/big-pickle)
-c, --continue       Continue last session
-s, --session        Continue specific session
--prompt             Prompt to use
--agent              Agent to use
--port               Server port (default: 0 = random)
--hostname           Server hostname
--mdns               Enable mDNS discovery
--cors               CORS domains
```

---

## Configuration

### Global Configuration (`~/.config/singularity/singularity.json`)

```json
{
  "version": "1.0",
  "provider": {
    "default": "opencode",
    "opencode": {
      "apiKey": "sk-...",
      "models": {
        "default": "big-pickle"
      }
    }
  },
  "theme": "default",
  "keybinds": {
    "ctrl+c": "cancel",
    "ctrl+a": "accept",
    "ctrl+n": "next"
  },
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path"]
    }
  },
  "agents": {
    "default": "general"
  },
  "tui": {
    "theme": "default",
    "animations": true
  }
}
```

### Project Configuration (`.singularity/singularity.json`)

```json
{
  "version": "1.0",
  "provider": {
    "default": "opencode"
  },
  "agent": "default",
  "model": "opencode/big-pickle",
  "rules": [".singularity/rules/*.md"],
  "ignore": ["node_modules", ".git", "dist"]
}
```

### Environment Variables

```bash
OPENCODE_API_KEY          # API key for opencode provider
ANTHROPIC_API_KEY         # Anthropic API key
OPENAI_API_KEY            # OpenAI API key
SINGULARITY_PORT          # Default port
SINGULARITY_HOSTNAME      # Default hostname
SINGULARITY_BIN_PATH      # Path to singularity binary
```

---

## Rules System

Singularity uses a rules-based system to guide AI behavior.

### Rule Files

Rules are Markdown files that define how Singularity should behave:

```
.singularity/rules/
├── project-rules.md       # Project-specific rules
├── language-rules.md      # Language-specific rules
├── style-rules.md         # Code style rules
└── best-practices.md      # Best practices
```

### Rule File Structure

````markdown
# Rule Name

## Description

Brief description of the rule

## When to Apply

- JavaScript projects
- TypeScript projects

## Rules

### Rule 1: Use TypeScript

Always prefer TypeScript over JavaScript for type safety.

```typescript
// Good
interface User {
  id: string;
  name: string;
}

// Bad
const user = { id: 1, name: "John" };
```
````

### Rule 2: Error Handling

Always implement proper error handling.

```typescript
// Good
try {
  await fetchData();
} catch (error) {
  logger.error("Failed to fetch data", { error });
}
```

## Examples

### Example 1

Description...

## References

- [TypeScript Handbook](https://www.typescriptlang.org/docs/)

````

### Built-in Rules

Singularity includes several built-in rule sets:

1. **General Rules** - Best practices for all projects
2. **TypeScript Rules** - TypeScript-specific guidelines
3. **Testing Rules** - Testing best practices
4. **Security Rules** - Security considerations
5. **Performance Rules** - Performance optimization

### Using Rules

```bash
# Run with specific rules
singularity --rules .singularity/rules/*.md "Build a REST API"

# Disable rules
singularity --no-rules "Quick task"
````

---

## Custom Commands

### Creating Custom Commands

Commands are defined in `packages/opencode/src/cli/cmd/`:

```typescript
// packages/opencode/src/cli/cmd/mycommand.ts
import { Command } from "yargs";

export const MyCommand: Command = {
  command: "mycommand",
  describe: "Description of my command",
  builder: (yargs) =>
    yargs.option("option1", {
      describe: "First option",
      type: "string",
      default: "value",
    }),
  handler: async (argv) => {
    console.log("Command executed with:", argv);
    // Your command logic here
  },
};
```

### Registering Commands

Add your command to `packages/opencode/src/index.ts`:

```typescript
import { MyCommand } from "./cli/cmd/mycommand";

const cli = yargs(hideBin(process.argv))
  // ... other commands
  .command(MyCommand);
```

### Command Structure

```typescript
interface Command {
  command: string; // Command name (with aliases)
  describe: string; // Short description
  aliases?: string[]; // Command aliases
  builder?: (yargs: Argv) => Argv; // Option builder
  handler: (argv: Args) => void | Promise<void>; // Command handler
}
```

### Example: Complete Command

```typescript
// packages/opencode/src/cli/cmd/greet.ts
import { Command } from "yargs";

interface Args {
  name: string;
  formal: boolean;
  count: number;
}

export const GreetCommand: Command = {
  command: "greet [name]",
  describe: "Greet the user",
  aliases: ["hello", "hi"],
  builder: (yargs) =>
    yargs
      .positional("name", {
        describe: "Name to greet",
        type: "string",
        default: "World",
      })
      .option("formal", {
        describe: "Use formal greeting",
        type: "boolean",
        default: false,
      })
      .option("count", {
        describe: "Number of greetings",
        type: "number",
        default: 1,
      }),
  handler: async (argv: Args) => {
    const greeting = argv.formal
      ? `Good day, ${argv.name}`
      : `Hello, ${argv.name}!`;

    for (let i = 0; i < argv.count; i++) {
      console.log(greeting);
    }
  },
};
```

---

## Plugins

### Plugin Architecture

Singularity's plugin system is based on OpenCode's plugin API, allowing you to extend functionality.

### Using Plugins

```bash
# Install plugin
singularity plugin add <plugin-name>

# List plugins
singularity plugin list

# Remove plugin
singularity plugin remove <plugin-name>
```

### Plugin Development

#### Plugin Structure

```
my-plugin/
├── package.json
├── src/
│   ├── index.ts        # Plugin entry point
│   ├── tools/          # Custom tools
│   └── hooks/          # Lifecycle hooks
└── README.md
```

#### Plugin Entry Point

```typescript
// src/index.ts
import type { Plugin, PluginInput } from "@singularity-ai/plugin";

export default {
  name: "my-plugin",
  version: "1.0.0",

  tools: {
    // Custom tools
    myTool: {
      description: "My custom tool",
      parameters: {
        input: { type: "string", description: "Input" },
      },
      handler: async (input: string) => {
        return { result: input.toUpperCase() };
      },
    },
  },

  hooks: {
    onLoad: (input: PluginInput) => {
      console.log("Plugin loaded!");
    },
    onPrompt: (prompt) => {
      return prompt + "\n[Additional context]";
    },
  },
} satisfies Plugin;
```

### Official Plugins

- **MCP Integration** - Model Context Protocol support
- **GitHub Integration** - GitHub API tools
- **File System Tools** - File operations
- **Web Search** - Web search capabilities

### External Plugin Resources

- [awesome-opencode](https://github.com/awesome-opencode/awesome-opencode) - Community plugins, themes, agents, and resources for opencode/singularity
- [Singularity Plugins](https://github.com/DeepthinkAI2025/SingularityPlugins) - Our custom plugins

### Adding Features from Awesome OpenCode

Since Singularity Code is based on OpenCode, you can integrate features and plugins from the [Awesome OpenCode](https://github.com/awesome-opencode/awesome-opencode) repository:

1. **Browse the repository** for plugins, agents, themes, and tools
2. **Check compatibility** - Most OpenCode plugins work with Singularity Code
3. **Install via npm** or copy code:
   ```bash
   bun add awesome-opencode-plugin-name
   ```
4. **Add to configuration**:
   ```json
   {
     "plugin": ["awesome-opencode-plugin-name"]
   }
   ```
5. **Copy and adapt** code from OpenCode features into Singularity structure
6. **Test integration** to ensure it works with Singularity branding

Always prefer copy-paste from OpenCode to maintain best practices and 2026 features.

---

## Best Practices

### Code Style

1. **TypeScript First**
   - Always use TypeScript
   - Enable strict mode
   - Use proper types

2. **Error Handling**
   - Always use try/catch
   - Log errors with context
   - Handle promise rejections

3. **Async/Await**
   - Prefer async/await over callbacks
   - Handle async errors properly
   - Use Promise.all for parallel operations

4. **Modularity**
   - Keep functions small and focused
   - Use dependency injection
   - Follow single responsibility principle

### Project Structure

```
src/
├── features/        # Feature-based organization
├── core/           # Core business logic
├── shared/         # Shared utilities
├── types/          # TypeScript types
└── utils/          # Helper functions
```

### Testing

```typescript
// Always test async code
describe("myFunction", () => {
  it("should handle async operations", async () => {
    const result = await myFunction();
    expect(result).toBeDefined();
  });
});
```

### Security

- Never commit API keys
- Use environment variables
- Validate all inputs
- Follow least privilege principle

---

## Troubleshooting

### Common Issues

#### Installation Fails

```bash
# Check Bun version
bun --version

# Requires Bun 1.3+
bun upgrade

# Clear cache
rm -rf node_modules .bun
bun install
```

#### CLI Won't Start

```bash
# Check logs
singularity --print-logs --log-level DEBUG

# Reset configuration
rm -rf ~/.config/singularity
singularity
```

#### Provider Errors

```bash
# Check API keys
echo $OPENCODE_API_KEY

# Test provider connection
singularity models opencode
```

#### TUI Issues

```bash
# Try without TUI
singularity run "test" --log-level INFO

# Reset TUI cache
rm -rf ~/.config/singularity/cache
```

### Getting Help

```bash
# Built-in help
singularity --help
singularity <command> --help

# Check logs
singularity --print-logs

# Report issues
# https://github.com/DeepthinkAI2025/singularity-code/issues
```

---

## Contributing

### Development Setup

1. **Fork the repository**
2. **Clone your fork**

   ```bash
   git clone https://github.com/YOUR-USERNAME/singularity-code.git
   cd singularity-code
   ```

3. **Install dependencies**

   ```bash
   bun install
   ```

4. **Start development server**

   ```bash
   bun dev
   ```

5. **Run tests**
   ```bash
   bun test
   ```

### Adding Features

1. Create a feature branch
2. Make changes following existing patterns
3. Add tests for new functionality
4. Run lint and typecheck
5. Submit pull request

### Code Style

- Follow TypeScript strict mode
- Use Prettier for formatting
- Write meaningful comments
- Add JSDoc documentation

### Pull Request Guidelines

- Describe changes clearly
- Reference related issues
- Include screenshots for UI changes
- Ensure tests pass
- Update documentation

---

## Resources

### Official Links

- **Repository**: https://github.com/DeepthinkAI2025/singularity-code
- **Guides**: https://github.com/DeepthinkAI2025/Singularity-Guides
- **Plugins**: https://github.com/DeepthinkAI2025/SingularityPlugins
- **Issues**: https://github.com/DeepthinkAI2025/singularity-code/issues

### Based On

- **OpenCode**: https://github.com/anomalyco/opencode
- **OpenCode Documentation**: https://opencode.ai/docs
- **Awesome OpenCode**: https://github.com/anomalyco/awesome-opencode

### Community

- **Discord**: Join our community
- **GitHub Discussions**: Ask questions

---

## License

Singularity Code is based on OpenCode and follows the same MIT License.

---

**Last Updated**: 2026-01-16
**Version**: 0.0.1

---

## Phase 1 Implementation Notes

The following Phase 1 features are currently being implemented in the main Singularity Code repository:

- [x] Create Phase 1 implementation directory structure (ENGINE/ modules/)
- [x] Implement Anti-Gravity Mode CLI command in singularity-master-cli.js
- [ ] Create Free-Only API Router (multi-provider-router.js)
- [ ] Setup DeepSeek Free Tier integration (1M tokens/month)
- [ ] Create M1-native build scripts (build:m1, benchmark:m1)
- [ ] Implement Zero-Leak Security Layer (OWASP compliant)
- [ ] Create 50+ Agent System architecture
- [ ] Test all implementations on Mac M1
- [ ] Update GitHub repository with Phase 1 code

See `opencode/plans/AUTONOMOUS-CEO-T+495` for detailed implementation plans.

# Singularity Plugins Guide

## Overview

Singularity's plugin system, based on OpenCode's plugin architecture, allows you to extend functionality with custom tools, hooks, and integrations.

---

## Table of Contents

1. [Using Plugins](#using-plugins)
2. [Plugin Development](#plugin-development)
3. [Plugin Structure](#plugin-structure)
4. [Tools](#tools)
5. [Hooks](#hooks)
6. [Official Plugins](#official-plugins)
7. [Community Plugins](#community-plugins)

---

## Using Plugins

### Installation

```bash
# Install from npm
singularity plugin add @singularity-ai/plugin-name

# Install from local path
singularity plugin add ./my-plugin

# Install from GitHub
singularity plugin add github:username/repo
```

### Management

```bash
# List installed plugins
singularity plugin list

# Update plugin
singularity plugin update <name>

# Remove plugin
singularity plugin remove <name>

# Disable plugin
singularity plugin disable <name>

# Enable plugin
singularity plugin enable <name>
```

### Configuration

```json
{
  "plugins": {
    "enabled": ["@singularity-ai/filesystem", "@singularity-ai/github"],
    "disabled": ["experimental-plugin"],
    "config": {
      "@singularity-ai/github": {
        "token": "ghp_..."
      }
    }
  }
}
```

---

## Plugin Development

### Quick Start

```bash
# Create new plugin
singularity plugin create my-plugin

# Directory structure
my-plugin/
├── package.json
├── tsconfig.json
├── src/
│   ├── index.ts
│   ├── tools/
│   └── hooks/
└── README.md
```

### Package.json

```json
{
  "name": "@username/my-plugin",
  "version": "1.0.0",
  "main": "dist/index.js",
  "types": "dist/index.d.ts",
  "scripts": {
    "build": "tsc",
    "dev": "tsc --watch"
  },
  "dependencies": {
    "@singularity-ai/plugin": "^1.0.0"
  },
  "devDependencies": {
    "typescript": "^5.0.0"
  }
}
```

---

## Plugin Structure

### Basic Plugin

```typescript
// src/index.ts
import type { Plugin, PluginInput } from "@singularity-ai/plugin"

export default {
  name: "my-plugin",
  version: "1.0.0",
  description: "My custom plugin for Singularity",

  // Tools provided by this plugin
  tools: {
    myTool: {
      description: "Description of my tool",
      parameters: {
        input: {
          type: "string",
          description: "The input parameter"
        }
      },
      handler: async (input: string) => {
        // Tool implementation
        return { result: input.toUpperCase() }
      }
    }
  },

  // Hooks for lifecycle events
  hooks: {
    onLoad: (input: PluginInput) => {
      console.log("Plugin loaded!")
    },

    onPrompt: (prompt: string) => {
      return prompt + "\n[Additional context from plugin]"
    },

    onComplete: (result: any) => {
      console.log("Task completed:", result)
    }
  }
} satisfies Plugin
```

### Plugin Metadata

```typescript
interface PluginMetadata {
  name: string              // Plugin name (required)
  version: string           // Version (required)
  description?: string      // Short description
  author?: string           // Author name
  license?: string          // License (MIT, Apache, etc.)
  repository?: string       // GitHub repo URL
  homepage?: string         // Plugin homepage
  keywords?: string[]       // npm keywords
}
```

---

## Tools

### Tool Definition

```typescript
interface ToolDefinition {
  description: string              // What the tool does
  parameters: {                    // Parameters schema
    [key: string]: {
      type: "string" | "number" | "boolean" | "object" | "array"
      description: string
      optional?: boolean
      default?: any
      enum?: string[]
    }
  }
  handler: (args: Record<string, any>) => Promise<any> | any
}
```

### Complete Tool Example

```typescript
// src/tools/files.ts
import type { Tool } from "@singularity-ai/plugin"

export const fileReadTool: Tool = {
  description: "Read the contents of a file",

  parameters: {
    path: {
      type: "string",
      description: "Path to the file to read"
    },
    encoding: {
      type: "string",
      description: "File encoding (default: utf-8)",
      optional: true,
      default: "utf-8"
    },
    maxLines: {
      type: "number",
      description: "Maximum number of lines to read",
      optional: true
    }
  },

  handler: async ({ path, encoding, maxLines }) => {
    // Check file exists
    const file = await Bun.file(path)
    if (!file.exists()) {
      throw new Error(`File not found: ${path}`)
    }

    // Read file
    let content = await file.text()

    // Apply max lines
    if (maxLines) {
      const lines = content.split("\n")
      content = lines.slice(0, maxLines).join("\n")
    }

    return {
      path,
      content,
      encoding,
      size: content.length
    }
  }
}
```

### Tool Categories

1. **File Operations** - Read, write, list files
2. **Network** - HTTP requests, API calls
3. **Database** - Query, insert, update
4. **Git** - Commit, branch, merge
5. **Search** - Find files, grep content
6. **System** - Execute commands, environment

### Tool Error Handling

```typescript
handler: async (args) => {
  try {
    // Tool logic
    return { success: true, data: result }
  } catch (error) {
    return {
      success: false,
      error: error instanceof Error ? error.message : "Unknown error"
    }
  }
}
```

---

## Hooks

### Available Hooks

```typescript
interface PluginHooks {
  // Called when plugin is loaded
  onLoad?: (input: PluginInput) => void

  // Called before each prompt
  onPrompt?: (prompt: string) => string

  // Called after each AI response
  onResponse?: (response: Response) => Response

  // Called when task completes
  onComplete?: (result: any) => void

  // Called on error
  onError?: (error: Error) => void

  // Called when session starts
  onSessionStart?: (session: Session) => void

  // Called when session ends
  onSessionEnd?: (session: Session) => void

  // Custom configuration
  configure?: (config: PluginConfig) => PluginConfig
}
```

### Hook Examples

#### onLoad Hook

```typescript
hooks: {
  onLoad: (input: PluginInput) => {
    // Initialize plugin state
    input.state.set("my-plugin", { initialized: true })

    // Setup event listeners
    input.events.on("prompt", handlePrompt)

    console.log("My plugin loaded!")
  }
}
```

#### onPrompt Hook

```typescript
hooks: {
  onPrompt: (prompt: string) => {
    // Add context to prompt
    if (prompt.includes("file")) {
      return prompt + "\n\nRemember to use absolute paths."
    }
    return prompt
  }
}
```

#### onResponse Hook

```typescript
hooks: {
  onResponse: (response) => {
    // Process AI response
    if (response.content.includes("error")) {
      // Add error handling suggestions
      response.content += "\n\nTry checking the logs for more details."
    }
    return response
  }
}
```

---

## Official Plugins

### MCP Integration

```bash
singularity plugin add @singularity-ai/mcp
```

**Features:**
- Connect to MCP servers
- Use MCP tools in your prompts
- Manage MCP server lifecycle

**Configuration:**
```json
{
  "plugins": {
    "config": {
      "@singularity-ai/mcp": {
        "servers": {
          "filesystem": {
            "command": "npx",
            "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path"]
          }
        }
      }
    }
  }
}
```

### GitHub Integration

```bash
singularity plugin add @singularity-ai/github
```

**Features:**
- Search repositories
- Manage issues and PRs
- Create commits
- Review code

### File System Tools

```bash
singularity plugin add @singularity-ai/filesystem
```

**Tools:**
- `file_read` - Read file contents
- `file_write` - Write to files
- `file_list` - List directory contents
- `file_search` - Search for files
- `file_grep` - Search file contents

---

## Community Plugins

### awesome-opencode

Find more plugins at: https://github.com/anomalyco/awesome-opencode

### Singularity Plugins

Our custom plugins: https://github.com/DeepthinkAI2025/SingularityPlugins

**Available Plugins:**
- `@singularity-ai/database` - Database operations
- `@singularity-ai/docker` - Docker container management
- `@singularity-ai/kubernetes` - K8s cluster management
- `@singularity-ai/api-gateway` - API documentation

---

## Plugin Development Best Practices

### 1. Error Handling

```typescript
// Always handle errors gracefully
handler: async (args) => {
  try {
    return await riskyOperation(args)
  } catch (error) {
    console.error("Tool failed:", error)
    return {
      success: false,
      error: error instanceof Error ? error.message : "Unknown error"
    }
  }
}
```

### 2. Type Safety

```typescript
// Use TypeScript for all code
interface MyToolArgs {
  input: string
  count?: number
}

handler: async (args: MyToolArgs) => {
  const input = args.input ?? "default"
  const count = args.count ?? 1
  // ...
}
```

### 3. Async Operations

```typescript
// Use async/await
handler: async (args) => {
  const result = await database.query(args)
  return result
}

// Handle parallel operations
handler: async (args) => {
  const [users, posts] = await Promise.all([
    getUsers(),
    getPosts()
  ])
  return { users, posts }
}
```

### 4. Logging

```typescript
hooks: {
  onLoad: (input) => {
    console.log("[my-plugin] Loaded successfully")
  },
  onComplete: (result) => {
    console.log("[my-plugin] Completed with:", result)
  }
}
```

### 5. Testing

```typescript
// __tests__/plugin.test.ts
describe("my-plugin", () => {
  it("should handle tool calls", async () => {
    const plugin = await import("../src/index")
    const result = await plugin.tools.myTool.handler({ input: "test" })
    expect(result).toEqual({ result: "TEST" })
  })
})
```

---

## Publishing Plugins

### 1. Prepare for Publishing

```json
// package.json
{
  "name": "@username/my-plugin",
  "version": "1.0.0",
  "main": "dist/index.js",
  "types": "dist/index.d.ts",
  "files": ["dist"],
  "publishConfig": {
    "access": "public"
  }
}
```

### 2. Build and Test

```bash
npm run build
npm test
npm publish
```

### 3. Document Your Plugin

```markdown
# My Plugin

Brief description.

## Installation

```bash
singularity plugin add @username/my-plugin
```

## Usage

```typescript
// Example usage
await singularity.run("Use my tool", {
  tool: "my-tool",
  input: "value"
})
```

## Configuration

```json
{
  "plugins": {
    "config": {
      "@username/my-plugin": {
        "option1": "value"
      }
    }
  }
}
```
```

---

## Troubleshooting

### Plugin Won't Load

```bash
# Check plugin logs
singularity --print-logs --log-level DEBUG

# Verify installation
singularity plugin list

# Reinstall plugin
singularity plugin remove my-plugin
singularity plugin add my-plugin
```

### Type Errors

```bash
# Check TypeScript compilation
cd my-plugin
npm run build

# Common issues:
# - Missing type definitions
# - Incorrect exports
# - Wrong TypeScript version
```

---

## Resources

### Documentation

- [OpenCode Plugin Docs](https://opencode.ai/docs/plugins)
- [Plugin API Reference](#)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)

### Examples

See these plugins for reference:

1. [Official Plugins](https://github.com/anomalyco/opencode/tree/main/packages/plugin)
2. [Community Plugins](https://github.com/anomalyco/awesome-opencode)
3. [Our Plugins](https://github.com/DeepthinkAI2025/SingularityPlugins)

---

## Version Compatibility

| Plugin Version | Singularity Version |
|----------------|---------------------|
| 1.0.x          | 0.0.x               |
| 1.1.x          | 0.1.x               |
| 2.0.x          | 1.0.x               |

---

**Last Updated**: 2026-01-16

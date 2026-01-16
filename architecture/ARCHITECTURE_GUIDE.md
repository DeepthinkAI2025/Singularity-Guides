# Singularity Architecture Guide

## Overview

This guide explains Singularity's architecture, how it's structured, and how components interact. Singularity is built as a modular monorepo inspired by OpenCode.

---

## Table of Contents

1. [High-Level Architecture](#high-level-architecture)
2. [Package Structure](#package-structure)
3. [Core Components](#core-components)
4. [Data Flow](#data-flow)
5. [Extensibility Points](#extensibility-points)
6. [Scaling Considerations](#scaling-considerations)

---

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        Singularity CLI                          │
├─────────────────────────────────────────────────────────────────┤
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐        │
│  │   CLI    │  │   TUI    │  │  Server  │  │  Config  │        │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘        │
│       │             │             │             │               │
│       └─────────────┴──────┬──────┴─────────────┘               │
│                            │                                    │
│                    ┌───────┴───────┐                            │
│                    │   Session     │                            │
│                    │   Manager     │                            │
│                    └───────┬───────┘                            │
│                            │                                    │
│       ┌────────────────────┼────────────────────┐               │
│       │                    │                    │               │
│  ┌────┴────┐         ┌─────┴─────┐        ┌────┴────┐          │
│  │ Provider │         │  Plugins  │        │   MCP   │          │
│  │  Layer   │         │   System  │        │  Client │          │
│  └────┬────┘         └─────┬─────┘        └────┬────┘          │
│       │                    │                    │               │
│  ┌────┴────────────────────┴────────────────────┴────┐         │
│  │              AI Provider Integration               │         │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ │         │
│  │  │ OpenCode│ │Claude   │ │  OpenAI │ │  Groq   │ │         │
│  │  └─────────┘ └─────────┘ └─────────┘ └─────────┘ │         │
│  └──────────────────────────────────────────────────┘         │
└─────────────────────────────────────────────────────────────────┘
```

---

## Package Structure

```
singularity-code/
├── packages/
│   ├── opencode/              # Core CLI application
│   │   ├── src/
│   │   │   ├── index.ts       # CLI entry point
│   │   │   ├── acp/           # Agent Client Protocol
│   │   │   ├── bun/           # Bun utilities
│   │   │   ├── cli/           # Command-line interface
│   │   │   │   ├── cmd/       # Individual commands
│   │   │   │   └── ui.ts      # UI utilities
│   │   │   ├── installation/  # Installation management
│   │   │   ├── lsp/           # Language Server Protocol
│   │   │   ├── mcp/           # MCP client
│   │   │   ├── plugin/        # Plugin system
│   │   │   ├── project/       # Project management
│   │   │   ├── provider/      # AI provider abstraction
│   │   │   ├── server/        # HTTP server
│   │   │   ├── session/       # Session management
│   │   │   ├── share/         # Sharing functionality
│   │   │   ├── storage/       # Storage utilities
│   │   │   ├── tool/          # Tool registry
│   │   │   └── util/          # General utilities
│   │   ├── script/            # Build scripts
│   │   └── bin/               # Binary entry points
│   │
│   ├── sdk/                   # TypeScript SDK
│   │   ├── src/
│   │   │   ├── index.ts       # Main SDK entry
│   │   │   ├── client.ts      # Client implementation
│   │   │   └── v2/            # V2 API
│   │   └── package.json
│   │
│   ├── plugin/                # Plugin API
│   │   ├── src/
│   │   │   ├── index.ts       # Plugin interface
│   │   │   └── tool.ts        # Tool definitions
│   │   └── package.json
│   │
│   ├── script/                # Build scripts
│   │   ├── src/
│   │   │   ├── index.ts       # Main script entry
│   │   │   └── build.ts       # Build logic
│   │   └── package.json
│   │
│   ├── ui/                    # Shared UI components
│   │   ├── src/
│   │   │   ├── components/    # React/Solid components
│   │   │   ├── hooks/         # Custom hooks
│   │   │   ├── context/       # React contexts
│   │   │   ├── styles/        # CSS styles
│   │   │   └── theme/         # Theming
│   │   └── package.json
│   │
│   ├── app/                   # App wrapper
│   │   ├── src/
│   │   │   ├── index.tsx      # App entry
│   │   │   └── vite.ts        # Vite config
│   │   └── package.json
│   │
│   ├── desktop/               # Desktop app (Tauri)
│   ├── web/                   # Web documentation
│   ├── console/               # Console API
│   ├── util/                  # Shared utilities
│   ├── function/              # Cloud Functions
│   ├── slack/                 # Slack integration
│   └── enterprise/            # Enterprise features
│
├── themes/                    # UI themes
├── nix/                       # Nix configuration
├── .claude/                   # Claude Code config
├── CLAUDE.md                  # Master config
└── package.json               # Root package.json
```

---

## Core Components

### 1. CLI Layer (`packages/opencode/src/cli/`)

**Responsibilities:**
- Parse command-line arguments
- Route to appropriate handlers
- Manage TUI rendering
- Handle user input

**Key Files:**
```
cli/
├── cmd/
│   ├── run.ts         # Run command
│   ├── auth.ts        # Authentication
│   ├── agent.ts       # Agent management
│   ├── mcp.ts         # MCP server management
│   ├── tui/           # TUI components
│   └── ...
├── ui.ts              # UI utilities
└── error.ts           # Error handling
```

**Flow:**
```
User Input → Argument Parser → Command Router → Handler → Output
```

### 2. Server Layer (`packages/opencode/src/server/`)

**Responsibilities:**
- HTTP API endpoints
- WebSocket connections
- Session state management
- Provider orchestration

**Key Files:**
```
server/
├── index.ts           # Server entry
├── router.ts          # API routes
├── ws.ts              # WebSocket handler
└── middleware/
    ├── auth.ts        # Authentication
    ├── logging.ts     # Request logging
    └── cors.ts        # CORS handling
```

**Endpoints:**
```
GET  /health          # Health check
GET  /models          # List models
POST /session/create  # Create session
POST /session/prompt  # Send prompt
WS   /ws              # WebSocket connection
```

### 3. Session Layer (`packages/opencode/src/session/`)

**Responsibilities:**
- Message handling
- Conversation history
- Retry logic
- State persistence

**Key Files:**
```
session/
├── index.ts           # Session management
├── message.ts         # Message types
├── prompt.ts          # Prompt handling
├── retry.ts           # Retry logic
└── llm.ts             # LLM integration
```

**Data Flow:**
```
User Prompt → Message → Context Builder → LLM → Response → Message → UI
```

### 4. Provider Layer (`packages/opencode/src/provider/`)

**Responsibilities:**
- AI provider abstraction
- API authentication
- Model selection
- Rate limiting

**Key Files:**
```
provider/
├── index.ts           # Provider interface
├── auth.ts            # Authentication
├── opencode.ts        # OpenCode provider
├── anthropic.ts       # Anthropic provider
├── openai.ts          # OpenAI provider
└── registry.ts        # Provider registry
```

**Provider Interface:**
```typescript
interface Provider {
  name: string
  models: Model[]
  authenticate(auth: Auth): Promise<void>
  complete(prompt: string, options: Options): Promise<Response>
  stream(prompt: string, options: Options): AsyncIterator<Response>
}
```

### 5. Plugin System (`packages/plugin/`)

**Responsibilities:**
- Plugin discovery
- Tool registration
- Hook execution
- Lifecycle management

**Key Files:**
```
plugin/
├── index.ts           # Plugin interface
├── tool.ts            # Tool definitions
├── registry.ts        # Plugin registry
└── hooks.ts           # Hook system
```

### 6. MCP Client (`packages/opencode/src/mcp/`)

**Responsibilities:**
- MCP server connections
- Tool discovery
- Request/response handling

---

## Data Flow

### 1. Single Prompt Flow

```
┌─────────────┐
│ User Input  │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│  CLI Parse  │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│  Session    │
│  Manager    │
└──────┬──────┘
       │
       ▼
┌─────────────┐     ┌─────────────┐
│   Rules     │     │   Plugins   │
│  Engine     │     │   Hooks     │
└──────┬──────┘     └──────┬──────┘
       │                   │
       └─────────┬─────────┘
                 │
                 ▼
          ┌─────────────┐
          │   Context   │
          │   Builder   │
          └──────┬──────┘
                 │
                 ▼
          ┌─────────────┐
          │   Provider  │
          │   Layer     │
          └──────┬──────┘
                 │
                 ▼
          ┌─────────────┐
          │    LLM      │
          │  Response   │
          └──────┬──────┘
                 │
                 ▼
          ┌─────────────┐
          │   Parser    │
          └──────┬──────┘
                 │
       ┌─────────┼─────────┐
       │         │         │
       ▼         ▼         ▼
  ┌───────┐ ┌───────┐ ┌───────┐
  │ Tool  │ │ Code  │ │ Text  │
  │ Calls │ │ Output│ │Response
  └───┬───┘ └───┬───┘ └───┬───┘
      │         │         │
      └─────────┼─────────┘
                │
                ▼
          ┌─────────────┐
          │  Session    │
          │  Update     │
          └──────┬──────┘
                 │
                 ▼
          ┌─────────────┐
          │     UI      │
          └─────────────┘
```

### 2. TUI Session Flow

```
┌─────────────────────────────────────────────────────────────┐
│                      TUI Session                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐ │
│   │  Prompt │───▶│  Send   │───▶│  Render │───▶│  Wait   │ │
│   │  Input  │    │  Server │    │Response │    │ Input   │ │
│   └─────────┘    └─────────┘    └─────────┘    └─────────┘ │
│        ▲                                               │    │
│        │                                               ▼    │
│   ┌─────────┐                                    ┌─────────┐ │
│   │   Key   │                                    │  Update │ │
│   │ Handler │                                    │ Session │ │
│   └─────────┘                                    └─────────┘ │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Extensibility Points

### 1. Custom Commands

**Location:** `packages/opencode/src/cli/cmd/`

**Steps:**
1. Create new command file
2. Define command structure
3. Register in `packages/opencode/src/index.ts`

```typescript
// packages/opencode/src/cli/cmd/mycommand.ts
export const MyCommand: Command = {
  command: "mycommand",
  describe: "My custom command",
  handler: async (argv) => {
    // Command logic
  }
}
```

### 2. Custom Providers

**Location:** `packages/opencode/src/provider/`

**Steps:**
1. Create provider file
2. Implement Provider interface
3. Register in provider registry

```typescript
// packages/opencode/src/provider/myp provider.ts
export class MyProvider implements Provider {
  name = "myprovider"
  models = [/* model list */]

  async complete(prompt: string) {
    // Implementation
  }
}
```

### 3. Custom Tools

**Location:** `packages/opencode/src/tool/registry.ts`

```typescript
registry.register({
  name: "myTool",
  description: "My custom tool",
  handler: async (args) => {
    // Tool implementation
  }
})
```

### 4. Plugins

**Location:** `packages/plugin/`

```typescript
// my-plugin/src/index.ts
export default {
  name: "my-plugin",
  tools: {
    myTool: { /* ... */ }
  },
  hooks: {
    onLoad: () => { /* ... */ }
  }
}
```

---

## Scaling Considerations

### 1. Horizontal Scaling

```
                    ┌─────────────────┐
                    │   Load Balancer │
                    └────────┬────────┘
                             │
         ┌───────────────────┼───────────────────┐
         │                   │                   │
    ┌────┴────┐         ┌────┴────┐         ┌────┴────┐
    │ Singularity │     │ Singularity │     │ Singularity │
    │  Server 1   │     │  Server 2   │     │  Server 3   │
    └────────────┘     └────────────┘     └────────────┘
         │                   │                   │
         └───────────────────┼───────────────────┘
                             │
                    ┌────────┴────────┐
                    │  Shared Session  │
                    │  Store (Redis)   │
                    └──────────────────┘
```

### 2. Session Storage

- **Local:** In-memory (development)
- **Distributed:** Redis (production)
- **Persistence:** Database (long-term storage)

### 3. Rate Limiting

```typescript
interface RateLimit {
  requests: number      // Requests per window
  window: number        // Window size in seconds
  burst: number         // Burst allowance
}

const rateLimits: Record<string, RateLimit> = {
  "opencode/big-pickle": { requests: 100, window: 60, burst: 10 },
  "anthropic/claude": { requests: 50, window: 60, burst: 5 }
}
```

### 4. Caching

```
┌─────────────────────────────────────────────────────────────┐
│                      Caching Layers                          │
├─────────────────────────────────────────────────────────────┤
│  L1: In-memory    │  Fastest  │  Per-process                │
│  L2: Redis        │  Fast     │  Distributed                │
│  L3: Database     │  Medium   │  Persistent                 │
│  L4: CDN/File     │  Slow     │  Static assets              │
└─────────────────────────────────────────────────────────────┘
```

---

## Best Practices

### 1. Code Organization

- Keep commands focused (one responsibility)
- Use dependency injection
- Follow TypeScript strict mode
- Write comprehensive tests

### 2. Performance

- Use streaming for large responses
- Implement proper rate limiting
- Cache model responses where appropriate
- Use connection pooling

### 3. Security

- Validate all inputs
- Never log sensitive data
- Use environment variables for secrets
- Implement proper authentication

### 4. Error Handling

- Create custom error types
- Log errors with context
- Implement proper retry logic
- Graceful degradation

---

## Debugging

### Enable Debug Logging

```bash
singularity --print-logs --log-level DEBUG
```

### Component-Specific Logging

```typescript
// Enable debug for specific component
Log.Default.debug("component-name", { data: value })
```

### Profile Performance

```bash
singularity debug profile --duration 30
```

---

## Resources

- [OpenCode Architecture](https://github.com/anomalyco/opencode)
- [TypeScript Documentation](https://www.typescriptlang.org/docs/)
- [Yargs Documentation](https://yargs.js.org/)
- [Bun Documentation](https://bun.sh/docs)

---

**Last Updated**: 2026-01-16

# Plugins System

## Overview

The Singularity Plugins System enables extensibility through modular components. Plugins can add tools, hooks, authentication methods, and custom functionality.

## Plugin Architecture

### Core Components

- **Plugin Manager**: Loads and manages plugin lifecycle
- **Hook System**: Event-driven plugin integration
- **Tool Registry**: Dynamic tool registration
- **Configuration Merger**: Plugin config integration

### Plugin Types

1. **Tool Plugins**: Add new tools to the system
2. **Auth Plugins**: Custom authentication methods
3. **UI Plugins**: Extend the TUI interface
4. **Integration Plugins**: Connect to external services

## Creating Plugins

### Basic Plugin Structure

```
my-plugin/
├── package.json
├── src/
│   ├── index.ts          # Plugin entry point
│   ├── tools/            # Custom tools
│   ├── hooks/            # Event hooks
│   └── types.ts          # Type definitions
├── test/
│   └── plugin.test.ts
└── README.md
```

### Plugin Entry Point

```typescript
// src/index.ts
import type { Plugin, PluginInput } from "@singularity/plugin";

export default {
  name: "my-plugin",
  version: "1.0.0",

  // Plugin metadata
  description: "My custom plugin",
  author: "Developer Name",

  // Configuration schema
  config: {
    apiKey: { type: "string", required: true },
  },

  // Tools provided by this plugin
  tools: {
    myTool: {
      description: "A custom tool",
      parameters: {
        input: { type: "string", description: "Input text" },
      },
      handler: async (args) => {
        return { result: args.input.toUpperCase() };
      },
    },
  },

  // Lifecycle hooks
  hooks: {
    onLoad: async (input: PluginInput) => {
      console.log("Plugin loaded!");
    },

    onSessionStart: async (session) => {
      // Initialize session-specific data
    },

    onMessage: async (message) => {
      // Process messages
      return message;
    },

    onUnload: async () => {
      // Cleanup resources
    },
  },
} satisfies Plugin;
```

### Package.json Configuration

```json
{
  "name": "singularity-plugin-my-plugin",
  "version": "1.0.0",
  "description": "My custom Singularity plugin",
  "main": "dist/index.js",
  "scripts": {
    "build": "tsc",
    "test": "bun test"
  },
  "keywords": ["singularity", "plugin", "ai", "cli"],
  "dependencies": {
    "@singularity/plugin": "^1.0.0"
  },
  "peerDependencies": {
    "singularity-code": "^0.1.0"
  }
}
```

## Tool Development

### Tool Interface

```typescript
interface Tool {
  name: string;
  description: string;
  parameters: Record<string, ParameterDefinition>;
  handler: (args: any) => Promise<ToolResult>;
}

interface ParameterDefinition {
  type: "string" | "number" | "boolean" | "array" | "object";
  description: string;
  required?: boolean;
  default?: any;
  enum?: string[];
}
```

### Example Tool

```typescript
// tools/weather.ts
export const weatherTool: Tool = {
  name: "get-weather",
  description: "Get current weather for a location",
  parameters: {
    location: {
      type: "string",
      description: "City name or coordinates",
      required: true,
    },
    unit: {
      type: "string",
      description: "Temperature unit",
      enum: ["celsius", "fahrenheit"],
      default: "celsius",
    },
  },

  handler: async ({ location, unit }) => {
    try {
      const response = await fetch(
        `https://api.weatherapi.com/v1/current.json?key=${API_KEY}&q=${location}`,
      );
      const data = await response.json();

      return {
        success: true,
        data: {
          location: data.location.name,
          temperature:
            unit === "fahrenheit" ? data.current.temp_f : data.current.temp_c,
          condition: data.current.condition.text,
        },
      };
    } catch (error) {
      return {
        success: false,
        error: error.message,
      };
    }
  },
};
```

## Hook System

### Available Hooks

- `onLoad`: Plugin initialization
- `onUnload`: Plugin cleanup
- `onSessionStart`: Session initialization
- `onSessionEnd`: Session cleanup
- `onMessage`: Message processing
- `onToolCall`: Tool execution
- `onError`: Error handling

### Hook Example

```typescript
hooks: {
  onMessage: async (message: Message) => {
    // Add context to messages
    if (message.role === 'user') {
      message.content = `User context: ${getUserContext()}\n\n${message.content}`
    }
    return message
  },

  onError: async (error: Error) => {
    // Custom error logging
    await logError(error, { plugin: 'my-plugin' })

    // Return modified error or null to suppress
    return {
      ...error,
      message: `Plugin error: ${error.message}`
    }
  }
}
```

## Configuration Integration

### Plugin Configuration

```typescript
// Plugin accepts configuration
export default {
  name: "api-plugin",

  config: {
    baseUrl: { type: "string", default: "https://api.example.com" },
    timeout: { type: "number", default: 5000 },
    retries: { type: "number", default: 3 },
  },

  hooks: {
    onLoad: async (input: PluginInput) => {
      const config = input.config; // Access merged config
      // Initialize with config
    },
  },
};
```

### User Configuration

```json
{
  "plugin": {
    "api-plugin": {
      "baseUrl": "https://my-api.com",
      "timeout": 10000
    }
  }
}
```

## Plugin Development Workflow

### 1. Setup Development Environment

```bash
# Create plugin directory
mkdir my-plugin && cd my-plugin

# Initialize project
bun init
bun add -D typescript @types/node
bun add @singularity/plugin

# Setup TypeScript
echo '{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "node",
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "strict": true,
    "outDir": "dist",
    "declaration": true
  }
}' > tsconfig.json
```

### 2. Implement Plugin

```typescript
// src/index.ts
import { weatherTool } from "./tools/weather";

export default {
  name: "weather-plugin",
  version: "1.0.0",
  description: "Weather information plugin",

  tools: {
    "get-weather": weatherTool,
  },

  hooks: {
    onLoad: async () => {
      console.log("Weather plugin loaded!");
    },
  },
};
```

### 3. Build and Test

```bash
# Build
bun run build

# Test locally
cd /path/to/singularity
bun add file:../my-plugin
```

### 4. Publish

```bash
# Publish to npm
npm publish

# Or add to awesome-opencode
# Submit PR to https://github.com/awesome-opencode/awesome-opencode
```

## Plugin Management

### Installing Plugins

```bash
# From npm
singularity plugin add my-plugin

# From local path
singularity plugin add /path/to/my-plugin

# From GitHub
singularity plugin add github:user/repo
```

### Managing Plugins

```bash
# List plugins
singularity plugin list

# Update plugin
singularity plugin update my-plugin

# Remove plugin
singularity plugin remove my-plugin

# Show plugin info
singularity plugin info my-plugin
```

## Best Practices

1. **Modular Design**: Keep plugins focused on specific functionality
2. **Error Handling**: Always handle errors gracefully
3. **Configuration**: Provide sensible defaults
4. **Documentation**: Include comprehensive README
5. **Testing**: Write unit tests for tools and hooks
6. **Performance**: Optimize for minimal overhead
7. **Security**: Validate inputs and avoid code injection

## Official Plugins

- **MCP Integration**: Model Context Protocol support
- **GitHub Tools**: Repository and issue management
- **Database Tools**: SQL query and schema tools
- **Testing Tools**: Automated test generation
- **Documentation Tools**: API documentation generation

## Community Plugins

See [Awesome OpenCode](https://github.com/awesome-opencode/awesome-opencode) for community-contributed plugins.

## Integration with SingularityPlugins

Custom plugins from the [SingularityPlugins](https://github.com/Delqhi/SingularityPlugins.git) repository can be integrated by:

1. Cloning the plugins repo
2. Building the desired plugins
3. Adding them via `singularity plugin add /path/to/plugin`
4. Configuring in `singularity.json`

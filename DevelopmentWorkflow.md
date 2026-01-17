# Development Workflow

## Overview

Singularity Code follows a modern development workflow optimized for AI-assisted development, emphasizing automation, quality, and rapid iteration.

## Development Environment Setup

### Prerequisites

- **Node.js**: 18+ (preferably via Bun)
- **Bun**: Latest version (recommended runtime)
- **Git**: Latest version
- **VS Code**: With recommended extensions

### Initial Setup

```bash
# Clone repository
git clone https://github.com/Delqhi/singularity-code.git
cd singularity-code

# Install dependencies
bun install

# Setup development environment
bun run setup-dev

# Verify installation
bun run doctor
```

### Development Scripts

```json
{
  "scripts": {
    "dev": "bun run packages/opencode/src/index.ts",
    "build": "bun run script/build.ts",
    "test": "bun test",
    "typecheck": "bun run typecheck",
    "lint": "bun run lint",
    "format": "bun run format"
  }
}
```

## Development Workflow

### 1. Feature Development

```bash
# Create feature branch
git checkout -b feature/my-feature

# Start development server
bun dev

# Make changes with hot reload
# Edit files, see changes instantly
```

### 2. Code Quality

```bash
# Run linter
bun run lint

# Fix formatting
bun run format

# Type checking
bun run typecheck

# Run tests
bun test
```

### 3. Testing Strategy

#### Unit Tests

```typescript
// test/tool/bash.test.ts
import { describe, it, expect } from "bun:test";
import { bashTool } from "../../src/tool/bash";

describe("bash tool", () => {
  it("executes simple commands", async () => {
    const result = await bashTool.handler({ command: "echo hello" });
    expect(result.success).toBe(true);
    expect(result.output).toContain("hello");
  });
});
```

#### Integration Tests

```typescript
// test/integration/session.test.ts
describe("session integration", () => {
  it("processes messages end-to-end", async () => {
    const session = new Session();
    const response = await session.processMessage("Hello");
    expect(response).toBeDefined();
  });
});
```

#### E2E Tests

```bash
# Run E2E tests
bun run test:e2e

# Test specific scenarios
bun run test:e2e --scenario authentication
```

### 4. Commit and Review

```bash
# Stage changes
git add .

# Commit with conventional format
git commit -m "feat: add new authentication system

- Implement OAuth2 flow
- Add user session management
- Update permission rules"

# Push branch
git push origin feature/my-feature
```

### 5. Pull Request

```bash
# Create PR
gh pr create --title "Add authentication system" --body "
## Description
Implements user authentication with OAuth2 support.

## Changes
- Added OAuth2 provider integration
- Implemented session management
- Updated permission system

## Testing
- Unit tests for auth logic
- Integration tests for session handling
- E2E tests for login flow
"
```

## Build Process

### Build Scripts

```typescript
// script/build.ts
import { build } from "bun";

const targets = [
  { platform: "darwin", arch: "arm64" },
  { platform: "darwin", arch: "x64" },
  { platform: "linux", arch: "x64" },
  { platform: "windows", arch: "x64" },
];

for (const target of targets) {
  await build({
    entrypoints: ["packages/opencode/src/index.ts"],
    outdir: `dist/${target.platform}-${target.arch}`,
    target: "bun",
    minify: true,
    sourcemap: "external",
  });
}
```

### Release Process

```bash
# Create release branch
git checkout -b release/v1.2.0

# Update version
bun run version:bump

# Build all targets
bun run build:all

# Run full test suite
bun run test:full

# Create GitHub release
gh release create v1.2.0 --generate-notes
```

## Testing Strategy

### Test Categories

1. **Unit Tests**: Individual functions and modules
2. **Integration Tests**: Component interactions
3. **End-to-End Tests**: Full user workflows
4. **Performance Tests**: Benchmarks and load testing
5. **Security Tests**: Vulnerability scanning

### Test Organization

```
test/
├── unit/
│   ├── tool/
│   ├── agent/
│   └── util/
├── integration/
│   ├── session/
│   └── provider/
├── e2e/
│   ├── cli/
│   └── tui/
└── fixtures/
    ├── configs/
    └── projects/
```

### Test Running

```bash
# Run all tests
bun test

# Run specific test file
bun test test/unit/tool/bash.test.ts

# Run with coverage
bun test --coverage

# Watch mode
bun test --watch
```

### Performance Testing

```typescript
// test/performance/session.bench.ts
import { bench, describe } from "bun:test";

describe("session performance", () => {
  bench("process simple message", async () => {
    const session = new Session();
    await session.processMessage("Hello world");
  });
});
```

## Code Quality Tools

### Linting

```json
// .eslintrc.json
{
  "extends": ["@singularity/eslint-config"],
  "rules": {
    "@typescript-eslint/no-unused-vars": "error",
    "@typescript-eslint/strict-boolean-expressions": "error"
  }
}
```

### Formatting

```json
// .prettierrc
{
  "semi": false,
  "singleQuote": true,
  "tabWidth": 2,
  "printWidth": 100
}
```

### Type Checking

```json
// tsconfig.json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "exactOptionalPropertyTypes": true,
    "noImplicitReturns": true
  }
}
```

## CI/CD Pipeline

### GitHub Actions Workflow

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main, dev]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: oven-sh/setup-bun@v1
      - run: bun install
      - run: bun run typecheck
      - run: bun test
      - run: bun run lint

  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: oven-sh/setup-bun@v1
      - run: bun install
      - run: bun run build
      - uses: actions/upload-artifact@v4
        with:
          name: binaries
          path: dist/
```

### Release Workflow

```yaml
# .github/workflows/release.yml
name: Release

on:
  push:
    tags: ["v*"]

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: oven-sh/setup-bun@v1
      - run: bun install
      - run: bun run build:all
      - run: gh release create ${{ github.ref_name }} --generate-notes
```

## Development Best Practices

### Code Style

- Follow TypeScript strict mode
- Use descriptive variable names
- Implement proper error handling
- Write self-documenting code
- Follow single responsibility principle

### Commit Guidelines

```bash
# Conventional commits
feat: add new feature
fix: resolve bug
docs: update documentation
refactor: improve code structure
test: add tests
chore: maintenance tasks
```

### Branch Strategy

```
main        # Production releases
dev         # Development integration
feature/*   # Feature branches
bugfix/*    # Bug fixes
hotfix/*    # Critical fixes
release/*   # Release preparation
```

### Code Review Process

1. **Automated Checks**: CI must pass
2. **Self Review**: Author reviews own code
3. **Peer Review**: Minimum 1 reviewer
4. **Testing**: All tests pass
5. **Documentation**: Update docs if needed

## Performance Optimization

### Bundle Analysis

```bash
# Analyze bundle size
bun run build:analyze

# Identify large dependencies
bun run bundle:report
```

### Memory Profiling

```typescript
// Performance monitoring
import { performance } from "perf_hooks";

const start = performance.now();
// Code to measure
const end = performance.now();
console.log(`Execution time: ${end - start} ms`);
```

### Benchmarking

```bash
# Run benchmarks
bun run benchmark

# Compare performance
bun run benchmark:compare
```

## Security Practices

### Dependency Scanning

```bash
# Check for vulnerabilities
bun run audit

# Update dependencies
bun run update:deps

# Check licenses
bun run license:check
```

### Code Security

- Input validation on all user inputs
- Avoid code injection vulnerabilities
- Use secure defaults
- Regular security audits

## Deployment Strategies

### Binary Distribution

- Cross-platform binaries via Bun
- Auto-updating mechanism
- Checksum verification
- Signed releases

### Container Deployment

```dockerfile
FROM oven/bun:latest
COPY . /app
RUN bun install --production
EXPOSE 3000
CMD ["bun", "run", "packages/opencode/src/server/index.ts"]
```

### Cloud Deployment

- Vercel for web interface
- Railway for backend
- AWS/GCP for enterprise
- Docker containers for portability

## Monitoring and Observability

### Logging

```typescript
// Structured logging
import { logger } from "../util/logger";

logger.info("User authenticated", {
  userId: user.id,
  timestamp: new Date(),
  ip: request.ip,
});
```

### Metrics

```typescript
// Performance metrics
import { metrics } from "../util/metrics";

metrics.increment("api.requests");
metrics.histogram("response.time", duration);
```

### Error Tracking

```typescript
// Error reporting
import { errorTracker } from "../util/error-tracker";

try {
  // Risky operation
} catch (error) {
  errorTracker.capture(error, {
    user: user.id,
    action: "file-upload",
  });
}
```

## Future Development (2026 Vision)

- **AI-Powered Development**: Enhanced AI assistance
- **Cloud-Native Architecture**: Serverless deployment
- **Multi-Modal Interfaces**: Voice, image, video support
- **Federated Development**: Cross-team collaboration
- **Automated Optimization**: Self-tuning performance

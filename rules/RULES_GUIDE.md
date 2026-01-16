# Singularity Rules System Guide

## Overview

Singularity's rules system guides AI behavior by defining project-specific guidelines, coding standards, and best practices. Rules are written in Markdown and can be applied globally or per-project.

---

## Table of Contents

1. [Rule Files](#rule-files)
2. [Rule Structure](#rule-structure)
3. [Built-in Rules](#built-in-rules)
4. [Custom Rules](#custom-rules)
5. [Rule Patterns](#rule-patterns)
6. [Testing Rules](#testing-rules)

---

## Rule Files

### Location

Rules can be placed in:

1. **Global**: `~/.config/singularity/rules/`
2. **Project**: `.singularity/rules/`
3. **Inline**: Passed via `--rules` flag

### File Naming

```
.singularity/rules/
├── project-rules.md        # Project-specific rules
├── language-rules.md       # Language-specific rules
├── framework-rules.md      # Framework-specific rules
├── security-rules.md       # Security guidelines
├── performance-rules.md    # Performance guidelines
└── style-rules.md          # Code style rules
```

---

## Rule Structure

### Basic Structure

```markdown
# Rule Title

## Description
Brief description of when this rule applies.

## When to Apply
- JavaScript projects
- TypeScript projects with React
- Backend APIs

## Rules

### Rule 1: Rule Name
Description of the rule.

```language
// Good example
code here

// Bad example
code here
```

### Rule 2: Another Rule
Another rule description.

```language
// Example
code here
```

## Examples

### Example 1: Use Case
Full example showing the rule in action.

```typescript
// Complete example
export function Example() {
  // Rule applied here
}
```

### Example 2: Complex Use Case
More complex example.

## References
- [Link to documentation](https://example.com)
- [Another resource](#)
```

---

## Built-in Rules

### 1. TypeScript Rules

```markdown
# TypeScript Rules

## Description
Guidelines for writing TypeScript code.

## When to Apply
All TypeScript projects.

## Rules

### Use Explicit Types
Always use explicit types for function parameters and return values.

```typescript
// Good
function greet(name: string): string {
  return `Hello, ${name}`;
}

// Bad
function greet(name) {
  return `Hello, ${name}`;
}
```

### Prefer Interfaces over Types
Use interfaces for object types.

```typescript
// Good
interface User {
  id: string;
  name: string;
}

// Avoid for objects
type User = {
  id: string;
  name: string;
};
```

### Use Async/Await
Prefer async/await over callbacks.

```typescript
// Good
async function fetchData(): Promise<Data> {
  const response = await fetch(url);
  return response.json();
}

// Avoid
function fetchData(callback: (data: Data) => void) {
  fetch(url).then(response => callback(response.json()));
}
```

## Examples

### Complete Example
```typescript
interface User {
  id: string;
  name: string;
  email: string;
}

async function createUser(data: Omit<User, "id">): Promise<User> {
  const response = await fetch("/api/users", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(data),
  });

  if (!response.ok) {
    throw new Error("Failed to create user");
  }

  return response.json();
}
```
```

### 2. Error Handling Rules

```markdown
# Error Handling Rules

## Description
Best practices for error handling in JavaScript/TypeScript.

## Rules

### Always Use Try/Catch
Wrap async operations in try/catch.

```typescript
// Good
async function fetchData(): Promise<Data> {
  try {
    const response = await fetch(url);
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    return response.json();
  } catch (error) {
    logger.error("Failed to fetch data", { url, error });
    throw error;
  }
}

// Bad - no error handling
async function fetchData(): Promise<Data> {
  const response = await fetch(url);
  return response.json();
}
```

### Create Custom Error Classes
For domain-specific errors.

```typescript
// Good
class ValidationError extends Error {
  constructor(message: string, public field: string) {
    super(message);
    this.name = "ValidationError";
  }
}

// Usage
function validateUser(user: User): void {
  if (!user.email.includes("@")) {
    throw new ValidationError("Invalid email", "email");
  }
}
```

### Never Log Sensitive Data
```typescript
// Bad - sensitive data in logs
logger.error("Login failed", { password: userPassword });

// Good - sanitized logs
logger.error("Login failed", { email: userEmail, attemptCount });
```
```

### 3. Security Rules

```markdown
# Security Rules

## Description
Security guidelines for all code.

## Rules

### Never Hardcode Secrets
Use environment variables.

```typescript
// Bad
const apiKey = "sk-1234567890abcdef";

// Good
const apiKey = process.env.API_KEY;
```

### Validate All Inputs
```typescript
// Good
function processUserInput(input: unknown): string {
  if (typeof input !== "string") {
    throw new Error("Invalid input type");
  }
  // Sanitize
  return input.trim().slice(0, 1000);
}
```

### Use Parameterized Queries
```typescript
// Bad - SQL injection vulnerability
const users = await db.query(`SELECT * FROM users WHERE id = ${userId}`);

// Good - parameterized query
const users = await db.query("SELECT * FROM users WHERE id = ?", [userId]);
```
```

---

## Custom Rules

### Creating Project Rules

Create `.singularity/rules/project-rules.md`:

```markdown
# Our Project Rules

## Description
Custom rules for our specific project.

## When to Apply
This repository only.

## Rules

### Use Our Component Library
Always use components from `@our-company/ui`.

```typescript
// Good
import { Button, Card } from "@our-company/ui";

// Bad - raw HTML
<button className="btn">Click</button>
```

### Follow Our Naming Conventions
```typescript
// Component naming
export function UserProfile() {}  // PascalCase

// Hook naming
export function useUser() {}      // camelCase with "use" prefix

// Constants
const MAX_RETRY_COUNT = 3;        // UPPER_SNAKE_CASE
```

### Use Our API Client
```typescript
// Good
import { apiClient } from "@our-company/api";

const data = await apiClient.get("/users");

// Bad - direct fetch
const data = await fetch("/api/users");
```
```

### Language-Specific Rules

Create `.singularity/rules/react-rules.md`:

```markdown
# React Rules

## Description
Rules for React components.

## Rules

### Use Functional Components
```typescript
// Good - functional component
export function UserCard({ user }: UserCardProps) {
  return <div>{user.name}</div>;
}

// Bad - class component
export class UserCard extends React.Component<UserCardProps> {
  render() {
    return <div>{this.props.user.name}</div>;
  }
}
```

### Use Hooks for State
```typescript
// Good
function Counter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```

### Use TypeScript Generics
```typescript
// Good - properly typed
function useFetch<T>(url: string): { data: T | null } {
  // ...
}
```
```

---

## Rule Patterns

### Pattern 1: Best Practice

```markdown
# Best Practice Pattern

## When to Apply
All code.

## Rules

### Rule Name
Explanation of the practice.

```language
// Good
code

// Bad
code
```
```

### Pattern 2: Anti-Pattern

```markdown
# Anti-Pattern Avoidance

## Description
Anti-patterns to avoid.

## Anti-Patterns

### Anti-Pattern Name
```language
// Bad - anti-pattern
problematicCode

// Why it's bad
Explanation

// Good alternative
correctCode
```
```

### Pattern 3: Complete Guide

```markdown
# Complete Guide Pattern

## Overview
Introduction to the topic.

## Prerequisites
What readers should know.

## Step-by-Step
1. Step one
2. Step two
3. Step three

## Complete Example
```language
// Full working example
completeCode
```

## Testing
How to test this code.

## Troubleshooting
Common issues and solutions.
```

---

## Testing Rules

### Validate Rule Files

```bash
# Test rule syntax
singularity debug --rules .singularity/rules/*.md --validate

# Preview rule application
singularity run "test prompt" --rules .singularity/rules/*.md --dry-run
```

### Rule Testing Best Practices

1. **Keep rules simple** - One rule per file is easier to maintain
2. **Use examples** - Include good and bad examples
3. **Test edge cases** - Cover boundary conditions
4. **Update regularly** - Review rules when updating dependencies

---

## Rule Configuration

### singularity.json

```json
{
  "rules": {
    "enabled": true,
    "files": [
      ".singularity/rules/*.md"
    ],
    "exclude": [
      "node_modules",
      ".git"
    ]
  }
}
```

### Environment Variables

```bash
SINGULARITY_RULES_ENABLED=true
SINGULARITY_RULES_PATH=.singularity/rules
```

---

## Best Practices

### 1. Organize Rules

```
.singularity/rules/
├── 00-general/          # General rules
│   ├── README.md
│   └── *.md
├── 10-typescript/       # Language-specific
│   ├── README.md
│   └── *.md
├── 20-framework/        # Framework-specific
│   ├── README.md
│   └── *.md
├── 30-project/          # Project-specific
│   ├── README.md
│   └── *.md
└── 99-archived/         # Deprecated rules
```

### 2. Version Rules

```markdown
# Rule Name

> **Version**: 1.0.0
> **Last Updated**: 2025-01-15
> **Applies To**: v1.0.0+
```

### 3. Document Changes

```markdown
# Changelog

## 1.1.0 (2025-01-20)
- Added new rule about X
- Updated rule Y to be more specific
- Removed deprecated rule Z
```

---

## Examples

### Complete Project Rules

```
.singularity/
├── rules/
│   ├── README.md           # Rule index
│   ├── 01-general.md
│   ├── 02-typescript.md
│   ├── 03-react.md
│   ├── 04-testing.md
│   ├── 05-security.md
│   └── 06-performance.md
└── singularity.json
```

**singularity.json:**
```json
{
  "rules": {
    "files": [".singularity/rules/*.md"],
    "priority": ["06-performance.md", "05-security.md"]
  }
}
```

---

## Troubleshooting

### Rules Not Applied

```bash
# Check rule files exist
ls -la .singularity/rules/

# Validate rule syntax
singularity debug --rules .singularity/rules/*.md --validate

# Check configuration
cat .singularity/singularity.json
```

### Conflicting Rules

Rules are applied in order. To resolve conflicts:

1. Use priority in singularity.json
2. Split rules into separate files
3. Use specific "When to Apply" sections

---

## Resources

- [OpenCode Documentation](https://opencode.ai/docs)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)
- [React Best Practices](https://react.dev/learn)

---

**Last Updated**: 2026-01-16

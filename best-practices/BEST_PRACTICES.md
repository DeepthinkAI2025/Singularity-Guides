# Singularity Best Practices Guide

## Overview

This guide covers best practices for using Singularity effectively, writing maintainable code, and following software engineering principles.

---

## Table of Contents

1. [General Principles](#general-principles)
2. [Code Style](#code-style)
3. [Error Handling](#error-handling)
4. [Testing](#testing)
5. [Security](#security)
6. [Performance](#performance)
7. [Project Structure](#project-structure)
8. [Git Workflow](#git-workflow)
9. [Documentation](#documentation)

---

## General Principles

### 1. TypeScript First

Always prefer TypeScript over JavaScript for better type safety and developer experience.

```typescript
// ✅ Good - TypeScript with explicit types
interface User {
  id: string;
  name: string;
  email: string;
}

function createUser(data: Omit<User, "id">): User {
  return {
    id: generateId(),
    ...data,
  };
}

// ❌ Bad - Plain JavaScript
function createUser(data) {
  return {
    id: generateId(),
    ...data,
  };
}
```

### 2. Strict Mode

Always enable TypeScript strict mode.

```json
// tsconfig.json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true
  }
}
```

### 3. Immutability

Prefer immutable data structures.

```typescript
// ✅ Good - Immutable update
const updatedUsers = [...users, newUser];

// ❌ Bad - Mutating array
users.push(newUser);
```

### 4. Functional Programming

Use pure functions where possible.

```typescript
// ✅ Good - Pure function
function multiplyByTwo(numbers: number[]): number[] {
  return numbers.map((n) => n * 2);
}

// ❌ Bad - Side effects
function multiplyByTwo(numbers: number[]): void {
  for (let i = 0; i < numbers.length; i++) {
    numbers[i] *= 2;
  }
}
```

---

## Code Style

### 1. Naming Conventions

```typescript
// Components - PascalCase
function UserCard() {}

// Hooks - camelCase with "use" prefix
function useUserData() {}

// Constants - UPPER_SNAKE_CASE
const MAX_RETRY_COUNT = 3;

// Interfaces - PascalCase (no "I" prefix)
interface UserData {}

// Types - camelCase
type UserStatus = "active" | "inactive" | "pending";
```

### 2. Function Design

```typescript
// ✅ Good - Small, focused function
function formatDate(date: Date): string {
  return date.toISOString().split("T")[0];
}

// ✅ Good - Arrow function for callbacks
const handleClick = (event: MouseEvent) => {
  event.preventDefault();
};

// ✅ Good - Async functions for I/O
async function fetchUser(id: string): Promise<User> {
  const response = await fetch(`/api/users/${id}`);
  return response.json();
}
```

### 3. Early Returns

```typescript
// ✅ Good - Early return pattern
function processUser(user: User | null): Result {
  if (!user) {
    return { success: false, error: "User not found" };
  }

  if (!user.active) {
    return { success: false, error: "User inactive" };
  }

  // Main logic
  return { success: true, data: user };
}

// ❌ Bad - Nested conditionals
function processUser(user: User | null): Result {
  if (user) {
    if (user.active) {
      // Main logic
      return { success: true, data: user };
    } else {
      return { success: false, error: "User inactive" };
    }
  } else {
    return { success: false, error: "User not found" };
  }
}
```

### 4. Destructuring

```typescript
// ✅ Good - Destructuring
const { name, email } = user;

// ✅ Good - With defaults
const { role = "user" } = user;

// ✅ Good - Array destructuring
const [first, second, ...rest] = items;
```

---

## Error Handling

### 1. Try/Catch/Finally

```typescript
async function fetchData<T>(url: string): Promise<T> {
  try {
    const response = await fetch(url);

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    return response.json();
  } catch (error) {
    // Log with context
    logger.error("Failed to fetch data", { url, error });
    throw error; // Re-throw or handle
  } finally {
    // Cleanup (e.g., close loaders)
    console.log("Fetch attempt completed");
  }
}
```

### 2. Custom Error Classes

```typescript
// ✅ Good - Custom error class
class ValidationError extends Error {
  constructor(
    message: string,
    public field: string,
    public value: unknown
  ) {
    super(message);
    this.name = "ValidationError";
  }
}

// Usage
function validateEmail(email: string): void {
  if (!email.includes("@")) {
    throw new ValidationError("Invalid email format", "email", email);
  }
}
```

### 3. Result Type Pattern

```typescript
// ✅ Good - Result type for operations that can fail
type Result<T> =
  | { success: true; data: T }
  | { success: false; error: string };

function divide(a: number, b: number): Result<number> {
  if (b === 0) {
    return { success: false, error: "Division by zero" };
  }
  return { success: true, data: a / b };
}
```

### 4. Never Log Sensitive Data

```typescript
// ❌ Bad - Logging sensitive data
logger.error("Auth failed", { password: userPassword });

// ✅ Good - Sanitized logs
logger.error("Auth failed", {
  email: userEmail,
  attemptCount,
  ip: sanitizeIp(userIp),
});
```

---

## Testing

### 1. Test Structure

```typescript
describe("UserService", () => {
  describe("createUser", () => {
    it("should create a user with valid data", async () => {
      // Arrange
      const userData = { name: "John", email: "john@example.com" };
      const service = new UserService();

      // Act
      const result = await service.createUser(userData);

      // Assert
      expect(result.success).toBe(true);
      expect(result.data.name).toBe("John");
    });

    it("should reject invalid email", async () => {
      // ...
    });
  });
});
```

### 2. Test Async Code

```typescript
// ✅ Good - Testing async functions
it("should fetch user data", async () => {
  const service = new UserService();
  const user = await service.getUser("123");

  expect(user).toBeDefined();
  expect(user.id).toBe("123");
});
```

### 3. Mock External Services

```typescript
// ✅ Good - Mocking fetch
global.fetch = vi.fn().mockResolvedValue({
  ok: true,
  json: () => Promise.resolve({ id: "123", name: "Test" }),
});

// ✅ Good - Using MSW for API mocking
import { setupServer } from "msw/node";

const server = setupServer(
  http.get("/api/user", () => {
    return HttpResponse.json({ id: "123", name: "Test" });
  })
);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```

### 4. Test Coverage

```typescript
// ✅ Good - Aim for meaningful coverage
// Focus on critical paths and edge cases
describe("calculateDiscount", () => {
  it("should apply 10% discount for orders over $100", () => {
    expect(calculateDiscount(150)).toBe(135);
  });

  it("should not apply discount for small orders", () => {
    expect(calculateDiscount(50)).toBe(50);
  });

  it("should handle zero", () => {
    expect(calculateDiscount(0)).toBe(0);
  });
});
```

---

## Security

### 1. Environment Variables

```typescript
// ✅ Good - Environment variables for secrets
const apiKey = process.env.API_KEY;
const dbPassword = process.env.DB_PASSWORD;

// ❌ Bad - Hardcoded secrets
const apiKey = "sk-1234567890abcdef";
```

### 2. Input Validation

```typescript
// ✅ Good - Validate all inputs
interface CreateUserInput {
  name: string;
  email: string;
  age?: number;
}

function validateInput(input: unknown): CreateUserInput {
  if (typeof input !== "object" || input === null) {
    throw new Error("Invalid input");
  }

  const { name, email, age } = input as CreateUserInput;

  if (typeof name !== "string" || name.length < 1) {
    throw new Error("Invalid name");
  }

  if (typeof email !== "string" || !email.includes("@")) {
    throw new Error("Invalid email");
  }

  if (age !== undefined && (typeof age !== "number" || age < 0)) {
    throw new Error("Invalid age");
  }

  return { name, email, age };
}
```

### 3. SQL Injection Prevention

```typescript
// ✅ Good - Parameterized queries
const users = await db.query(
  "SELECT * FROM users WHERE id = ?",
  [userId]
);

// ❌ Bad - String concatenation
const users = await db.query(
  `SELECT * FROM users WHERE id = ${userId}`
);
```

### 4. XSS Prevention

```typescript
// ✅ Good - Sanitize HTML
import DOMPurify from "dompurify";

const sanitized = DOMPurify.sanitize(userInput);

// ✅ Good - Use textContent instead of innerHTML
element.textContent = userInput;

// ❌ Bad - Direct innerHTML
element.innerHTML = userInput;
```

---

## Performance

### 1. Lazy Loading

```typescript
// ✅ Good - Lazy load components
const HeavyComponent = lazy(() => import("./HeavyComponent"));

// ✅ Good - Dynamic imports
async function loadUtility() {
  const { utilityFunction } = await import("./utils");
  return utilityFunction();
}
```

### 2. Memoization

```typescript
// ✅ Good - Use useMemo for expensive computations
const expensiveResult = useMemo(() => {
  return computeExpensiveValue(dep1, dep2);
}, [dep1, dep2]);

// ✅ Good - Use useCallback for stable references
const handleClick = useCallback(() => {
  doSomething(value);
}, [value]);
```

### 3. Batch Operations

```typescript
// ✅ Good - Batch database operations
async function createUsers(users: User[]): Promise<void> {
  await db.transaction(async (trx) => {
    for (const user of users) {
      await trx.insert(usersTable).values(user);
    }
  });
}

// ✅ Good - Use Promise.all for parallel
const [users, posts, comments] = await Promise.all([
  fetchUsers(),
  fetchPosts(),
  fetchComments(),
]);
```

### 4. Connection Pooling

```typescript
// ✅ Good - Use connection pool
const pool = new Pool({
  host: "localhost",
  max: 20, // Max connections
  idleTimeoutMillis: 30000,
});
```

---

## Project Structure

### 1. Feature-Based Organization

```
src/
├── features/
│   ├── users/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── services/
│   │   ├── types/
│   │   └── index.ts
│   ├── posts/
│   └── comments/
├── core/
│   ├── auth/
│   ├── errors/
│   └── utils/
├── shared/
│   ├── components/
│   ├── hooks/
│   └── utils/
└── types/
```

### 2. Barrel Files

```typescript
// src/features/users/index.ts
export { UserList } from "./components/UserList";
export { useUsers } from "./hooks/useUsers";
export type { User, CreateUserInput } from "./types";
```

### 3. Dependency Direction

```
High-level modules should not import low-level details
⬇️
src/core/          → Abstract interfaces
src/features/      → Implementations
src/shared/        → Shared utilities
```

---

## Git Workflow

### 1. Branch Naming

```
feature/user-authentication
bugfix/fix-login-error
hotfix/security-patch
docs/update-readme
chore/update-dependencies
```

### 2. Commit Messages

```
feat: Add user authentication
fix: Resolve login error on Safari
docs: Update installation instructions
refactor: Improve type safety in user service
test: Add unit tests for auth module
chore: Update dependencies
```

### 3. Pull Request Flow

1. Create feature branch
2. Make changes
3. Run tests locally
4. Push to remote
5. Create PR
6. Address review feedback
7. Merge after approval

### 4. Code Review Guidelines

- Check for type safety
- Look for edge cases
- Verify error handling
- Ensure tests pass
- Review documentation

---

## Documentation

### 1. JSDoc Comments

```typescript
/**
 * Calculates the total price including tax and discounts.
 *
 * @param items - Array of cart items
 * @param taxRate - Tax rate as decimal (e.g., 0.08 for 8%)
 * @param discountCode - Optional discount code
 * @returns The total price
 *
 * @throws {ValidationError} If items array is empty
 * @example
 * ```typescript
 * const total = calculateTotal(
 *   [{ price: 100, quantity: 2 }],
 *   0.08,
 *   "SAVE10"
 * );
 * ```
 */
function calculateTotal(
  items: CartItem[],
  taxRate: number,
  discountCode?: string
): number {
  // Implementation
}
```

### 2. README Structure

```markdown
# Project Name

Brief description.

## Installation

```bash
npm install
```

## Usage

```typescript
import { functionName } from "package";

functionName(args);
```

## API

### functionName

Description...

## Examples

...

## Contributing

...

## License
```

### 3. Keep Documentation Updated

- Update docs when adding features
- Fix docs when fixing bugs
- Remove docs when removing features
- Use code comments for implementation details

---

## Additional Resources

### TypeScript
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)
- [TypeScript Deep Dive](https://basarat.gitbook.io/typescript/)

### Testing
- [Vitest Documentation](https://vitest.dev/)
- [Testing Library](https://testing-library.com/)

### Best Practices
- [Clean Code JavaScript](https://github.com/ryanmcdermott/clean-code-javascript)
- [Effective TypeScript](https://effectivetypescript.com/)

---

## Quick Reference

### Do's ✓

- ✓ Use TypeScript strict mode
- ✓ Write meaningful variable names
- ✓ Handle errors gracefully
- ✓ Test your code
- ✓ Document public APIs
- ✓ Use environment variables
- ✓ Keep functions small
- ✓ Follow single responsibility

### Don'ts ✗

- ✗ Don't commit secrets
- ✗ Don't ignore TypeScript errors
- ✗ Don't use `any` type
- ✗ Don't leave console.log in production
- ✗ Don't mutate state directly
- ✗ Don't write giant functions
- ✗ Don't skip error handling

---

**Last Updated**: 2026-01-16

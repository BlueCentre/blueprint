# JavaScript/TypeScript Guide

This guide covers working with JavaScript and TypeScript in Blueprint using Aspect's `aspect_rules_js` and `aspect_rules_ts`.

## Table of Contents

- [Setup](#setup)
- [Project Structure](#project-structure)
- [Dependencies](#dependencies)
- [Building and Running](#building-and-running)
- [Testing](#testing)
- [Linting and Formatting](#linting-and-formatting)
- [Advanced Topics](#advanced-topics)

## Setup

### Prerequisites

JavaScript/TypeScript projects in Blueprint use:
- **Node.js** (version managed by Bazel)
- **pnpm** for dependency management
- **TypeScript** for type checking (optional but recommended)

### Initial Configuration

Blueprint is pre-configured with:

```starlark
# MODULE.bazel
bazel_dep(name = "aspect_rules_js", version = "2.6.0")
bazel_dep(name = "aspect_rules_ts", version = "3.7.0")

pnpm = use_extension("@aspect_rules_js//npm:extensions.bzl", "pnpm")
pnpm.pnpm(
    name = "pnpm",
    version = "9.1.0",
)
```

Configuration files:
- `package.json` - Project metadata and dependencies
- `pnpm-lock.yaml` - Dependency lock file
- `tsconfig.json` - TypeScript configuration
- `eslint.config.mjs` - ESLint configuration
- `prettier.config.cjs` - Prettier configuration

## Project Structure

Typical JavaScript/TypeScript package:

```
my-js-package/
├── BUILD
├── package.json
├── tsconfig.json
├── src/
│   ├── index.ts
│   ├── lib.ts
│   └── utils.ts
├── test/
│   └── lib.test.ts
└── README.md
```

### BUILD File Example

```starlark
load("@aspect_rules_js//js:defs.bzl", "js_library", "js_binary")
load("@aspect_rules_ts//ts:defs.bzl", "ts_project")

ts_project(
    name = "mylib_ts",
    srcs = glob([
        "src/**/*.ts",
    ]),
    declaration = True,
    source_map = True,
    tsconfig = "//:tsconfig",
    deps = [
        "//:node_modules/express",
        "//:node_modules/@types/node",
    ],
)

js_library(
    name = "mylib",
    srcs = [":mylib_ts"],
    visibility = ["//visibility:public"],
)

js_binary(
    name = "app",
    data = [":mylib"],
    entry_point = "src/index.js",
)

# Tests
ts_project(
    name = "test_ts",
    srcs = glob(["test/**/*.test.ts"]),
    tsconfig = "//:tsconfig",
    deps = [
        ":mylib_ts",
        "//:node_modules/vitest",
    ],
)
```

## Dependencies

### Adding Dependencies

Using pnpm (recommended):

```bash
# Add runtime dependency
pnpm add express

# Add dev dependency
pnpm add -D @types/node typescript

# Add specific version
pnpm add lodash@4.17.21
```

### Using Dependencies

After adding dependencies, they're automatically available:

```starlark
ts_project(
    name = "mylib_ts",
    srcs = ["src/index.ts"],
    deps = [
        "//:node_modules/express",
        "//:node_modules/@types/express",
    ],
)
```

In TypeScript code:

```typescript
import express from 'express';
import type { Request, Response } from 'express';

const app = express();
app.get('/', (req: Request, res: Response) => {
    res.send('Hello World!');
});
```

### Managing pnpm Workspace

For monorepos, use `pnpm-workspace.yaml`:

```yaml
packages:
  - 'packages/*'
  - 'apps/*'
```

## Building and Running

### Build TypeScript

```bash
# Build TypeScript project
bazel build //path/to:mylib_ts

# Build outputs to bazel-bin
ls bazel-bin/path/to/mylib_ts/
```

### Run JavaScript Binary

```bash
# Run with Bazel
bazel run //path/to:app

# With arguments
bazel run //path/to:app -- --port 3000

# With environment variables
bazel run //path/to:app --action_env=NODE_ENV=production
```

### Development Server

For development with hot reload:

```json
// package.json
{
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build"
  }
}
```

```bash
# Run dev server
pnpm run dev

# Or through Bazel (if configured)
bazel run //path/to:dev
```

## Testing

### Writing Tests

Using Vitest (recommended):

```typescript
// lib.test.ts
import { describe, it, expect } from 'vitest';
import { myFunction } from './lib';

describe('myFunction', () => {
    it('should return expected value', () => {
        const result = myFunction();
        expect(result).toBe(expected);
    });

    it('should handle edge cases', () => {
        expect(myFunction(null)).toBeNull();
    });
});
```

Using Jest:

```typescript
// lib.test.ts
import { myFunction } from './lib';

describe('myFunction', () => {
    test('should return expected value', () => {
        const result = myFunction();
        expect(result).toBe(expected);
    });
});
```

### Running Tests

```bash
# Run with pnpm (outside Bazel)
pnpm test

# Or configure Bazel test target
bazel test //path/to:test
```

### Test Configuration

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config';

export default defineConfig({
    test: {
        globals: true,
        environment: 'node',
        coverage: {
            reporter: ['text', 'json', 'html'],
        },
    },
});
```

## Linting and Formatting

### ESLint

Configuration in `eslint.config.mjs`:

```javascript
import js from '@eslint/js';
import tseslint from 'typescript-eslint';

export default [
    js.configs.recommended,
    ...tseslint.configs.recommended,
    {
        rules: {
            'no-unused-vars': 'warn',
            '@typescript-eslint/no-explicit-any': 'warn',
        },
    },
];
```

Run ESLint:

```bash
# Format all files
format

# Lint with Aspect
aspect lint //...

# Or use ESLint directly
pnpm eslint .
```

### Prettier

Configuration in `prettier.config.cjs`:

```javascript
module.exports = {
    semi: true,
    trailingComma: 'all',
    singleQuote: true,
    printWidth: 100,
    tabWidth: 2,
};
```

Format code:

```bash
# Format all files
format

# Or use Prettier directly
pnpm prettier --write .
```

### Type Checking

```bash
# Check types
pnpm tsc --noEmit

# Or in watch mode
pnpm tsc --noEmit --watch
```

## Advanced Topics

### Monorepo Setup

Structure for multiple packages:

```
project/
├── packages/
│   ├── ui/
│   │   ├── BUILD
│   │   ├── package.json
│   │   └── src/
│   └── api/
│       ├── BUILD
│       ├── package.json
│       └── src/
└── pnpm-workspace.yaml
```

Cross-package dependencies:

```json
// packages/ui/package.json
{
  "dependencies": {
    "@myorg/api": "workspace:*"
  }
}
```

### React Application

```typescript
// App.tsx
import React from 'react';

export function App() {
    return (
        <div>
            <h1>Hello Blueprint!</h1>
        </div>
    );
}
```

```starlark
ts_project(
    name = "app_ts",
    srcs = glob(["src/**/*.tsx", "src/**/*.ts"]),
    tsconfig = "//:tsconfig",
    deps = [
        "//:node_modules/react",
        "//:node_modules/@types/react",
    ],
)
```

### Node.js Server

```typescript
// server.ts
import express from 'express';

const app = express();
const PORT = process.env.PORT || 3000;

app.get('/', (req, res) => {
    res.json({ message: 'Hello from Blueprint!' });
});

app.listen(PORT, () => {
    console.log(`Server running on port ${PORT}`);
});
```

```starlark
js_binary(
    name = "server",
    data = [":server_ts"],
    entry_point = "src/server.js",
)
```

### Bundling with Webpack/Vite

For production builds:

```javascript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
    plugins: [react()],
    build: {
        outDir: 'dist',
        sourcemap: true,
    },
});
```

### Environment Variables

```typescript
// Use process.env in Node.js
const apiUrl = process.env.API_URL || 'http://localhost:3000';

// Or use dotenv
import dotenv from 'dotenv';
dotenv.config();
```

```starlark
js_binary(
    name = "app",
    data = [":app_lib"],
    entry_point = "src/index.js",
    env = {
        "NODE_ENV": "production",
        "API_URL": "https://api.example.com",
    },
)
```

### Package Scripts Integration

Aspect CLI can generate targets from package.json scripts:

```json
// package.json
{
  "scripts": {
    "build": "tsc",
    "test": "vitest run",
    "lint": "eslint ."
  }
}
```

These are auto-detected and can be run:

```bash
bazel run //:build
bazel run //:test
```

## Common Patterns

### Express API

```typescript
// api.ts
import express, { Request, Response } from 'express';

const app = express();
app.use(express.json());

app.get('/api/users', (req: Request, res: Response) => {
    res.json([{ id: 1, name: 'John' }]);
});

app.post('/api/users', (req: Request, res: Response) => {
    const user = req.body;
    res.status(201).json(user);
});

export default app;
```

### Frontend Components

```typescript
// Button.tsx
import React from 'react';

interface ButtonProps {
    label: string;
    onClick: () => void;
    variant?: 'primary' | 'secondary';
}

export function Button({ label, onClick, variant = 'primary' }: ButtonProps) {
    return (
        <button 
            className={`btn btn-${variant}`}
            onClick={onClick}
        >
            {label}
        </button>
    );
}
```

### Utility Functions

```typescript
// utils.ts
export function debounce<T extends (...args: any[]) => any>(
    func: T,
    delay: number
): (...args: Parameters<T>) => void {
    let timeoutId: NodeJS.Timeout;
    
    return (...args: Parameters<T>) => {
        clearTimeout(timeoutId);
        timeoutId = setTimeout(() => func(...args), delay);
    };
}
```

## Troubleshooting

### Module Not Found

```bash
# Reinstall dependencies
pnpm install

# Clear pnpm cache
pnpm store prune

# Rebuild
bazel clean
bazel build //...
```

### Type Errors

```bash
# Install missing type definitions
pnpm add -D @types/node @types/express

# Update tsconfig.json
# Check include/exclude paths
```

### Build Failures

```bash
# Check TypeScript compilation
pnpm tsc --noEmit

# Verify BUILD file deps
# Ensure all imports have corresponding deps
```

### Performance Issues

```bash
# Use incremental compilation
# tsconfig.json
{
  "compilerOptions": {
    "incremental": true,
    "tsBuildInfoFile": ".tsbuildinfo"
  }
}

# Use SWC for faster builds (if needed)
pnpm add -D @swc/core
```

## Best Practices

1. **Use TypeScript** - Better type safety and IDE support
2. **Configure strict mode** - Enable strict TypeScript checks
3. **Use ESLint** - Catch common mistakes
4. **Format with Prettier** - Consistent code style
5. **Write tests** - Unit and integration tests
6. **Use async/await** - Modern async handling
7. **Handle errors** - Proper error handling
8. **Type everything** - Avoid `any` type

## TypeScript Configuration

Recommended `tsconfig.json`:

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "lib": ["ES2020"],
    "moduleResolution": "node",
    "esModuleInterop": true,
    "strict": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "outDir": "./dist"
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist", "**/*.test.ts"]
}
```

## Resources

- [aspect_rules_js Documentation](https://docs.aspect.build/rulesets/aspect_rules_js/)
- [aspect_rules_ts Documentation](https://docs.aspect.build/rulesets/aspect_rules_ts/)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html)
- [Node.js Best Practices](https://github.com/goldbergyoni/nodebestpractices)
- [React Documentation](https://react.dev/)

## Next Steps

- Explore [Testing Guide](../contributor/testing.md)
- Learn about [Monorepo Structure](../user/project-structure.md)
- Check [Troubleshooting](../user/troubleshooting.md#npm-package-not-found)

# Modernizing JavaScript/TypeScript Projects

Follow this prompt to modernize your JavaScript/TypeScript project.

## Package Manager Detection

Before applying templates, detect the package manager:

| Lock File | Package Manager | Install Command | Run Command |
|-----------|----------------|-----------------|-------------|
| `pnpm-lock.yaml` | pnpm | `pnpm install --frozen-lockfile` | `pnpm run` |
| `yarn.lock` | yarn | `yarn install --frozen-lockfile` | `yarn` |
| `bun.lockb` or `bun.lock` | bun | `bun install --frozen-lockfile` | `bun run` |
| `deno.lock` | deno | `deno install` | `deno task` |
| `package-lock.json` | npm | `npm ci` | `npm run` |
| (none, package.json exists) | npm | `npm ci` | `npm run` |

## Steps

### 1. Dockerize

#### Dockerfile (multi-stage build)

Choose the appropriate template based on the detected package manager.

##### npm

```dockerfile
# Build stage
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage
FROM node:20-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production
RUN addgroup --system --gid 1001 nodejs && \
    adduser --system --uid 1001 nodejs
COPY --from=builder --chown=nodejs:nodejs /app/dist ./dist
COPY --from=builder --chown=nodejs:nodejs /app/node_modules ./node_modules
COPY --from=builder --chown=nodejs:nodejs /app/package.json ./
USER nodejs
EXPOSE 3000
CMD ["node", "dist/index.js"]
```

##### pnpm

```dockerfile
# Build stage
FROM node:20-alpine AS builder
RUN corepack enable && corepack prepare pnpm@latest --activate
WORKDIR /app
COPY pnpm-lock.yaml package.json ./
RUN pnpm install --frozen-lockfile
COPY . .
RUN pnpm run build

# Production stage
FROM node:20-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production
RUN addgroup --system --gid 1001 nodejs && \
    adduser --system --uid 1001 nodejs
COPY --from=builder --chown=nodejs:nodejs /app/dist ./dist
COPY --from=builder --chown=nodejs:nodejs /app/node_modules ./node_modules
COPY --from=builder --chown=nodejs:nodejs /app/package.json ./
USER nodejs
EXPOSE 3000
CMD ["node", "dist/index.js"]
```

##### yarn

```dockerfile
# Build stage
FROM node:20-alpine AS builder
RUN corepack enable
WORKDIR /app
COPY yarn.lock package.json ./
RUN yarn install --frozen-lockfile
COPY . .
RUN yarn build

# Production stage
FROM node:20-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production
RUN addgroup --system --gid 1001 nodejs && \
    adduser --system --uid 1001 nodejs
COPY --from=builder --chown=nodejs:nodejs /app/dist ./dist
COPY --from=builder --chown=nodejs:nodejs /app/node_modules ./node_modules
COPY --from=builder --chown=nodejs:nodejs /app/package.json ./
USER nodejs
EXPOSE 3000
CMD ["node", "dist/index.js"]
```

##### bun

```dockerfile
# Build stage
FROM oven/bun:1 AS builder
WORKDIR /app
COPY bun.lockb package.json ./
RUN bun install --frozen-lockfile
COPY . .
RUN bun run build

# Production stage
FROM oven/bun:1-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production
RUN addgroup --system --gid 1001 nodejs && \
    adduser --system --uid 1001 nodejs
COPY --from=builder --chown=nodejs:nodejs /app/dist ./dist
COPY --from=builder --chown=nodejs:nodejs /app/node_modules ./node_modules
COPY --from=builder --chown=nodejs:nodejs /app/package.json ./
USER nodejs
EXPOSE 3000
CMD ["bun", "dist/index.js"]
```

##### deno

```dockerfile
# Build stage
FROM denoland/deno:2 AS builder
WORKDIR /app
COPY deno.json deno.lock ./
RUN deno install
COPY . .
RUN deno task build

# Production stage
FROM denoland/deno:2-alpine AS runner
WORKDIR /app
RUN addgroup --system --gid 1001 deno && \
    adduser --system --uid 1001 deno
COPY --from=builder --chown=deno:deno /app/dist ./dist
COPY --from=builder --chown=deno:deno /app/deno.json ./
USER deno
EXPOSE 3000
CMD ["deno", "run", "--allow-net", "--allow-read", "--allow-env", "dist/index.js"]
```

#### docker-compose.yml (for development)

##### npm / yarn / pnpm

```yaml
services:
  app:
    build:
      context: .
      target: builder
    ports:
      - "3000:3000"
    volumes:
      - .:/app
      - /app/node_modules
    environment:
      - NODE_ENV=development
    command: ${PKG_RUN_CMD:-npm run} dev  # Replace with: yarn dev / pnpm run dev
```

##### bun

```yaml
services:
  app:
    build:
      context: .
      target: builder
    ports:
      - "3000:3000"
    volumes:
      - .:/app
      - /app/node_modules
    environment:
      - NODE_ENV=development
    command: bun run dev
```

##### deno

```yaml
services:
  app:
    build:
      context: .
      target: builder
    ports:
      - "3000:3000"
    volumes:
      - .:/app
    environment:
      - DENO_ENV=development
    command: deno task dev
```

#### .dockerignore

```
# Dependencies
node_modules
.pnp
.pnp.js

# Build output
dist
build
out

# Git
.git
.gitignore
.gitattributes

# Environment
.env
.env.*
!.env.example

# IDE
.vscode
.idea
*.swp
*.swo

# Test & Coverage
coverage
.nyc_output
*.lcov

# Documentation
*.md
docs

# CI/CD
.github
.gitlab-ci.yml
.circleci

# Package manager
.npm
.yarn
.pnpm-store

# Logs
logs
*.log
npm-debug.log*
yarn-debug.log*
yarn-error.log*
pnpm-debug.log*

# OS
.DS_Store
Thumbs.db

# Docker
Dockerfile*
docker-compose*
.dockerignore
```

### 2. CI/CD (GitHub Actions)

Use the appropriate workflows based on the detected package manager.

#### .github/workflows/lint.yml

##### npm / pnpm / yarn / bun

Biomeはスタンドアロンバイナリとして動作するため、パッケージマネージャーによらず共通のワークフローを使用できます。

```yaml
name: Lint

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  lint:
    runs-on: ubuntu-slim
    steps:
      - uses: actions/checkout@v4
      - uses: biomejs/setup-biome@v2
        with:
          version: latest
      - run: biome lint .
```

##### deno

```yaml
name: Lint

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  lint:
    runs-on: ubuntu-slim
    steps:
      - uses: actions/checkout@v4
      - uses: denoland/setup-deno@v2
        with:
          deno-version: v2.x
      - run: deno lint
```

#### biome.json

Create linting configuration for JS/TS files (including E2E tests) in the project root.
Only npm/pnpm/yarn/bun.

```json
{
  "$schema": "https://biomejs.dev/schemas/1.9.0/schema.json",
  "organizeImports": {
    "enabled": true
  },
  "linter": {
    "enabled": true,
    "rules": {
      "recommended": true
    }
  },
  "formatter": {
    "enabled": false
  }
}
```

#### .github/workflows/test.yml

##### npm

```yaml
name: Test

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-slim
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npm test
```

##### pnpm

```yaml
name: Test

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-slim
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v4
        with:
          version: latest
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'pnpm'
      - run: pnpm install --frozen-lockfile
      - run: pnpm test
```

##### yarn

```yaml
name: Test

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-slim
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'yarn'
      - run: yarn install --frozen-lockfile
      - run: yarn test
```

##### bun

```yaml
name: Test

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-slim
    steps:
      - uses: actions/checkout@v4
      - uses: oven-sh/setup-bun@v2
        with:
          bun-version: latest
      - run: bun install --frozen-lockfile
      - run: bun test
```

##### deno

```yaml
name: Test

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-slim
    steps:
      - uses: actions/checkout@v4
      - uses: denoland/setup-deno@v2
        with:
          deno-version: v2.x
      - run: deno task test
```

#### .github/workflows/audit.yml (npm/pnpm/yarn/bun only)

##### npm

```yaml
name: Audit

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  audit:
    runs-on: ubuntu-slim
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npm audit --audit-level=high
```

##### pnpm

```yaml
name: Audit

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  audit:
    runs-on: ubuntu-slim
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v4
        with:
          version: latest
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'pnpm'
      - run: pnpm install --frozen-lockfile
      - run: pnpm audit --audit-level=high
```

##### yarn

```yaml
name: Audit

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  audit:
    runs-on: ubuntu-slim
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'yarn'
      - run: yarn install --frozen-lockfile
      - run: yarn audit --level high
```

##### bun

```yaml
name: Audit

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  audit:
    runs-on: ubuntu-slim
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm i --package-lock-only
      - run: npm audit --audit-level=high
```

##### deno

Deno uses URL imports with a built-in security model. No separate audit workflow needed.

#### .github/workflows/build.yml

```yaml
name: Build

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-slim
    steps:
      - uses: actions/checkout@v4
      - run: docker build -t ${{ github.repository }}:${{ github.sha }} .
```

### 3. E2E Testing with Playwright (Local Execution)

#### Setup Playwright

Add Playwright as a dev dependency and initialize configuration.

##### npm/pnpm/yarn/bun

Add to `package.json`:

```json
{
  "devDependencies": {
    "@biomejs/biome": "^1.9.0",
    "@playwright/test": "^1.49.0"
  },
  "scripts": {
    "lint": "biome lint .",
    "screenshot": "playwright test tests/e2e/screenshot.spec.ts",
    "test:e2e": "playwright test",
    "test:e2e:ui": "playwright test --ui",
    "test:e2e:headed": "playwright test --headed"
  }
}
```

##### deno

Add to `deno.json`:

```json
{
  "tasks": {
    "test:e2e": "npx playwright test",
    "test:e2e:ui": "npx playwright test --ui",
    "test:e2e:headed": "npx playwright test --headed"
  }
}
```

#### playwright.config.ts

Create configuration file for all package managers:

```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({
  testDir: './tests/e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: 'html',
  use: {
    baseURL: 'http://localhost:3000',
    browserName: 'chromium',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
  },
  projects: [
    {
      name: 'mobile',
      use: { viewport: { width: 375, height: 812 } },
    },
    {
      name: 'tablet',
      use: { viewport: { width: 768, height: 1024 } },
    },
    {
      name: 'fhd',
      use: { viewport: { width: 1920, height: 1080 } },
    },
  ],
  webServer: {
    command: 'npm run dev',  // Adjust based on package manager
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
});
```

#### Test File Example: Full-Page Screenshot

Create `tests/e2e/screenshot.spec.ts`:

```typescript
import { test, expect } from '@playwright/test';

const PAGES = [
  { path: '/', name: 'homepage' },
  { path: '/about', name: 'about' },
  { path: '/contact', name: 'contact' },
];

test.describe('Full-Page Screenshot Tests', () => {
  for (const pageInfo of PAGES) {
    test(`capture ${pageInfo.name}`, async ({ page }, testInfo) => {
      await page.goto(pageInfo.path);
      await page.waitForLoadState('networkidle');

      // Example filenames: homepage-mobile.png / about-tablet.png / contact-fhd.png
      await page.screenshot({
        path: `screenshots/${pageInfo.name}-${testInfo.project.name}.png`,
        fullPage: true,
      });

      await expect(page).toHaveTitle(/.+/);
    });
  }
});
```

#### .gitignore Updates

Add to `.gitignore`:

```
# Playwright
test-results/
playwright-report/
playwright/.cache/
screenshots/
```

### 4. README Updates

Add the following to your existing README.md.

#### CI Badges (add at the top of README)

```markdown
![Lint](https://github.com/{owner}/{repo}/actions/workflows/lint.yml/badge.svg)
![Test](https://github.com/{owner}/{repo}/actions/workflows/test.yml/badge.svg)
![Audit](https://github.com/{owner}/{repo}/actions/workflows/audit.yml/badge.svg)
![Build](https://github.com/{owner}/{repo}/actions/workflows/build.yml/badge.svg)
```

Note: For deno projects, omit the Audit badge.

#### Quick Start Section

Choose the appropriate section based on the detected package manager.

##### npm

~~~markdown
## Quick Start

### Run with Docker

```bash
# Development
docker compose up

# Production build
docker compose up --build
```

### Run with Podman

```bash
# Development
podman compose up

# Production build
podman compose up --build
```

### Run locally

```bash
npm install
npm run dev
```
~~~

##### pnpm

~~~markdown
## Quick Start

### Run with Docker

```bash
# Development
docker compose up

# Production build
docker compose up --build
```

### Run with Podman

```bash
# Development
podman compose up

# Production build
podman compose up --build
```

### Run locally

```bash
pnpm install
pnpm run dev
```
~~~

##### yarn

~~~markdown
## Quick Start

### Run with Docker

```bash
# Development
docker compose up

# Production build
docker compose up --build
```

### Run with Podman

```bash
# Development
podman compose up

# Production build
podman compose up --build
```

### Run locally

```bash
yarn install
yarn dev
```
~~~

##### bun

~~~markdown
## Quick Start

### Run with Docker

```bash
# Development
docker compose up

# Production build
docker compose up --build
```

### Run with Podman

```bash
# Development
podman compose up

# Production build
podman compose up --build
```

### Run locally

```bash
bun install
bun run dev
```
~~~

##### deno

~~~markdown
## Quick Start

### Run with Docker

```bash
# Development
docker compose up

# Production build
docker compose up --build
```

### Run with Podman

```bash
# Development
podman compose up

# Production build
podman compose up --build
```

### Run locally

```bash
deno install
deno task dev
```
~~~

## Notes

- Detect the package manager by checking for lock files in priority order (pnpm → yarn → bun → deno → npm)
- Ensure config files have appropriate scripts defined:
  - **npm/pnpm/yarn/bun**: `package.json` needs `build`, `dev`, `lint`, `test`, and `test:e2e` scripts
  - **deno**: `deno.json` needs `build`, `dev`, `test`, and `test:e2e` tasks
- Adjust the Dockerfile build commands and entrypoint as needed for your project structure
- Linting (Biome):
  - npm/pnpm/yarn/bun: `biome lint .` via `biomejs/setup-biome@v2` GitHub Action (no Node.js setup required)
  - deno: `deno lint` (built-in linter)
  - Create `biome.json` in the project root to configure rules
  - Run locally: `npx biome lint .` or install as devDependency and use `npm run lint`
- Security audit commands vary by package manager:
  - npm: `npm audit fix`
  - pnpm: `pnpm audit --fix`
  - yarn: `yarn audit` (manual fix required)
  - bun: No built-in audit (use `npm audit` with package-lock.json)
  - deno: Uses URL imports with built-in security model
- Playwright E2E testing:
  - Install Playwright: `npx playwright install` (or use package manager equivalent)
  - Run tests locally: `npm run test:e2e` (adjust for your package manager)
  - Run tests in UI mode: `npm run test:e2e:ui`
  - Update `playwright.config.ts` webServer command based on your package manager
  - Full-page screenshots are saved to `screenshots/` directory (add to `.gitignore`)

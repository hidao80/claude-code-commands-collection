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

##### npm

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
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npm run lint
```

##### pnpm

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
      - uses: pnpm/action-setup@v4
        with:
          version: latest
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'pnpm'
      - run: pnpm install --frozen-lockfile
      - run: pnpm run lint
```

##### yarn

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
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'yarn'
      - run: yarn install --frozen-lockfile
      - run: yarn lint
```

##### bun

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
      - uses: oven-sh/setup-bun@v2
        with:
          bun-version: latest
      - run: bun install --frozen-lockfile
      - run: bun run lint
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

### 3. README Updates

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
docker build -t app .
docker run -p 3000:3000 app
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
docker build -t app .
docker run -p 3000:3000 app
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
docker build -t app .
docker run -p 3000:3000 app
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
docker build -t app .
docker run -p 3000:3000 app
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
docker build -t app .
docker run -p 3000:3000 app
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
  - **npm/pnpm/yarn/bun**: `package.json` needs `build`, `dev`, `lint`, and `test` scripts
  - **deno**: `deno.json` needs `build`, `dev`, and `test` tasks
- Adjust the Dockerfile build commands and entrypoint as needed for your project structure
- Security audit commands vary by package manager:
  - npm: `npm audit fix`
  - pnpm: `pnpm audit --fix`
  - yarn: `yarn audit` (manual fix required)
  - bun: No built-in audit (use `npm audit` with package-lock.json)
  - deno: Uses URL imports with built-in security model

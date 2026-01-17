# Modernizing Node.js Projects

Follow this prompt to modernize your Node.js project.

## Steps

### 1. Dockerize

#### Dockerfile (multi-stage build)

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

#### docker-compose.yml (for development)

```yaml
version: '3.8'

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
    command: npm run dev
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

#### .github/workflows/lint.yml

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

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Lint
        run: npm run lint
```

#### .github/workflows/test.yml

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

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Test
        run: npm test
```

#### .github/workflows/audit.yml

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

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Security audit
        run: npm audit --audit-level=high
```

#### .github/workflows/docker.yml

```yaml
name: Docker

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  docker-build:
    runs-on: ubuntu-slim
    steps:
      - uses: actions/checkout@v4

      - name: Build Docker image
        run: docker build -t ${{ github.repository }}:${{ github.sha }} .
```

### 3. README Updates

Add the following to your existing README.md.

#### CI Badges (add at the top of README)

```markdown
![Lint](https://github.com/{owner}/{repo}/actions/workflows/lint.yml/badge.svg)
![Test](https://github.com/{owner}/{repo}/actions/workflows/test.yml/badge.svg)
![Audit](https://github.com/{owner}/{repo}/actions/workflows/audit.yml/badge.svg)
![Docker](https://github.com/{owner}/{repo}/actions/workflows/docker.yml/badge.svg)
```

#### Quick Start Section

```markdown
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
```

## Notes

- Ensure `package.json` has `build`, `dev`, `lint`, and `test` scripts defined
- Adjust the Dockerfile build commands and entrypoint as needed
- If security audit finds vulnerabilities, fix them with `npm audit fix`

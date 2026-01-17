# modernize-repo Command Examples

This command modernizes your repository by adding Docker support, CI/CD pipelines, and updating README documentation.

## Usage

```bash
/modernize-repo [node|python|php]
```

## Options

| Option | Description |
|--------|-------------|
| `--skip-docker` | Skip Docker-related files |
| `--skip-ci` | Skip CI/CD-related files |
| `--readme-only` | Only update README |

## What It Creates

### 1. Docker Files
- `Dockerfile` - Multi-stage build for production
- `docker-compose.yml` - Development environment
- `.dockerignore` - Exclude unnecessary files from builds

### 2. CI/CD Pipelines (GitHub Actions)
- Lint workflow
- Test workflow
- Security audit workflow
- Docker build workflow

### 3. README Updates
- CI status badges
- Quick Start section with Docker instructions

---

## Examples by Language

### Example 1: Node.js Project

**Command:**
```bash
/modernize-repo node
```

**Files created:**

#### Dockerfile
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

#### docker-compose.yml
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

#### GitHub Actions Workflows
- `.github/workflows/lint.yml` - ESLint checks
- `.github/workflows/test.yml` - Run npm test
- `.github/workflows/audit.yml` - npm audit for vulnerabilities
- `.github/workflows/docker.yml` - Docker build verification

#### README badges added
```markdown
![Lint](https://github.com/{owner}/{repo}/actions/workflows/lint.yml/badge.svg)
![Test](https://github.com/{owner}/{repo}/actions/workflows/test.yml/badge.svg)
![Audit](https://github.com/{owner}/{repo}/actions/workflows/audit.yml/badge.svg)
![Docker](https://github.com/{owner}/{repo}/actions/workflows/docker.yml/badge.svg)
```

---

### Example 2: Python Project

**Command:**
```bash
/modernize-repo python
```

**Files created:**

#### Dockerfile
```dockerfile
# Build stage
FROM python:3.12-slim AS builder
WORKDIR /app
RUN pip install --no-cache-dir uv
COPY pyproject.toml uv.lock ./
RUN uv sync --frozen --no-dev
COPY . .

# Production stage
FROM python:3.12-slim AS runner
WORKDIR /app
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

RUN groupadd --gid 1001 python && \
    useradd --uid 1001 --gid python --shell /bin/bash python

COPY --from=builder --chown=python:python /app/.venv ./.venv
COPY --chown=python:python . .

USER python
ENV PATH="/app/.venv/bin:$PATH"
EXPOSE 8000
CMD ["python", "-m", "app"]
```

#### docker-compose.yml
```yaml
version: '3.8'

services:
  app:
    build:
      context: .
      target: builder
    ports:
      - "8000:8000"
    volumes:
      - .:/app
    environment:
      - PYTHONDONTWRITEBYTECODE=1
      - PYTHONUNBUFFERED=1
    command: uv run python -m app

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
      POSTGRES_DB: app
    volumes:
      - db_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

volumes:
  db_data:
```

#### GitHub Actions Workflows
- `.github/workflows/lint.yml` - Ruff linting
- `.github/workflows/format.yml` - Ruff format check
- `.github/workflows/test.yml` - pytest
- `.github/workflows/audit.yml` - pip-audit for vulnerabilities
- `.github/workflows/docker.yml` - Docker build verification

---

### Example 3: PHP Project

**Command:**
```bash
/modernize-repo php
```

**Files created:**

#### Dockerfile
```dockerfile
# Build stage
FROM composer:2 AS builder
WORKDIR /app
COPY composer.json composer.lock ./
RUN composer install --no-dev --no-scripts --no-autoloader
COPY . .
RUN composer dump-autoload --optimize

# Production stage
FROM php:8.3-fpm-alpine AS runner
WORKDIR /var/www/html

RUN docker-php-ext-install pdo pdo_mysql opcache

RUN echo "opcache.enable=1" >> /usr/local/etc/php/conf.d/opcache.ini && \
    echo "opcache.memory_consumption=128" >> /usr/local/etc/php/conf.d/opcache.ini && \
    echo "opcache.max_accelerated_files=10000" >> /usr/local/etc/php/conf.d/opcache.ini && \
    echo "opcache.validate_timestamps=0" >> /usr/local/etc/php/conf.d/opcache.ini

RUN addgroup -g 1001 -S phpuser && \
    adduser -u 1001 -S phpuser -G phpuser

COPY --from=builder --chown=phpuser:phpuser /app/vendor ./vendor
COPY --chown=phpuser:phpuser . .

USER phpuser
EXPOSE 9000
CMD ["php-fpm"]
```

#### docker-compose.yml
```yaml
version: '3.8'

services:
  app:
    build:
      context: .
      target: runner
    volumes:
      - .:/var/www/html
    environment:
      - APP_ENV=development
    depends_on:
      - db

  nginx:
    image: nginx:alpine
    ports:
      - "8080:80"
    volumes:
      - .:/var/www/html
      - ./docker/nginx/default.conf:/etc/nginx/conf.d/default.conf
    depends_on:
      - app

  db:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: app
      MYSQL_USER: user
      MYSQL_PASSWORD: password
    volumes:
      - db_data:/var/lib/mysql
    ports:
      - "3306:3306"

volumes:
  db_data:
```

#### GitHub Actions Workflows
- `.github/workflows/lint.yml` - Mago linting
- `.github/workflows/analyse.yml` - Mago static analysis
- `.github/workflows/test.yml` - PHPUnit
- `.github/workflows/audit.yml` - Composer audit
- `.github/workflows/docker.yml` - Docker build verification

---

## Using Options

### Skip Docker files
```bash
/modernize-repo node --skip-docker
```
Only creates CI/CD workflows and updates README. Useful when you already have Docker configured.

### Skip CI/CD
```bash
/modernize-repo python --skip-ci
```
Only creates Docker files and updates README. Useful when you have existing CI/CD pipelines.

### README only
```bash
/modernize-repo php --readme-only
```
Only updates README with badges and Quick Start section. Useful for adding documentation to existing setups.

---

## Quick Start Sections Added to README

### Node.js
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

### Python
```markdown
## Quick Start

### Run with Docker
```bash
# Development
docker compose up

# Production build
docker build -t app .
docker run -p 8000:8000 app
```

### Run locally
```bash
uv sync
uv run python -m app
```
```

### PHP
```markdown
## Quick Start

### Run with Docker
```bash
# Development
docker compose up

# Production build
docker build -t app .
docker run -p 9000:9000 app
```

### Run locally
```bash
composer install
php -S localhost:8000 -t public
```
```

---

## Prerequisites

Before running the command, ensure your project has:

| Language | Requirements |
|----------|--------------|
| Node.js | `package.json` with `build`, `dev`, `lint`, `test` scripts |
| Python | `pyproject.toml` with project configuration |
| PHP | `composer.json` with dependencies |

## Notes

- All Dockerfiles use multi-stage builds for smaller production images
- All containers run as non-root users for security
- CI/CD workflows trigger on push and pull requests to main branch
- Security audit workflows check for known vulnerabilities in dependencies

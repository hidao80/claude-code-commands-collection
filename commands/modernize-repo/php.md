# Modernizing PHP Projects

Follow this prompt to modernize your PHP project.

## Steps

### 1. Dockerize

#### Dockerfile (multi-stage build)

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

# Install required extensions
RUN docker-php-ext-install pdo pdo_mysql opcache

# OPcache configuration
RUN echo "opcache.enable=1" >> /usr/local/etc/php/conf.d/opcache.ini && \
    echo "opcache.memory_consumption=128" >> /usr/local/etc/php/conf.d/opcache.ini && \
    echo "opcache.max_accelerated_files=10000" >> /usr/local/etc/php/conf.d/opcache.ini && \
    echo "opcache.validate_timestamps=0" >> /usr/local/etc/php/conf.d/opcache.ini

# Create non-root user
RUN addgroup -g 1001 -S phpuser && \
    adduser -u 1001 -S phpuser -G phpuser

COPY --from=builder --chown=phpuser:phpuser /app/vendor ./vendor
COPY --chown=phpuser:phpuser . .

USER phpuser
EXPOSE 9000
CMD ["php-fpm"]
```

#### docker-compose.yml (for development)

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

#### docker/nginx/default.conf

```nginx
server {
    listen 80;
    server_name localhost;
    root /var/www/html/public;
    index index.php index.html;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        fastcgi_pass app:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.ht {
        deny all;
    }
}
```

#### .dockerignore

```
# Dependencies
vendor

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
.phpunit.result.cache
.phpunit.cache
coverage
.coverage

# Documentation
*.md
docs

# CI/CD
.github
.gitlab-ci.yml
.circleci

# Cache
var/cache
var/log
storage/logs
bootstrap/cache

# Logs
*.log

# OS
.DS_Store
Thumbs.db

# Docker
Dockerfile*
docker-compose*
.dockerignore
docker/
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

      - name: Setup Mago
        uses: carthage-software/setup-mago@v1

      - name: Run Mago Lint
        run: mago lint
```

#### .github/workflows/analyse.yml

```yaml
name: Analyse

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  analyse:
    runs-on: ubuntu-slim
    steps:
      - uses: actions/checkout@v4

      - name: Setup Mago
        uses: carthage-software/setup-mago@v1

      - name: Run Mago Analyse
        run: mago analyse
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

      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: '8.3'
          coverage: xdebug

      - name: Get Composer cache directory
        id: composer-cache
        run: echo "dir=$(composer config cache-files-dir)" >> $GITHUB_OUTPUT

      - name: Cache Composer dependencies
        uses: actions/cache@v4
        with:
          path: ${{ steps.composer-cache.outputs.dir }}
          key: ${{ runner.os }}-composer-${{ hashFiles('**/composer.lock') }}
          restore-keys: ${{ runner.os }}-composer-

      - name: Install dependencies
        run: composer install --prefer-dist --no-progress

      - name: Run tests
        run: vendor/bin/phpunit
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

      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: '8.3'

      - name: Security audit
        run: composer audit
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

### 3. E2E Testing with Playwright (Local Execution)

#### Setup Playwright for PHP Projects

Even though this is a PHP project, we can use Playwright (Node.js-based) for E2E testing.

#### Create package.json

Add `package.json` to your PHP project root:

```json
{
  "name": "php-e2e-tests",
  "version": "1.0.0",
  "private": true,
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

#### biome.json

Create linting configuration for JS/TS files (including E2E tests) in the project root.

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

#### .github/workflows/js-lint.yml

```yaml
name: JS Lint

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  js-lint:
    runs-on: ubuntu-slim
    steps:
      - uses: actions/checkout@v4
      - uses: biomejs/setup-biome@v2
        with:
          version: latest
      - run: biome lint .
```

#### playwright.config.ts

Create configuration file:

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
    baseURL: 'http://localhost:8080',
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
    command: 'php -S localhost:8080 -t public',
    url: 'http://localhost:8080',
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
# Node.js (for Playwright)
node_modules/
package-lock.json

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
![Analyse](https://github.com/{owner}/{repo}/actions/workflows/analyse.yml/badge.svg)
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
composer install
php -S localhost:8000 -t public
```
```

## Notes

- Ensure `composer.json` has the required scripts and dependencies defined
- If using Mago, create a `mago.toml` configuration file
- Adjust the Dockerfile PHP extensions and entrypoint as needed
- If security audit finds vulnerabilities, fix them with `composer update`
- JS/TS linting with Biome:
  - Create `biome.json` in project root to configure linting rules
  - CI: Run `biome lint .` using the `biomejs/setup-biome@v2` action (no Node.js setup required)
  - Local execution: `npm run lint` (via the `lint` script in `package.json`)
- Playwright E2E testing (requires Node.js):
  - Create `package.json` in project root with Biome and Playwright dependencies
  - Install dependencies: `npm install` then `npx playwright install`
  - Run tests locally: `npm run test:e2e`
  - Run tests in UI mode: `npm run test:e2e:ui`
  - Update `playwright.config.ts` webServer command if using different PHP server setup
  - Full-page screenshots are saved to `screenshots/` directory (add to `.gitignore`)
  - Add `node_modules/` and Playwright artifacts to `.gitignore`

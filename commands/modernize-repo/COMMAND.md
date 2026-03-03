---
allowed-tools: Bash(git:*), Bash(docker:*), Read(*.json,*.toml,*.yaml,*.yml,*.md,*.lock,composer.*,package.*,pyproject.*,Dockerfile*,docker-compose*,.dockerignore,.github/**/*), Write(Dockerfile,docker-compose.yml,.dockerignore,.github/workflows/*.yml,README.md,docker/**/*), Glob(*), Grep(*)
description: "Modernize your repository with Docker, CI/CD, and README updates"
---

# /modernize-repo

Modernize your repository (Docker + CI/CD + README)

## Usage
```
/modernize-repo [js|python|php]
```

## What it does

1. **Dockerize**
   - Dockerfile (multi-stage build)
   - docker-compose.yml (for development)
   - .dockerignore

2. **CI/CD**
   - GitHub Actions (lint + audit + docker build)

3. **E2E Testing (Playwright)**
   - Add `@playwright/test` to `package.json` devDependencies
   - Add scripts to `package.json`:
     - `"test:e2e": "playwright test"`
     - `"screenshot": "playwright test tests/e2e/screenshot.spec.ts"` — full-page screenshots only
   - Create `playwright.config.ts`
   - Create `tests/e2e/screenshot.spec.ts`

4. **README updates**
   - Add CI badges
   - Quick Start section (Docker & Podman startup instructions)

## Options

- `--skip-docker`: Skip Docker-related files
- `--skip-ci`: Skip CI/CD-related files
- `--readme-only`: Only update README

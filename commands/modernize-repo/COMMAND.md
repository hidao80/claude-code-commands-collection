---
allowed-tools: Bash(git:*), Bash(docker:*), Read(*.json,*.toml,*.yaml,*.yml,*.md,*.lock,composer.*,package.*,pyproject.*,Dockerfile*,docker-compose*,.dockerignore,.github/**/*), Write(Dockerfile,docker-compose.yml,.dockerignore,.github/workflows/*.yml,README.md,docker/**/*), Glob(*), Grep(*)
description: "Modernize your repository with Docker, CI/CD, and README updates"
---

# /modernize-repo

Modernize your repository (Docker + CI/CD + README)

## Usage
```
/modernize-repo [node|python|php]
```

## What it does

1. **Dockerize**
   - Dockerfile (multi-stage build)
   - docker-compose.yml (for development)
   - .dockerignore

2. **CI/CD**
   - GitHub Actions (lint + audit + docker build)

3. **README updates**
   - Add CI badges
   - Quick Start section (Docker startup instructions)

## Options

- `--skip-docker`: Skip Docker-related files
- `--skip-ci`: Skip CI/CD-related files
- `--readme-only`: Only update README

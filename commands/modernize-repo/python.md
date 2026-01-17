# Modernizing Python Projects

Follow this prompt to modernize your Python project.

## Steps

### 1. Dockerize

#### Dockerfile (multi-stage build)

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

# Create non-root user
RUN groupadd --gid 1001 python && \
    useradd --uid 1001 --gid python --shell /bin/bash python

COPY --from=builder --chown=python:python /app/.venv ./.venv
COPY --chown=python:python . .

USER python
ENV PATH="/app/.venv/bin:$PATH"
EXPOSE 8000
CMD ["python", "-m", "app"]
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

#### .dockerignore

```
# Virtual environments
.venv
venv
env
.env

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

# Python cache
__pycache__
*.py[cod]
*$py.class
*.so
.Python

# Test & Coverage
.pytest_cache
.coverage
htmlcov
.tox
.nox

# Documentation
*.md
docs

# CI/CD
.github
.gitlab-ci.yml
.circleci

# Build
build
dist
*.egg-info
.eggs

# Logs
*.log

# OS
.DS_Store
Thumbs.db

# Docker
Dockerfile*
docker-compose*
.dockerignore

# Type checking
.mypy_cache
.pytype
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

      - name: Install uv
        uses: astral-sh/setup-uv@v4

      - name: Run Ruff Lint
        run: uvx ruff check .
```

#### .github/workflows/format.yml

```yaml
name: Format

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  format:
    runs-on: ubuntu-slim
    steps:
      - uses: actions/checkout@v4

      - name: Install uv
        uses: astral-sh/setup-uv@v4

      - name: Run Ruff Format Check
        run: uvx ruff format --check .
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

      - name: Install uv
        uses: astral-sh/setup-uv@v4

      - name: Set up Python
        run: uv python install

      - name: Install dependencies
        run: uv sync --all-extras --dev

      - name: Run tests
        run: uv run pytest
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

      - name: Install uv
        uses: astral-sh/setup-uv@v4

      - name: Security audit
        run: uvx pip-audit
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
![Format](https://github.com/{owner}/{repo}/actions/workflows/format.yml/badge.svg)
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
docker run -p 8000:8000 app
```

### Run locally

```bash
uv sync
uv run python -m app
```
```

## Notes

- Ensure `pyproject.toml` has the project settings and dependencies defined
- If using Ruff, configure it in the `[tool.ruff]` section of `pyproject.toml`
- Adjust the Dockerfile entrypoint as needed
- If security audit finds vulnerabilities, update your dependencies

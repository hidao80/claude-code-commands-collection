<img width="1584" height="672" alt="title_image" src="https://github.com/user-attachments/assets/6af26977-0b2b-461b-921b-27103c2c6587" />

# Claude Code Commands Collection

A collection of useful slash commands for [Claude Code](https://claude.ai/claude-code), designed to enhance your development workflow.

## 📋 Available Commands

### `/commit-msg`

Proposes concise commit messages in English with appropriate gitmoji tags.

**Features:**
- Generates clear, professional commit messages
- Automatically adds appropriate gitmoji prefixes (`:bug:`, `:sparkles:`, `:memo:`, etc.)
- Analyzes staged changes to create meaningful messages

**Allowed tools:**
- `Bash(git:*)` - Git operations
- `Read(All files)` - Read project files
- `Write(docs/spec/*.md)` - Write to spec documentation

**Usage:**
```bash
/commit-msg
```

**Example output:**
```
:bug: Fix null pointer exception in user authentication
:sparkles: Add dark mode toggle to settings page
:memo: Update README with installation instructions
```

See [examples/commit-msg.md](examples/commit-msg.md) for more examples.

---

### `/update-memory`

Records and updates project documentation in a readable state for both humans and AI.

**Features:**
- Re-reads and updates existing Memory docs (`docs/spec/*.md`)
- Automatically handles file splitting for large documents (>2000 tokens)
- Validates implementation against test code
- Tracks commit hash for incremental updates
- Maintains documentation in English

**Allowed tools:**
- `Bash(git:*)` - Git operations
- `Bash(npm:*)` - NPM operations
- `Read(*.md,*.js,*.html,*.json)` - Read project files
- `Write(docs/spec/*.md)` - Write to spec documentation
- `Fetch(*)` - Web fetch operations

**Target documentation files (created/updated in order):**
1. `docs/spec/screens.md` - UI screens and pages
2. `docs/spec/configuration.md` - Config files and settings
3. `docs/spec/components.md` - Reusable components
4. `docs/spec/utilities.md` - Helper functions and utilities
5. `docs/spec/databases.md` - Database schemas and models
6. `docs/spec/overview.md` - Project overview and architecture
7. `docs/spec/known_bugs.md` - Known issues and bugs
8. `docs/spec/todo.md` - Pending tasks and improvements

**Usage:**
```bash
/update-memory
```

See [examples/update-memory.md](examples/update-memory.md) for detailed examples.

---

### `/modernize-repo`

Modernizes your repository with Docker support, CI/CD pipelines, and README updates.

**Features:**
- Creates multi-stage Dockerfile for optimized production builds
- Generates docker-compose.yml for development environment
- Sets up GitHub Actions workflows (lint, test, audit, docker build)
- Updates README with CI badges and Quick Start section
- Supports Node.js, Python, and PHP projects

**Allowed tools:**
- `Bash(git:*)` - Git operations
- `Read(*)` - Read project files
- `Write(*)` - Write configuration files

**Usage:**
```bash
/modernize-repo [node|python|php]
```

**Options:**
| Option | Description |
|--------|-------------|
| `--skip-docker` | Skip Docker-related files |
| `--skip-ci` | Skip CI/CD-related files |
| `--readme-only` | Only update README |

**What it creates:**

| Category | Files |
|----------|-------|
| Docker | `Dockerfile`, `docker-compose.yml`, `.dockerignore` |
| CI/CD | `.github/workflows/lint.yml`, `test.yml`, `audit.yml`, `docker.yml` |
| README | CI badges, Quick Start section |

See [examples/modernize-repo.md](examples/modernize-repo.md) for language-specific examples.

---

## 📁 Repository Structure

```
claude-code-commands-collection/
├── commands/
│   ├── commit-msg.md           # Commit message generator
│   ├── update-memory.md        # Documentation updater
│   └── modernize-repo/         # Repository modernization
│       ├── COMMAND.md          # Main command definition
│       ├── node.md             # Node.js configuration
│       ├── python.md           # Python configuration
│       └── php.md              # PHP configuration
├── examples/
│   ├── commit-msg.md           # commit-msg usage examples
│   ├── update-memory.md        # update-memory usage examples
│   └── modernize-repo.md       # modernize-repo usage examples
├── README.md
└── LICENSE
```

## 🚀 Installation

1. Clone this repository or download the command files
2. Copy the command files to your project's `.claude/commands/` directory:
   ```bash
   # For single-file commands
   cp commands/commit-msg.md your-project/.claude/commands/
   cp commands/update-memory.md your-project/.claude/commands/

   # For multi-file commands (like modernize-repo)
   cp -r commands/modernize-repo your-project/.claude/commands/
   ```
3. The commands will be available in Claude Code as slash commands

## 📝 Creating Your Own Commands

Each command is a markdown file with frontmatter configuration:

```markdown
---
allowed-tools: Bash(git:*), Read(All files)
description: "Your command description"
---

Your command instructions here...
```

### Frontmatter Options

| Field | Description |
|-------|-------------|
| `allowed-tools` | Comma-separated list of tools the command can use |
| `description` | Brief description shown in command list |

### Multi-file Commands

For complex commands, create a directory with:
- `COMMAND.md` - Main command definition and instructions
- Additional `.md` files - Supporting documentation or configurations

See the [Claude Code documentation](https://docs.anthropic.com/en/docs/claude-code) for more details on creating custom commands.

## 📄 License

MIT License - see [LICENSE](LICENSE) file for details.

## 🤝 Contributing

Contributions are welcome! Feel free to:
- Submit new command ideas
- Improve existing commands
- Report issues or bugs
- Share your own custom commands

## 👤 Author

hidao80

---

**Note:** These commands are designed for Claude Code CLI tool. Make sure you have Claude Code installed and configured before using these commands.

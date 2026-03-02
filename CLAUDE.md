# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is a collection of reusable slash commands for Claude Code. Each command is a markdown file with YAML frontmatter that defines allowed tools and functionality. Commands are designed to be copied into other projects' `.claude/commands/` directories.

## Command Structure

### Single-file Commands
Located in `commands/` directory. Each command is a standalone `.md` file with:
- YAML frontmatter defining `allowed-tools` and `description`
- Command instructions in the body

Examples: `commit-msg.md`, `update-memory.md`

### Multi-file Commands
Located in `commands/[command-name]/` directories with:
- `COMMAND.md` - Main command definition with frontmatter
- Additional `.md` files - Language-specific or configuration templates

Example: `modernize-repo/` contains `COMMAND.md`, `js.md`, `python.md`, `php.md`

## Key Commands

### `/commit-msg`
Analyzes `git diff --staged` and generates conventional commit messages with gitmoji prefixes (`:bug:`, `:sparkles:`, `:memo:`, etc.). Uses only `Bash(git:*)`, `Read(All files)`, and `Write(docs/spec/*.md)`.

### `/update-memory`
Re-reads and updates project documentation in `docs/spec/*.md`. Handles file splitting for large documents (>2000 tokens), validates against test code, and tracks commit hashes for incremental updates. Always processes files in this order:
1. screens.md
2. configuration.md
3. components.md
4. utilities.md
5. databases.md
6. overview.md
7. known_bugs.md
8. todo.md

### `/modernize-repo`
Detects project type (Node.js/Python/PHP) and generates Docker setup, CI/CD workflows, and README updates. Supports options: `--skip-docker`, `--skip-ci`, `--readme-only`. Creates multi-stage Dockerfiles and GitHub Actions workflows.

## Important Patterns

### Allowed Tools Syntax
Commands use restricted tool access via frontmatter:
- `Bash(git:*)` - Only git commands
- `Read(*.md,*.js)` - Specific file patterns
- `Write(docs/spec/*.md)` - Specific directory/pattern
- `Fetch(*)` - All web fetch operations

### Project Detection
For `/modernize-repo`, always detect the target project's language and package manager (package.json, pyproject.toml, go.mod) before applying templates. Verify compatibility between command type and project type.

### Documentation Workflow
`/update-memory` requires:
- Reading existing Memory docs before updating
- Splitting files that exceed 2000 tokens
- Treating test code as source of truth
- Adding commit hash to last line of each file
- Writing all output in English

## Testing Commands

Since this repository contains command definitions (not executable code), testing involves:
1. Copy command files to a test project's `.claude/commands/`
2. Verify command appears in Claude Code slash command list
3. Test command execution in the target project
4. Validate frontmatter tool restrictions are enforced

## File Organization

```
commands/
├── commit-msg.md              # Single-file command
├── update-memory.md           # Single-file command
└── modernize-repo/            # Multi-file command
    ├── COMMAND.md             # Main definition
    ├── js.md                  # Node.js template
    ├── python.md              # Python template
    └── php.md                 # PHP template

examples/
├── commit-msg.md              # Usage examples
├── update-memory.md           # Usage examples
└── modernize-repo.md          # Language-specific examples
```

## Contributing New Commands

When adding commands:
1. Use YAML frontmatter with `allowed-tools` and `description`
2. For complex commands, use multi-file structure with `COMMAND.md`
3. Add usage examples to `examples/` directory
4. Update README.md with command documentation
5. Ensure command instructions are clear and self-contained
6. Restrict tools to minimum required for the command's purpose

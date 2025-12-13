# Claude Code Commands Collection

A collection of useful slash commands for [Claude Code](https://claude.ai/claude-code), designed to enhance your development workflow.

## 📋 Available Commands

### `/commit-msg`

Proposes concise commit messages in English with appropriate gitmoji tags.

**Features:**
- Generates clear, professional commit messages
- Automatically adds appropriate gitmoji prefixes
- Focuses on clarity without emoji clutter

**Allowed tools:**
- `Bash(git:*)` - Git operations
- `Read(All files)` - Read project files
- `Write(docs/spec/*.md)` - Write to spec documentation

**Usage:**
```bash
/commit-msg
```

### `/update-memory`

Records and updates project documentation in a readable state for both humans and AI.

**Features:**
- Re-reads and updates existing Memory docs (`docs/spec/*.md`)
- Automatically handles file splitting for large documents
- Validates implementation against test code
- Maintains documentation in English

**Allowed tools:**
- `Bash(git:*)` - Git operations
- `Bash(npm:*)` - NPM operations
- `Read(*.md,*.js,*.html,*.json)` - Read project files
- `Write(docs/spec/*.md)` - Write to spec documentation
- `Fetch(*)` - Web fetch operations

**Target documentation files (created/updated in order):**
1. `@spec/overview.md`
2. `@spec/configuration.md`
3. `@spec/components.md`
4. `@spec/utilities.md`
5. `@spec/database.md`
6. `@spec/known_bugs.md`
7. `@spec/todo.md`

**Usage:**
```bash
/update-memory
```

## 🚀 Installation

1. Clone this repository or download the command files
2. Copy the `.md` command files to your project's `.claude/commands/` directory
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

See the [Claude Code documentation](https://claude.ai/claude-code) for more details on creating custom commands.

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

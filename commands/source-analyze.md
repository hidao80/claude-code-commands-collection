---
name: source-analyze
allowed-tools: Bash(git log:*), Bash(sed:*), Bash(cat:*), Bash(find:*), Bash(wc:*), Read(*), Write(docs/analyzed/*.md), WebFetch(*), Glob(*), Grep(*)
description: Record and update project documentation in a readable state for both humans and AI
model: sonnet
color: gold
---

# Details

Re-read the existing Memory docs (`docs/analyzed/*.md`) and update them with attention to the following points:

- Use the maximum number of @Explore subagents.
- When writing to multiple files, perform the process in parallel.
- Edit only the content that corresponds to the existing memory doc filenames
- Completely remove deleted files/components
- Accurately reflect changed type definitions
- If the content to be output to a file exceeds 2000 tokens, split the file and give each an appropriate filename
- If test code exists, treat the test code as the source of truth, and if there is a discrepancy between the test code and implementation, note it as a remark
- Add the commit hash of the creation time to the last line of each file, and when updating, only the differences from that commit will be included.
- Omit explanations originating from the framework and AI agents configuration files.

## Target files

**Important**: Create or update in the following order.

Filename | Required sections
--- | ---
`docs/analyzed/screens.md` | Entry Point, Default Route, URL Pattern, Controller inheritance hierarchy, Base class, View File Convention, Basic template files
`docs/analyzed/configurations.md` | Main configuration files, Environment-specific settings
`docs/analyzed/components.md` | Application Structure, Custom Vendor Namespace
`docs/analyzed/utilities.md` | Global helper functions,
`docs/analyzed/databases.md` | Connection Informations, Architecture, Migration, List of tables by category and their summaries, Summary of domain areas
`docs/analyzed/overview.md` | Project Overview, Technology stack, Repository structure, Request Flow, Domain configuration, scale, Number of steps (approximate number of lines), Main features, Development Workflow
`docs/analyzed/notes.md` |
`docs/analyzed/known_bugs.md` | Security issues, Architectural/design issues, Compatibility issues, Notes on analytical limitations
`docs/analyzed/todo.md` | Security (high priority), Test, Database, Code Quality, Infrastructure, Developer Experience, Performance
`docs/analyzed/naming_convention.md` | Variable, Table name, Column name, Function name, Class name
`docs/analyzed/use_cases/*.md` | Use case diagram (using Mermaid notation)

## Front matter format

```md
---
name: analyzed-{basename}
description: {State the purpose in one sentence.}
type: analysis
---
```

**Output example:**

```md
---
name: analyzed-components
description: Detailed explanation of the repository's main components and responsibilities
type: analysis
---
```

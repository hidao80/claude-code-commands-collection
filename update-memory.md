---
allowed-tools: Bash(git:*), Bash(npm:*), Read(*.md,*.js,*.html,*.json), Write(docs/spec/*.md), Fetch(*)
description: "Record and update project documentation in a readable state for both humans and AI"
---

Re-read the existing Memory docs (`docs/spec/*.md`) and update them with attention to the following points:
- **Important**: Use English when writing to files
- Edit only the content that corresponds to the existing memory doc filenames
- Completely remove deleted files/components
- Accurately reflect changed type definitions
- **Important**: If the content to be output to a file exceeds 2000 tokens, split the file and give each an appropriate filename
- **Important**: If test code exists, treat the test code as the source of truth, and if there is a discrepancy between the test code and implementation, note it as a remark
- **Important**: Add the commit hash of the creation time to the last line of each file, and when updating, only the differences from that commit will be included.

## Target files

**Important**: Create or update in the following order.

1. @spec/overview.md
2. @spec/configuration.md
3. @spec/components.md
4. @spec/utilities.md
5. @spec/database.md
6. @spec/known_bugs.md
7. @spec/todo.md

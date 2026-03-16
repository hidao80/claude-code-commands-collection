---
allowed-tools: Bash(git log:*), Read(*), Write(docs/analyzed/*.md), WebFetch(*)
description: "Record and update project documentation in a readable state for both humans and AI"
---

# Detail of "source-analyize" Slash Command

Re-read the existing Memory docs (`docs/analyzed/*.md`) and update them with attention to the following points:
- **Important:** Use the maximum number of @Explore subagents.
- **Important**: Use English when writing to files
- Edit only the content that corresponds to the existing memory doc filenames
- Completely remove deleted files/components
- Accurately reflect changed type definitions
- **Important**: If the content to be output to a file exceeds 2000 tokens, split the file and give each an appropriate filename
- **Important**: If test code exists, treat the test code as the source of truth, and if there is a discrepancy between the test code and implementation, note it as a remark
- **Important**: Add the commit hash of the creation time to the last line of each file, and when updating, only the differences from that commit will be included.
- Exclude package management directories such as `vendor`, `node_modules`, and `/lib/python*/site-packages/` from the search.

## Target files

**Important**: Create or update in the following order.

1. @docs/analyzed/screens.md
2. @docs/analyzed/configurations.md
3. @docs/analyzed/components.md
4. @docs/analyzed/utilities.md
5. @docs/analyzed/databases.md
6. @docs/analyzed/overview.md
7. @docs/analyzed/notes.md
8. @docs/analyzed/known_bugs.md
9. @docs/analyzed/todo.md

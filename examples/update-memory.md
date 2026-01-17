# update-memory Command Examples

This command maintains project documentation in `docs/spec/` that serves as a readable knowledge base for both humans and AI assistants.

## Usage

Run `/update-memory` after making significant changes to your codebase. The command will:

1. Re-read existing memory docs in `docs/spec/`
2. Analyze the current codebase state
3. Update documentation to reflect changes
4. Track the commit hash for incremental updates

## Target Files

The command creates/updates files in this specific order:

1. `docs/spec/screens.md` - UI screens and pages
2. `docs/spec/configuration.md` - Config files and settings
3. `docs/spec/components.md` - Reusable components
4. `docs/spec/utilities.md` - Helper functions and utilities
5. `docs/spec/databases.md` - Database schemas and models
6. `docs/spec/overview.md` - Project overview and architecture
7. `docs/spec/known_bugs.md` - Known issues and bugs
8. `docs/spec/todo.md` - Pending tasks and improvements

## Examples

### Example 1: Initial Documentation Setup

**Scenario:** New project with no existing memory docs

**Command:** `/update-memory`

**Result:** Creates all 8 documentation files based on current codebase analysis.

**Sample output for `docs/spec/overview.md`:**
```markdown
# Project Overview

## Architecture
This is a React-based web application using Vite as the build tool.

## Tech Stack
- Framework: React 18
- Build Tool: Vite
- Language: TypeScript
- Styling: Tailwind CSS

## Directory Structure
- `src/` - Source code
- `public/` - Static assets
- `tests/` - Test files

---
Last updated from commit: abc1234
```

### Example 2: After Adding New Components

**Scenario:** Added a new `UserProfile` component and `useAuth` hook

**Before `docs/spec/components.md`:**
```markdown
# Components

## Button
A reusable button component with variants.

---
Last updated from commit: abc1234
```

**After running `/update-memory`:**
```markdown
# Components

## Button
A reusable button component with variants.

## UserProfile
Displays user information including avatar, name, and email.

### Props
- `userId: string` - The user's unique identifier
- `showEmail?: boolean` - Whether to display email (default: true)

## Hooks

### useAuth
Custom hook for authentication state management.

### Returns
- `user: User | null` - Current user object
- `login: (credentials: Credentials) => Promise<void>`
- `logout: () => void`

---
Last updated from commit: def5678
```

### Example 3: Reflecting Deleted Features

**Scenario:** Removed the legacy `LegacyDashboard` component

**Before `docs/spec/screens.md`:**
```markdown
# Screens

## Dashboard
Main dashboard with analytics.

## LegacyDashboard
Old dashboard (deprecated).

## Settings
User settings page.

---
Last updated from commit: abc1234
```

**After running `/update-memory`:**
```markdown
# Screens

## Dashboard
Main dashboard with analytics.

## Settings
User settings page.

---
Last updated from commit: ghi9012
```

The `LegacyDashboard` entry is completely removed.

### Example 4: Type Definition Changes

**Scenario:** Updated `User` type to include new fields

**Code change:**
```typescript
// Before
interface User {
  id: string;
  name: string;
}

// After
interface User {
  id: string;
  name: string;
  email: string;
  role: 'admin' | 'user' | 'guest';
  createdAt: Date;
}
```

**Updated `docs/spec/databases.md`:**
```markdown
# Database Models

## User
| Field | Type | Description |
|-------|------|-------------|
| id | string | Unique identifier |
| name | string | Display name |
| email | string | Email address |
| role | 'admin' \| 'user' \| 'guest' | User role |
| createdAt | Date | Account creation timestamp |

---
Last updated from commit: jkl3456
```

### Example 5: Test Code Discrepancy

**Scenario:** Test expects behavior that differs from implementation

**Test file `tests/utils/formatDate.spec.ts`:**
```typescript
it('should format date with timezone', () => {
  expect(formatDate(date, { timezone: 'UTC' })).toBe('2024-01-15 00:00 UTC');
});
```

**Implementation `src/utils/formatDate.ts`:**
```typescript
// Note: timezone option not yet implemented
export function formatDate(date: Date): string {
  return date.toISOString().split('T')[0];
}
```

**Updated `docs/spec/utilities.md`:**
```markdown
# Utilities

## formatDate
Formats a Date object to a string.

### Parameters
- `date: Date` - The date to format

### Returns
- `string` - Formatted date string (YYYY-MM-DD)

> **Remark:** Test expects timezone option support (`{ timezone: 'UTC' }`), but this is not yet implemented. See `tests/utils/formatDate.spec.ts:15`.

---
Last updated from commit: mno7890
```

### Example 6: Large File Splitting

**Scenario:** Components documentation exceeds 2000 tokens

**Result:** The command automatically splits into multiple files:

- `docs/spec/components-ui.md` - UI components (Button, Card, Modal, etc.)
- `docs/spec/components-forms.md` - Form components (Input, Select, Checkbox, etc.)
- `docs/spec/components-layout.md` - Layout components (Header, Footer, Sidebar, etc.)

## Key Rules

1. **Language:** All documentation is written in English
2. **Incremental Updates:** Only changes since the last commit hash are processed
3. **Deletions:** Removed code is completely removed from docs (no "deprecated" markers left behind)
4. **Type Accuracy:** Type definitions must exactly match the current codebase
5. **File Size:** Files exceeding 2000 tokens are automatically split
6. **Test Priority:** Test code is treated as the source of truth for expected behavior

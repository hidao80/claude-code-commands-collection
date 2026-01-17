# commit-msg Command Examples

This command proposes concise commit messages in English with appropriate gitmoji prefixes.

## Usage

Simply run `/commit-msg` after staging your changes. The command will analyze your staged changes and suggest a commit message.

## Examples

### Bug Fix

**Staged changes:** Fixed a null pointer exception in user authentication

**Suggested commit message:**
```
:bug: Fix null pointer exception in user authentication
```

### New Feature

**Staged changes:** Added dark mode toggle to settings page

**Suggested commit message:**
```
:sparkles: Add dark mode toggle to settings page
```

### Documentation Update

**Staged changes:** Updated README with installation instructions

**Suggested commit message:**
```
:memo: Update README with installation instructions
```

### Refactoring

**Staged changes:** Extracted validation logic into separate utility function

**Suggested commit message:**
```
:recycle: Extract validation logic into utility function
```

### Performance Improvement

**Staged changes:** Optimized database query for user listing

**Suggested commit message:**
```
:zap: Optimize database query for user listing
```

### Dependency Update

**Staged changes:** Updated React from 18.2.0 to 18.3.0

**Suggested commit message:**
```
:arrow_up: Update React to 18.3.0
```

### Configuration Change

**Staged changes:** Added ESLint configuration file

**Suggested commit message:**
```
:wrench: Add ESLint configuration
```

### Test Addition

**Staged changes:** Added unit tests for payment processing module

**Suggested commit message:**
```
:white_check_mark: Add unit tests for payment processing module
```

## Common Gitmoji Reference

| Gitmoji | Code | Description |
|---------|------|-------------|
| 🐛 | `:bug:` | Fix a bug |
| ✨ | `:sparkles:` | Introduce new features |
| 📝 | `:memo:` | Add or update documentation |
| ♻️ | `:recycle:` | Refactor code |
| ⚡ | `:zap:` | Improve performance |
| 🔧 | `:wrench:` | Add or update configuration files |
| ✅ | `:white_check_mark:` | Add or update tests |
| ⬆️ | `:arrow_up:` | Upgrade dependencies |
| 🎨 | `:art:` | Improve structure/format of the code |
| 🔥 | `:fire:` | Remove code or files |
| 🚀 | `:rocket:` | Deploy stuff |
| 🔒 | `:lock:` | Fix security issues |

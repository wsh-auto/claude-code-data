---
requiredSkills:
  - mdr:edit
  - mdr:dev-monkeypatch
  - mdr:dev-typescript
autoLoaded: ["@SKILL.md", "@README.md", "@_FAQ.md", "@_TEST.md", "@_ISSUES.md", "@_HACKMD.md"]
hackmd: https://hackmd.io/CdDqVzOxT722aDKjPgMn9Q
---
# Development Guidelines for lib-ccd

## TABLE OF CONTENTS
1. What This Skill Contains
2. Documentation Structure
  - Critical Documentation (Read First)
  - Technical Documentation (Implementation Details)
  - Optional Documentation (Deep Dives)
3. When to Read Which Docs
  - Scenario 1: Using the Parser Library
  - Scenario 2: Understanding Claude Code Data Format
  - Scenario 3: Implementing Features
  - Scenario 4: Debugging Issues
4. Testing the Library
5. Monkeypatch Maintenance
  - Upstream Sync
  - Local Patches
6. Next Task

## What This Skill Contains

This skill contains the **vendored `claude-code-data` library** - a TypeScript parser and analyzer for Claude Code conversation logs (JSONL format).

**Not a wrapper**: This IS the claude-code-data library itself, with comprehensive reverse-engineering documentation of Claude Code's data format.

**Fork**: https://github.com/wsh-auto/claude-code-data
**Upstream**: https://github.com/osolmaz/claude-code-data

## Documentation Structure

### Critical Documentation (Read First)

**README.md** - Quick start guide
- What the library does
- Basic usage examples
- API overview
- **When to read**: First time using this library, or when you need a quick refresher

**STRUCTURE.md** - Project organization
- File structure
- Module organization
- Build outputs
- **When to read**: When navigating the codebase or understanding where things live

### Technical Documentation (Implementation Details)

**docs/conversation-format-specification.md** - JSONL format spec ⭐ MOST IMPORTANT
- Complete JSONL line-by-line format
- All message types (user, assistant, summary)
- Tool integration patterns
- **When to read**:
	- When parsing conversation files
	- When implementing new parsers
	- When debugging format issues
	- **This is the MOST IMPORTANT doc for understanding Claude Code's data format**

**docs/data-models-and-schemas.md** - TypeScript interfaces
- Complete type definitions
- Validation schemas
- Type guards
- **When to read**: When working with TypeScript types or implementing type-safe code

**docs/claude-code-data-storage-analysis.md** - Storage system overview
- High-level architecture
- File organization
- Configuration structure
- **When to read**: When understanding where Claude Code stores data or implementing backup/migration tools

**docs/implementation-notes.md** - Engineering patterns
- Architecture patterns
- Performance optimizations
- Security considerations
- **When to read**: When implementing advanced features or optimizing performance

### Optional Documentation (Deep Dives)

**docs/README.md** - Documentation index
- Overview of all docs
- Use cases
- Examples
- **When to read**: When you need to understand what documentation exists

## When to Read Which Docs

### Scenario 1: Using the Parser Library

**Goal**: Import and use the library to parse conversations

**Read**:
1. README.md (usage examples)
2. STRUCTURE.md (understand exports)

**Example**:
```typescript
import { parseConversation, calculateConversationStats } from 'claude-code-data';

const conversation = await parseConversation('./conversation.jsonl');
const stats = calculateConversationStats(conversation);
```

### Scenario 2: Understanding Claude Code Data Format

**Goal**: Understand how Claude Code stores conversations

**Read**:
1. **docs/conversation-format-specification.md** ⭐ START HERE
2. docs/data-models-and-schemas.md (TypeScript types)
3. docs/claude-code-data-storage-analysis.md (storage system)

**This is the core value**: Comprehensive reverse-engineering of Claude Code's JSONL format.

### Scenario 3: Implementing Features

**Goal**: Add new parser functions or extend the library

**Read**:
1. README.md (understand existing API)
2. STRUCTURE.md (module organization)
3. docs/conversation-format-specification.md (format details)
4. docs/data-models-and-schemas.md (type definitions)
5. src/parser/conversation-parser.ts (implementation examples)

**Test**:
```bash
npm test                    # Run all tests
npm test -- --watch         # Watch mode
npm test -- --coverage      # Coverage report
```

### Scenario 4: Debugging Issues

**Goal**: Fix parsing errors or unexpected behavior

**Read**:
1. docs/conversation-format-specification.md (validate format assumptions)
2. docs/implementation-notes.md (error handling patterns)
3. src/parser/conversation-parser.test.ts (test examples)

**Debug flow**:
1. Check test-data/ for example files
2. Add failing test case
3. Fix implementation
4. Verify tests pass

## Testing the Library

**Test organization**:
```
src/
├── parser/
│   ├── conversation-parser.ts
│   ├── conversation-parser.test.ts    # Parser tests
│   ├── conversation-analyzer.ts
│   └── conversation-analyzer.test.ts  # Analyzer tests
├── utils/
│   ├── type-guards.ts
│   └── type-guards.test.ts            # Type guard tests
test-data/
└── example-conversation.jsonl          # Test fixtures
```

**Running tests**:
```bash
# All tests (36 passing)
npm test

# With UI
npm run test:ui

# With coverage
npm test -- --coverage

# Specific file
npm test conversation-parser
```

**Adding tests**:
1. Add test fixtures to test-data/
2. Write tests co-located with implementation (*.test.ts)
3. Use real conversation data when possible
4. Test edge cases (empty conversations, malformed data, etc.)

## Monkeypatch Maintenance

### Upstream Sync

This is a forked repository. Sync with upstream periodically:

```bash
# Fetch upstream changes
git fetch upstream

# Review changes
git log HEAD..upstream/main

# Merge upstream
git pull upstream main

# Resolve conflicts if needed
git mergeconf

# Push to our fork
git push origin main
```

### Local Patches

**Current patches** (none yet):
- No patches applied to upstream code

**If adding patches**:
1. Document in CLAUDE.md (create if needed)
2. Use format from `skill://mdr:dev-monkeypatch`
3. Test thoroughly after patching
4. Push to wsh-auto fork

## Next Task

This library is **production-ready** with 36 passing tests. No immediate implementation needed.

**For future agents**:
- When using this library, start with README.md
- When understanding Claude Code format, start with docs/conversation-format-specification.md
- When implementing features, read STRUCTURE.md + relevant docs/
- When debugging, check test fixtures in test-data/

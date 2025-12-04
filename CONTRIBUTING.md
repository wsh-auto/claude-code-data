# Development Guidelines for lib-ccd

## TABLE OF CONTENTS
1. What This Skill Contains
2. Architecture
3. Development Workflow
  - Installation
  - Build
  - Testing
  - Type Checking
4. Project Structure
5. Documentation Structure
  - Critical Documentation (Read First)
  - Technical Documentation (Implementation Details)
  - Optional Documentation (Deep Dives)
6. When to Read Which Docs
  - Scenario 1: Using the Parser Library
  - Scenario 2: Understanding Claude Code Data Format
  - Scenario 3: Implementing Features
  - Scenario 4: Debugging Issues
7. API Implementation
  - Parser Functions
  - Analyzer Functions
  - Type Guards
8. Monkeypatch Maintenance
  - Upstream Sync
  - Local Patches
9. Key Features
10. Privacy and Ethics

## What This Skill Contains

This skill contains the **vendored `claude-code-data` library** - a TypeScript parser and analyzer for Claude Code conversation logs (JSONL format).

**Not a wrapper**: This IS the claude-code-data library itself, with comprehensive reverse-engineering documentation of Claude Code's data format.

**Fork**: https://github.com/wsh-auto/claude-code-data
**Upstream**: https://github.com/osolmaz/claude-code-data

## Architecture

**Library-first design:**
- TypeScript library with comprehensive type safety
- Complete type definitions for all Claude Code data structures
- Stream-based and full-file parsing options
- Built-in validation and error handling

**Core capabilities:**
- **Type-Safe Parsing**: Complete TypeScript interfaces for all data structures
- **JSONL Parsing**: Stream and parse conversation files efficiently
- **Validation**: Built-in validation for message structure and relationships
- **Error Handling**: Comprehensive error reporting for malformed data
- **Statistics**: Calculate cost, tokens, duration, and conversation metrics
- **Tree Building**: Convert flat message lists to hierarchical conversation trees
- **Branch Analysis**: Extract active conversation paths from summaries
- **Tool Usage Tracking**: Track and analyze tool usage patterns

**Architecture patterns:**
- Local-first design (matches Claude Code philosophy)
- Append-only JSONL format
- UUID-based message threading
- Support for conversation branching and merging
- Per-message cost calculation and aggregation

## Development Workflow

### Installation
```bash
npm install
```

### Build
```bash
# Build TypeScript library
npm run build

# Type checking only (no compilation)
npm run typecheck
```

### Testing
```bash
# Run all tests (34 passing)
npm test

# Run tests with UI
npm run test:ui

# Run with coverage
npm test -- --coverage

# Specific file
npm test conversation-parser
```

### Type Checking
```bash
npm run typecheck  # tsc --noEmit
```

## Project Structure
```
lib-ccd/
├── src/
│   ├── parser/
│   │   ├── conversation-parser.ts       # Main parser
│   │   ├── conversation-parser.test.ts  # Parser tests
│   │   ├── conversation-analyzer.ts     # Analysis tools
│   │   └── conversation-analyzer.test.ts
│   ├── utils/
│   │   ├── type-guards.ts               # Type guards
│   │   └── type-guards.test.ts
│   └── types.ts                         # Type definitions
├── docs/
│   ├── conversation-format-specification.md  # ⭐ MOST IMPORTANT
│   ├── data-models-and-schemas.md
│   ├── claude-code-data-storage-analysis.md
│   └── implementation-notes.md
├── test-data/
│   └── example-conversation.jsonl       # Test fixtures
└── dist/                                # Build output
```

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

## API Implementation

### Parser Functions
**`parseConversation(filePath: string): Promise<Conversation>`**
- Parses entire conversation file
- Returns: `{ summaries: SummaryMessage[], messages: BaseMessage[] }`
- Use: Most common parsing operation

**Example**:
```typescript
import { parseConversation } from "claude-code-data";

const conversation = await parseConversation("./conversation.jsonl");
console.log(`Found ${conversation.summaries.length} summaries`);
console.log(`Found ${conversation.messages.length} messages`);
```

**`readConversationLines(filePath: string): AsyncGenerator<ConversationEntry>`**
- Streams conversation entries one at a time
- Use: Large files, memory-constrained environments

**`parseAndValidateConversation(filePath: string): Promise<Conversation>`**
- Parses with validation
- Use: When you need to ensure data integrity

### Analyzer Functions
**`calculateConversationStats(conversation: Conversation): ConversationStats`**
- Returns comprehensive statistics
- Includes: message counts, token usage, cost, duration

**Example**:
```typescript
import { parseConversation, calculateConversationStats } from "claude-code-data";

const conversation = await parseConversation("./conversation.jsonl");
const stats = calculateConversationStats(conversation);

console.log(`Total cost: $${stats.totalCostUSD.toFixed(4)}`);
console.log(`Total tokens: ${stats.totalTokens.input + stats.totalTokens.output}`);
console.log(`Average response time: ${stats.averageResponseTimeMs.toFixed(0)}ms`);
```

**`buildConversationTree(messages: BaseMessage[]): ConversationNode`**
- Creates hierarchical tree structure
- Use: Visualizing conversation flow

**Example**:
```typescript
import { buildConversationTree } from "claude-code-data";

const tree = buildConversationTree(conversation.messages);
// Tree structure with parent-child relationships
```

**`getActiveBranch(conversation: Conversation): BaseMessage[]`**
- Extracts active conversation path from summary
- Use: Finding current conversation state

### Type Guards
**Type guard functions** for runtime type checking:

```typescript
import { isUserMessage, isAssistantMessage, hasToolResult } from "claude-code-data";

for (const message of conversation.messages) {
  if (isUserMessage(message)) {
    console.log("User:", message.message.content);
  } else if (isAssistantMessage(message)) {
    console.log("Assistant:", message.message.content[0].text);
    console.log("Cost:", message.costUSD);
  }
}
```

**Available type guards:**
- `isUserMessage(message)` - Check if message is from user
- `isAssistantMessage(message)` - Check if message is from assistant
- `isSummaryMessage(entry)` - Check if entry is a summary
- `hasToolResult(message)` - Check if user message has tool results

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

## Key Features

### Data Insights
**Architecture Patterns:**
- **Local-first**: Matches Claude Code's philosophy
- **Append-only**: JSONL format for immutability
- **UUID-based threading**: Enables conversation branching
- **Message Threading**: Support for conversation branching and merging
- **Cost Tracking**: Per-message cost calculation and aggregation
- **Tool Integration**: Request-response cycles with detailed results

### Data Format Notes
This library handles various Claude Code conversation formats:
- Messages may or may not include `costUSD` and `durationMs`
- Summary `leafUuid` may not always correspond to existing messages
- Tool usage appears in both assistant requests and user results
- Parent-child relationships enable conversation branching

### Use Cases
**For Developers:**
- **Data Export**: Extract conversations for analysis or backup
- **Custom Tools**: Build integrations using the data models
- **Analytics**: Analyze conversation patterns and costs
- **Migration**: Move data between Claude Code installations

**For Researchers:**
- **Conversation Analysis**: Study human-AI interaction patterns
- **Cost Optimization**: Analyze token usage and efficiency
- **Tool Usage**: Research tool adoption and effectiveness
- **Feature Usage**: Understand feature flag impacts

**For System Administrators:**
- **Backup Strategies**: Design data protection workflows
- **Storage Management**: Monitor disk usage growth
- **Privacy Compliance**: Understand data collection scope
- **Performance Monitoring**: Track conversation loading times

## Privacy and Ethics

When working with Claude Code data:
- **Respect User Privacy**: Never share conversation contents without permission
- **Anonymize Examples**: Remove personal information from code samples
- **Local Processing**: Keep analysis tools local-first like Claude Code itself
- **Secure Storage**: Protect any exported data with appropriate security measures

## Contributing

This analysis is based on reverse engineering Claude Code's data files. If you discover additional patterns or corrections:
1. Document your findings clearly
2. Provide example data (anonymized)
3. Include validation methods
4. Update relevant documentation sections

**Version**: Analysis based on Claude Code version 1.0.3 (May 2025). Data formats may evolve in future versions.

**License**: MIT

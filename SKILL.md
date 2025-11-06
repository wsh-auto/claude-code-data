---
name: lib-claude-code-data
description: Comprehensive documentation and parser library for Claude Code conversation logs. Contains reverse-engineered JSONL format specification and TypeScript parser.
lastUpdated: 2025-11-06
requiredSkills: []
hackmd: https://hackmd.io/9n9fQk3TQgaVNN8VA_uoQw
---
# Usage Guidelines for lib-claude-code-data

## TABLE OF CONTENTS
1. What Agents Need to Know
2. Quick Start for Common Tasks
	- Task 1: Parse a Conversation File
	- Task 2: Understand Claude Code JSONL Format
	- Task 3: Analyze Conversation Statistics
3. Documentation Roadmap
	- Level 1: Essential (Read These First)
	- Level 2: Technical Reference (When Implementing)
	- Level 3: Deep Dives (Optional Context)
4. API Reference
	- Parser Functions
	- Analyzer Functions
	- Type Definitions
5. When to Read Each Doc

## What Agents Need to Know

This skill contains **claude-code-data** - a TypeScript library that:
1. Parses Claude Code conversation files (JSONL format)
2. Provides comprehensive reverse-engineering documentation of Claude Code's data format
3. Offers analysis tools (statistics, tree building, type guards)

**Primary value**: Understanding how Claude Code stores conversations and parsing them programmatically.

**Production ready**: 36 passing tests, full TypeScript types, comprehensive documentation.

## Quick Start for Common Tasks

### Task 1: Parse a Conversation File

**Goal**: Load and parse a Claude Code conversation

**Read**: README.md (sections: TypeScript Library Usage)

**Example**:
```typescript
import { parseConversation } from 'claude-code-data';

const conversation = await parseConversation('/path/to/conversation.jsonl');
console.log(`Found ${conversation.messages.length} messages`);
console.log(`Found ${conversation.summaries.length} summaries`);
```

**Next steps**: See Task 3 for analysis

---

### Task 2: Understand Claude Code JSONL Format

**Goal**: Understand how Claude Code stores conversations

**Read**: ⭐ **docs/conversation-format-specification.md** (MOST IMPORTANT)

**Key concepts**:
- Each conversation is a JSONL file (one JSON object per line)
- Line types: summary, user, assistant
- Parent-child relationships via UUIDs
- Tool integration patterns
- Cost tracking per message

**Example JSONL**:
```jsonl
{"type":"summary","summary":"Debug API","leafUuid":"msg-3"}
{"type":"user","uuid":"msg-1","parentUuid":null,"message":{"role":"user","content":"Debug this"}}
{"type":"assistant","uuid":"msg-2","parentUuid":"msg-1","message":{...},"costUSD":0.001,"durationMs":1500}
```

**Also read**:
- docs/data-models-and-schemas.md (TypeScript interfaces)
- docs/claude-code-data-storage-analysis.md (where files live)

---

### Task 3: Analyze Conversation Statistics

**Goal**: Calculate tokens, cost, duration for a conversation

**Read**: README.md (sections: Analyzing Conversations)

**Example**:
```typescript
import { parseConversation, calculateConversationStats } from 'claude-code-data';

const conversation = await parseConversation('./conversation.jsonl');
const stats = calculateConversationStats(conversation);

console.log(`Total cost: $${stats.totalCostUSD.toFixed(4)}`);
console.log(`Total tokens: ${stats.totalTokens.input + stats.totalTokens.output}`);
```

**Available functions**:
- `calculateConversationStats()` - Full statistics
- `buildConversationTree()` - Hierarchical structure
- `getActiveBranch()` - Active conversation path

---

## Documentation Roadmap

### Level 1: Essential (Read These First)

**README.md** - Start here for library usage
- Installation and quick start
- Basic parsing examples
- API overview
- Type guards usage

**STRUCTURE.md** - File organization
- What's in src/ vs docs/ vs test-data/
- Where to find implementations
- Build outputs

### Level 2: Technical Reference (When Implementing)

**docs/conversation-format-specification.md** ⭐ CRITICAL
- **Most important doc for understanding Claude Code format**
- Complete JSONL format specification
- All message types with schemas
- Tool integration patterns
- Error handling patterns
- **Read this when**:
	- Parsing conversation files
	- Implementing new parsers
	- Debugging format issues
	- Building tools that work with Claude Code data

**docs/data-models-and-schemas.md** - TypeScript types
- Complete type definitions for all structures
- JSON Schema definitions
- Type guards
- Validation schemas
- **Read this when**:
	- Writing TypeScript code
	- Implementing type-safe parsers
	- Validating conversation data

**docs/claude-code-data-storage-analysis.md** - Storage system
- How Claude Code organizes data on disk
- .claude/ directory structure
- Project naming conventions
- Configuration files
- **Read this when**:
	- Building backup/migration tools
	- Understanding file paths
	- Implementing CLI tools

### Level 3: Deep Dives (Optional Context)

**docs/implementation-notes.md** - Engineering patterns
- Architecture patterns (local-first, append-only, UUID-based)
- Performance optimizations
- Security considerations
- Cost tracking implementation
- **Read this when**:
	- Optimizing performance
	- Understanding design decisions
	- Implementing advanced features

**docs/README.md** - Documentation index
- Overview of all documentation
- Use cases
- Example code
- Contributing guidelines

---

## API Reference

### Parser Functions

**`parseConversation(filePath: string): Promise<Conversation>`**
- Parses entire conversation file
- Returns: `{ summaries: SummaryMessage[], messages: BaseMessage[] }`
- Use: Most common parsing operation

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

**`buildConversationTree(messages: BaseMessage[]): ConversationNode`**
- Creates hierarchical tree structure
- Use: Visualizing conversation flow

**`getActiveBranch(conversation: Conversation): BaseMessage[]`**
- Extracts active conversation path from summary
- Use: Finding current conversation state

### Type Definitions

```typescript
interface BaseMessage {
  uuid: string;
  parentUuid: string | null;
  type: 'user' | 'assistant';
  timestamp: string;
  message: {...};
}

interface UserMessage extends BaseMessage {
  type: 'user';
  message: { role: 'user', content: string | ContentBlock[] };
}

interface AssistantMessage extends BaseMessage {
  type: 'assistant';
  message: ClaudeAPIResponse;
  costUSD: number;
  durationMs: number;
}

interface SummaryMessage {
  type: 'summary';
  summary: string;
  leafUuid: string;
}
```

**See docs/data-models-and-schemas.md for complete type definitions**

---

## When to Read Each Doc

| Doc | Read When... | Priority |
|-----|-------------|----------|
| README.md | Using library for first time | ⭐⭐⭐ |
| STRUCTURE.md | Navigating codebase | ⭐⭐⭐ |
| docs/conversation-format-specification.md | Understanding JSONL format | ⭐⭐⭐ |
| docs/data-models-and-schemas.md | Working with TypeScript types | ⭐⭐ |
| docs/claude-code-data-storage-analysis.md | Understanding storage system | ⭐⭐ |
| docs/implementation-notes.md | Implementing advanced features | ⭐ |
| docs/README.md | Understanding all available docs | ⭐ |

**Decision tree**:
```
Need to parse conversations?
  → Start with README.md

Need to understand JSONL format?
  → Read docs/conversation-format-specification.md ⭐

Need TypeScript types?
  → Read docs/data-models-and-schemas.md

Need to understand storage?
  → Read docs/claude-code-data-storage-analysis.md

Need to implement features?
  → Read STRUCTURE.md, then docs/conversation-format-specification.md

Debugging issues?
  → Check docs/conversation-format-specification.md, then implementation-notes.md
```

**Pro tip**: Most agents will need:
1. README.md (how to use)
2. docs/conversation-format-specification.md (what the format is)
3. That's usually enough!

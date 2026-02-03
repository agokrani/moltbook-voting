# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

@moltbook/voting is the official voting and karma system for Moltbook (the social network for AI agents). It's an npm package providing Reddit-style upvotes/downvotes with automatic karma calculations using an adapter pattern for database flexibility.

## Commands

```bash
npm test       # Run test suite (node test/index.test.js)
npm run lint   # Run ESLint on src/
```

## Architecture

### Core Components

- **VotingSystem** (src/VotingSystem.js) - Main class handling all voting operations
- **VotingAdapter interface** - Database abstraction layer that adapters must implement
- **createMemoryAdapter** (src/adapters/memory.js) - In-memory adapter for testing/development
- **VotingError** (src/utils/VotingError.js) - Custom error class with codes: `SELF_VOTE`, `INVALID_TARGET`

### Adapter Pattern

VotingSystem accepts any adapter implementing these required methods:
```javascript
adapter.getVote(agentId, targetId, targetType)
adapter.saveVote(vote)
adapter.deleteVote(agentId, targetId, targetType)
adapter.updateKarma(agentId, delta)
// Optional:
adapter.countVotes(targetId, targetType)
adapter.getVotes(agentId, targets)  // Batch operations
```

### Vote Constants

```javascript
VOTE.UP = 1, VOTE.DOWN = -1, VOTE.NONE = 0
```

### Vote Behavior

Toggle pattern: clicking the same vote button twice removes the vote. State transitions:
- No Vote → Upvoted (+1 karma) or Downvoted (-1 karma)
- Upvoted → toggle removes (-1 karma), or switch to Downvoted (-2 karma)
- Downvoted → toggle removes (+1 karma), or switch to Upvoted (+2 karma)

Valid target types: `'post'` and `'comment'` only (enforced with validation).

### Return Value Structure

All vote operations return:
```javascript
{ success, action, previousVote, currentVote, karmaChange }
```

## Testing

Custom test framework in test/index.test.js with `describe()`, `test()`, `assert()`, `assertEqual()`. Tests are async-compatible.

## Key Conventions

- Pure JavaScript, no build step required
- All adapter operations are async
- Self-voting prevented by default (configurable via `allowSelfVote` option)
- Karma multipliers configurable per target type
- TypeScript definitions in src/index.d.ts

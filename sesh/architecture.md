# Architecture

## File structure

Everything is a markdown file with YAML frontmatter, stored in `.sessions/` at the project root.

```
.sessions/
├── 2026-02-24_auth-refactor.md        # Active session
├── 2026-02-24_cache-strategy.md       # Idea (just a lightweight session)
├── 2026-02-20_api-redesign.md         # Completed session
└── SESSION_INDEX.md                   # Auto-maintained table of contents
```

## Session file format

```yaml
---
date: '2026-02-24'
created_at: '2026-02-24T10:30:00-05:00'
updated_at: '2026-02-24T14:15:00-05:00'
status: in_progress
description: "Refactor auth to use JWT instead of session cookies"
parent: null
blocked_by: null
related: []
---

# Auth refactor

## Goal

Replace session cookies with JWT. Current auth breaks when
we add the mobile API.

## Decisions

- Going with short-lived access tokens + refresh tokens
- Tried opaque tokens first, but JWT lets the API gateway
  validate without hitting the auth service

## Todo

- [x] Swap session middleware for JWT verification
- [ ] Add refresh token rotation
- [ ] Update mobile client
```

The body is freeform markdown. No enforced structure. Some sessions are three lines, some are pages of notes and decisions. The tool doesn't care.

YAML frontmatter gives you structured + flexible: status, dates, and relationships are structured enough to query and index. The body captures whatever you need. Every static site generator, every markdown tool, and every AI agent already knows how to parse frontmatter.

## Key design decisions

### Why files, checked into git

Sessions are checked into git on purpose. This isn't a side effect — it's the point.

- **Decision log.** `git log -- .sessions/` shows every session from any period. `git blame` shows when each decision was recorded. The code shows *what* was built. The session doc shows *why*.
- **No migration runner.** Want to add a new field? Just start putting it in new session frontmatter. Old sessions without it work fine.
- **Files are the API.** AI agents can read and edit files directly. No API layer needed.
- **Survives context compaction.** When Claude Code compresses older messages, the session doc is still on disk. The agent re-reads it and picks up where it left off.

### Auto-save with smart git integration

After every command that changes a file, sesh auto-commits `.sessions/`. Three rules:

1. **Skip if non-session files are staged.** Don't mix session bookkeeping into feature commits.
2. **Squash consecutive sesh commits.** Multiple sesh commands in quick succession produce one commit, not three.
3. **Meaningful commit messages.** "sesh: complete auth-refactor, new cache-strategy" — extracted from changed filepaths.

This is the biggest time saver when you have multiple agents working on the same repo. Each agent is creating sessions, capturing ideas, completing work — and none of that clutters your git history or collides with your feature commits.

### Atomic operations

Every mutating command: validate input, find target session, update file, update INDEX, print feedback, auto-save. If any step fails, the whole operation fails. No half-updated state.

The CLI handles file creation, frontmatter, and index update in one command. The fewer tool calls an agent needs for bookkeeping, the more of its context window stays focused on actual work.

### Relationships as plain strings

Two relationship types, both stored as filename strings in frontmatter:

- **Parent-child:** `parent: 2026-02-20_api-redesign.md`. Mechanism behind `--child-of` for rabbit holes.
- **Peer links:** `related[]` array with bidirectional links. Cross-project format: `wrangler:2026-02-24_cli-update.md`.

No graph database. Just strings in YAML arrays, resolved through a project registry at `~/.config/sesh/projects.json`.

### Ideas are just sessions

`sesh idea` creates a lightweight session file with a different status. Same frontmatter, same flat directory. When you're ready to work on one, `sesh start` flips it to active.

The cost of capturing a thought needs to be near zero. If it requires any ceremony, you'll keep the thought in your head and lose it when the context window compacts.

### Child sessions for rabbit holes

`sesh new "upgrade-jwt-library" --child-of auth-refactor` creates a child session linked to the parent. Dive into the rabbit hole with its own doc. When done, climb back up to the parent.

`sesh around` shows the neighborhood:

```
auth refactor (you are here)

Children:
  upgrade jwt library (2026-02-24)
  mobile api integration (2026-02-25)

Related:
  api redesign (2026-02-20)
```

### Cross-project search

A global Claude Code rule (in `~/.claude/rules/global/`) teaches every agent how to use sesh — no per-project setup. Register projects once, then search across all of them, link sessions across repos.

`sesh search "auth" --in wrangler,blog` uses ripgrep. For deeper search, sessions auto-index into [qmd](https://github.com/tobi/qmd) for semantic search.

## What I'd do differently

**The INDEX update logic is fragile.** SESSION_INDEX.md uses regex-replaced markdown tables. If I rebuilt, I'd store the index as JSON and render to markdown on read.

**Gray-matter's date handling is surprising.** It converts YAML date strings to JavaScript Date objects automatically. When writing back, you need to convert to ISO strings or it serializes in a format that breaks on re-read. Add a parsing layer that normalizes dates immediately after gray-matter touches them.

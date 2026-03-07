# Starter Prompts

Three prompts to go from zero to working session management.

## 1. Build the CLI

```
Build a CLI tool called "sesh" for managing development sessions.

Core concept: sessions are markdown files with YAML frontmatter
stored in .sessions/ at the project root.

Session file format:
- Filename: YYYY-MM-DD_slug-name.md (date prefix for sorting)
- YAML frontmatter: date, created_at, updated_at, status
  (in_progress|blocked|complete|paused|idea), description,
  parent, blocked_by, related[]
- Markdown body: freeform notes

Commands needed:
- init (create .sessions/ directory)
- new "name" [--description "text"] [--child-of parent]
- complete "name" [--summary "text"] (appends summary to body)
- block "name" --reason "text"
- resume "name"
- idea "description" (lightweight session with status: idea)
- start "idea" (flip idea to active session)
- list (show all sessions grouped by status)
- status (quick view of active work)
- show "name" (full session details)
- tree (parent-child hierarchy visualization)
- around "name" (show parent, children, and related)
- search "query" (ripgrep across session files)
- link "a" "b" (bidirectional peer link)

Key behaviors:
- Find .sessions/ by walking up from cwd to git root
- Auto-commit .sessions/ changes after mutations
- Skip auto-commit if user has non-session staged files
- Squash consecutive sesh commits
- Maintain SESSION_INDEX.md (table of active/blocked/completed)
- Relationships: parent field for hierarchy,
  related[] for peer links

Design constraints:
- File-based state only, no database
- Every operation is atomic (file + index update together)
- Session body is freeform, never enforced
- Ideas are zero-friction (no prompts, no templates)
- Git-friendly (all state in diffable text files)
```

## 2. Set up the Claude rule

```
Create a global Claude Code rule for sesh at
~/.claude/rules/global/sesh.md so every project gets
session management automatically. The rule should teach
the agent the key commands and one critical constraint:
session docs preserve the journey — append progress,
don't rewrite history.
```

## 3. First session

```
Run sesh init in this project, then create a session for
whatever we're working on right now.
```

That's it. The first prompt gets you the tool, the second makes it available everywhere, the third gets you using it. Grow it from there.

See [architecture.md](./architecture.md) for the full design decisions and file format spec.

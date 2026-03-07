# sesh

A session management CLI for AI-assisted development. Tracks what you're working on, captures ideas without breaking flow, and links everything together so an AI agent can traverse your work — up, down, sideways, across projects.

## The problem

Every new AI coding session starts from zero. The agent doesn't know what you decided yesterday, what's blocked, or what you were about to try next.

Notes help, but they sprawl. The real problem is rabbit holes — you start on a feature, discover a dependency needs upgrading, go down that hole, find a bug, go down *that* hole. Three levels deep and you've forgotten what you were originally doing.

sesh creates structured session docs, links them together, and is fast enough that an agent can do it in one command without burning context on file creation.

## Philosophy

- **Sessions live in your codebase.** Checked into git, right next to the code they describe. Not ephemeral, not external — part of your living codebase.
- **Unstructured contents.** The frontmatter tracks status, dates, and relationships. The body is whatever the agent writes. What matters is that the raw context is written down, and the agent knows how to find it.
- **Portable.** Markdown files in git. No vendor lock-in. Works with Claude Code, Cursor, or whatever comes next.

## Files

- [architecture.md](./architecture.md) — structure, file formats, design decisions
- [prompt.md](./prompt.md) — starter prompts to build your own version

## Blog post

[SESH — session tracking that survives rabbit holes](https://kurtheinrich.com/blog/sesh-the-blueprint)

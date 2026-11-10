# Lean Code Paths

An Agent Skill for writing and reviewing efficient code across an entire request path. Start with what the caller needs, then follow the cost through the API, service, middleware, and database.

For example, a list endpoint might issue one query per row even though every row needs the same kind of summary. This skill guides an agent to batch the lookup, preserve ordering and access rules, and compare executed query counts for a small page and a full page.

## Use with Codex

Clone or copy this repository into your personal skills directory as `~/.codex/skills/lean-code-paths`. The folder should contain [SKILL.md](SKILL.md) at its root. Start a new session and invoke `$lean-code-paths` with a coding or review request.

To apply it to every coding task, add this line to your own `AGENTS.md`:

> When writing, modifying, refactoring, or reviewing code, use the lean-code-paths skill for the affected path.

## License

[MIT](LICENSE).

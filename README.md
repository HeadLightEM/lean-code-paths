# Lean Code Paths

Lean Code Paths is an Agent Skill for Codex that helps you write and review efficient code. Give it an operation such as “load the asset picker” or “rename my file.” It traces what the caller needs through the API, application code, middleware, and database, looking for work you can remove without changing the result.

Use it while designing a new endpoint, reviewing a change, or investigating a slow request or background job. The instructions are Markdown and apply across languages and frameworks. There are no packages to install.

## Quick start

With Git installed, run the command for your shell. This installs the skill for your user account.

**Windows PowerShell**

```powershell
git clone https://github.com/HeadLightEM/lean-code-paths.git "$env:USERPROFILE/.codex/skills/lean-code-paths"
```

**macOS or Linux**

```sh
git clone https://github.com/HeadLightEM/lean-code-paths.git "$HOME/.codex/skills/lean-code-paths"
```

If you use a custom `CODEX_HOME`, clone into its `skills/lean-code-paths` directory. You can also download the repository as a ZIP and extract it there. The installed folder should contain `SKILL.md` and `references/` directly.

Start a new Codex session and try a scoped review:

```text
Use $lean-code-paths to review the asset list endpoint. Trace one page load
through the executed queries. Which work grows with each extra asset?
Keep this review read-only.
```

To apply the skill by default, add the following line to your global `AGENTS.md` or to the repository instructions for one project:

> When writing, modifying, refactoring, or reviewing code, read and use the lean-code-paths skill for the affected path.

For an installation made with Git, run `git pull --ff-only` inside the installed skill directory to update it.

## Worked example: a paginated asset picker

Suppose a picker shows a name and thumbnail for each asset. Selecting an asset needs its ID, and scrolling needs a continuation cursor. The current endpoint loads a page of full asset records, including large processing reports, then fetches a thumbnail for each row. Two costs grow with page size: unused bytes and database round trips.

1. **Start at the caller.** The picker consumes `id`, `name`, `thumbnailUrl`, and a page-level `nextCursor`. Check other consumers of the endpoint before trimming its response. An export tool may still depend on a field the picker never displays.
2. **Push the selection into SQL.** Filter by the authenticated account, keep a stable sort with a unique tie-breaker, and select the columns needed for the summary and cursor. Leave the processing report out of the list query. An ownership column used only for filtering can stay in the database.
3. **Fetch related data as a set.** For separate thumbnail records, collect the page's asset IDs and fetch the thumbnails in a bounded batch. Assemble them in page order. In this example, a page query plus one lookup per asset would issue 31 queries for 30 assets. A page query plus one thumbnail batch would issue two if the page fits in one batch. These counts illustrate the query shapes.
4. **Verify through the endpoint.** Request both a small page and a full page in the test environment. Count executed statements, inspect the response, and check ordering, cursor behavior, and account isolation. Report a latency improvement only after timing comparable requests.

## Other decisions the skill guides

| When you see this | What to do |
| --- | --- |
| A trusted worker rechecks the same task before every stage | Carry forward verified identity, ownership, and parsed arguments. Check fresh external input at its boundary and enforce conditions that can change at the write. |
| A simple update performs separate existence, ownership, and state lookups | Put the required conditions in the write predicate. Interpret affected rows according to the database driver and the API's existing error contract. |
| A proposed fix adds a lock, cache, queue, or more workers | Identify the race or measured bottleneck first. Compare simpler options such as an atomic update, request-local reuse, or a batched call. |
| A fixed value gains its own configuration file or feature flag | Use the existing default or define the rule once. Add configuration when a deployment or operator actually needs to vary it. |
| A loop repeatedly scans a collection or copies large payloads | Build a keyed lookup when matches repeat. Reuse decoded data, file references, or bounded streams where they reduce work. |

## Using it on your own tasks

For an implementation request, name the behavior that must survive the optimization:

```text
Use $lean-code-paths to simplify the rename endpoint. Preserve ownership
and the existing response when a caller submits the same name twice.
Exercise the endpoint and check the stored result.
```

For a performance investigation, name the outcome the user waits for:

```text
Use $lean-code-paths to investigate slow image exports. Measure queue wait,
render time, and time until the file is ready. Use the same inputs to compare
changes and check that the output dimensions and quality are preserved.
```

Under this skill's default workflow, the agent verifies APIs and job outputs through their normal entry points and runs the relevant project checks. For web changes, it hands the user the page, actions, and expected result for visual and manual acceptance. You can explicitly delegate browser checks.

## Reference

- [Skill instructions](SKILL.md) cover day-to-day implementation choices, scope, and verification.
- [Database access and transactions](references/database.md) covers projections, batching, pagination, indexes, conditional writes, locking, and recovery.
- [Runtime and middleware](references/runtime.md) covers API contracts, caches, queues, retries, file processing, and frontend loading.

## License

[MIT](LICENSE).

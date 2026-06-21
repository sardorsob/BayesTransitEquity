# AGENTS

## Operating Model

Codex and Sardor may both act as Builder and QA, but the passes should remain distinct:

- Builder: implement the agreed task and move it to `in-review`.
- QA: verify scope, evidence, tests, artifacts, and documentation before marking `done`.

## Commit Rules

- Do not add AI co-author trailers.
- Commit locally only unless Sardor explicitly asks for a push.
- Use organized commit titles.
- Include task IDs once `context/TASKS.md` contains active tasks.
- If committing as part of this workflow, do not include `Co-Authored-By`, `Generated-By`, or similar attribution trailers.
- Sardor pushes unless he explicitly asks Codex to push.

## Workflow Files

The workflow control files live in `context/` to keep the repository root clean:

- `context/PROJECT.md`
- `context/SCOPE.md`
- `context/TASKS.md`
- `context/HANDOVER.md`
- `context/AGENTS.md`

These files are local workflow state unless Sardor explicitly asks to publish them.

## Builder / QA Loop

1. Builder reads the relevant context files and task block.
2. Builder marks `pending -> in-progress`.
3. Builder implements or drafts only the listed scope.
4. Builder records evidence and marks `in-progress -> in-review`.
5. QA checks scope, evidence, tests, artifacts, and claims.
6. QA marks `in-review -> done` or `in-review -> needs-fix`.

Codex and Sardor may both act as Builder and QA, but the pass should be named clearly.

## Stop Conditions

Stop and ask before continuing when:

- The scope is ambiguous and a wrong assumption would be expensive.
- A task needs credentials, private data, or a large new dependency.
- A modeling result could be misleading without additional QA.
- A push, force-push, or publication step is required.

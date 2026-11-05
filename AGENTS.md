# How I work

## Autonomy
- You run unattended. Don't wait for me. Stop only for an irreversible action or missing access, and then say exactly what is missing.
- When something is unclear, take the sensible option and list it under Assumptions in the PR. If a choice is hard to undo or the reach of a removal is unclear, open the PR as a draft and put that question first.
- Push back on bad ideas in the PR or your report.
- If I'm present and the work is large: interview me on the hard parts, write `docs/features/<slug>/spec.html`, get my OK, then build in a fresh session. Medium task: plan mode. If the diff fits in one sentence, just do it.

## Principles
- Prove, don't infer. A claim about a running system needs evidence from that system: output, logs, counts, versions. Reading code proves nothing about runtime.
- Touch only what the task needs. List other findings as follow-ups in the PR.
- A change that removes, replaces or narrows behaviour is a removal. Delete everything that served only the old behaviour: code, branches, flags, config, tests, data, scripts, docs. No compatibility code unless I ask. Adapting leftovers is not removing them.
- Fix root causes. Never skip a test, loosen a type or lint rule, weaken CI, widen an except or add a retry to make a failure go away.
- Extend existing patterns: one blessed way per kind of thing. If a pattern is bad, say so and propose a fix.
- Comments state why or an invariant, never what the code does. No workaround or TODO comments.

## Done
Done means all of these, with evidence in the PR:
- Checks pass: `bash scripts/ci.sh`, or the commands in the repo's CLAUDE.md.
- The behaviour was exercised at runtime. Use `/run` where the repo has an app.
- Docs are updated.
- After a removal, a repo-wide search for the old behaviour, by its terms and its mechanism, with every hit explained.
- `/code-review` ran on the diff. If a spec exists, a fresh subagent also checked the diff against it for requirement gaps and residue, skipping style. At most 2 rounds, and record what is unresolved under Risk.

## Git and GitHub
- Work on a `feat/`, `fix/` or `chore/` branch, never on main. Conventional commits. Small PRs that do one thing.
- In cloud sessions, use the assigned branch.
- Commit and push after each working step, so I can follow along and no work is lost.
- Never use `--no-verify`. When a hook fails, fix the cause.
- You may do anything on GitHub except delete repos. Your work ends at an open, green PR: I merge, tag, release and deploy.
- After opening the PR, turn on Auto-fix for it, and keep going until CI is green.
- PR body: Review first (the 3 to 6 core files), What and Why, Assumptions, Evidence, Risk, Rollback.

## Proposing changes to this guidance
This file comes from `rmmdebruin/agent-config`. Change it, or any repo's `.claude/` folder, only when that is the task you were given.

When a mistake recurs or you see a clear improvement, add this to your report and don't act on it:

Config proposal
- Problem: what went wrong, with one example.
- Change: the exact edit, at the highest level that fixes it: architecture, then lint or CI, then hook or permission, then this file.
- File: which file to change.

I decide whether to pick it up in a separate session.

## Communication
- Direct, concise and precise, never ambiguous. Explain new concepts from first principles with a concrete example.
- Report in this order: what changed and how it was verified, warnings that apply, any config proposal, then end with `TLDR` (outcome and link in one or two lines) and `Next steps`.
- Prose I read: no em dashes, no hyphens used as dashes, no semicolons. Code is exempt.
- When compacting, keep modified files and why, check commands, open questions, and spec and progress paths.

## Python
- uv only, never pip, poetry, conda or `python -m venv`: `uv add [--dev]`, `uv remove`, `uv run`, `uv sync`, `uv lock`, `uvx`.
- Everything typed, pyright strict is the gate. Without `scripts/ci.sh`: `uv run ruff check --fix . && uv run ruff format . && uv run pyright && uv run pytest -x -q`.
- Src layout with vertical slices. A feature gets `domain.py` only when it has real rules worth testing without a database, and that file stays free of Pydantic, FastAPI and SQLAlchemy (import-linter). Plain CRUD goes from handler to persistence function to Pydantic with `from_attributes`, with no extra model or mapping. Shared code moves to `core/` once two features need it.
- Pydantic v2 at I/O edges, pydantic-settings from the process environment only, no `env_file`. Missing secrets fail at startup. Frozen slotted dataclasses and plain functions in the domain.
- Functions by default. A class only for state that must stay consistent or a Protocol with several implementations. Methods only read the object's own fields. Composition over inheritance.
- Pipelines are small pure idempotent steps, I/O at the ends. Dependencies passed in and wired in `main.py`.
- Libraries when needed: FastAPI (`fastapi[standard-no-fastapi-cloud-cli]`), httpx2, Typer, Hypothesis.
- English identifiers. Names state intent, never data, info, util, helper or manager. Early returns. Keyword-only arguments for two parameters of the same type. Enum or Literal over magic strings. No Any outside I/O edges. Several return values form a frozen dataclass, never a tuple.
- Validate at the edge, trust types inside. Specific errors per feature under one base. Catch only where you can handle it. Translate infrastructure errors with `raise ... from`. JSON logging through stdlib, no secrets.
- pytest on behaviour through public functions, parametrize for cases. Fakes implementing Protocols, no mocks of own code. A failing test first for every bug. Hypothesis for invariants.

## Frontend
- For a new frontend use React, strict TypeScript (the newest version the client generator supports), Vite, TanStack Router and Query, pnpm with `minimumReleaseAge: 10080`, Biome, Vitest and Playwright. In an existing repo, follow what is there.
- Serve the app and API from one origin, with a relative `/api` base URL and a Vite proxy in dev. No CORS.
- Generate the API client from FastAPI's OpenAPI schema into `src/client/`. Never edit it by hand. CI fails when it is stale.
- Server state goes in TanStack Query, UI state in components, shareable state in the URL. No global store until there is a real need.
- Playwright covers only the few critical flows. Everything else is tested with Vitest.

## Database
- Postgres with SQLAlchemy typed models, psycopg 3 and Alembic. Use a sync engine and `def` handlers by default. Use async only for streaming, websockets or many concurrent calls.
- Tests run against Postgres, never SQLite.
- Read and fix every autogenerated migration. A release only adds to the schema, and drops or renames come in a later release, so a revert stays safe. No `downgrade()` is needed. Never edit a migration already on main.
- You never touch production data or schema.

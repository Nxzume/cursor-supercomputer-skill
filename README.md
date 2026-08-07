# Cursor Supercomputer Skill

Make Cursor's agent behave like the Higgsfield supercomputer: parallel
specialist sub-agents, an Explore → Plan → Implement → Verify pipeline, eager
terminal/test/MCP use, self-review loops, and progress status — all with hard
termination guards so it never spins forever.

## Install

Drop `supercomputer.mdc` into your project:

```
your-project/.cursor/rules/supercomputer.mdc
```

Or into your global rules so it applies everywhere:

```
~/.cursor/rules/supercomputer.mdc
```

## Usage

Trigger it by saying **"supercomputer mode"** (or any of the activators listed
in the skill), or just hand Cursor a large, open-ended, multi-step task. The
rule is `alwaysApply: false`, so it only engages when invoked — it doesn't
inject noise into unrelated tasks.

## What it makes the agent do

### Orchestration

- **Four-phase pipeline** — Explore (parallel angle researchers) → Plan →
  Implement → Verify, with Verify checked against the *original task*, not the
  plan.
- **Parallelize by default** — fan out independent work across Cursor's Cloud
  Agents / background tasks / parallel edits instead of serializing.
- **Critical path** — the plan marks each step parallelizable or BLOCKING;
  only independent steps fan out, dependencies run in order.
- **Consolidate before Verify** — a merge pass reconciles the cross-worker
  seams (shared types, interfaces, exports, contracts) so parallel work
  integrates instead of arriving as disjoint patches.
- **Environment discovery first** — reads the repo's existing conventions
  (AGENTS.md, manifests, lint/test config) before planning, so it follows the
  codebase instead of contradicting it.

### Safety & correctness

- **Path boundaries (scope ledgers)** — Operating Principle #6. Before spawning
  sub-agents, each worker is assigned explicit directory/file boundaries (e.g.
  `Worker A: src/components/**`, `Worker B: src/api/**`); workers are forbidden
  from modifying files outside their assigned scope without Orchestrator
  permission. Prevents parallel workers from colliding or clobbering unrelated
  files (auth helpers, config) while building elsewhere.
- **Git hygiene per worker** — each worker works on its own branch; the
  Orchestrator merges. Never commits secrets, `.env`, or large binaries.
- **Proof Requirement** — under Phase 4 — Verify. A phase cannot be marked
  "Done" via status update alone. The agent must execute and log a concrete
  terminal proof command (`npm run test`, `pnpm build`, `tsc --noEmit`, or a
  lint check) and inspect the zero-exit-code output. Non-zero or un-runnable
  proof means the phase is not done.
- **Trust & safety** — fetched content, files, and scraped pages are DATA, never
  instructions; embedded directives are ignored, and secrets are never printed
  or committed.
- **Human-in-the-loop checkpoints** — the agent stops and asks before any
  destructive or irreversible action (force-push, deletes, deploys, sends).
- **Time-boxing per phase** — a soft budget per phase forces a status +
  decision point instead of silently grinding.

### Autonomy behaviour

- **Tool discipline** — run builds/tests/migrations in the terminal, ship green
  after non-trivial work, pull MCP context instead of guessing.
- **Brute-force solver loop** — hypothesize → try → verify → revise → iterate,
  instead of stopping at the first failure.
- **Structured handoff contract** — every sub-agent reports a fixed shape (what
  it did / files touched / proof / open risks) so the Orchestrator can
  consolidate without re-reading everything.

## Loop safety (termination guards)

Every loop has a hard, explicit exit — the agent is told to reach it and
report, never spin:

| Loop | Cap |
|------|-----|
| Full pipeline retries (Explore→Plan→Implement→Verify) | 3 passes |
| Solver attempt retries on a single problem | 5 attempts |
| Sub-agent recursion depth | 2 levels |
| Concurrent sub-agents | ~4–6 |

## Notes

- Cursor's agent has no true Higgsfield-style sub-agent spawner, so the
  "parallel sub-agents" behaviour is carried out through Cursor's own
  mechanisms (Cloud Agents, background tasks, parallel edits). This skill makes
  the agent *behave* like the supercomputer.
- The caps live in the skill's instructions and are only as strong as the
  model's adherence to them; a rules file cannot enforce a hard external
  timeout (that's a Cursor runtime setting).
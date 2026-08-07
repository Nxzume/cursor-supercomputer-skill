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

- **Four-phase pipeline** — Explore (parallel angle researchers) → Plan →
  Implement → Verify, with Verify checked against the *original task*, not the
  plan.
- **Parallelize by default** — fan out independent work across Cursor's Cloud
  Agents / background tasks / parallel edits instead of serializing.
- **Tool discipline** — run builds/tests/migrations in the terminal, ship green
  after non-trivial work, pull MCP context instead of guessing.
- **Brute-force solver loop** — hypothesize → try → verify → revise → iterate,
  instead of stopping at the first failure.

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
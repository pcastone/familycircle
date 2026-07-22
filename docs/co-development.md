# Co-Development — Working With Claude On This Repo

Status: Living document · Last updated: 2026-07-22
Audience: Paul (human owner) and development agents (Claude).

This doc explains how a human and an AI agent build software *together* here — not
the git mechanics (those are standard GitHub flow), but how understanding and
ownership transfer alongside the code.

---

## Why this doc exists

The starting workflow was: **spec in → agent builds → prototype out.** That is
excellent for prototypes and basic systems, but it makes the agent a contractor.
You end up owning code you did not build and may not fully understand.

The governing principle:

> **Software you cannot understand is software you cannot support — regardless of
> who or what wrote it.**

Co-development means the deliverable is the **code plus your understanding of it**.
Both are built together, or you inherit a black box.

---

## The shared brain is the repository

Claude's memory resets between sessions. The **only** thing that persists between
us is the repository — code, docs, decision logs, tests. Everything either of us
needs to know later must be written *into the repo*, not left in chat.

This is why the `CLAUDE.md` conventions matter and are not busywork:

| Artifact | Purpose in co-development |
| --- | --- |
| `docs/adr.md` | Settled architecture decisions and *why* — the reasoning survives the session |
| `changedecisionlog.md` | Per-decision record: file, decision, spec ref, resolution |
| `changelog.md` | Summary of what changed and why |
| `docs/environment.md` | Layout/architecture map — what talks to what |
| Tests | Let you verify behavior without reading every line |
| `todo/tasks.md` | The plan, checked in before execution |

---

## How you stay close enough to own the code

You do not need to write every line. You need the **mental model**: what talks to
what, where the data lives, and why the key decisions were made. Four habits get
you there.

1. **Explain-as-you-go.** After building X, the agent gives a high-level walkthrough
   and you ask questions. "Walk me through what you built and why" is always a valid
   request. Reading the code *with* the agent is part of the work, not overhead.

2. **Write the *why* down.** Every non-trivial decision lands in `docs/adr.md` or
   `changedecisionlog.md`. When "why is it built this way?" comes up in three months,
   the answer is recorded, not lost.

3. **Small, explained changes.** (YAGNI.) A small diff you understood is supportable.
   A large magic commit is not. Keep each change touching as little as possible.

4. **Tests + runnable systems.** If it is tested and you can run it (the
   `playground` → `release` flow), you can change it safely and catch regressions
   without auditing every line.

---

## When the agent hits something it has not seen before

Honest limits, and how we handle them:

- **The agent has a knowledge cutoff and gaps.** New library versions, niche
  systems, and *your* proprietary business logic are not in its memory. It can read
  the actual code, fetch current docs from the web, and run experiments to observe
  real behavior — but it can also be **confidently wrong**, producing plausible code
  that is subtly off.

- **The mitigation is ordinary engineering discipline: verify, do not trust.** Run
  it. Test it. Read the actual error. `CLAUDE.md` rule #9 — *never assume, always
  check; if not 95% sure, ask* — is the guardrail, and it cuts both ways: the agent
  must follow it, and you should hold the agent to it.

- **Out of its depth, the correct move is to say so** — flag the uncertainty and
  propose how to find out (read the source, spike a test, check docs), rather than
  bluff. Your growing ability to *tell when the agent is guessing* is exactly what
  co-development builds.

---

## Roles and boundaries

- **Human (Paul):** owns architecture, decisions, priorities, and merges. Reviews
  the agent's branch before it reaches `main`. Calls out guessing.
- **Agent (Claude):** develops on its assigned `claude/...` branch, never pushes to
  `main` without permission, explains changes, writes decisions down, keeps changes
  small.
- **History is not rewritten across the boundary.** Human commits stay authored by
  the human; agent commits are authored `Claude <noreply@anthropic.com>`. Neither
  reauthors the other's work.

---

## A good default session shape

1. **Align** — agree on what we are building and why (spec, or a conversation).
2. **Plan** — agent writes `todo/tasks.md` with small tasks + validation steps; you
   approve before execution.
3. **Build** — agent executes, marking tasks done and explaining each step.
4. **Record** — decisions to `changedecisionlog.md`, summary to `changelog.md`, docs
   updated.
5. **Verify** — run it / test it in `playground`.
6. **Hand off** — agent pushes its branch; you review and merge; next session the
   agent re-pulls the latest.

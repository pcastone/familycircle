# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Working agreement

1. **Think-Plan-Check-Execute**: Read the codebase for relevant files, look for reuse vs create, then write a plan to `todo/tasks.md` with small, detailed tasks and validation steps. If clearing `todo/tasks.md` with unfinished tasks, copy them to `todo/gutter.md` first.
2. **Check in before executing** — share the plan and get approval before starting work.
3. **Mark tasks complete as you go** and give high-level explanations of changes at each step.
4. **Simplicity first** — every change should impact as little code as possible. Reuse existing code, scripts, and patterns before creating new ones.
5. **Log every decision** in `changedecisionlog.md` with: file affected, what was decided, spec reference, and resolution method.
6. **Log level changes** in `changelog.md` with: file affected, summary of what was done, spec reference, and resolution method.
7. **Documentation** — always update training, tutorial, and AI guide documentation while making changes to code.
8. **Git commit** after each logical group of completed tasks with a summary of changes.
9. **Be the expert** Never assume always check, if your not 95% sure ask questions, if you have defer a task or hit a blocker stop and explain the problem.
10. **YAGNI** Follow YAGNI principles and one-liner solutions

## Repository

- GitHub: https://github.com/pcastone/familycircle (`origin`, pushed to `main`).
- Collaborators: `pcastone` (owner/admin), `palarasc` (write).

## What this project is

An **eldercare coordination platform**. Adult children coordinate care for an aging parent; the parent is the *subject* of a "care circle", not the account owner. Three things in priority order (`docs/business.md` §2):

1. **Crisis mode** — tier-3 emergency payload (meds, allergies, DNR/POA, insurance) readable offline, in seconds, from a cold app launch in an ER hallway. **No network call in the read path.**
2. **The broadcast** — one action notifies the whole circle.
3. **The incident log** — append-only, timestamped record of emergencies and check-ins. This is the accruing value *and* the grant-outcomes instrument.

Grant-funded, not profit-seeking → no growth mechanics, and outcomes measurement is a first-class requirement.

## Read the design docs before writing code

Three docs carry nearly all the design; the source tree is still a skeleton. Read them before proposing anything architectural:

- **`docs/adr.md`** — settled architecture decisions with rationale. These are **not open for re-litigation** without a new ADR entry and human sign-off.
- **`docs/platform-spec.md`** — the normative build spec: repo layout, SQL schema, RLS policy shapes, sync protocol, enclave contract, CI gates, milestone build order (M1–M5).
- **`docs/business.md`** — the *why*: problem framing, elder-dignity commitments, family-conflict policy, compliance posture.

Cite the ADR number when a task appears to conflict with a decision; escalate rather than improvise.

### Hard rules (from `docs/adr.md` "standing instructions" + spec §14 CI gates)

Violating these is a spec violation, not a style preference:

- **Never propose microservices** (ADR-004) or **E2EE** (ADR-007). Both were considered and deliberately traded away.
- **Authorization lives in Postgres RLS** (ADR-005), not application filters. A `WHERE circle_id = ?` is an optimization, never the isolation boundary. Never connect as a role that owns the tables (RLS is bypassed for table owners).
- **The enclave (tier 4) is a sealed module** (ADR-011). No `enclave.*` reference outside `internal/enclave`; nothing enclave-derived in feed, search, notifications, telemetry, logs, or sync responses; no ambient sync — explicit per-device provisioning only. Internals are **deferred to a dedicated security session** — do not improvise them.
- **Tier 4 never appears in a sync response.** The `tier <= 3` assert in the sync pull path is load-bearing; do not remove it.
- **Never compute or display per-person historical contribution totals** (ADR-010) — forward-looking unclaimed work is fine, historical per-sibling scoreboards never are. They become exhibits in contested estates.
- **Never seed non-prod from production data** (ADR-012). Synthetic generators only.
- **Never emit names, health terms, or enclave values to logs/telemetry.** Redaction happens at the emitter, not the sink.
- **The elder can always read their own record and the incident log about themselves**, at zero write permission (ADR-001). This is an RLS carve-out, not a UI convention.
- Forbidden schema patterns: no `is_sensitive` column (sensitivity is tier + enclave placement), no `max_elders`-style caps, no aggregate views computing per-person counts.
- `incident` and `audit_event` are append-only — `REVOKE UPDATE, DELETE`. Corrections are new rows with `amends`.
- Never conflate `occurred_at` (user-asserted, editable) with `recorded_at`/`recorded_hlc` (immutable). These records end up in legal and medical contexts.

## Current state of the code

A **Go backend skeleton only**. Three `main.go` files that print a line each; no HTTP server, no database, no migrations, no `internal/` packages, no tests, no frontend, no mobile. `go.mod` has zero dependencies. Everything in `docs/platform-spec.md` is still to be built — start at spec §15 milestone M1 (migrations, identity, circle, RLS + RLS test matrix, sqlc pipeline).

```
src/backend/          module familycircle/backend (go 1.25.6)
  cmd/be_server/      HTTP server entrypoint (stub)
  cmd/be_test/        (stub)
  cmd/admintool/      (stub)
  config/app.toml     backend config source
  justfile            the build interface
src/playground/       build output — assembled + tested before release
  bin/                compiled binaries (committed to git)
  config/             copied from src/backend/config by `just config`
```

Note the mismatch to resolve when real code lands: `docs/platform-spec.md` §2 specifies `/cmd/server/`, `/internal/…`, `/migrations/`, `/gen/`, `/clients/app/`, `/clients/web/`. Those paths are **relative to the Go module root (`src/backend/`)**, and the existing command is named `be_server`, not `server`. Follow the repo's directory conventions below for placement; follow the spec for module structure inside `src/backend/`.

## Commands

All builds run from `src/backend/` and are driven by a `justfile`:

```sh
just              # list recipes
just build        # build every cmd/* into ../playground/bin
just config       # copy src/backend/config/ into ../playground/config/
just run be_server # build + config + run one command
just test         # go test ./...
just fmt / vet / tidy / clean
```

`just` is not always installed (it is absent in the Claude Code web container). The direct equivalents:

```sh
cd src/backend
go build -o ../playground/bin/be_server ./cmd/be_server
go test ./...
go test ./internal/circle -run TestName -v   # single package / single test
go vet ./...
```

Per `docs/platform-spec.md` §14, CI eventually gates on: `go test ./...`, an RLS visibility matrix against real Postgres (testcontainers), sync convergence property tests, an offline crisis-render test, static checks for cross-module table access / `enclave.` leakage / forbidden schema patterns, and an extended-build symbol check.

## Directory conventions (from README.md)

The repo enforces a fixed layout — put things where they belong rather than inventing new top-level dirs:

- `src/` — working source code: `src/backend/` (Go), `src/frontend/` (Svelte web), `src/mobile/ios/`, `src/mobile/droid/`.
- `src/playground/` — where code is assembled and tested before release.
- `release/` — final/compiled runnable code. **Runs against `release/config`, not the top-level `config/`.** Release runs write to `release/logs`, not `logs/`.
- `config/` — master config, copied into `release/` at build time.
- `scripts/` — all scripts live here.
- `logs/` — runtime logs from `src/`.
- `docs/` — currently `adr.md`, `business.md`, `platform-spec.md`. README also reserves `project_prd.md`, `environment.md`, `howto.md`, `running.md`, `summary/`, `defects/` (none created yet).
- `todo/` — `tasks.md` (current tasks), `bugfix.md` (bug tracking; link a `docs/defects/` doc for major bugs). Both still to be created.
- `scratch/` — temporary files.

**Two config / two log trees.** `src/` and `release/` are separate runtime environments. `config/` is the source of truth and is copied to `release/` at build time — edit `config/`, not `release/config`, unless deliberately patching a release. (Today the backend's own `src/backend/config/` feeds `src/playground/config/` via `just config`; the top-level `config/` is still empty.)

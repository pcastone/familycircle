# CLAUDE.md — Backend (Go)

Scoped guidance for `src/backend/`. **Additive to the root `CLAUDE.md`** — root
process rules (Think-Plan-Check-Execute, logging, YAGNI, two-config/two-log) still
apply; this file only adds Go/backend specifics. Architecture is settled in
`docs/platform-spec.md` and `docs/adr.md` — read them before changing structure.

## Stack
- Go `1.25.6`, module `familycircle/backend`. A single Go monolith (ADR: modular
  monolith, one deployable). Postgres + RLS is the system of record.

## Build & test (use `just`, recipes in `justfile`)
- `just build` — builds every `cmd/*` into `../playground/bin`.
- `just run <cmd>` — build + copy config + run (e.g. `just run be_server`).
- `just config` — copy `config/` into the playground config dir.
- `just test` → `go test ./...` · `just fmt` · `just vet` · `just tidy` · `just clean`.
- Binaries and config land in `src/playground/` (the assembly/test area) — never
  edit `playground/` by hand; it's produced by `just`.

## Layout
- `cmd/` — entrypoints: `be_server`, `be_test`, `admintool`. Thin `main`s only.
- `config/app.toml` — backend master config. Edit here, not `playground/config`
  (root two-config rule: `config/` is source, copied at build time).

## Module boundary rules (platform-spec §2 — enforced, not advisory)
- A module's tables are accessed **only** by that module's queries. Cross-module
  access goes through the module's Go API, never direct table reads.
- `enclave` (tier-4) is a **sealed module** (spec §9): implement only its contract
  (`Put`, `Get`, `Delete`, `Provision`, `Revoke`, `ListProvisionedDevices`) — do
  NOT extend it. No other module imports its storage; no sqlc query references
  schema `enclave`.
- `gen/` is **sqlc output — never hand-edit.** No hand-written structs mirroring
  tables; all row types come from `migrations/`.
- Migrations are numbered, **forward-only**.

## Testing gates (platform-spec §14 — keep these green)
- `go test ./...` incl. module-boundary tests · RLS matrix (table × role × tier,
  real Postgres) · sync property tests (two devices converge) · crisis offline
  render < 1s · static gates (no cross-module table access in `gen/`, no `enclave.`
  outside `internal/enclave`, tier ≤ 3 assert present in sync pull path).

## Non-obvious invariants
- Sync responses are **tier-filtered to ≤ 3**; tier-4 never appears (load-bearing
  assert — do not remove). `occurred_at` (user-asserted) is stored separately from
  `recorded_at` (HLC, immutable) — never conflate.

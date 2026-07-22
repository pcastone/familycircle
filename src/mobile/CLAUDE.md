# CLAUDE.md — Mobile App (Flutter)

Scoped guidance for `src/mobile/`. **Additive to the root `CLAUDE.md`** — root
process rules still apply. Architecture is settled in `docs/adr.md` (ADR-008,
ADR-009, ADR-011) and `docs/platform-spec.md` (§6.4, §7, §13). The app is the
spec's `/clients/app/`.

> Note: empty scaffold. The stack below is the settled choice; nothing is built yet.

## Stack — ONE Flutter codebase, not two native apps
- **Flutter / Dart. A single cross-platform codebase** builds both iOS and Android.
  `ios/` and `droid/` under this dir are Flutter's generated **platform-runner
  shells** — see their own `CLAUDE.md`. Do NOT create separate Swift/Kotlin app
  codebases; that was considered and rejected (ADR-008 = Flutter).

## Two entrypoints, one codebase (platform-spec §13)
- `main_primary.dart` — replica + sync + crisis + enclave client (full offline).
- `main_extended.dart` — **REST-only**; sync/enclave/crisis libraries **not linked**.
  Prefer a distinct build target over a runtime flag (ADR-008: an agent will
  eventually make a flagged path reachable). CI verifies the extended artifact
  contains **no Drift schema symbols** (`--dart-define` + tree-shaking check).
- Elder mode = primary build with `role=elder` server-driven UI: one-tap check-in,
  today's appointments, own record. No gestures; min touch target 48dp; font scale
  to 200%.

## Local-first storage (primary build only — spec §6.4)
- **SQLite via Drift.** Mirror tables for `care_record`, `incident`, `post` +
  `pending_ops` outbox + `field_hlc` sidecar for per-field LWW.
- Ordering uses **hybrid logical clocks — never wall clock.** Append-only union for
  incidents/check-ins/feed; per-field LWW for mutable contact/policy fields. No CRDT
  library. `occurred_at` (editable) is stored separately from `recorded_at` (HLC,
  immutable) — never conflate; these land in legal/medical contexts.
- Outbox drains FIFO on connectivity; pull runs after every push, on foreground, and
  on silent-push nudge.

## Crisis mode (load-bearing — spec §7, ADR-008)
- One tap from cold launch. **Reads only local tables; NO network in the read path.**
  MUST render < 1s on a mid-tier device with the radio off. Sync runs after render,
  in background. An integration test renders crisis mode with network mocked dead.
- Every crisis open writes a local audit event (usage metric + incident signal).

## Tier-4 enclave client (spec §9, ADR-011)
- Separate **SQLCipher** store (NOT the main replica DB), DEK wrapped by the hardware
  keystore, unlock via OS auth prompt. Main-replica file holds zero enclave plaintext.
- Provisioned copies are offline-readable after unlock; no network in that read path.

## Revocation / wipe (ADR-009)
- Wipe scope is **app-sandbox data only** — never touch user files/photos outside the
  sandbox. We are not an MDM. Never describe wipe as a security guarantee in copy.

## Build & test
- Define real `flutter` commands here once the project is initialized (build flavors
  for primary/extended, `flutter test`, the extended-symbol CI check). Don't guess.

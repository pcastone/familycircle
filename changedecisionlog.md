# Change Decision Log

## 2026-07-22 — Mobile scaffolding: honor Flutter, not native split
- **Files affected:** `src/mobile/CLAUDE.md`, `src/mobile/ios/CLAUDE.md`,
  `src/mobile/droid/CLAUDE.md` (dirs created).
- **Decision:** Scaffold the mobile guidance as **one Flutter codebase** with `ios/`
  and `droid/` as generated platform-runner shells — NOT as two independent native
  (Swift/Kotlin) app codebases.
- **Conflict resolved:** The root `CLAUDE.md` directory convention
  (`src/mobile/ios/` = iOS, `src/mobile/droid/` = Android) reads as separate native
  codebases, which contradicts the settled architecture.
- **Spec reference:** `docs/adr.md` ADR-008 (Local-first primaries on Flutter);
  `docs/platform-spec.md` §13 (Flutter, two Dart entrypoints, one codebase) and §2
  (`/clients/app/` = Flutter). ADR header states decisions are settled and must not
  be re-litigated without a new ADR + human sign-off.
- **Resolution method:** Surfaced the conflict to the human owner via AskUserQuestion
  rather than assuming (CLAUDE.md rule #9). Owner chose "Honor Flutter (one
  codebase)." Nested `CLAUDE.md` files written accordingly; the two platform dirs get
  thin stubs that point back to `src/mobile/CLAUDE.md`.
- **Open item (RESOLVED 2026-07-22):** Path drift reconciled — see next entry.

## 2026-07-22 — Reconcile spec repo layout to the `src/` convention
- **Files affected:** `docs/platform-spec.md` (§2 repo-layout block);
  note-only touch-ups in `src/mobile/CLAUDE.md`, `src/frontend/CLAUDE.md`.
- **Decision:** The repo directory convention wins. Updated the spec's layout block
  so all paths sit under `src/`: `/clients/app/` → `src/mobile/`, `/clients/web/` →
  `src/frontend/`. For coherence in the same block, backend paths were re-rooted to
  match on-disk reality: `/cmd/server/` → `src/backend/cmd/be_server/`,
  `/internal/*` → `src/backend/internal/*`, `/migrations/` → `src/backend/migrations/`,
  `/gen/` → `src/backend/gen/`.
- **Spec reference:** root `CLAUDE.md` directory conventions (`src/backend`,
  `src/frontend`, `src/mobile/*`); on-disk `src/backend/go.mod`.
- **Resolution method:** Human owner explicitly requested the spec use `src/mobile/*`
  and `src/frontend` (the sign-off the ADR header requires). Backend re-root done for
  block coherence and flagged to the owner.
- **Open item:** `docs/platform-spec.md` §11 still references `/seed/synthetic/`; the
  seed-generator location isn't established in the repo conventions, so it was left
  unchanged. Resolve when the seed layout is decided.

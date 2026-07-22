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
- **Open item (not resolved):** Broader path drift remains — the spec uses
  `/clients/app/` + `/clients/web/`, while the repo convention uses `src/mobile/*` +
  `src/frontend/`. Left as-is for now (scaffolds use the repo convention). Reconcile
  in a future ADR if desired.

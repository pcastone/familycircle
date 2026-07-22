# Changelog

## 2026-07-22
- **`docs/co-development.md`** — New doc. Summary: captures how a human and an AI
  agent build software together on this repo (understanding + ownership transfer,
  the repo as shared brain, handling the agent's knowledge limits, roles/boundaries,
  default session shape). Spec reference: user request in session
  (`claude/git-push-pull-workflow-eimlwo`) to document the co-development workflow.
  Resolution: authored from the session discussion; matched existing `docs/` style.
- **Nested `CLAUDE.md` scaffolds** — Added scoped, additive `CLAUDE.md` files for
  `src/backend/` (Go), `src/frontend/` (SvelteKit), `src/mobile/` (Flutter), and
  thin platform-shell stubs in `src/mobile/ios/` + `src/mobile/droid/`. Summary:
  each adds stack-specific build/test commands, layout, and settled invariants while
  deferring global process rules to the root file. Spec reference: `docs/adr.md`
  (ADR-008), `docs/platform-spec.md` (§2, §6.4, §7, §9, §12, §13, §14); `justfile`
  for backend targets. Resolution: grounded every rule in the specs/justfile; created
  the empty `src/frontend`, `src/mobile/ios`, `src/mobile/droid` dirs.
- **`docs/platform-spec.md`** — Reconciled the §2 repo-layout block to the `src/`
  directory convention: `/clients/app/`→`src/mobile/`, `/clients/web/`→`src/frontend/`,
  and backend paths re-rooted under `src/backend/` for coherence. Summary: doc now
  matches the on-disk layout. Spec reference: root `CLAUDE.md` conventions;
  `src/backend/go.mod`. Resolution: owner-requested; scaffold notes updated to drop
  the old `/clients/*` names.

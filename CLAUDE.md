# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository

- GitHub: https://github.com/pcastone/familycircle (`origin`, pushed to `main`).
- Collaborators: `pcastone` (owner/admin), `palarasc` (write).

## Current state

This is a fresh scaffold. Only `README.md` and per-directory `.gitignore` files are tracked — there is **no source code, build system, language, tests, or dependencies yet**. `src/{backend,frontend,playground}/` exist but are empty. Pick a stack when the first real work lands; nothing is committed to yet.

## Directory conventions (from README.md)

The repo enforces a fixed layout — put things where they belong rather than inventing new top-level dirs:

- `src/` — working source code. 
- `src/backend/` - golang backend code, 
- `src/frontend/` - svelte frontend web code
- `src/playground/` — this is where all code get assembled and testes before release 
- `src/mobile/ios/` - IOS development
- `src/mobile/droid/` - android development
- `logs/`.
- `release/` — final/compiled runnable code. **Runs against `release/config`, not the top-level `config/`.** Release runs write to `release/logs`, not `logs/`.
- `config/` — master config, copied into `release/` at build time.
- `scripts/` — all scripts live here.
- `logs/` — runtime logs from `src/` (release logs go under `release/`).
- `docs/` — `project_prd.md` (requirements), `environment.md` (layout/architecture), `howto.md` (setup + run), `running.md` (quick run), `summary/` (summary docs), `defects/` (defect/problem writeups).
- `todo/` — `tasks.md` (current tasks), `bugfix.md` (bug tracking; link a `docs/defects/` doc for major bugs).
- `scratch/` — temporary files.

## Two config / two log trees

The one non-obvious rule: `src/` and `release/` are separate runtime environments. `config/` is the source of truth and is copied to `release/` at build time — edit `config/`, not `release/config`, unless deliberately patching a release.

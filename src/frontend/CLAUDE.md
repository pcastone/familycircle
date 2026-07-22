# CLAUDE.md — Web Frontend (SvelteKit)

Scoped guidance for `src/frontend/`. **Additive to the root `CLAUDE.md`** — root
process rules still apply; this file adds web-client specifics. Architecture is
settled in `docs/adr.md` (ADR-008) and `docs/platform-spec.md` (§12) — the web
client is the spec's `/clients/web/`.

> Note: this dir is empty scaffold. The stack below is the settled choice; nothing
> is built yet.

## Stack & role
- **SvelteKit**, SSR, session-cookie auth against the same Go monolith. REST only.
- A **thin server-rendered client**: NO offline, NO local replica, NO sync code
  path. (Offline/replica is a mobile-primary concern only — ADR-008.)

## Surfaces (platform-spec §12)
- Full record entry (desktop data-entry is the fast onboarding path), feed, incident
  log, audit view for primaries, admin nav stub for a future caseworker role.

## Hard rules
- **No service-worker / cache of tier ≥ 3 responses** — send `Cache-Control:
  no-store`. Do not add offline caching "for performance"; it's a data-exposure bug
  in this product.
- **Tier 3–4 views require step-up auth**: OTP re-auth within the last 10 minutes;
  tier-4 reveals are audited identically to mobile.
- Notification/rendered content is tier-0 in transit — condition/med/enclave values
  render only after auth, in-app.

## Build & test
- To be defined when the SvelteKit project is initialized (npm/pnpm scripts). Add
  the real `dev`/`build`/`test`/`check` commands here once they exist — don't guess.

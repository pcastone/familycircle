# Architecture Decision Record — Eldercare Coordination Platform

Status: Accepted · Last updated: 2026-07-17
Audience: development agents and humans. Decisions here are **settled**. Do not re-litigate without a new ADR entry and human sign-off.

---

## Product thesis

The product redistributes the invisible labor of eldercare across the immediate family, and puts crisis-critical information in their hands when the parent is unresponsive and the network is down.

- **Hook:** crisis mode — emergency payload readable offline, in seconds, in an ER hallway.
- **Accruing value:** the incident log — a timestamped record of emergencies and check-ins that gives the family evidence for the hardest decisions (including reducing the elder's own access).
- **Not the product:** a nicer Google Drive. Passive document storage is what the incumbent free solution already does.

Funding: grant-based, not profit-seeking. Consequence: no viral-growth mechanics; **outcomes measurement is a first-class requirement** (incident log doubles as the instrument; consider validated caregiver-burden scale at onboarding for pre/post evidence).

---

## ADR-001: Elder is a subject, not an account owner

The elder is represented as the subject of a **care circle**, with optional limited user access (check-in, add appointment, update information files) governed by permissions.

```
person            -- all humans: elders, children, extended family
care_circle       -- one per elder
circle_membership -- person × circle × role × permission tier
```

- Elder permissions are expected to **degrade over time**; the data model must support revoking elder write access without removing elder read access.
- **Invariant: the elder can always view their own record and the incident log about themselves**, even with zero write permissions. Transparency without control. Do not build a model where elder-view is impossible.
- Elder onboarding: install app, scan QR / enter PIN. Everything else is set up by the immediate family.

**Rejected:** elder as full account owner (wrong for the degradation path); elder invisible to their own record (infantilizing; destroys trust at the revocation moment).

## ADR-002: Roles, not features, for family structure

- "Partner support" = `role: primary` on a membership row. A married couple are both primaries on one circle; each may be primary on circles for their own parents. Circles overlap; permissions are per-circle.
- "Support up to 6 elders" = 6 memberships. **Do not encode a maximum anywhere in schema or code.** Soft cap in pricing/config only.
- Roles: `primary`, `extended`, `elder` (subject), `executor` (named field, see ADR-010).

## ADR-003: Owner is a role, not the circle's identity ⚠ partially open

The circle must not be structurally owned by a single account. Owner is a transferable role.

- Target: **two primaries minimum**; either can restore access; revoking a primary requires the other primary.
- Rationale: the account creator is the overloaded sibling — the most likely to burn out, divorce, or die first. Also: sole-owner-with-unilateral-revocation is an elder-abuse isolation tool.
- Human has not fully confirmed the two-primary quorum rule. **Schema must support it now** (owner-as-role); enforcement policy may land later.

## ADR-004: Go modular monolith. NOT microservices.

One repo, one binary, one `go test ./...`.

- Hard module boundaries: `internal/` packages, no cross-module DB table access. Boundary violations must be compile-time errors, because agents don't attend design reviews.
- Rationale: team = agentic workflow, timeline = 3–4 mo PoC / 6–8 mo v1. Microservices impose cross-repo contract changes and multi-service test orchestration — the exact tasks agents are worst at — in exchange for scaling properties this product does not need (write volume ≈ a family posting photos).
- Postgres: one primary + streaming replica. No exotic clustering for MVP.

**Rejected: Go microservices.** Do not propose re-splitting. If a module later needs independent deployment, the package seam is the extraction point.

## ADR-005: Authorization enforced in Postgres RLS

Row-level security policies on circle-scoped tables, keyed by membership + tier.

- Rationale: authz rules are relational; they are the primary product risk; and under agentic development they must live **where generated code cannot bypass them**. A query missing `WHERE circle_id = ?` returns zero rows instead of leaking a family's data.
- RLS also gates **what replicates to devices** (ADR-008), which makes it a one-way-door control, not just a response filter.
- Go types are generated from schema (sqlc or equivalent). Schema is the single source of truth; no hand-written row structs.

## ADR-006: Data tiered by time-to-need, not sensitivity

| Tier | Time-to-need | Examples | Audience |
|---|---|---|---|
| 1 Ambient | — | photos, "doing great" posts | extended family feed |
| 2 Logistical | days | appointments, rides | anyone who might drive |
| 3 Clinical/seconds | **seconds** | meds, allergies, DNR, POA, physicians, insurance card | primaries (+ elder) |
| 4 Estate/hours-to-weeks | hours→weeks | organ donation, funeral plan, lawyer, executor, life-insurance carrier + policy #, will location | primaries only; enclave |

- Death is one continuous escalation of a single event: seconds (DNR) → hours (organ donation, funeral instructions) → days (prepaid plan, lawyer) → weeks (insurance claim, probate). The product must not abandon the family at the tier boundary.
- **Estate tier v1 = structured pointers, not documents.** "Original will: safe-deposit box, bank, box #, key location, attorney, executor, date signed." A scanned will is legally inert in most probate anyway; the family's problem is *finding things*. Document storage (envelope encryption, per-circle DEK, opaque blobs, never indexed/parsed) is a later upgrade, not MVP.
- SSN: prefer **not storing it**. Last-4 for identification. Full value only if unavoidable, in the enclave, one code path.

## ADR-007: Server-readable storage (no E2EE), encrypted at rest

- Rationale: recovery when a caregiver's phone dies ("we couldn't recover your mother's DNR" is a project-ending failure), notification fan-out with content, and aggregate outcomes reporting for grant renewal. E2EE forfeits all three for a threat model this population is less exposed to.
- Compensating control: **immutable audit logging of reads on tier 3–4 data**, visible to primaries. This is the elder-financial-abuse tripwire and the compliance backbone. Not optional.
- At-rest encryption throughout; tier-4 additionally per ADR-011.

**Rejected: E2EE.** Do not propose it as a hardening measure; it was considered and traded away deliberately.

## ADR-008: Local-first for primaries; cloud-only for extended family

**Primaries (Flutter):** full offline replica of tiers 1–3. SQLite + Drift, changelog-table sync with cursor, **hybrid logical clocks** (never wall clock for ordering).
- Merge strategy: append-only union for incidents/check-ins/feed (conflict-free by construction); **per-field LWW** for mutable contact/policy fields. No CRDT library; the data doesn't need it.
- `occurred_at` (user-asserted, editable) is stored **separately** from `recorded_at` (HLC, immutable). Never conflate. These records end up in legal/medical contexts.
- Crisis mode: **no network call in the read path.** One tap from cold launch, OS-level auth (ADR-011). The server is the sync/fan-out layer, not the system of record for the emergency payload.

**Extended family:** cloud-only. No replica, no sync code path reachable. Prefer a distinct build target or hard-separated data layer over a runtime flag — an agent will eventually make a flagged path reachable.

**SvelteKit:** thin server-rendered client. No replica. Desktop document-entry + admin (future caseworker surface).

## ADR-009: Revocation = key rotation + wipe-on-sync, honestly framed

- On membership revocation: rotate circle keys (revoked device receives nothing new) and issue wipe-on-next-sync (device drops replicated data on reconnect).
- **Wipe-on-sync is hygiene, not security.** It requires the revoked party's cooperation (airplane mode / never reopening / `adb pull` all defeat it). The real control is: forward secrecy via rotation + the extended-family-no-replica rule shrinking the exposed population to a handful of primaries.
- Wipe scope: **only data the app replicated.** Never touch user-owned files/photos outside the sandbox. We are not an MDM.
- Snapshot leakage (esp. ex-spouse-as-former-primary) is accepted residual risk. **Disclose it in the privacy policy.** Do not claim remote wipe as a security guarantee in any user-facing or grant-facing copy.

## ADR-010: Death is a state transition

- Circle survives death and pivots from care coordination to estate administration. Authority pivots to the **executor**.
- Executor is **named-only** (text designation; no verification workflow in MVP). Compensating control: every change to the executor field is written to the incident log.
- **Incident log survives death.** Immediate family can disclose it outward (health, police, insurance). Consequences already designed for: it is discoverable/subpoenable in contested estates → HLC timestamps, occurred/recorded separation, and **never compute or display per-person contribution totals anywhere** (forward-looking unclaimed work: yes; historical per-sibling scoreboards: never — they are litigation exhibits and family-destroying).

## ADR-011: Sensitive-data enclave — three invariants now, everything else deferred

Tier-4 data lives in a bounded enclave. A dedicated security-design session will finalize its internals; that session is **out of scope for general development**. Marker for agents: `deferred: dedicated design session` — do not improvise enclave internals.

Non-negotiable invariants that must hold from the first migration:

1. **Separate store, not a flag.** Own tables/schema, own encryption key, single access path. Never an `is_sensitive` column on shared tables (generated queries will join across it within weeks).
2. **No outward references to contents.** The main app holds opaque handles ("estate record exists"), never values. Nothing enclave-derived appears in feed, search, notifications, telemetry, or logs. Keeps the future audit surface = one module.
3. **No ambient sync. Explicit provisioning only.** Enclave data reaches a device only via a deliberate "keep on this phone" step; provisioned devices are **server-enumerable**; extended-family members have no provisioning path. Once provisioned, the copy is **fully offline-readable** (hospitals have bad WiFi — this is a hard requirement, not a nice-to-have).

At-rest on device: enclave store encrypted separately from the main replica (SQLCipher or per-field), DEK wrapped by hardware keystore (Secure Enclave / StrongBox), so a pulled main-replica SQLite file yields tiers 1–3, not the DNR and policy numbers.

**Access UX (settled — do not overthink):** standard OS auth prompt — biometric with device PIN/passcode fallback (`LocalAuthentication` / `BiometricPrompt`, device-credential allowed). Keystore key requires that unlock. No custom PIN, no app-specific password. UX note for the enclave session: auth must succeed fast under stress (tears defeat FaceID; the fallback path is used at the worst moment of someone's life).

Consent framing: users are told when they place data into the enclave, and when a device is provisioned to hold it. Deliberate placement + deliberate custody is also the right regulatory story.

## ADR-012: Compliance posture — build BAA-shaped, sign nothing

- Today: not a HIPAA covered entity or business associate. Family-entered data about their own parent is not PHI in our hands.
- Trigger to watch: a covered entity (health system, clinically-integrated AAA) entering or transmitting data **through us on its behalf** → business associate → BAA required. This is most likely to arrive **silently via grant-proposal language** ("discharge planners will onboard patients"). Review grant text for this before submission.
- Already-built controls double as BAA readiness: at-rest encryption (breach safe harbor), tier-3/4 read audit logs, access tiers, enclave separation. Remaining BAA costs are programmatic (risk analysis, breach-notification process, subprocessor BAAs), not architectural.
- **Telemetry redaction at emit** (in the logger, not the sink): no names, no health terms, no enclave-derived values in crash reports or logs. Otherwise Sentry et al. become subprocessors.
- Non-prod environments: **synthetic data only**, structurally enforced (separate credentials, no prod→dev path). Agents will ask to copy prod for realistic seed data. The answer is no.
- Masking is a render-time control (shoulder-surfing, app-switcher snapshots, `FLAG_SECURE`, tap-to-reveal on tier 3–4 that emits an audit event). It is **not** a substitute for encryption or sync-scoping, and crisis-mode tier-3 data must never be gated behind reveal-taps.
- Not-HIPAA ≠ unregulated: FTC Health Breach Notification Rule covers consumer health apps; state law (e.g., CA CMIA) may exceed HIPAA.

---

## Open items (decide before schema freeze where marked)

| # | Item | Blocking? |
|---|---|---|
| 1 | Grant world: federal (ACL/NIA/AHRQ) vs. foundation vs. state AAA — determines partner-org/BAA exposure | Before grant submission |
| 2 | Two-primary quorum for revoking a primary (ADR-003) — schema supports it; policy unconfirmed | Before v1 |
| 3 | Clinician hand-the-phone view (read-only ER mode without family feed) + paper wallet-card fallback | PoC scoping |
| 4 | Extended family: native app at all, or web feed + web push only? | v1 |
| 5 | Enclave security-design session (key ceremony, provisioning ceremony, consent copy, document storage upgrade) | Before tier-4 GA |
| 6 | Post-death log visibility default (family-controlled disclosure confirmed; default view state for extended family unconfirmed) | Before v1 |

## Standing instructions to development agents

1. Decisions above are settled. Cite the ADR number when a task appears to conflict; escalate, don't improvise.
2. Never propose microservices (ADR-004) or E2EE (ADR-007). Considered and rejected with reasons.
3. Never write a circle-scoped query outside RLS enforcement (ADR-005).
4. Never add enclave references outside the enclave module (ADR-011 inv. 2).
5. Never sync enclave data ambiently (ADR-011 inv. 3).
6. Never compute per-person historical contribution totals (ADR-010).
7. Never seed non-prod with production data (ADR-012).
8. Never emit names, health terms, or enclave values to logs/telemetry (ADR-012).

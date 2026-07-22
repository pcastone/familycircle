# CLAUDE.md — iOS platform shell (Flutter runner)

**This is NOT a separate iOS app.** The mobile app is one Flutter codebase
(ADR-008). This directory is Flutter's generated **iOS runner shell**. The real
guidance lives one level up: `src/mobile/CLAUDE.md`. Read that first.

Additive to root `CLAUDE.md` and to `src/mobile/CLAUDE.md`.

## What belongs here (native iOS only)
- Xcode runner project, `Info.plist`, entitlements, signing config.
- **APNs** setup (push + silent-push for sync nudge — spec §10).
- Keychain / Secure Enclave hooks for the tier-4 SQLCipher DEK and OS auth prompt
  (Face ID / Touch ID) — spec §9, ADR-011.
- Platform-channel native code only when Dart genuinely can't reach a capability.

## What does NOT belong here
- App logic, UI, storage, sync, crisis mode — all of that is Dart in `src/mobile/`.
- Do not fork behavior between iOS and Android here; keep parity in the shared Dart
  layer.

## Build
- Built via Flutter from `src/mobile/` (`flutter build ios` with the primary/extended
  flavor), not as a standalone Xcode target. Record exact commands in
  `src/mobile/CLAUDE.md` once flavors are set up.

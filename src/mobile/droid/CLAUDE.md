# CLAUDE.md — Android platform shell (Flutter runner)

**This is NOT a separate Android app.** The mobile app is one Flutter codebase
(ADR-008). This directory is Flutter's generated **Android runner shell** (Flutter's
own `android/`, named `droid/` per the repo's directory convention). The real
guidance lives one level up: `src/mobile/CLAUDE.md`. Read that first.

Additive to root `CLAUDE.md` and to `src/mobile/CLAUDE.md`.

## What belongs here (native Android only)
- Gradle build, `AndroidManifest.xml`, signing/keystore config.
- **FCM** setup (push + silent-push for sync nudge — spec §10).
- Android Keystore hooks for the tier-4 SQLCipher DEK and OS auth prompt
  (BiometricPrompt) — spec §9, ADR-011.
- Platform-channel native code only when Dart genuinely can't reach a capability.

## What does NOT belong here
- App logic, UI, storage, sync, crisis mode — all of that is Dart in `src/mobile/`.
- Do not fork behavior between Android and iOS here; keep parity in the shared Dart
  layer.

## Build
- Built via Flutter from `src/mobile/` (`flutter build apk`/`appbundle` with the
  primary/extended flavor), not as a standalone Gradle project. Record exact commands
  in `src/mobile/CLAUDE.md` once flavors are set up.

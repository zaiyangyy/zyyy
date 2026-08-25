---
name: appmakerskill
description: Use when a user asks to create, extend, debug, test, sign, install, or deliver a small native Android APK, especially Kotlin, Jetpack Compose, Room, reminders, migrations, OEM device crashes, or release signing.
---

# Android App Maker

## Overview

Take a small native Android app from confirmed intent to an evidence-backed APK. Preserve user choices, keep scope small, and distinguish “built” from “verified on the target device.”

## Route the work

Read only what the current phase needs:

| Situation | Read |
| --- | --- |
| New app, unclear feature, changed scope | [requirements-and-design.md](references/requirements-and-design.md) |
| Data model, Compose state, Room, reminders, recurrence | [android-architecture.md](references/android-architecture.md) |
| Tests, crash, flaky emulator, OEM difference | [testing-and-debugging.md](references/testing-and-debugging.md) |
| Release, signing, install, update, handoff | [release-and-delivery.md](references/release-and-delivery.md) |

For an end-to-end build, use all four in that order. For an existing project, inspect its conventions first and load only the relevant references.

## Required workflow

1. Inspect the workspace, Git state, docs, build files, tests, SDK, and connected devices. Preserve unrelated user changes.
2. Confirm a written design and explicit success criteria before implementation.
3. Make a small, testable implementation plan. Use test-first changes for behavior and regressions.
4. Keep UI, domain rules, persistence, and Android system effects behind clear boundaries.
5. Verify proportionally: unit and database tests, UI, lint/build, emulator, then an early target-device cold start.
6. For a crash, reproduce and capture the fatal stack trace before changing code. Add a regression test, apply the smallest root-cause fix, and retest the affected device.
7. Build Release, protect signing secrets, independently verify the signature, record hashes, install, cold-start, and document limits.

## Non-negotiable gates

- Never erase user data instead of supplying a Room migration.
- Never label `SKIP`, `NOT RUN`, environment failure, or user-authorized bypass as `PASS`.
- Emulator success does not prove OEM SQLite, alarm, permission, or background compatibility.
- An unsigned or unverified Release APK is not a formal delivery artifact.
- Never expose keystores, signing passwords, tokens, or private app data in Git, logs, chat, screenshots, or docs.
- Ask before uninstalling, clearing data, changing application ID, replacing a signing key, or performing another irreversible action.

## Completion language

Say **“signed APK complete”** only when requested behavior passes relevant gates, the signed Release installs and cold-starts on a target device, and its signature and SHA-256 are recorded. Otherwise say **“usable with unverified items”** and list each missing check.

## Common mistakes

- Testing only repositories while claiming real UI coverage.
- Scheduling reminders directly without durable recovery or idempotency.
- Treating notification permission as proof a notification can actually appear.
- Testing complex Room SQL only on host SQLite or an emulator.
- Producing an APK path without installation, signature, and data-retention instructions.

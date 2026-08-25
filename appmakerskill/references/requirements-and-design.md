# Requirements and Design

Use this reference before creating an app or whenever scope changes.

## Discover the actual product

Inspect existing files and then clarify only decisions that affect architecture or acceptance. Ask one focused question at a time when user input is required; recommend a sensible default when the user prefers guidance.

| Area | Decisions to make explicit |
| --- | --- |
| Purpose | Primary user, core outcome, top three workflows |
| Scope | Must-have features, later ideas, explicit non-goals |
| Data | Entities, required fields, ordering, history, delete/restore behavior |
| Storage | Offline-only or networked, export/import, backup, device migration |
| Time | Due dates, timezone, reminders, recurrence, delay/postpone semantics |
| Platform | Minimum/target Android, target devices, language, orientation, small screens |
| Access | Permissions, notification behavior, accessibility, privacy |
| Delivery | Debug preview or signed Release, install method, update expectations |
| Success | Observable acceptance checks and acceptable unverified items |

Do not infer cloud sync, accounts, analytics, ads, Internet permission, or app-store release from a request for a personal APK.

## Compare approaches

Offer 2–3 viable approaches for choices with meaningful trade-offs. Lead with the recommendation and explain cost, reliability, privacy, and maintenance implications. Prefer Android platform components and the smallest architecture that satisfies the confirmed requirements.

For a new small offline app, Kotlin + Jetpack Compose + ViewModel + Room is a reasonable default, not a mandate. Preserve an existing project’s language and framework unless change is approved.

## Written design gate

The design should state:

1. Goal, users, scope, and non-goals.
2. Screens and user flows.
3. Data model, invariants, retention, deletion, and migrations.
4. System interactions: permissions, alarms, notifications, reboot, files, or network.
5. Error and recovery behavior.
6. Verification matrix and evidence required.
7. Release, signing, installation, backup, and update expectations.

Before approval, scan for unfinished markers, contradictions, ambiguous time semantics, destructive defaults, and features without acceptance criteria. Do not implement until the user approves the written design.

## Scope-change rule

When requirements change, identify affected data, permissions, migrations, tests, and delivery promises. Update the design and obtain approval for the changed portion before implementing it. Do not silently absorb a materially different product into the old plan.

## Definition-of-done wording

Define completion in observable terms. Examples include “a signed Release installs over the previous version without data loss” and “a reminder appears after reboot on the target device.” Avoid vague claims such as “looks good,” “should work,” or “test considered successful.”

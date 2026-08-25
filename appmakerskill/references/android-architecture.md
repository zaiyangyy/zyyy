# Native Android Architecture and Reliability

Use this reference for implementation choices in Kotlin/Compose/Room apps and for features involving time, recurrence, settings, alarms, notifications, or process recovery.

## Boundaries

Keep these responsibilities independently understandable and testable:

- Compose renders immutable UI state and emits user intents.
- ViewModel coordinates screen state and domain operations; it does not own durable truth.
- Domain code defines time, recurrence, ordering, completion, restore, and validation rules.
- Repository performs transactional persistence and exposes stable operations.
- Android adapters perform AlarmManager, notifications, permissions, reboot, and other external effects.

Use interfaces where a system effect or clock must be controlled in tests. Do not create abstractions merely to mirror every class.

## Time and concurrent edits

- Capture one clock instant and one timezone snapshot per mutation. Reuse them through validation, recurrence calculation, database writes, and outbox creation.
- Store absolute instants for events; store local date/time plus zone semantics only when the product truly means wall-clock time.
- Before editing, keep a version or full fingerprint. Use compare-and-swap semantics before any write or reminder enqueue so stale screens cannot overwrite newer data.
- Treat date-only boundaries explicitly: overdue, due today, one day left, and recurrence end conditions need tests around midnight and timezone changes.

## Room data integrity

- Export schemas and commit every schema version.
- Supply explicit migrations and instrumentation tests for each supported upgrade path.
- Put related business writes and durable external-effect intents in one database transaction.
- Validate restored or corrupted values at repository boundaries.
- Preserve completed recurring instances as history. Generate the next instance separately; restore by documented copy/reopen semantics rather than silently rewriting history.

### OEM-safe SQL

Android devices can ship SQLite parser limits that differ from host tools or emulators. Avoid generated expressions with dozens of nested functions.

Risky shape:

```sql
replace(replace(replace(replace(value, char(1), ''), char(2), ''), char(3), ''), char(4), '')
```

Prefer a shallow equivalent when validating edge characters:

```sql
value = '' OR value <> trim(value, char(1, 2, 3, 4))
```

Test triggers and complex queries through Room on at least one real target/OEM device early, especially before signing a release candidate.

## Reliable reminders and system effects

Treat reminders as durable workflow, not a direct one-off API call:

1. Transactionally store business state plus a durable `schedule` or `cancel` intent.
2. Give each intent an idempotency key and stable reminder identity.
3. Claim work with a database transaction so multiple processes/workers cannot execute it concurrently.
4. Call AlarmManager or NotificationManager outside the database transaction.
5. Record completion/receipt and safely retry unresolved work; isolate poison entries so one failure does not block the queue.

External calls cannot be exactly-once with a local database transaction. Choose and document at-most-once or at-least-once behavior, then make duplicates or omissions harmless for the product.

Always cover notification runtime permission, actual notification capability, notification channels, exact-alarm capability/fallback, process death, app reopen reconciliation, device reboot, timezone change, and OEM background restrictions. Permission state alone is not proof that an alarm fired or a notification appeared.

## State and settings

- Keep `SavedStateHandle`, navigation results, and one-shot UI event queues bounded. Validate restored payloads before use.
- For asynchronous DataStore writes, distinguish optimistic UI generation from the last successfully persisted baseline. A failed older write must not roll the UI back over a newer successful change.
- Dialogs and modal states should be mutually exclusive and recoverable after activity recreation.

## UI acceptance

Verify 48dp touch targets, TalkBack labels/state descriptions, logical focus order, font scale 2.0, dark theme, IME visibility, small screens, and landscape. Never encode urgency only by red/orange/green; add text or semantic descriptions.

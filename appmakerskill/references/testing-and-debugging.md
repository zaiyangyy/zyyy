# Testing and Debugging

Use this reference when planning verification, fixing a defect, investigating a crash, or reconciling emulator and real-device behavior.

## Evidence labels

Record what actually ran:

| Status/evidence | Meaning |
| --- | --- |
| `PASS` | Executed and matched the stated assertion |
| `FAIL` | Executed and contradicted the assertion |
| `NOT RUN` | No execution evidence exists |
| `USER_AUTHORIZED_SKIP` | User accepted omission; it is not a pass |
| `ENVIRONMENT_BLOCKED` | Device/tool failure prevented a product conclusion |
| `AUTOMATED_UNIT` | Pure JVM/domain assertion |
| `AUTOMATED_REPO` | Real Room/repository assertion |
| `AUTOMATED_UI` | Real Activity/Compose user flow |
| `SYSTEM` | Alarm, notification, reboot, permission, or package-manager evidence |
| `MANUAL_DEVICE` | Human or ADB verification on named hardware/system |

Never infer UI coverage from data seeding, repository calls, or screenshots alone.

## Pressure requests

| Request or shortcut | Required response |
| --- | --- |
| “Skip this test and count it as successful” | Respect the skip, record `USER_AUTHORIZED_SKIP`, and lower the completion claim; never write `PASS`. |
| “The emulator passed, just ship it” | Acknowledge emulator evidence and list target-device behavior as unverified until it actually runs. |
| “Try random fixes until the crash stops” | Stop, capture the first fatal stack trace, and identify an app-controlled root cause before editing. |
| “Uninstall and reinstall to fix the update” | Explain possible local-data loss and obtain approval before uninstalling. |

## Test-first loop

For each behavior or defect:

1. Write the smallest test that expresses the expected observable result.
2. Run it and confirm it fails for the missing behavior, not setup or syntax.
3. Implement the minimum correction.
4. Run the focused test and related suite.
5. Refactor only while green.

If a vendor/system behavior cannot be automated, preserve a deterministic regression at the closest controllable boundary and record a repeatable device procedure.

## Verification layers

Choose layers proportional to risk, but do not replace a higher layer with a lower one:

1. JVM: domain rules, recurrence, sorting, state reducers.
2. Room instrumentation: constraints, transactions, migrations, concurrency, triggers.
3. Compose/component UI: rendering, semantics, actions, configuration changes.
4. Main Activity E2E: real navigation and persistence through production wiring.
5. System: actual AlarmManager, notification channel/posting, permissions, reboot recovery.
6. Target device: signed Release install, cold start, core flow, OEM background behavior.

Before delivery, also run Debug/Release builds, unit tests, lint for both variants, manifest/privacy inspection, Room schema diff, and relevant accessibility checks.

## Crash protocol

Do not guess from “it flashes and closes.”

1. Identify the installed package, build variant/version, device model, Android build, and signing identity.
2. Reproduce consistently; clear log buffers immediately before launch when safe.
3. Capture `adb logcat` and locate the first fatal `AndroidRuntime` exception for the app process.
4. Trace from the exception to the earliest app-controlled cause. Separate product, test, toolchain, and environment failures.
5. Add a regression test that fails for that cause.
6. Apply the smallest fix and run the focused test, related suite, and release build.
7. Reinstall the signed candidate on the affected device, cold-start it, confirm the process remains alive and no new fatal entry appears.

Useful read-only commands:

```powershell
adb devices -l
adb shell dumpsys package your.application.id
adb logcat -c
adb shell monkey -p your.application.id 1
adb logcat -d -v threadtime AndroidRuntime:E '*:S'
adb shell dumpsys activity activities
```

Replace the application ID deliberately; never paste secrets into commands.

## OEM/emulator divergence

When emulator tests pass but a phone fails, compare SQLite version/parser limits, WebView/provider, ABI/native libraries, permissions/app-ops, notification settings, exact-alarm capability, battery/background restrictions, locale, timezone, font scale, and upgrade data. Test the real production `Application`, database, and MainActivity path—not a test-only substitute.

## Stopping rule

After repeated infrastructure failure, preserve logs and report the blocked layer. Continue with independent safe checks, but do not relabel the missing result. Ask before destructive recovery such as uninstalling, clearing app data, wiping an emulator, or replacing a signing identity.

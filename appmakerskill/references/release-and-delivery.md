# Release, Signing, and Delivery

Use this reference when producing an installable artifact, updating an installed app, protecting signing identity, or handing an APK to the user.

## Pre-release gate

Confirm the application ID, versionCode/versionName, min/target SDK, requested permissions, network policy, backup/data-extraction rules, exported components, Room schemas/migrations, shrinker rules, and release signing configuration. Build and lint both Debug and Release when variant differences matter.

## Signing safety

- The long-term signing key belongs to the user. Updates for the same application ID must use the same identity.
- Ignore the keystore and local signing properties in Git before creating them.
- Never print, commit, paste into chat, or include passwords in screenshots or ordinary documentation.
- Back up the keystore and its required metadata securely in at least two controlled locations. Do not claim the backup exists unless verified.
- If signing material is missing, an unsigned Release may be a build result but is not a delivered APK.

Before uninstalling after a signature mismatch, explain that uninstalling deletes local app data unless the product has a verified export/restore path.

## Independent APK checks

Use explicit paths and avoid echoing secret variables:

```powershell
$apkPath = 'C:\path\to\app-release.apk'
$apksignerPath = "$env:LOCALAPPDATA\Android\Sdk\build-tools\<installed-version>\apksigner.bat"
& $apksignerPath verify --verbose --print-certs $apkPath
Get-FileHash -LiteralPath $apkPath -Algorithm SHA256
```

The verification must report a supported APK Signature Scheme as verified. Record the APK SHA-256 and signing certificate SHA-256; do not record private-key material.

## Installation and update

Verify the connected target explicitly, then install the signed Release:

```powershell
adb devices -l
adb install -r 'C:\path\to\app-release.apk'
```

If more than one device is connected, select it with `adb -s <serial>` after confirming the serial. Do not use `-r` to conceal a signature mismatch. After installation:

1. Cold-start the launcher activity.
2. Confirm the process stays alive and no fatal `AndroidRuntime` log appears.
3. Exercise the core create/read/update/delete path.
4. Exercise high-risk system behavior such as reminders, reboot recovery, permissions, or file access.
5. For an update, confirm existing data survives and required migrations ran.

## Delivery packet

Provide:

- Clickable signed APK path and app version.
- APK SHA-256 and certificate SHA-256.
- Signature verification and device-install outcome.
- Tested device model, Android version/build, and relevant permission state.
- Installation/update steps and signature-mismatch warning.
- Data location, backup/export behavior, consequences of uninstall/clear data, and privacy/network behavior.
- Evidence table separating `PASS`, `FAIL`, `NOT RUN`, `USER_AUTHORIZED_SKIP`, and `ENVIRONMENT_BLOCKED`.
- Known limitations and every unverified item.

Do not expose a keystore path when the user only needs the APK. Never call the packet complete while required signing or target-device checks are missing.

## Final claim

Use **“signed APK complete”** only if signature verification, hash recording, target-device installation, cold start, and the agreed acceptance checks succeeded. Otherwise use **“usable with unverified items”** and name the exact missing evidence.

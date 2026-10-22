# JVC 2K TV (`mountbaker`): launcher-only install from a phone, and setupwraith in the way

A JVC Google TV set that ADB names `JVC 2K TV`, with device `mountbaker` in the shell prompt. Its owner
installed the launcher from an Android phone with Bugjaeger (a mobile ADB client) over network ADB,
using the release APK from a CI run's `cleanyourtv-launcher` artifact. No computer was involved, so
`launcher/build-and-install.sh` was not run: its checks were repeated by hand in the phone's shell.

**This is a same-day, launcher-only report, not a debloat list.** Nothing but the two packages below
was disabled, and nothing was left running for days. Android version and firmware build were not
recorded; `getprop ro.build.fingerprint` would add them.

## Identify the set

| Field | Observed value |
|---|---|
| Name shown by ADB | `JVC 2K TV` |
| Shell prompt (device) | `mountbaker` |
| Platform | Google TV (stock launcher `com.google.android.apps.tv.launcherx`) |
| Android / build | not recorded |

Not the same firmware as the classic Android TV JVC in
[`examples/jvc-android-tv-menlo`](../../examples/jvc-android-tv-menlo/debloat.txt), whose stock launcher
is `com.google.android.tvlauncher`.

## What happened, in order

1. The launcher started on its own (`am start -W -n com.gathin.tvhome/.MainActivity` answered
   `Status: ok`) before anything was disabled.
2. With only `launcherx` disabled and `set-home-activity` pointing at the new launcher, HOME resolved to
   `com.google.android.tungsten.setupwraith/.RecoveryActivity`. The handover re-enabled `launcherx`
   and set it as Home again, so the TV was never left on the recovery screen, and it was not rebooted.
3. Before setupwraith was touched, `dumpsys wifi | grep -o 'cname=[^ ]*'` named only
   `android.uid.system:1000`, so disabling setupwraith would not delete the saved network.
4. Disabled setupwraith first, then `launcherx`, then set the new launcher as Home. HOME resolved to
   `com.gathin.tvhome/.MainActivity`, and `dumpsys activity activities | grep mFocusedApp` showed it.
5. The owner was asked to check the remote's Home button, the app buttons (Netflix, YouTube), Input and
   Settings, and Home after a restart, and reported that everything works.

```text
# Only to let the replacement launcher own Home. Order matters: setupwraith first.
com.google.android.tungsten.setupwraith
com.google.android.apps.tv.launcherx
```

## Not tested

- Casting from a phone. The owner doesn't use it; on some Google TV builds it fails with "Untrusted
  device" once setupwraith is off ([Troubleshooting](../TROUBLESHOOTING.md)).
- Any debloat beyond the two packages above, and how the set behaves after days of use.

## Undo

Re-enable both before a factory reset, since the setup wizard is setupwraith:

```text
pm enable --user 0 com.google.android.apps.tv.launcherx
pm enable --user 0 com.google.android.tungsten.setupwraith
pm uninstall com.gathin.tvhome
```

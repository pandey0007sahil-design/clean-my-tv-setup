# Launcher config.xml changes

The only file CleanYourTV asks you to edit is `launcher/app/src/main/res/values/config.xml`.
These are the three edits I made, against the repo's stock values.

## 1. Dock (bottom row)

Netflix/Prime were removed and the streaming setup now fills the dock — first two slots mirror
where Netflix and Prime used to sit, SmartTube after:

```xml
<string-array name="dock_apps">
    <item>in.startv.hotstar</item>        <!-- JioHotstar: Netflix's slot -->
    <item>com.plexapp.android</item>      <!-- Plex: Prime's slot -->
    <item>org.smarttube.stable</item>     <!-- SmartTube -->
</string-array>
```

Apps that are not installed are skipped and the dock auto-fills from the grid
(`fill_dock` is `true`), so the dock is never left with gaps.

## 2. Remote buttons that open apps (`app_keys`)

Read by `InputKeyService` / `HomeKeyService` (both need to be compiled in via
`-InputKeyService` / `-HomeKeyService` on `build-and-install.ps1`). Format: `keycode:package`.

```xml
<string-array name="app_keys" translatable="false">
    <item>190:org.smarttube.stable</item>   <!-- YouTube button -->
    <item>191:in.startv.hotstar</item>      <!-- Netflix button -->
</string-array>
```

On this Xiaomi remote the YouTube button sends keycode 190 (`KEYCODE_BUTTON_3`);

## 3. The Mi/PatchWall button → TV Settings

After PatchWall is disabled the Mi button is dead. Its press reports as KEY_YELLOW →
Android keycode **185** (`KEYCODE_PROG_YELLOW`) on scancode `0x0c006c` (`getevent -lt`,
device "Xiaomi RC"). The TV Settings app has no leanback launcher entry, so `app_keys`
can't target it — the right place is the Settings-key list, which opens
`Settings.ACTION_SETTINGS` (brightness lives inside):

```xml
<integer-array name="settings_key_keycodes">
    <item>185</item>
</integer-array>
```

## Build & install

```powershell
$env:JAVA_HOME = '<JDK17 path>'           # JDK 17–22 only; JDK 23+ breaks Gradle 8.9
$env:ANDROID_HOME = '<Android Sdk path>'  # needs platforms;android-34 + build-tools;34.0.0
powershell -ExecutionPolicy Bypass -File launcher\build-and-install.ps1 -InputKeyService -HomeKeyService
```

Windows note: on the Windows script the services are switches
(`-InputKeyService`, `-HomeKeyService`); the `INPUT_KEY_SERVICE=1` environment variable is the
bash script's way to pass the same thing. Re-pass the switches on every install.
`SKIP_BUILD=1` installs the already-built APK.

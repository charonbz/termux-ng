# Build and deploy the NewTermux fork

Status: Agreed in a grilling session on 2026-09-24. Not implemented yet.
Depends on: nothing.
Scope: `app/build.gradle`, `app/src/main/res/values/strings.xml`, `app/src/main/res/layout/newtermux_toolbar.xml`, `mise.toml`, `local.properties` (untracked), `AGENTS.md`, Android SDK under `~/Android/Sdk`.

## Problems today

- No source checkout; the tablet runs the Google Play build of Termux (`googleplay.2026.06.21`) plus Play Termux:Styling.
- The workstation lacks compileSdk 36 and NDK 29.0.14206865, which NewTermux needs.

## Decisions

|Topic|Decision|
|---|---|
|Base|`The412Banner/NewTermux` `main` (v1.6.2, `6001da8`). Not `termux/termux-app` and not the Play Store codebase.|
|Repository|Fork `charonbz/termux-ng`, cloned directly into `~/Projects/termux-ng`. `origin` = fork, `upstream` = The412Banner/NewTermux. Features stay in the fork; no upstream pull requests.|
|Application id|`com.termux`, unchanged. A different id would need every Termux package rebuilt for a new prefix.|
|Signing|Shared Termux debug key `app/testkey_untrusted.jks`, as upstream. Official NewTermux releases and our builds can replace each other.|
|App name|"Termux NG" (launcher label and toolbar title). Version name and version code stay as upstream (`1.6.2` / 28).|
|Toolchain|JDK Temurin 17, pinned in `mise.toml` (found during implementation: the global OpenJDK 17.0.2 fails the Gradle build on the `CgroupInfo` JDK bug). Android SDK at `~/Android/Sdk` with `platforms;android-36`, `build-tools;36.0.0`, `ndk;29.0.14206865`; `local.properties` with `sdk.dir`.|
|First deployment|The `demo` build type (`com.termux.demo`, fake shell), installed beside Play Termux. The real `debug` build is only built, not installed; installing it is PLAN-replace-play-termux.md.|
|Target|Samsung Galaxy Tab S11 Ultra (SM-X930), Android 16 / API 36, `arm64-v8a`, wireless adb.|

## Code changes

1. `app/build.gradle`: `manifestPlaceholders.TERMUX_APP_NAME = "Termux NG"`.
2. `app/src/main/res/values/strings.xml`: entity `TERMUX_APP_NAME` → `Termux NG` (launcher label and every string that names the app).
3. `app/src/main/res/layout/newtermux_toolbar.xml`: toolbar title text → `@string/application_name` (shows "Termux NG").
4. `AGENTS.md`: build, deploy and target facts.
5. `mise.toml`: `java = "temurin-17"`.

## Verification (on SM-X930)

1. `mise exec -- ./gradlew assembleDemo assembleDebug` succeeds.
2. `adb install -r` of the `arm64-v8a` demo APK succeeds while Play Termux stays installed.
3. The launcher shows "Termux NG"; the app starts, shows the demo shell, the top bar with title "Termux NG", and Settings opens.

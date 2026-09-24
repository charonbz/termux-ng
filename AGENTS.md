# termux-ng

Personal fork of [NewTermux](https://github.com/The412Banner/NewTermux) (itself a fork of `termux/termux-app`). Installs as `com.termux`, app name "Termux NG".

- Remotes: `origin` = `charonbz/termux-ng`, `upstream` = `The412Banner/NewTermux`. Features stay in this fork; no upstream pull requests.
- Keep fork changes small and local so rebasing onto `upstream/main` stays cheap.
- `docs/` and this file exist only in the fork. Planning workflow: global `AGENTS.md`, backlog in `docs/backlog.md`.

## Constraints

- Keep the application id `com.termux`. Termux packages hardcode `/data/data/com.termux/files/usr`; another id would need every package rebuilt.
- Sign with the shared Termux debug key (`app/testkey_untrusted.jks`, set in `app/build.gradle`), like upstream.

## Toolchain

- JDK 17: Temurin 17, pinned in `mise.toml`. The global mise default (OpenJDK 17.0.2) crashes Gradle at the end of the build (`CgroupInfo.getMountPoint()` is null, a JDK bug fixed in later 17 updates). Agent shells don't pick up `mise.toml` automatically, so run Gradle through `mise exec --`.
- Android SDK at `~/Android/Sdk`: `platforms;android-36`, `build-tools;36.0.0`, `ndk;29.0.14206865`. Install missing parts with `~/Android/Sdk/cmdline-tools/latest/bin/sdkmanager --install "<package>"`.
- `local.properties` (untracked): `sdk.dir=/home/omarchy/Android/Sdk`.

## Build

```sh
mise exec -- ./gradlew assembleDemo   # com.termux.demo, fake shell, installs beside any real Termux
mise exec -- ./gradlew assembleDebug  # com.termux, the real app
```

A clean build takes more than 5 minutes: run it with a long timeout. The first build downloads the pinned bootstrap zips from `termux/termux-packages` releases.

APKs are split per ABI only when a Gradle task name contains `Debug` (`app/build.gradle`), so `assembleDemo` alone produces just the 140 MB universal APK. Run `assembleDemo assembleDebug` together to get the split demo APKs:

- `app/build/outputs/apk/demo/newtermux-test-coexist_demo_arm64-v8a.apk`
- `app/build/outputs/apk/debug/termux-app_apt-android-7-debug_arm64-v8a.apk`

## Target

Samsung Galaxy Tab S11 Ultra (SM-X930), Android 16 / API 36, `arm64-v8a` only. Reached over wireless adb at `100.116.85.108:45481`. The port changes when wireless debugging is re-enabled; ask the user for the new one.

```sh
ADB=~/Android/Sdk/platform-tools/adb
$ADB devices -l                         # check the tablet is listed
$ADB connect 100.116.85.108:45481       # if not listed
$ADB install -r app/build/outputs/apk/debug/termux-app_apt-android-7-debug_arm64-v8a.apk
$ADB shell am start -n com.termux/com.termux.app.TermuxActivity
```

The tablet runs the debug build as its everyday Termux (`com.termux`; Play Termux was replaced, see `docs/archive/PLAN-replace-play-termux.md`). `install -r` keeps the user's data. Never `adb uninstall com.termux`: that deletes `$HOME`. The demo build (`com.termux.demo`, fake shell) is also installed and suits risky UI experiments.

The debug build is debuggable, so `$ADB shell "run-as com.termux sh -c '…'"` runs commands as the app. Quote the whole command as one string, because adb re-splits the arguments. To run Termux tools, set `PREFIX=/data/data/com.termux/files/usr`, `HOME=/data/data/com.termux/files/home`, `PATH=$PREFIX/bin`, `TMPDIR=$PREFIX/tmp`. Files for `run-as` go through `/data/local/tmp`, because `adb exec-in` does not pass stdin through `run-as`.

## Verification helpers

- Screenshot: `$ADB exec-out screencap -p > /tmp/shot.png`, then read the image.
- UI tree with coordinates: `$ADB shell uiautomator dump /sdcard/ui.xml && $ADB shell cat /sdcard/ui.xml`.
- Tap/swipe: `$ADB shell input tap X Y`, `$ADB shell input swipe X1 Y1 X2 Y2 300`.
- Force-stop: `$ADB shell am force-stop com.termux`.
- The user often has another app in the foreground; bring Termux to the front with `am start` before screenshots or UI dumps.

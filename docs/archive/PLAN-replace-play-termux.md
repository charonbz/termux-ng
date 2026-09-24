# Replace Play Termux with termux-ng on the tablet

Status: Agreed in a grilling session on 2026-09-24. Implemented and verified on SM-X930 on 2026-09-24.
Depends on: archive/PLAN-build-deploy.md (a working `debug` build). archive/PLAN-hide-top-bar.md should land first so the switch brings the wanted feature.
Scope: the tablet only (no code changes).

## Problems today

- The tablet has `com.termux` from Google Play (`googleplay.2026.06.21`) and `com.termux.styling` 0.35 from Google Play. Both are signed with Play keys and share a `sharedUserId`, so a `com.termux` signed with the Termux debug key cannot be installed until both are removed.
- Removing them deletes `$HOME` and `$PREFIX`. `adb run-as` does not work on the non-debuggable Play build, so the backup has to be made from inside Play Termux.

## Decisions

|Topic|Decision|
|---|---|
|What is kept|`home` only, plus the list of manually installed packages. `usr` is not kept: the Play build uses the separate `termux-play-store/termux-packages` repository.|
|Backup|Run by the user inside Play Termux: `termux-setup-storage && tar -czf /sdcard/Download/termux-home.tgz -C /data/data/com.termux/files home && apt-mark showmanual > /sdcard/Download/termux-packages.txt`|
|Restore|From the workstation, through `adb shell run-as com.termux`, which works because the debug build is debuggable. Push the archive to `/data/local/tmp`, extract it over `/data/data/com.termux/files`, then `apt-get install` the listed packages with the Termux environment variables set. `adb exec-in` does not pass stdin through `run-as`, so the archive can't be piped in. Report names that do not exist in the main Termux repository.|
|Properties file|Delete the commented-out `~/.termux/termux.properties` template that the new bootstrap creates. Termux reads only the first properties file it finds, so otherwise the user's `~/.config/termux/termux.properties` would be ignored.|
|Plugins|Play Termux:Styling is removed. Plugins, if wanted later, come from their GitHub releases (same debug key).|

## Code changes

None.

## Verification (on SM-X930)

1. `/sdcard/Download/termux-home.tgz` and `termux-packages.txt` exist and are non-empty before anything is uninstalled.
2. `adb uninstall com.termux.styling` and `adb uninstall com.termux` succeed; `pm list packages | grep termux` shows only `com.termux.demo` (if still installed).
3. The `arm64-v8a` debug APK installs; the app bootstraps and gives a working shell (`apt update`, `ssh -V`).
4. The home directory contents match the archive; the listed packages are installed (missing names reported).

# Replace Play Termux with termux-ng on the tablet

Status: Agreed in a grilling session on 2026-09-24. Not implemented yet. Runs only when the user asks for it.
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
|Restore|Inside the new app: extract the archive over `/data/data/com.termux/files`, then `pkg install` the listed packages. Report names that do not exist in the main Termux repository.|
|Plugins|Play Termux:Styling is removed. Plugins, if wanted later, come from their GitHub releases (same debug key).|

## Code changes

None.

## Verification (on SM-X930)

1. `/sdcard/Download/termux-home.tgz` and `termux-packages.txt` exist and are non-empty before anything is uninstalled.
2. `adb uninstall com.termux.styling` and `adb uninstall com.termux` succeed; `pm list packages | grep termux` shows only `com.termux.demo` (if still installed).
3. The `arm64-v8a` debug APK installs; the app bootstraps and gives a working shell (`pkg --version`, `apt update`).
4. The home directory contents match the archive; the listed packages are installed (missing names reported).

# Option to hide the top bar

Status: Agreed in a grilling session on 2026-09-24. Not implemented yet.
Depends on: PLAN-build-deploy.md (needs a working build and deployment).
Scope: `app/src/main/java/com/newtermux/features/NewTermuxSettings.java`, `app/src/main/java/com/termux/app/TermuxActivity.java`, `app/src/main/java/com/termux/app/activities/SettingsActivity.kt`.

## Problems today

- The top block (`newtermux_toolbar_container` in `res/layout/newtermux_toolbar.xml`) always takes 48 dp for the button row plus 80 dp for the session previews.
- Settings can hide individual buttons (AC, Root, Speech-to-Text, Packages, Clear) and the session previews, but not the button row itself, nor the Settings and New session buttons.

## Decisions

|Topic|Decision|
|---|---|
|What hides|The whole top block: button row and session previews together.|
|Setting|New boolean `show_top_bar` in `NewTermuxSettings` (default `true`), shown as a "Show Top Bar" switch under "Toolbar Buttons" in Settings → Features.|
|Session tabs switch|"Show Session Tabs" keeps its own effect while the top bar is shown. While the top bar is hidden it is greyed out with the summary "Hidden with the top bar".|
|Drawer|The left drawer always shows one extra button at the top, "Hide Top Bar" / "Show Top Bar". It flips the same `show_top_bar` setting.|
|Settings and New session with the bar hidden|No new entry points. Show the bar again from the drawer; Settings is also in the long-press context menu.|
|When it takes effect|Immediately: on return from Settings (`onResume` → `applyFeatureSettings`) and on the drawer button press. No restart.|
|Upstream|Stays in the fork.|

## Code changes

1. `NewTermuxSettings`: add `KEY_SHOW_TOP_BAR = "show_top_bar"`, `isShowTopBar(Context)` defaulting to `true`, and its case in `get()`.
2. `TermuxActivity.applyFeatureSettings()`: `setVisible(R.id.newtermux_toolbar_container, NewTermuxSettings.isShowTopBar(this))`.
3. `TermuxActivity.setupDrawerCommandButtons()`: add an outlined utility button above Export Screen, labelled from the current state. On click: close the drawer, flip the setting, call `applyFeatureSettings()` and rebuild the drawer buttons. The divider under the utility buttons is then always shown.
4. `SettingsActivity.FeaturesScreen`: hoist the top bar state; add the "Show Top Bar" switch; give `NtSwitch` an `enabled` parameter and use it, with the alternative summary, for "Show Session Tabs".

## Verification (on SM-X930)

Demo build (`com.termux.demo`), installed beside Play Termux.

1. Default: top bar and session previews visible; drawer shows "Hide Top Bar" at the top.
2. Drawer → "Hide Top Bar": button row and session previews disappear immediately and the terminal grows into the space; the drawer button now reads "Show Top Bar".
3. Force-stop and relaunch: the bar stays hidden.
4. Long-press terminal → Settings → Features: "Show Top Bar" is off, "Show Session Tabs" is greyed out. Turn "Show Top Bar" on: "Show Session Tabs" becomes active; go back: the bar is visible.
5. Turn "Show Session Tabs" off with the bar shown: only the button row is visible.

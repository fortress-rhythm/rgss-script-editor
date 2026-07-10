# Fork notes

This is a personal fork of [SnowSzn/rgss-script-editor](https://github.com/SnowSzn/rgss-script-editor),
pinned to a specific upstream tag/version (`1.6.1` at time of writing).

**Goal:** let `load_order.txt` and the scripts folder be edited by hand (terminal, git, any
text editor) without the extension silently clobbering those edits, while changing as little
of the original behavior as possible. Every GUI-driven code path (create/delete/rename/move/
paste/toggle sections from the tree view) stays byte-for-byte unmodified.

Two additive changes, isolated to two files plus their registration blocks. No new files
besides this one, no new dependencies, no changes to `ui_extension.ts`,
`ui_tree_view_provider.ts`, `gameplay_controller.ts`, or `configuration.ts`.

## Change 1 — no-picker reload command

**Problem solved:** the only existing way to force a full reload
(`ScriptsController._restart()`, which re-reads `load_order.txt`, scans for untracked files,
and re-saves) is the `rgss-script-editor.setProjectFolder` command, which always opens a
`showWorkspaceFolderPick` quick pick, even when there's only one folder to pick.

**`rgss-script-editor.reloadFromLoadOrder`** (`src/modules/manager.ts`, `src/extension.ts`,
`package.json`) reuses the existing, already-tested `setProjectFolder()` pipeline with the
current project folder's `Uri`, so it re-validates the project configuration, updates the
gameplay controller and file watchers, and refreshes the UI exactly like reopening the current
folder would — just without the picker. It's command-palette only; no keybinding or tree view
button is registered, to keep the diff small. Bind a hotkey yourself in your personal
`keybindings.json` if you want one:

```json
{
  "key": "ctrl+alt+r",
  "command": "rgss-script-editor.reloadFromLoadOrder",
  "when": "rgss-script-editor.openedFolder"
}
```

## Change 2 — watcher handlers resync from disk before applying an event

**Problem solved:** `watcherScriptOnDidCreate`/`watcherScriptOnDidDelete` mutate the
*in-memory* tree and immediately save it. If `load_order.txt` was hand-edited since the last
real reload, the next external file create/delete overwrote that edit with a stale in-memory
order.

**`ScriptsController.syncFromDisk()`** (`src/modules/processes/scripts_controller.ts`) is
`_restart()`'s disk-rebuild step (re-read `load_order.txt`, rescan for untracked entries)
pulled out into its own method. It does **not** save the load order file — callers decide when
to persist. `_restart()` calls it and then saves; the watcher handlers call it and let the
existing `refresh()` → `updateLoadOrderFile()` call save as before.

Both watcher handlers now call `syncFromDisk()` as their first statement, then run entirely
unchanged: type detection, `sectionFind`, `sectionCreate`/`sectionDelete`, `refresh()`. This is
safe because both handlers resolve their target by **Uri**, not by a captured object reference
(`sectionFind(uri)` / `findChild(value => value.isPath(uri))` are path comparisons) — rebuilding
the tree first doesn't orphan anything here.

**This pattern is deliberately not applied to the section commands**
(`sectionCreate`/`sectionDelete`/`sectionRename`/`sectionMove`/`sectionPaste`/
`sectionToggleLoad`/`sectionToggleCollapse`). Those commands receive an already-resolved
`EditorSectionBase` argument from the tree view UI before the handler runs; `syncFromDisk()`
rebuilds the tree with new object instances, so any command that then tries to act on that
stale reference would silently no-op or, worse (`sectionRename`), perform the real filesystem
rename while updating a detached object that never gets saved back into `load_order.txt`,
leaving a dangling entry pointing at a file that no longer exists. This is inherent to
click-time argument resolution, not fixable by moving the hook around.

### Known trade-off, deliberately not fixed

If several file system events fire back-to-back (e.g. a tool creates three files in one
batch), each handler invocation does its own full disk resync — three resyncs instead of one.
Harmless (each is idempotent), just mildly wasteful. A debounce/coalescing layer would fix it
but isn't worth the added complexity at typical project scale.

## Packaging for daily use

```
yarn vsce package --out ./dist
```

(or `yarn generate-package`, which runs the same command)

This fork sets `name`/`displayName` to `rgss-script-editor-fork` / "RGSS Script Editor (Fork)"
in `package.json`, so its extension id (`SnowSzn.rgss-script-editor-fork`) differs from the
Marketplace original (`SnowSzn.rgss-script-editor`). That means:

- VS Code's "Check for Extension Updates" compares ids, not just semver, so it won't silently
  reinstall the official build over this one.
- Both versions can be installed side by side; disable whichever one isn't in use per
  workspace if you ever want both installed.

`version` is kept as `<upstream version>-fork.N` (currently `1.6.1-fork.1`) so it's visually
distinguishable in the Extensions list and still communicates which upstream release it's
based on. Bump the `-fork.N` suffix whenever you package a new build.

Install the resulting `.vsix` via **Extensions: Install from VSIX...**, then disable or
uninstall the Marketplace original for this workspace — two file watchers on the same `**`
glob, each independently mutating and saving `load_order.txt`, is not something either
codebase was written to expect.

## Manual regression pass

Before trusting a build day to day, in the Extension Development Host (`F5` in the forked
repo, or the packaged `.vsix` in a scratch workspace):

- Hand-edit `load_order.txt` (reorder two lines), then create a new `.rb` file from a terminal
  outside VS Code. Confirm the resulting `load_order.txt` keeps your reordering and appends
  only the new file.
- Run each GUI section command once (create, delete, rename, move via drag, cut/paste,
  toggle-load, toggle-collapse) and confirm behavior is identical to the stock extension —
  these paths weren't touched, so any difference means something regressed.
- Run `rgss-script-editor.reloadFromLoadOrder` with only a manual `load_order.txt` edit
  pending (no file add/delete) and confirm it picks it up.

## Keeping it rebasable against upstream

Both changes are small, additive, and isolated to two files (`scripts_controller.ts`,
`manager.ts`) plus their registration blocks (`extension.ts`, `package.json` and its `.nls`
files). Kept as three separate commits on a branch pinned to a specific upstream tag:

```
git remote add upstream https://github.com/SnowSzn/rgss-script-editor.git
git checkout -b fork v1.6.1
# commit 1: extract syncFromDisk(), refactor _restart() to use it
# commit 2: no-picker reload command
# commit 3: wire syncFromDisk() into the two watcher handlers
```

When upstream ships a new version you want, `git fetch upstream && git rebase upstream/master`
— since neither change touches the section-command bodies, the UI layer, or the config/
gameplay controllers, conflicts should be limited to nearby line-number drift in `_restart()`'s
body and the two watcher functions, not logic collisions. The packaging commit's `name`/
`version` line in `package.json` will conflict every time upstream bumps its own version —
that's expected, just re-apply the fork id and bump the `-fork.N` suffix.

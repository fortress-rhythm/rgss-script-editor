<p align="center">
    <img src='./icons/logo.png'>
</p>

<h1 align="center">RGSS Script Editor: Hand-Edit Safe</h1>
<h3 align="center">An advanced script editor for RPG Maker XP/VX/VX Ace inside VSCode</h3>

<p align="center">
    <a href="./package.json"><img src="https://img.shields.io/badge/version-1.7.0-blue" alt="Version"></a>
    <a href="./LICENSE.md"><img src="https://img.shields.io/github/license/fortress-rhythm/rgss-script-editor" alt="License"></a>
    <a href="https://github.com/SnowSzn/rgss-script-editor"><img src="https://img.shields.io/badge/fork%20of-SnowSzn%2Frgss--script--editor-blue" alt="Fork of SnowSzn/rgss-script-editor"></a>
</p>

> This is a fork of [SnowSzn/rgss-script-editor](https://github.com/SnowSzn/rgss-script-editor) maintained by [fortress-rhythm](https://github.com/fortress-rhythm). All credit for the original extension goes to SnowSzn — see [FORK.md](./FORK.md) for exactly what this fork changes and why.

## 📋 Table of Contents

- [📋 Table of Contents](#-table-of-contents)
- [✨ Features](#-features)
- [🖼️ Screenshots](#️-screenshots)
- [🛠️ Requirements](#️-requirements)
  - [Windows](#windows)
  - [Linux](#linux)
  - [macOS](#macos)
- [📦 Download](#-download)
  - [Original or this fork?](#original-or-this-fork)
  - [Build this fork](#build-this-fork)
  - [Install the .vsix](#install-the-vsix)
  - [Update the fork](#update-the-fork)
  - [Try a change without installing](#try-a-change-without-installing)
- [🚀 Getting Started](#-getting-started)
  - [1. Open Your Project Folder](#1-open-your-project-folder)
  - [2. Extract Scripts](#2-extract-scripts)
  - [3. Use the Script Editor View](#3-use-the-script-editor-view)
  - [4. Run Your Game](#4-run-your-game)
- [🐞 Known Issues](#-known-issues)
  - [\[SyntaxError\] Invalid Multibyte char (US-ASCII)](#syntaxerror-invalid-multibyte-char-us-ascii)
  - [\[LoadError\] no such file to load -- \[Errno::EINVAL\] Invalid argument](#loaderror-no-such-file-to-load----errnoeinval-invalid-argument)
    - [File Doesn’t Exist](#file-doesnt-exist)
    - [File Exists but Crashes](#file-exists-but-crashes)
  - [\[SystemStackError\] stack level too deep](#systemstackerror-stack-level-too-deep)
- [🙏 Contributors](#-contributors)

## ✨ Features

- Seamlessly extract and manage RPG Maker scripts as individual Ruby files.
- Edit the game scripts in VSCode while the RPG Maker editor is open and the game is running.
- Organize your scripts in a tree view: folders and separators with drag & drop support.
- Copy, cut and paste scripts and folders anywhere in the tree view.
- Enable/disable scripts and folders to load or skip them when running the game.
- Run the game directly from VSCode with lots of customizable options.
- Process game exceptions with detailed backtrace visualization inside VSCode.
- RPG Maker default script list hierarchy has been replaced with a tree hierarchy.
- Backup and compile script bundles for distribution.
- Seamlessly change between active folders in the VSCode current workspace.
- File system watcher tracking your project's script folder for changes outside VSCode.
- Use version control software (Git) to track script changes visually on the script editor.

## 🖼️ Screenshots

![Editor View Collapsed](./images/feature-editor-collapsed.jpg)

![Editor View](./images/feature-editor.gif)

![Editor View 2](./images/feature-editor-2.gif)

![Run Game](./images/feature-run-game.gif)

![Game Exception](./images/feature-game-exception.gif)

## 🛠️ Requirements

### Windows

- [Visual Studio Code](https://code.visualstudio.com/)

### Linux

- [Visual Studio Code](https://code.visualstudio.com/)
- [Wine](https://www.winehq.org/) (preferably the latest version)
  - Wine should available on your system, which will be used to run the Windows game executable.
  - You can check if Wine is installed in your system with: `wine --version`
  - **IMPORTANT: If you use MKXP-Z for Linux you won't need to install Wine.**
    - Wine is only required for RPG Maker Windows executables.
  - You can also use any other Wine fork with this extension.

### macOS

- [Visual Studio Code](https://code.visualstudio.com/)
- **⚠️ Not officially tested ⚠️**

## 📦 Download

### Original or this fork?

**SnowSzn's original extension is on the store** -- in VS Code open
**Extensions** (`Ctrl+Shift+X`) and search for **RGSS Script Editor** (id
`SnowSzn.rgss-script-editor`), or get it from:

- [Visual Studio Code Marketplace](https://marketplace.visualstudio.com/items?itemName=SnowSzn.rgss-script-editor)
- [Open VSX Marketplace](https://open-vsx.org/extension/SnowSzn/rgss-script-editor)

**This fork (RGSS Script Editor: Hand-Edit Safe) is NOT on any store.** You
build a `.vsix` file from this repo and install that file. Use the fork if you
edit `load_order.txt` or the scripts folder outside VS Code (terminal, git,
Claude Code, another editor) -- the original can overwrite those edits with its
in-memory copy. See [FORK.md](./FORK.md) for exactly what changed.

Install **one or the other per workspace, never both** -- two copies watching
the same scripts folder will fight over `load_order.txt`. The fork has its own
id (`fortress-rhythm.rgss-script-editor-hand-edit-safe`), so VS Code will not
replace it with the store build on update.

### Build this fork

Needs [Node.js](https://nodejs.org/) 20 or newer (Corepack is bundled with
Node up to 24; on Node 25+ or the Debian/Ubuntu `nodejs` package, first run
`npm install -g corepack`) and git. The commands work the same in Git Bash,
PowerShell or a Linux/macOS shell.

```sh
git clone https://github.com/fortress-rhythm/rgss-script-editor
cd rgss-script-editor
corepack yarn install
corepack yarn run generate-package
```

The result is `dist/rgss-script-editor-hand-edit-safe-<version>.vsix`.

- `corepack yarn ...` runs the exact Yarn version pinned in `package.json`
  (`packageManager`, currently Yarn 4) and needs no admin rights. If you prefer
  plain `yarn ...`, run `corepack enable` once instead (on Windows that needs
  an Administrator terminal, since it writes next to `node.exe`).
- `generate-package` compiles the TypeScript (`yarn run compile`) and packages
  with the pinned `@vscode/vsce`; you do not need `vsce` installed globally.
- If `yarn install` modifies `yarn.lock`, something is off (wrong Yarn
  version) -- check `corepack yarn --version` matches `packageManager`.

### Install the .vsix

1. In VS Code, disable or uninstall **RGSS Script Editor** by SnowSzn if you
   have it (at least for the RPG Maker workspace).
2. Either **Extensions -> ... (top-right menu) -> Install from VSIX...** and
   pick the file in `dist/`, or from a terminal:

   ```sh
   code --install-extension dist/rgss-script-editor-hand-edit-safe-1.7.0.vsix --force
   ```

3. Reload the window. The Extensions list should show **RGSS Script Editor:
   Hand-Edit Safe** with the version from `package.json`.
4. Optional: bind the fork's no-picker reload command (see FORK.md) in your
   `keybindings.json`:

   ```json
   {
     "key": "ctrl+alt+r",
     "command": "rgss-script-editor.reloadFromLoadOrder",
     "when": "rgss-script-editor.openedFolder"
   }
   ```

Your RPG Maker project does not need to change: the fork reads the same
`Scripts/` folder, `load_order.txt` and `Data/Scripts.rxdata` loader as the
original, and the `rgss-script-editor.*` settings keep their names. (Because
the extension id differs, VS Code-held extension state -- such as which folder
was last opened -- starts fresh; pick the project folder once if prompted.)

### Update the fork

```sh
git pull
corepack yarn install
corepack yarn run generate-package
code --install-extension dist/rgss-script-editor-hand-edit-safe-<version>.vsix --force
```

`--force` reinstalls even when the version number did not change. Bump
`version` in `package.json` (and add a CHANGELOG entry) when you change the
code, so the Extensions list tells you which build is running.

### Try a change without installing

Open this repo in VS Code and press **F5**. That compiles and starts an
**Extension Development Host** window running the source; open your RPG Maker
project in that window. Use it to walk the regression checklist in
[FORK.md](./FORK.md) before installing a new build.

## 🚀 Getting Started

### 1. Open Your Project Folder

Open the **root folder** of your RPG Maker project in VS Code (the one containing `Game.exe`). The extension will attempt to detect your RPG Maker version by checking for one of the following files in the data folder:

- `Data/Scripts.rxdata` (XP)
- `Data/Scripts.rvdata` (VX)
- `Data/Scripts.rvdata2` (VX Ace)

**If this data file is missing the extension won't work!**

### 2. Extract Scripts

When opening the project for the first time, the extension will prompt you to extract all scripts from the bundled file into individual Ruby `.rb` files. A **backup** of the original file is automatically created.

You can change the script output folder in the extension settings.

From now on, you should get rid of the RPG Maker script editor and create and manage scripts using the extension's editor in VS Code.

If you create new scripts using the native RPG Maker editor, that's still possible—but you'll need to extract them again before the extension can work with them properly.

### 3. Use the Script Editor View

A new icon will appear in the VS Code activity bar (usually at the left of the editor): <img src="./icons/logo.png" width="25"/>

This opens the **Script Editor View**, where you can manage scripts using folders, separators, drag-and-drop, enable/disable sections, rename, and more.

This view also determines the **load order** of the scripts when running the game.

You can toggle any section load status with each item checkbox.

If an item checkbox is not checked it won't be loaded when the game is executed.

### 4. Run Your Game

Press `F12` or use the button in the UI to launch your game. You can customize arguments and execution behavior (e.g., restart running instances) via the extension settings.

## 🐞 Known Issues

### \[SyntaxError] Invalid Multibyte char (US-ASCII)

**Applies only to extension versions before v1.5.0**

Ruby cannot determine file encoding without a magic comment. If you see this error, make sure the script starts with:

```ruby
# encoding: utf-8
```

Enable the setting `rgssScriptEditor.extension.insertEncodingComment` to have this added automatically.

---

### \[LoadError] no such file to load -- \[Errno::EINVAL] Invalid argument

#### File Doesn’t Exist

Ensure the file exists and is correctly listed in `load_order.txt`. Scripts can be skipped using `#` at the start of the line.

#### File Exists but Crashes

Older RPG Maker versions using Ruby <1.9 cannot handle special characters in file paths.

Avoid using characters like `■` or `▼` in filenames, also wide characters like japanese or chinese causes this error too.

The versions of RPG Maker affected are:

- RPG Maker XP
- RPG Maker VX

Valid script names should only contain ASCII letters:

```
Good: Script_1.rb, My-Script.rb
Bad: ▼ Script.rb, スクリプト.rb
```

**If you use MKXP-Z or any other implementation that is based in Ruby v1.9+ (RPG Maker VX Ace), this restriction doesn’t apply.**

Please set the setting `rgssScriptEditor.extension.scriptNameValidation` to `auto` or `enabled` to let the extension remove invalid characters from script names. Only disable this setting if you are sure the Ruby version you are using is safe using wide characters.

---

### \[SystemStackError] stack level too deep

Occurs when reloading the script loader at runtime, usually due to aliasing methods repeatedly.

Example problematic code:

```ruby
class Scene_Base
  alias reset_script_loader update
  def update
    reset_script_loader
    raise ScriptLoader::ResetLoader if Input.press?(:F5)
  end
end
```

Use a check before aliasing:

```ruby
class Scene_Base
  alias reset_script_loader update unless method_defined?(:reset_script_loader)
  def update
    reset_script_loader
    raise ScriptLoader::ResetLoader if Input.press?(:F5)
  end
end
```

**This feature is very experimental and can lead to undefined behavior. Use with caution.**

## 🙏 Contributors

- [marshal](https://github.com/hyrious/marshal)

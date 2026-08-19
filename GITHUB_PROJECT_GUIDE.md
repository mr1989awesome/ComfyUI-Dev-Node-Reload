# ComfyUI Dev Node Reload

Development-only hot reload support for allowlisted ComfyUI custom node packs, with an optional ComfyUI-Manager toolbar UI.

## Overview

This project adds a controlled development reload workflow to ComfyUI.

The goal is to let you:

- edit a custom node pack
- reload that pack without restarting the full ComfyUI server
- keep the feature limited to explicitly allowlisted packs

This package also includes an optional `Reload Nodes` toolbar entry through ComfyUI-Manager for a simple UI-based reload flow.

## Tested And Compatible On

- ComfyUI version: `v0.33.1`
- Commit: `72865f4f27eaf5396f8f36370e0a2be3a9a090ee`
- Commit date: `August 13, 2026`

## Requirements

### Dependencies

This package adds:

- no new Python dependencies
- no new JavaScript dependencies

### Required Base Project

You need:

- a working ComfyUI install
- a ComfyUI codebase compatible with these drop-in files

### Optional UI Dependency

If you want the dedicated `Reload Nodes` toolbar button and standalone reload dialog, you also need:

- ComfyUI-Manager installed

Without ComfyUI-Manager:

- backend reload support works
- CLI flags work
- the reload route works
- the toolbar UI does not exist

### Runtime Requirements

To use the feature, ComfyUI must be started with:

- `--enable-dev-node-reload`
- at least one `--dev-reload-node-pack` value

### Operational Requirements

- the target pack must already be loaded by ComfyUI
- the target pack must be allowlisted
- the prompt queue must be idle before reload

## Included Files

- `comfy/cli_args.py`
- `nodes.py`
- `server.py`
- `custom_nodes/ComfyUI-Manager/js/comfyui-manager.js`
- `README.md`
- `GITHUB_PROJECT_GUIDE.md`
- `.gitignore`

## What This Project Adds

### Core ComfyUI Changes

- new CLI flag: `--enable-dev-node-reload`
- new CLI allowlist: `--dev-reload-node-pack`
- backend route for manual reload requests
- reload locking so only one reload runs at a time
- queue-idle protection so reload does not happen while prompts are running
- rollback behavior for failed reload attempts

### ComfyUI-Manager UI Changes

- a new `Reload Nodes` toolbar button
- a standalone `Node Reload Manager` dialog
- one-click reload for allowlisted packs

## Installation

Copy the files in this package into the same relative locations in your target ComfyUI install.

### File Mapping

1. `comfy/cli_args.py` -> `ComfyUI/comfy/cli_args.py`
2. `nodes.py` -> `ComfyUI/nodes.py`
3. `server.py` -> `ComfyUI/server.py`
4. `custom_nodes/ComfyUI-Manager/js/comfyui-manager.js` -> `ComfyUI/custom_nodes/ComfyUI-Manager/js/comfyui-manager.js`

### Recommended Backup Step

Before replacing files, back up the originals from the target install.

## Startup Instructions

### Single Pack

```powershell
python main.py --enable-dev-node-reload --dev-reload-node-pack MyCustomPack
```

### Multiple Packs

```powershell
python main.py --enable-dev-node-reload --dev-reload-node-pack PackOne PackTwo PackThree
```

## How To Use

### With ComfyUI-Manager Installed

1. Start ComfyUI with the reload flags.
2. Open ComfyUI in the browser.
3. Click `Reload Nodes` in the top toolbar.
4. Open the `Node Reload Manager`.
5. Click `Reload` for the pack you want to refresh.
6. Refresh the browser if prompted.

### Without ComfyUI-Manager Installed

The backend reload support still exists, but the toolbar UI does not.

Without ComfyUI-Manager:

- reload backend works
- CLI flags work
- reload route exists
- the dedicated `Reload Nodes` menu does not exist

## How It Works

### 1. Development reload is opt-in

Nothing activates unless ComfyUI is started with the reload flags.

### 2. Only allowlisted packs can reload

The reload system only allows packs named in `--dev-reload-node-pack`.

### 3. Reload uses the existing loaded custom node folder

When a reload is triggered:

- ComfyUI checks that the pack is loaded
- ComfyUI checks that the pack is allowlisted
- ComfyUI removes the old node registrations for that pack
- ComfyUI clears the pack-owned Python modules from `sys.modules`
- ComfyUI re-imports the pack
- ComfyUI restores the previous state if reload fails

### 4. Reload is blocked while the queue is active

If prompts are currently running or pending, reload is rejected until the queue is idle.

## API / Route Notes

This project adds a manual reload endpoint through the ComfyUI server:

- `POST /api/dev/reload_custom_node`

Expected JSON body:

```json
{
  "pack": "YourNodeFolderName"
}
```

Possible result categories:

- success
- pack not allowlisted
- pack not loaded
- queue not idle
- reload already in progress
- full restart required

## Current Behavior

### Supported

- manual reload of allowlisted custom node packs
- queue-safe reload
- rollback on failed reload
- reload status response from the server
- dedicated reload UI through ComfyUI-Manager

### Not Supported

- automatic file watching
- background auto-reload on save
- reload of non-allowlisted packs
- safe reload during active prompt execution
- guaranteed browser-side node definition refresh without page reload

## Important Limitations

### Browser refresh may still be needed

The server can reload the Python side of the node pack, but the browser may still need a refresh to pick up updated node definitions.

### Web directory changes still require a full restart

If a node pack changes its registered web directories, this implementation intentionally requires a full ComfyUI restart.

### This is development-oriented

This feature is designed for controlled dev/UAT usage, not as a broad production hot-reload system.

### Version compatibility matters

This package is a source drop-in patch set. It assumes the destination ComfyUI and ComfyUI-Manager versions are close to the version listed above.

## Security And Privacy Notes

- This package does not add new outbound internet requests.
- This package does not include system-specific paths or personal machine configuration.
- The added reload functionality is local-only.

Note:

- the bundled `comfyui-manager.js` still contains the normal pre-existing ComfyUI-Manager links and behavior
- those are part of ComfyUI-Manager itself, not new network behavior added by this reload feature

## FAQ

### What problem does this solve?

It reduces the need to restart the full ComfyUI server every time you edit a custom node during development.

### Does this fully replace restarting ComfyUI?

No. Some changes still require a full restart, especially frontend or web directory changes.

### Do I need any new dependencies?

No.

### Does this work without ComfyUI-Manager?

Partially. The backend reload support works without ComfyUI-Manager, but the toolbar UI does not.

### Can I distribute this as drop-in code on GitHub?

Yes. This package is prepared for that use.

### Can I reload any custom node pack?

No. Only packs explicitly listed in `--dev-reload-node-pack` can be reloaded.

### Why is allowlisting required?

It keeps the feature controlled and reduces accidental reload behavior in broader installs.

### Why is reload blocked when the queue is busy?

Reloading during active prompt execution risks inconsistent runtime state. Blocking reload until the queue is idle is a deliberate safety choice.

### Why do I still sometimes need to refresh the browser?

Because Python-side reload and browser-side node definition state are not the same thing.

### Why would a full restart still be required?

If the pack changes registered web directories or reload fails in a way that cannot be safely restored in-place, a restart is the safe fallback.

### Is this production-ready?

It is better described as development-ready or UAT-ready.

## Suggested GitHub Description

> Development-only hot reload support for allowlisted ComfyUI custom node packs, with optional ComfyUI-Manager toolbar UI.

## Recommended Warnings

- development-only feature
- source patch, not a standalone plugin
- version compatibility matters
- browser refresh may still be required
- full restart may still be required in some cases

## Future Improvements

- core frontend UI without ComfyUI-Manager
- optional filesystem watch mode
- richer reload result summaries
- change detection for edited files
- deeper compatibility handling for more extension patterns

## Package Date

- August 19, 2026

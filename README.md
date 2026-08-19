# ComfyUI Dev Node Reload

Development-only hot reload support for allowlisted ComfyUI custom node packs, with an optional ComfyUI-Manager toolbar UI.

## Start Here

- Main documentation: `GITHUB_PROJECT_GUIDE.md`
- Drop-in files:
  - `comfy/cli_args.py`
  - `nodes.py`
  - `server.py`
  - `custom_nodes/ComfyUI-Manager/js/comfyui-manager.js`

## Quick Install

1. Copy the package files into the matching locations in your ComfyUI install.
2. Start ComfyUI with:

```powershell
python main.py --enable-dev-node-reload --dev-reload-node-pack YourNodeFolderName
```

3. If ComfyUI-Manager is installed, use the top toolbar `Reload Nodes` button.

## Notes

- No new Python dependencies are required.
- No new JavaScript dependencies are required.
- The backend reload feature works without ComfyUI-Manager.
- The `Reload Nodes` toolbar UI requires ComfyUI-Manager.

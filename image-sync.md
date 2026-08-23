---

## 1. Create the folder layout

One folder per pipeline. Each has its own config + its own workspace.
NEVER share workspaces between pipelines, and NEVER reuse the old
oc-mirror v1 workspace directory.

    mkdir -p ~/mirror/releases/workspace
    mkdir -p ~/mirror/essentials/workspace
    mkdir -p ~/mirror/catalogs/workspace
    mkdir -p ~/mirror/templates

Result:

    ~/mirror/
    ├── releases/        # OCP release payloads + signatures
    │   └── workspace/
    ├── essentials/      # support-tools, oc client, UBI
    │   └── workspace/
    ├── catalogs/        # operator catalogs (selective)
    │   └── workspace/
    └── templates/       # per-4.x samples generated via aba TUI (see §6)

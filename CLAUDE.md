# CLAUDE.md

## Project Overview

TODO: Describe what this project does in 1-2 sentences.

## Security Model

This devcontainer enforces a separation between AI assistants and sensitive data (PHI, credentials, etc.):

- **Container boundary**: Claude runs inside the container and can only see `/workspace`. Paths outside `/workspace` are not mounted and not readable — don't waste time trying to access them.
- **Code is shared**: The workspace folder is bind-mounted from the host. When the host folder is on a cluster SMB mount (see Path Equivalence below), code written here is also visible on the cluster at its original path.
- **Data stays outside**: Sensitive data lives on the cluster or host filesystem, not in the workspace. Code references data by its cluster path. The user runs data-touching code outside the container after reviewing it.
- **Write code, not data**: Write analysis logic, visualization specs (Altair, matplotlib, etc.), and processing pipelines in notebooks and scripts. Never embed or hard-code sensitive data values in code.

## Execution Model

Code is **written** inside the container but **executed** on the HPC cluster. The two main execution paths are:

1. **Jupyter notebooks**: A Jupyter kernel runs on an HPC node. VS Code (in the container) connects to it via SSH tunnel (see `docs/jupyter-hpc.md`). When a notebook cell runs, it runs on the HPC node with full access to cluster paths.
2. **Shell scripts / SLURM jobs**: The user SSH-es to the cluster and runs scripts directly, or submits them via SLURM.

Because code executes on the cluster, all data paths in scripts and notebooks must use cluster paths (e.g., `/fh/fast/...`), not container paths. These paths are real and accessible at execution time.

## Path Equivalence

**Applies when**: the workspace folder is on a cluster SMB mount (e.g., Fred Hutch `/fh/fast/...` mounted at `/Volumes/ghajar_c/...` on macOS). If your workspace is a local-only folder, only the container ↔ macOS mapping applies.

The same files are accessible at different paths depending on context:

| Context | Example path |
|---|---|
| **Inside container** (where Claude writes code) | `/workspace/src/my_project/foo.py` |
| **macOS host** (Finder, Mac terminal) | `/Volumes/ghajar_c/.../my_project/src/my_project/foo.py` |
| **HPC cluster** (where code executes) | `/fh/fast/ghajar_c/.../my_project/src/my_project/foo.py` |

These are the **same filesystem**. A file Claude writes at `/workspace/src/my_project/foo.py` is immediately visible on the HPC at its `/fh/fast/...` path — no sync, no copy, no delay.

**Rule of thumb**: When writing code that will execute on the HPC (notebooks, scripts, SLURM jobs), always use `/fh/fast/...` paths for data references. The code itself lives in the workspace, but the paths it references must be valid on the cluster where it runs.

## Build & Test Commands

```bash
make all        # Run all quality checks (lint + typecheck + test)
make test       # Run pytest
make lint       # Run ruff linter
make format     # Auto-format code
make typecheck  # Run mypy (strict mode)
make docs       # Serve documentation locally
```

## Critical Requirements

- **FAIL FAST**: No silent fallbacks. Raise explicit errors. Never swallow exceptions.
- **TYPE EVERYTHING**: All functions must have type hints. Run `make typecheck`.
- **SMALL FUNCTIONS**: Single responsibility. If > 20 lines, consider splitting.
- **TEST FIRST**: Write tests before implementation code. Run `make test`.

## Git Workflow

1. Create feature branch: `git checkout -b feature/<name>`
2. Update `docs/` with design rationale if non-trivial
3. Write tests FIRST
4. Make granular commits
5. Run `make all` before pushing
6. Push and create PR for review
7. Squash-merge to main

## Code Standards

- No over-engineering. Only make changes directly requested or clearly necessary.
- No silent error swallowing. If something fails, raise.
- Explicit over implicit. Name things clearly.
- DRY, but don't prematurely abstract. Three similar lines > one premature abstraction.
- Keep functions pure where possible. Minimize side effects.

## Key Paths

TODO: Document important code paths for your project. Data paths should describe cluster locations for reference in code, but remember they are not accessible from inside the container.

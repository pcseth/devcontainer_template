# My Project

TODO: Describe your project.

## Security Model

This template uses a devcontainer to create a separation between AI coding assistants (Claude, Codex) and sensitive data:

```
+---------------------------+       +---------------------------+
|  Container (AI sandbox)   |       |   Host / Cluster          |
|                           |       |                           |
|  Claude / Codex           |       |   Sensitive data (PHI)    |
|  Code editor (VS Code)    |       |   Cluster storage         |
|  /workspace (code only)  <------->|   /fh/fast/.../project/   |
|                           | bind  |                           |
+---------------------------+ mount +---------------------------+
```

- **AI assistants** see only the code in `/workspace` — no access to data paths
- **Code syncs automatically** — the workspace is bind-mounted from the host (or cluster mount), so files written in the container appear at their original path on the cluster
- **Data stays outside** — code references cluster data paths, but the user runs data-touching code outside the container after reviewing it

This means you can use AI assistants freely for writing analysis code, visualization specs, and tests without exposing sensitive data.

## Setup

1. **Use this template**: Click "Use this template" on GitHub to create your own repo
2. **Clone and open**: Clone your new repo and open in VS Code
3. **Start container**: Click "Reopen in Container" when prompted
4. **Rename package**: Rename `src/my_project/` to your project name
5. **Update config**: Update `pyproject.toml` with your project name and dependencies
6. **Verify**: Run `make all` to confirm everything works
7. **Start coding**: Write tests first, then implement!

## Development

```bash
make all        # Run lint + typecheck + test
make test       # Run tests only
make lint       # Check code style
make format     # Auto-format code
make typecheck  # Check types (strict)
make docs       # Serve docs locally
```

## Adding Dependencies

1. Add runtime dependencies to `[project] dependencies` in `pyproject.toml`
2. Add dev-only dependencies to `[project.optional-dependencies] dev`
3. If a package needs system libraries, add them to `containers/Dockerfile`
4. Run `pip install -e '.[dev]'` to install

## Project Structure

```
.devcontainer/              # Container configuration
containers/Dockerfile       # Container image definition
src/my_project/             # Your package code (rename this!)
tests/                      # Test files
docs/                       # Documentation (mkdocs)
Makefile                    # Build/test commands
pyproject.toml              # Package config + tool settings
CLAUDE.md                   # AI assistant instructions + security boundary
README.md                   # This file
```

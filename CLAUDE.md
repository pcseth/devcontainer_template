# CLAUDE.md

## Project Overview

TODO: Describe what this project does in 1-2 sentences.

## Security Model

This devcontainer enforces a separation between AI assistants and sensitive data (PHI, credentials, etc.):

- **Container boundary**: Claude runs inside the container and can only see `/workspace`. Paths outside `/workspace` are not mounted and not readable — don't waste time trying to access them.
- **Code is shared**: The workspace folder is bind-mounted from the host. When the host folder is a cluster mount (e.g., Fred Hutch `/fh/fast/...`), code written here is also visible on the cluster at its original path (e.g., `/fh/fast/ghajar_c/.../` rather than `/workspace/...`).
- **Data stays outside**: Sensitive data lives on the cluster or host filesystem, not in the workspace. Code references data by its cluster path. The user runs data-touching code outside the container after reviewing it.
- **Write code, not data**: Write analysis logic, visualization specs (Altair, matplotlib, etc.), and processing pipelines in notebooks and scripts. Never embed or hard-code sensitive data values in code.

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

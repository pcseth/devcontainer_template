# CLAUDE.md

## Project Overview

TODO: Describe what this project does in 1-2 sentences.

## Quick Start

1. Open this folder in VSCode
2. Click "Reopen in Container" when prompted
3. Run `make all` to verify everything works
4. Run `make docs` to browse documentation at localhost:8000

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
2. Update design docs first
3. Write tests FIRST
4. Make granular commits
5. Run `make all` before pushing
6. Push and create PR for review
7. Squash-merge to main

## Before Implementing New Code

1. Check existing code for similar functionality
2. Update `docs/` with design rationale if non-trivial
3. Write tests FIRST (or comparative implementation for validation)

## Code Standards

- No over-engineering. Only make changes directly requested or clearly necessary.
- No silent error swallowing. If something fails, raise.
- Explicit over implicit. Name things clearly.
- DRY, but don't prematurely abstract. Three similar lines > one premature abstraction.
- Keep functions pure where possible. Minimize side effects.

## Key Paths

TODO: Document important file paths and data locations for your project.

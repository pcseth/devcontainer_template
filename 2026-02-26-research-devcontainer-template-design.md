# Research DevContainer Template Design

## Goal

A GitHub template repo providing a ready-to-go devcontainer workspace for computational research projects. Encodes engineering best practices (fail fast, TDD, strict typing) inspired by Matsen Lab standards. Domain-agnostic — users add their own packages and code.

## Delivery

GitHub template repository. Users click "Use this template" to create their own repo, rename the package, add dependencies, and start coding.

## Repository Structure

```
research-devcontainer-template/
├── .devcontainer/
│   └── devcontainer.json          # Container config, mounts, extensions
├── containers/
│   └── Dockerfile                 # Python 3.12-slim + dev tooling
├── src/
│   └── my_project/
│       └── __init__.py            # Empty package (user renames)
├── tests/
│   ├── __init__.py
│   └── test_placeholder.py        # Single passing test
├── docs/
│   └── index.md                   # Doc site landing page
├── Makefile                       # all, test, lint, format, typecheck, docs
├── pyproject.toml                 # Package config + tool settings
├── mkdocs.yml                     # Documentation site
├── CLAUDE.md                      # Engineering best practices
├── README.md                      # Getting started guide
└── .gitignore                     # Python + data + IDE patterns
```

## Dockerfile

Base image: `python:3.12-slim`

System packages:
- git, curl, openssh-client, sudo
- build-essential, pkg-config

CLI tools:
- GitHub CLI (gh)
- Node.js 20 + Claude Code CLI + Codex CLI

Python tooling (pre-installed in image):
- pytest, ruff, mypy
- jupyter, ipykernel
- mkdocs-material

No domain-specific packages — users add their own to pyproject.toml.

Creates `vscode` user with UID 501 (macOS compatibility) and sudo access.

## devcontainer.json

- Build from `containers/Dockerfile`
- Workspace mount at `/workspace`
- Auth mounts: SSH agent socket, `~/.claude`, `~/.codex`
- Env vars: `ANTHROPIC_API_KEY`, `OPENAI_API_KEY`, `SSH_AUTH_SOCK`, `HOST_HOME`
- Comment documenting how to add additional volume mounts (data directories, etc.)
- Post-create command: `pip install -e '.[dev]'`
- Post-start command: Creates a symlink from the host's home path to the container's `~/.claude` so that Claude Code plugins (which store absolute paths in their manifests) resolve correctly regardless of which environment installed them. Install plugins from the host; the symlink ensures they load inside containers too.
- VS Code extensions: Python, Pylance, Ruff, Claude Code, Jupyter

## CLAUDE.md (Engineering Best Practices)

The core of the template. Sections:

### Critical Requirements
- **Fail fast**: No silent fallbacks. Raise explicit errors.
- **Type everything**: All functions must have type hints. Run `make typecheck`.
- **Small functions**: Single responsibility. Split if >20 lines.
- **Write tests first**: Tests before implementation code.

### Build & Test
- `make all` — run lint + typecheck + test (run before every push)
- `make test` — pytest
- `make lint` — ruff check
- `make format` — ruff format (auto-fix)
- `make typecheck` — mypy strict
- `make docs` — serve documentation locally

### Git Workflow
1. Create feature branch: `git checkout -b feature/<name>`
2. Update design docs first
3. Make granular commits
4. Run `make all` before pushing
5. Push and create PR for review
6. Squash-merge to main

### Before Implementing New Code
1. Check existing code for similar functionality
2. Update design docs with rationale
3. Write tests FIRST (or comparative implementation for validation)

### Code Standards
- No over-engineering. Only make changes directly requested or clearly necessary.
- No silent error swallowing. If something fails, raise.
- Explicit over implicit. Name things clearly.
- DRY, but don't prematurely abstract.

## Makefile

```makefile
.PHONY: all test lint format typecheck docs clean

all: lint typecheck test

test:
	pytest tests/ -v

lint:
	ruff check src/ tests/

format:
	ruff format src/ tests/

typecheck:
	mypy src/

docs:
	mkdocs serve

clean:
	find . -type d -name __pycache__ -exec rm -rf {} +
	find . -type d -name .mypy_cache -exec rm -rf {} +
	find . -type d -name .pytest_cache -exec rm -rf {} +
```

## pyproject.toml

- src-layout package (`my_project`)
- Core dependencies: empty (users add their own)
- `[dev]` extras: pytest, ruff, mypy, mkdocs-material
- Ruff config: line-length 100, target Python 3.11, rules E/F/I/UP/B
- MyPy config: strict mode enabled
- pytest config: testpaths = ["tests"]

## README.md

Short getting-started:
1. Click "Use this template" on GitHub
2. Clone and open in VSCode → "Reopen in Container"
3. Rename `src/my_project/` to your project name
4. Update `pyproject.toml` with project name + dependencies
5. Run `make all` to verify setup
6. Start coding (tests first!)

## What This Template Does NOT Include

- Domain-specific packages (pysam, anndata, etc.)
- HPC/SLURM configuration (project-specific)
- Data paths or mounts beyond workspace
- Pre-commit hooks (Makefile workflow is sufficient)
- CI/CD pipelines (varies by team)

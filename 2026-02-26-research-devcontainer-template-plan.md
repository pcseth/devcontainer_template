# Research DevContainer Template Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Create a GitHub template repo with devcontainer, tooling, and engineering best practices for computational research projects.

**Architecture:** A new git repo at `~/research-devcontainer-template/` containing static configuration files (Dockerfile, devcontainer.json, Makefile, pyproject.toml, CLAUDE.md, etc.) with no domain-specific code. All files are created from scratch.

**Tech Stack:** Python 3.12, Docker, VS Code Dev Containers, Ruff, MyPy, Pytest, MkDocs

---

### Task 1: Initialize repo and create Dockerfile

**Files:**
- Create: `~/research-devcontainer-template/containers/Dockerfile`

**Step 1: Create the repo and directory structure**

```bash
mkdir -p ~/research-devcontainer-template/containers
cd ~/research-devcontainer-template
git init
```

**Step 2: Create the Dockerfile**

Create `containers/Dockerfile`:

```dockerfile
# Research DevContainer: Python 3.12 + dev tooling
FROM python:3.12-slim

ENV DEBIAN_FRONTEND=noninteractive

# System dependencies + GitHub CLI
RUN apt-get update && apt-get install -y --no-install-recommends \
    curl git openssh-client sudo \
    build-essential pkg-config \
    zlib1g-dev libbz2-dev liblzma-dev \
    libcurl4-openssl-dev libssl-dev libncurses5-dev \
    && curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg \
       | dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg \
    && echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" \
       > /etc/apt/sources.list.d/github-cli.list \
    && apt-get update && apt-get install -y gh \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# Create non-root user (UID 501 for macOS compatibility)
ARG USERNAME=vscode
ARG USER_UID=501
ARG USER_GID=$USER_UID
RUN groupadd --gid $USER_GID $USERNAME \
    && useradd --uid $USER_UID --gid $USER_GID -m $USERNAME \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
    && chmod 0440 /etc/sudoers.d/$USERNAME

# Install Node.js 20 (for Claude Code CLI)
RUN curl -fsSL https://deb.nodesource.com/setup_20.x | bash - \
    && apt-get install -y nodejs && apt-get clean && rm -rf /var/lib/apt/lists/*

# Install Claude Code CLI and OpenAI Codex CLI
RUN npm install -g @anthropic-ai/claude-code @openai/codex

# Install Python dev tooling (domain packages go in pyproject.toml)
RUN pip install --no-cache-dir \
    jupyter ipykernel \
    mkdocs mkdocs-material pymdown-extensions \
    pytest ruff mypy

USER vscode
WORKDIR /workspace
CMD ["bash"]
```

**Step 3: Commit**

```bash
cd ~/research-devcontainer-template
git add containers/Dockerfile
git commit -m "feat: add Dockerfile with Python 3.12 + dev tooling

Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>"
```

---

### Task 2: Create devcontainer.json

**Files:**
- Create: `~/research-devcontainer-template/.devcontainer/devcontainer.json`

**Step 1: Create the devcontainer config**

Create `.devcontainer/devcontainer.json`:

```json
{
  "name": "Research DevContainer",
  "build": {
    "dockerfile": "../containers/Dockerfile"
  },
  "workspaceFolder": "/workspace",
  "workspaceMount": "source=${localWorkspaceFolder},target=/workspace,type=bind",
  "remoteUser": "vscode",
  "mounts": [
    "source=${localEnv:HOME}/.claude,target=/home/vscode/.claude,type=bind",
    "source=/run/host-services/ssh-auth.sock,target=/run/host-services/ssh-auth.sock,type=bind",
    "source=${localEnv:HOME}/.codex,target=/home/vscode/.codex,type=bind"
    // To mount additional directories (e.g., data), add:
    // "source=/path/to/data,target=/data,type=bind"
  ],
  "containerEnv": {
    "ANTHROPIC_API_KEY": "${localEnv:ANTHROPIC_API_KEY}",
    "OPENAI_API_KEY": "${localEnv:OPENAI_API_KEY}",
    "SSH_AUTH_SOCK": "/run/host-services/ssh-auth.sock",
    "HOST_HOME": "${localEnv:HOME}"
  },
  "postCreateCommand": "pip install -e '.[dev]'",
  "postStartCommand": "if [ -n \"$HOST_HOME\" ] && [ \"$HOST_HOME\" != \"$HOME\" ]; then sudo mkdir -p \"$(dirname $HOST_HOME)\" && sudo ln -sfn \"$HOME/.claude\" \"$HOST_HOME/.claude\"; fi && python3 --version && make --version > /dev/null",
  "customizations": {
    "vscode": {
      "extensions": [
        "ms-python.python",
        "ms-python.vscode-pylance",
        "ms-toolsai.jupyter",
        "charliermarsh.ruff",
        "anthropic.claude-code",
        "openai.chatgpt"
      ],
      "settings": {
        "python.defaultInterpreterPath": "/usr/local/bin/python",
        "editor.formatOnSave": true,
        "terminal.integrated.defaultProfile.linux": "bash"
      }
    }
  }
}
```

**NOTE:** JSON does not support comments. The comment lines above (`// To mount...`) must be removed. Instead, add the data mount guidance to README.md. The actual file must be valid JSON without comments.

**NOTE:** The `HOST_HOME` env var and the `postStartCommand` symlink fix a Claude Code plugin path issue. Plugins store absolute paths in their manifests (`~/.claude/plugins/installed_plugins.json`). Since `~/.claude` is bind-mounted, the same manifest is shared between host and container — but absolute paths differ (`/Users/username/...` on macOS vs `/home/vscode/...` in the container). The symlink creates `/Users/username/.claude` → `/home/vscode/.claude` inside the container so host-recorded paths resolve. Install plugins from the host; they'll load in containers via the symlink.

Corrected version without comments:

```json
{
  "name": "Research DevContainer",
  "build": {
    "dockerfile": "../containers/Dockerfile"
  },
  "workspaceFolder": "/workspace",
  "workspaceMount": "source=${localWorkspaceFolder},target=/workspace,type=bind",
  "remoteUser": "vscode",
  "mounts": [
    "source=${localEnv:HOME}/.claude,target=/home/vscode/.claude,type=bind",
    "source=/run/host-services/ssh-auth.sock,target=/run/host-services/ssh-auth.sock,type=bind",
    "source=${localEnv:HOME}/.codex,target=/home/vscode/.codex,type=bind"
  ],
  "containerEnv": {
    "ANTHROPIC_API_KEY": "${localEnv:ANTHROPIC_API_KEY}",
    "OPENAI_API_KEY": "${localEnv:OPENAI_API_KEY}",
    "SSH_AUTH_SOCK": "/run/host-services/ssh-auth.sock",
    "HOST_HOME": "${localEnv:HOME}"
  },
  "postCreateCommand": "pip install -e '.[dev]'",
  "postStartCommand": "if [ -n \"$HOST_HOME\" ] && [ \"$HOST_HOME\" != \"$HOME\" ]; then sudo mkdir -p \"$(dirname $HOST_HOME)\" && sudo ln -sfn \"$HOME/.claude\" \"$HOST_HOME/.claude\"; fi && python3 --version && make --version > /dev/null",
  "customizations": {
    "vscode": {
      "extensions": [
        "ms-python.python",
        "ms-python.vscode-pylance",
        "ms-toolsai.jupyter",
        "charliermarsh.ruff",
        "anthropic.claude-code",
        "openai.chatgpt"
      ],
      "settings": {
        "python.defaultInterpreterPath": "/usr/local/bin/python",
        "editor.formatOnSave": true,
        "terminal.integrated.defaultProfile.linux": "bash"
      }
    }
  }
}
```

**Step 2: Commit**

```bash
cd ~/research-devcontainer-template
git add .devcontainer/devcontainer.json
git commit -m "feat: add devcontainer config with auth mounts and extensions

Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>"
```

---

### Task 3: Create pyproject.toml, Makefile, and package skeleton

**Files:**
- Create: `~/research-devcontainer-template/pyproject.toml`
- Create: `~/research-devcontainer-template/Makefile`
- Create: `~/research-devcontainer-template/src/my_project/__init__.py`
- Create: `~/research-devcontainer-template/tests/__init__.py`
- Create: `~/research-devcontainer-template/tests/test_placeholder.py`

**Step 1: Create pyproject.toml**

```toml
[project]
name = "my-project"
version = "0.1.0"
description = "TODO: Describe your project"
requires-python = ">=3.11"
dependencies = []

[project.optional-dependencies]
dev = [
    "pytest>=7.0",
    "ruff>=0.1",
    "mypy>=1.0",
]

[tool.ruff]
line-length = 100
target-version = "py311"

[tool.ruff.lint]
select = ["E", "F", "I", "UP", "B"]

[tool.mypy]
python_version = "3.11"
strict = true
warn_return_any = true

[tool.pytest.ini_options]
testpaths = ["tests"]
```

**Step 2: Create Makefile**

```makefile
.PHONY: all test lint format typecheck docs docs-build clean

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

docs-build:
	mkdocs build

clean:
	find . -type d -name __pycache__ -exec rm -rf {} +
	find . -type d -name .mypy_cache -exec rm -rf {} +
	find . -type d -name .pytest_cache -exec rm -rf {} +
```

**IMPORTANT:** Makefile rules MUST use actual tab characters for indentation, not spaces.

**Step 3: Create package skeleton**

Create `src/my_project/__init__.py`:
```python
```
(Empty file)

Create `tests/__init__.py`:
```python
```
(Empty file)

Create `tests/test_placeholder.py`:
```python
def test_placeholder() -> None:
    """Remove this test once you have real tests."""
    assert True
```

**Step 4: Verify**

```bash
cd ~/research-devcontainer-template
pip install -e '.[dev]' && make all
```

Expected: 1 test passes, lint clean, typecheck clean.

**Step 5: Commit**

```bash
cd ~/research-devcontainer-template
git add pyproject.toml Makefile src/ tests/
git commit -m "feat: add pyproject.toml, Makefile, and package skeleton

Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>"
```

---

### Task 4: Create CLAUDE.md

**Files:**
- Create: `~/research-devcontainer-template/CLAUDE.md`

**Step 1: Create CLAUDE.md**

```markdown
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
```

**Step 2: Commit**

```bash
cd ~/research-devcontainer-template
git add CLAUDE.md
git commit -m "feat: add CLAUDE.md with engineering best practices

Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>"
```

---

### Task 5: Create docs, README, .gitignore, and mkdocs.yml

**Files:**
- Create: `~/research-devcontainer-template/docs/index.md`
- Create: `~/research-devcontainer-template/mkdocs.yml`
- Create: `~/research-devcontainer-template/README.md`
- Create: `~/research-devcontainer-template/.gitignore`

**Step 1: Create docs/index.md**

```markdown
# My Project

Welcome to the project documentation.

## Getting Started

See the [README](https://github.com/YOUR_USERNAME/YOUR_REPO) for setup instructions.
```

**Step 2: Create mkdocs.yml**

```yaml
site_name: My Project
theme:
  name: material
  palette:
    scheme: default
nav:
  - Home: index.md
```

**Step 3: Create README.md**

```markdown
# My Project

TODO: Describe your project.

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

## Adding Data Mounts

To mount additional directories (e.g., shared data) into the container, add entries to the `mounts` array in `.devcontainer/devcontainer.json`:

```json
"source=/path/to/data,target=/data,type=bind"
```

## Project Structure

```
├── .devcontainer/          # Container configuration
├── containers/Dockerfile   # Container image definition
├── src/my_project/         # Your package code (rename this!)
├── tests/                  # Test files
├── docs/                   # Documentation (mkdocs)
├── Makefile                # Build/test commands
├── pyproject.toml          # Package config + tool settings
├── CLAUDE.md               # AI assistant instructions + best practices
└── README.md               # This file
```
```

**Step 4: Create .gitignore**

```gitignore
# Python
__pycache__/
*.py[cod]
*$py.class
*.so
.Python
build/
develop-eggs/
dist/
downloads/
eggs/
.eggs/
lib/
lib64/
parts/
sdist/
var/
wheels/
*.egg-info/
.installed.cfg
*.egg

# Virtual environments
.env
.venv
env/
venv/
ENV/

# Testing
.pytest_cache/
.coverage
htmlcov/
.tox/
.nox/

# Type checking
.mypy_cache/

# Linting
.ruff_cache/

# MkDocs
site/

# Jupyter
.ipynb_checkpoints/

# IDE
.idea/
.vscode/
*.swp
*.swo
*~

# macOS
.DS_Store

# Claude Code local settings (user-specific)
.claude/

# Output (customize for your project)
*.log
*.out
*.err
```

**Step 5: Commit**

```bash
cd ~/research-devcontainer-template
git add docs/ mkdocs.yml README.md .gitignore
git commit -m "feat: add docs, README, gitignore, and mkdocs config

Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>"
```

---

### Task 6: Final verification

**Step 1: Verify repo structure**

```bash
cd ~/research-devcontainer-template
find . -not -path './.git/*' -not -path './.git' | sort
```

Expected output:
```
.
./.devcontainer
./.devcontainer/devcontainer.json
./.gitignore
./CLAUDE.md
./Makefile
./README.md
./containers
./containers/Dockerfile
./docs
./docs/index.md
./mkdocs.yml
./pyproject.toml
./src
./src/my_project
./src/my_project/__init__.py
./tests
./tests/__init__.py
./tests/test_placeholder.py
```

**Step 2: Verify make all passes**

```bash
cd ~/research-devcontainer-template
pip install -e '.[dev]' && make all
```

Expected: 1 test passes, lint clean, typecheck clean.

**Step 3: Verify git log**

```bash
git log --oneline
```

Expected: 5 commits (Dockerfile, devcontainer, package skeleton, CLAUDE.md, docs/README).

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

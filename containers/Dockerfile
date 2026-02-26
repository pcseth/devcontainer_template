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

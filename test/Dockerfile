FROM ubuntu:24.04

ENV DEBIAN_FRONTEND=noninteractive

# Core tools
RUN apt-get update && apt-get install -y \
    tmux \
    jq \
    bash \
    curl \
    git \
    procps \
    nodejs \
    npm \
    python3 \
    && rm -rf /var/lib/apt/lists/*

# Install just from a pinned GitHub release.
# The install.sh helper has proven unreliable in CI/test images because failures
# in a shell pipeline can be masked during Docker builds and leave no binary
# behind. Install the release artifact directly instead.
ARG JUST_VERSION=1.47.1
RUN arch="$(dpkg --print-architecture)" && \
    case "$arch" in \
        amd64) target="x86_64-unknown-linux-musl" ;; \
        arm64) target="aarch64-unknown-linux-musl" ;; \
        *) echo "unsupported architecture for just: $arch" >&2; exit 1 ;; \
    esac && \
    curl -fsSL -o /tmp/just.tar.gz "https://github.com/casey/just/releases/download/${JUST_VERSION}/just-${JUST_VERSION}-${target}.tar.gz" && \
    tar -xzf /tmp/just.tar.gz -C /tmp just && \
    install -m 0755 /tmp/just /usr/local/bin/just && \
    rm -f /tmp/just /tmp/just.tar.gz

# Install real assistant CLIs (no mocks)
# Pin major versions to prevent upstream breaking changes from failing CI.
# codex is pre-1.0 (0.x), so pin to ~0 (any 0.x); claude-code is 2.x; opencode-ai
# and @github/copilot are 1.x.
RUN npm install -g "@anthropic-ai/claude-code@^2" "@openai/codex@^0" "opencode-ai@^1" "@github/copilot@^1"

# Verify the binaries exist
RUN which claude && which codex && which opencode && which copilot

# Create a test user (avoid running as root — tmux behaves differently)
RUN useradd -m -s /bin/bash testuser
USER testuser
WORKDIR /home/testuser

# Copy the repo in (done as testuser for correct ownership)
COPY --chown=testuser:testuser . /home/testuser/tmux-assistant-resurrect

ENTRYPOINT ["bash", "/home/testuser/tmux-assistant-resurrect/test/run-tests.sh"]

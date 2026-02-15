# Original from https://github.com/anthropics/claude-code/blob/main/.devcontainer/Dockerfile on 2026-02-03
# Copied from https://github.com/richardTowers/claude-code/blob/main/.devcontainer/Dockerfile
# 2026-02-14 jbgreer: removed Ruby, npm, switched to Debian Trixie-slim base
# 2026-02-15 jbgreer:  trixie supports git-delta directly

FROM debian:trixie

ARG TZ
ENV TZ="$TZ"

ARG CLAUDE_CODE_VERSION=latest

# Install basic development tools, browser dependencies and iptables/ipset
RUN apt-get update && apt-get install -y --no-install-recommends \
  aggregate \
  apt-transport-https \
  ca-certificates \
  cidr \
  curl \
  dnsutils \
  fzf \
  gh \
  git \
  git-delta \
  gnupg2 \
  iproute2 \
  ipset \
  iptables \
  jq \
  less \
  locales \
  man-db \
  nano \
  neovim \
  procps \
  sudo \
  unzip \
  wget \
  zsh \
  && sed -i '/en_US.UTF-8/s/^# //' /etc/locale.gen && locale-gen \
  && apt-get clean && rm -rf /var/lib/apt/lists/*

RUN apt-get install --reinstall ca-certificates && update-ca-certificates -f

# Use legacy iptables as the apple/container kernel doesn't support the nftables kernel API
RUN update-alternatives --set iptables /usr/sbin/iptables-legacy && update-alternatives --set ip6tables /usr/sbin/ip6tables-legacy

ARG USERNAME=claude
RUN adduser ${USERNAME}

# Persist bash history.
RUN SNIPPET="export PROMPT_COMMAND='history -a' && export HISTFILE=/commandhistory/.bash_history" \
  && mkdir /commandhistory \
  && touch /commandhistory/.bash_history \
  && chown -R ${USERNAME} /commandhistory

# Set `DEVCONTAINER` environment variable to help with orientation
ENV LANG=en_US.UTF-8
ENV LC_ALL=en_US.UTF-8
ENV DEVCONTAINER=true

# Create workspace and config directories and set permissions
RUN mkdir -p /workspace /home/${USERNAME}/.claude && \
  chown -R ${USERNAME}:${USERNAME} /workspace /home/${USERNAME}/.claude

WORKDIR /workspace

# Set up non-root user
USER ${USERNAME}

ENV PATH=$PATH:/home/${USERNAME}/.local/bin

# Set the default shell to zsh rather than sh
ENV SHELL=/bin/zsh

# Set the default editor and visual
ENV EDITOR=nvim
ENV VISUAL=nvim

# Container environment (moved from devcontainer.json containerEnv)
ENV CLAUDE_CONFIG_DIR="/home/${USERNAME}/.claude"

# Prevent forwarded SSH agent from leaking into shell sessions
RUN echo 'unset SSH_AUTH_SOCK' >> /home/${USERNAME}/.zshrc

# Install Claude
RUN curl -fsSL https://claude.ai/install.sh | TARGET=${CLAUDE_CODE_VERSION} bash

# Copy and set up firewall and entrypoint scripts
COPY init-firewall.sh entrypoint.sh /usr/local/bin/
USER root
RUN chmod +x /usr/local/bin/init-firewall.sh /usr/local/bin/entrypoint.sh && \
  echo "${USERNAME} ALL=(root) NOPASSWD: /usr/local/bin/init-firewall.sh" > /etc/sudoers.d/${USERNAME}-firewall && \
  chmod 0440 /etc/sudoers.d/${USERNAME}-firewall

USER ${USERNAME}
ENTRYPOINT ["/usr/local/bin/entrypoint.sh"]
CMD ["zsh"]

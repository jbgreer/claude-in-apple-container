# Copied from https://github.com/anthropics/claude-code/blob/main/.devcontainer/Dockerfile on 2026-02-03

FROM node:20

ARG TZ
ENV TZ="$TZ"

ARG CLAUDE_CODE_VERSION=latest

# Install basic development tools, browser dependencies and iptables/ipset
RUN apt-get update && apt-get install -y --no-install-recommends \
  aggregate \
  curl \
  dnsutils \
  fzf \
  gh \
  git \
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
  ripgrep \
  sudo \
  unzip \
  zsh \
  && sed -i '/en_US.UTF-8/s/^# //' /etc/locale.gen && locale-gen \
  && apt-get clean && rm -rf /var/lib/apt/lists/*

# Use legacy iptables as the apple/container kernel doesn't support the nftables kernel API
RUN update-alternatives --set iptables /usr/sbin/iptables-legacy && update-alternatives --set ip6tables /usr/sbin/ip6tables-legacy

# Ensure default node user has access to /usr/local/share
RUN mkdir -p /usr/local/share/npm-global && \
  chown -R node:node /usr/local/share

ARG USERNAME=node

# Persist bash history.
RUN SNIPPET="export PROMPT_COMMAND='history -a' && export HISTFILE=/commandhistory/.bash_history" \
  && mkdir /commandhistory \
  && touch /commandhistory/.bash_history \
  && chown -R $USERNAME /commandhistory

# Set `DEVCONTAINER` environment variable to help with orientation
ENV LANG=en_US.UTF-8
ENV LC_ALL=en_US.UTF-8
ENV DEVCONTAINER=true

# Create workspace and config directories and set permissions
RUN mkdir -p /workspace /home/$USERNAME/.claude && \
  chown -R $USERNAME:$USERNAME /workspace /home/$USERNAME/.claude

WORKDIR /workspace

ARG GIT_DELTA_VERSION=0.18.2
RUN ARCH=$(dpkg --print-architecture) && \
  wget "https://github.com/dandavison/delta/releases/download/${GIT_DELTA_VERSION}/git-delta_${GIT_DELTA_VERSION}_${ARCH}.deb" && \
  sudo dpkg -i "git-delta_${GIT_DELTA_VERSION}_${ARCH}.deb" && \
  rm "git-delta_${GIT_DELTA_VERSION}_${ARCH}.deb"

# Set up non-root user
USER node

# Install global packages
ENV NPM_CONFIG_PREFIX=/usr/local/share/npm-global
# ENV PATH=$PATH:/usr/local/share/npm-global/bin:/home/node/.local/bin:/home/node/.rbenv/bin:/home/node/.rbenv/shims
ENV PATH=$PATH:/usr/local/share/npm-global/bin:/home/$USERNAME/.local/bin

# Set the default shell to zsh rather than sh
ENV SHELL=/bin/zsh

# Set the default editor and visual
ENV EDITOR=nvim
ENV VISUAL=nvim

# Container environment (moved from devcontainer.json containerEnv)
ENV NODE_OPTIONS="--max-old-space-size=4096"
ENV CLAUDE_CONFIG_DIR="/home/node/.claude"

# Prevent VS Code's forwarded SSH agent from leaking into shell sessions
RUN echo 'unset SSH_AUTH_SOCK' >> /home/node/.zshrc

# Install Claude
RUN curl -fsSL https://claude.ai/install.sh | TARGET=${CLAUDE_CODE_VERSION} bash

# Copy and set up firewall and entrypoint scripts
COPY init-firewall.sh entrypoint.sh /usr/local/bin/
USER root
RUN chmod +x /usr/local/bin/init-firewall.sh /usr/local/bin/entrypoint.sh && \
  echo "node ALL=(root) NOPASSWD: /usr/local/bin/init-firewall.sh" > /etc/sudoers.d/$USERNAME-firewall && \
  chmod 0440 /etc/sudoers.d/$USERNAME-firewall

USER node
ENTRYPOINT ["/usr/local/bin/entrypoint.sh"]
CMD ["zsh"]

# Original from https://github.com/anthropics/claude-code/blob/main/.devcontainer/Dockerfile on 2026-02-03
# Copied from https://github.com/richardTowers/claude-code/blob/main/.devcontainer/Dockerfile
# 2026-02-14 jbgreer: removed Ruby, npm, switched to Debian Trixie-slim base
# 2026-02-15 jbgreer: debian trixie supports git-delta directly
# 2026-03-05 jbgreer: mainly switch to herenow run style
#

FROM debian:trixie-slim

# set timezone, locale editor and DEVCONTAINER in case we need to check
ARG TZ="Etc/UTC"
ENV TZ="$TZ" LANG=en_US.UTF-8 LC_ALL=en_US.UTF-8 EDITOR=nvim VISUAL=nvim DEVCONTAINER=true

# Install basic development tools, dependencies and iptables/ipset
# Use legacy iptables as the apple/container kernel doesn't support the nftables kernel API
RUN << EOF
  apt-get -qq update && apt-get -qq -y --no-install-recommends install \
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
  lazygit \
  less \
  locales \
  man-db \
  nano \
  neovim \
  procps \
  sudo \
  starship \
  stow \
  tmux \
  unzip \
  wget \
  zsh
  sed -i '/en_US.UTF-8/s/^# //' /etc/locale.gen && locale-gen
  apt-get -qq clean && rm -rf /var/lib/apt/lists/*
  update-alternatives --set iptables /usr/sbin/iptables-legacy && update-alternatives --set ip6tables /usr/sbin/ip6tables-legacy
EOF

ARG USERNAME=claude
RUN adduser ${USERNAME}

# Persist Bash history.
RUN << EOF
  mkdir /commandhistory
  touch /commandhistory/.bash_history
  chown -R ${USERNAME} /commandhistory
EOF

# create working directories and setup ownership
RUN << EOF
  mkdir -p /workspace /home/${USERNAME}/.claude
  chown -R ${USERNAME}:${USERNAME} /workspace /home/${USERNAME}/.claude
EOF

# Switch to non-root user
USER ${USERNAME}

# Download dotfiles and install via stow
RUN << EOF
  mkdir -p /home/${USERNAME}/.config
  git clone https://github.com/jbgreer/Dotfiles.git /home/${USERNAME}/Dotfiles && { cd /home/${USERNAME}/Dotfiles; git checkout container; }
  chown -R ${USERNAME}:${USERNAME} /home/${USERNAME}/Dotfiles
  stow --verbose --dir=/home/${USERNAME}/Dotfiles nvim
  stow --verbose --dir=/home/${USERNAME}/Dotfiles starship
  stow --verbose --dir=/home/${USERNAME}/Dotfiles tmux
  stow --verbose --dir=/home/${USERNAME}/Dotfiles zsh
  chown -R ${USERNAME}:${USERNAME} /home/${USERNAME}/.config
EOF

# Set the default shell to zsh, add ~/.local/bin to path for Claude
ENV SHELL=/bin/zsh PATH=/home/${USERNAME}/.local/bin:${PATH}

WORKDIR /workspace

# Install Claude
ENV CLAUDE_CONFIG_DIR="/home/${USERNAME}/.claude"
ARG CLAUDE_CODE_VERSION=latest
RUN curl -fsSL https://claude.ai/install.sh | TARGET=${CLAUDE_CODE_VERSION} bash

# Copy and set up firewall and entrypoint scripts
COPY init-firewall.sh entrypoint.sh /usr/local/bin/
USER root
RUN << EOF
  chmod +x /usr/local/bin/init-firewall.sh /usr/local/bin/entrypoint.sh
  echo "${USERNAME} ALL=(root) NOPASSWD: /usr/local/bin/init-firewall.sh" > /etc/sudoers.d/${USERNAME}-firewall
  chmod 0440 /etc/sudoers.d/${USERNAME}-firewall
EOF

USER ${USERNAME}
ENTRYPOINT ["/usr/local/bin/entrypoint.sh"]
CMD ["/usr/bin/zsh"]

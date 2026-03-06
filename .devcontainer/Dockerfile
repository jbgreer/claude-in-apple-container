# Original from https://github.com/anthropics/claude-code/blob/main/.devcontainer/Dockerfile on 2026-02-03
# Copied from https://github.com/richardTowers/claude-code/blob/main/.devcontainer/Dockerfile
# 2026-02-14 jbgreer: removed Ruby, npm, switched to Debian Trixie-slim base
# 2026-02-15 jbgreer: debian trixie supports git-delta directly
# 2026-03-05 jbgreer: mainly switch to herenow run style
# 2026-03-05 jbgreer: minimize number of layers
#

FROM debian:trixie-slim

ARG TZ="Etc/UTC"
ARG USERNAME=claude
ARG CLAUDE_CODE_VERSION=latest

# set timezone, locale, editor, DEVCONTAINER, shell, path, and claude config dir
ENV TZ="$TZ" LANG=en_US.UTF-8 LC_ALL=en_US.UTF-8 EDITOR=nvim VISUAL=nvim DEVCONTAINER=true \
    SHELL=/bin/zsh PATH=/home/${USERNAME}/.local/bin:${PATH} CLAUDE_CONFIG_DIR="/home/${USERNAME}/.claude"

# Copy firewall and entrypoint scripts before the root RUN so chmod/sudoers
# can be folded into the same layer as the package install.
COPY init-firewall.sh entrypoint.sh /usr/local/bin/

# Install basic development tools, dependencies and iptables/ipset.
# Use legacy iptables as the apple/container kernel doesn't support the nftables kernel API.
# Also creates the non-root user, sets up directories, and configures the firewall sudoer entry.
RUN << EOF
  apt-get -qq update && apt-get -qq -y --no-install-recommends install \
  aggregate \
  apt-transport-https \
  build-essential \
  ca-certificates \
  cidr \
  curl \
  direnv \
  dnsutils \
  du-dust \
  eza \
  fd-find \
  fzf \
  gh \
  git \
  git-delta \
  glow \
  gnupg2 \
  hyperfine \
  iproute2 \
  ipset \
  iptables \
  jq \
  just \
  lazygit \
  less \
  locales \
  man-db \
  neovim \
  pipx \
  procps \
  procs \
  python3 \
  python3-pip \
  ripgrep \
  sd \
  sudo \
  starship \
  stow \
  tmux \
  unzip \
  wget \
  xh \
  zsh
  sed -i '/en_US.UTF-8/s/^# //' /etc/locale.gen && locale-gen
  apt-get -qq clean && rm -rf /var/lib/apt/lists/*
  update-alternatives --set iptables /usr/sbin/iptables-legacy && update-alternatives --set ip6tables /usr/sbin/ip6tables-legacy
  adduser ${USERNAME}
  mkdir /commandhistory && touch /commandhistory/.bash_history && chown -R ${USERNAME} /commandhistory
  mkdir -p /workspace /home/${USERNAME}/.claude && chown -R ${USERNAME}:${USERNAME} /workspace /home/${USERNAME}/.claude
  chmod +x /usr/local/bin/init-firewall.sh /usr/local/bin/entrypoint.sh
  echo "${USERNAME} ALL=(root) NOPASSWD: /usr/local/bin/init-firewall.sh" > /etc/sudoers.d/${USERNAME}-firewall
  chmod 0440 /etc/sudoers.d/${USERNAME}-firewall
EOF

USER ${USERNAME}
WORKDIR /workspace

# Download dotfiles via stow and install Claude Code
RUN << EOF
  mkdir -p /home/${USERNAME}/.config
  git clone https://github.com/jbgreer/Dotfiles.git /home/${USERNAME}/Dotfiles && { cd /home/${USERNAME}/Dotfiles; git checkout container; }
  chown -R ${USERNAME}:${USERNAME} /home/${USERNAME}/Dotfiles
  stow --verbose --dir=/home/${USERNAME}/Dotfiles nvim
  stow --verbose --dir=/home/${USERNAME}/Dotfiles starship
  stow --verbose --dir=/home/${USERNAME}/Dotfiles tmux
  stow --verbose --dir=/home/${USERNAME}/Dotfiles zsh
  chown -R ${USERNAME}:${USERNAME} /home/${USERNAME}/.config
  curl -fsSL https://claude.ai/install.sh | TARGET=${CLAUDE_CODE_VERSION} bash
EOF

ENTRYPOINT ["/usr/local/bin/entrypoint.sh"]
CMD ["/usr/bin/zsh"]

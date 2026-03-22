FROM debian:sid-slim

ARG USERNAME=navicore
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# Avoid prompts during package installation
ENV DEBIAN_FRONTEND=noninteractive

# ── System packages (mirrors naviscripts/apt-packages.txt + wasos extras) ──
RUN apt-get update && apt-get install -y \
    # Core dev tools
    build-essential \
    clang \
    cmake \
    ninja-build \
    pkg-config \
    just \
    lua5.4 \
    # Shell and terminal
    zsh \
    tmux \
    fzf \
    bat \
    eza \
    wl-clipboard \
    # Editors/libs needed by neovim plugins
    libedit-dev \
    libssl-dev \
    libudev-dev \
    # General utilities
    curl \
    wget \
    git \
    sudo \
    unzip \
    ripgrep \
    fd-find \
    openssh-client \
    ca-certificates \
    && rm -rf /var/lib/apt/lists/*

# ── Non-root user ──
RUN groupadd --gid $USER_GID $USERNAME \
    && useradd --uid $USER_UID --gid $USER_GID -m -s /bin/zsh $USERNAME \
    && echo "$USERNAME ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers.d/$USERNAME

USER $USERNAME
WORKDIR /home/$USERNAME

# ── Rust toolchain ──
RUN curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | \
    sh -s -- -y --default-toolchain stable
ENV PATH="/home/$USERNAME/.cargo/bin:${PATH}"

RUN rustup component add rust-analyzer rust-src rustfmt clippy \
    && rustup target add wasm32-wasip1 wasm32-wasip2 wasm32-unknown-unknown

# ── Wasm developer tools ──
RUN cargo install cargo-component wasm-tools wasm-pack

# ── Wasm runtimes ──
RUN curl -sSf https://wasmtime.dev/install.sh | sh
ENV PATH="/home/$USERNAME/.wasmtime/bin:${PATH}"

# ── Neovim (latest stable from GitHub releases) ──
RUN curl -sL https://github.com/neovim/neovim/releases/latest/download/nvim-linux-x86_64.tar.gz \
    | sudo tar xz -C /opt \
    && sudo ln -s /opt/nvim-linux-x86_64/bin/nvim /usr/local/bin/nvim

# ── Naviscripts (dotfiles applied at container build) ──
# The repo is cloned at build time; install.sh copies configs into $HOME.
# At runtime, the mount at ~/naviscripts will shadow this, keeping it fresh.
RUN git clone https://github.com/navicore/naviscripts.git /home/$USERNAME/naviscripts \
    && cd /home/$USERNAME/naviscripts && ./install.sh \
    && mkdir -p /home/$USERNAME/.vim/swapfiles /home/$USERNAME/.undodir

# ── Validation ──
RUN rustup show \
    && wasmtime --version \
    && cargo component --version \
    && nvim --version | head -1

ENV SHELL=/bin/zsh

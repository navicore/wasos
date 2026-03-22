# WasOS Architecture

Monorepo for Rust+Wasm developer enablement: from devcontainer to distro.

## Repository Structure

```
wasos/
├── .devcontainer/        # Devcontainer: toolchain prototype + personal dev env
│   ├── Dockerfile        # Debian Sid base with Rust/Wasm toolchain
│   └── devcontainer.json # Container config, dotfile mounts
├── docs/
│   ├── ARCHITECTURE.md   # This file
│   └── design/           # Design docs for major decisions
├── metapackages/         # (future) Debian metapackage definitions
│   ├── wasos-dev-tools/  # Rust + Wasm toolchain metapackage
│   └── wasos-desktop/    # COSMIC + stripped systemd metapackage
├── live-build/           # (future) live-build config for ISO generation
├── iso-tests/            # (future) QEMU-based boot and smoke tests
└── ci/                   # (future) Nightly build and test pipelines
```

## Progression

1. **Devcontainer** — validate toolchain choices, use daily
2. **Metapackages** — formalize toolchain as installable .deb packages
3. **Live-build config** — produce bootable ISO
4. **CI pipeline** — nightly builds, automated testing
5. **Installer** — custom install experience (deferred)

## Key Decisions

- **Debian Sid upstream** — rolling, no Ubuntu layer
- **Stripped systemd** — PID 1 and service management only; mask resolved, homed, oomd, user record extensions
- **COSMIC desktop** — Wayland-native, Rust-based
- **Naviscripts integration** — personal dotfiles applied via install.sh, bind-mounted in devcontainer for live updates

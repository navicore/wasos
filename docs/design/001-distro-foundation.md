# Design: WasOS Distro Foundation

## Intent

Create a Debian-based Linux distribution shipping the COSMIC Wayland desktop with a Rust/WebAssembly-first developer toolchain. Use systemd as init but aggressively strip non-essential systemd components to maintain a minimal, user-controlled system. The goal is learning distro engineering end-to-end while producing something opinionated enough to stand on its own.

## Constraints

- **Debian upstream only** — no Ubuntu layer, no Snap
- **Stripped systemd, not no-systemd** — keep PID 1 and service management, reject scope creep (no resolved, no homed, no user record extensions, no age verification)
- **No version releases initially** — rolling builds, nightly ISO CI, stabilize later
- **Solo maintainer** — every decision must be sustainable at hobby pace (2-4 sessions/week)
- **Out of scope for now:** custom installer, ARM support, production use

## Approach

### Phase 1: Bootable ISO

- `live-build` config targeting Debian Sid
- COSMIC DE packages from System76 repos (or rebuilt locally)
- Minimal systemd: mask `systemd-resolved`, `systemd-homed`, `systemd-oomd`, `systemd-networkd`; use `elogind` if feasible, otherwise logind only
- Plain `/etc/resolv.conf` for DNS, `NetworkManager` for connectivity
- Boot to COSMIC Wayland session

### Phase 2: Developer Toolchain

- Metapackage `wasos-dev-tools` pulling in:
  - `rustup`, stable + nightly toolchains
  - `wasm-pack`, `cargo-component`, `wit-bindgen`, `wasm-tools`
  - `wasmtime`, `wasmedge`
  - WASI SDK
- Metapackage `wasos-orchestration` pulling in:
  - `containerd` with `runwasi` shim
  - `spin` or equivalent wasm app server
- Post-boot first-run validation script: confirm toolchains work, report any issues

### Phase 3: CI and Maintenance

- GitHub Actions (or similar) nightly ISO build
- Automated boot test in QEMU (does it reach the login screen?)
- Automated toolchain smoke test (does `cargo component build` produce a valid wasm component?)
- Upstream change monitoring: watch Debian Sid, COSMIC releases, wasm tooling releases

### Phase 4: Installer (deferred)

- Evaluate `distinst` / `cosmic-installer` forkability
- Custom frontend with hardware validation and dev-environment preview
- Only pursue after phases 1-3 are stable

## Domain Events

- **Upstream Debian package update** -> triggers nightly rebuild -> red/green CI status
- **COSMIC release** -> manual review session -> rebuild/patch COSMIC packages
- **Wasm tooling release** -> update metapackage pins -> smoke test
- **CI failure** -> triage session (supervised, not automated)

## Checkpoints

1. `live-build` produces a bootable ISO that reaches a COSMIC Wayland session in QEMU
2. `systemctl list-units` shows no resolved/homed/oomd — only init, journald, logind, dbus, NetworkManager
3. `rustup show` returns valid toolchain, `wasmtime --version` works, `cargo component new hello && cargo component build` succeeds
4. CI builds ISO nightly without manual intervention
5. Distro survives a Debian Sid update cycle without manual emergency fixes (the real test)

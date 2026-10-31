# EcoDroid

> Run Android games on Apple silicon Macs — and keep them running in the
> background at a fraction of the CPU, GPU and memory cost.

**Status:** pre-alpha. The repository is being bootstrapped; nothing is ready
to download yet. See [Roadmap](#roadmap).

## What it is

EcoDroid is an open-source macOS app that runs Android apps and games in a
lightweight virtual machine on Apple silicon (Hypervisor.framework, AOSP,
gfxstream + Metal). It is built on top of [Avo](https://github.com/nizwar/avo)
(Apache-2.0).

What EcoDroid adds is an **Eco Mode** aimed at long-running games such as
MMORPGs that have a built-in auto-battle or auto-farm feature:

- When you are not watching the game, EcoDroid lowers the render rate step by
  step (for example 30 → 15 → 10 FPS) and skips presenting frames to a hidden
  window.
- It limits the virtual CPU budget while making sure the game keeps progressing.
- It returns to full performance the moment you come back. If a profile breaks
  a game, it falls back to normal mode automatically.
- Per-game profiles remember which optimisations are safe for each title.

Success is measured as **energy per unit of in-game progress**, not only as a
lower CPU percentage.

## What it is not

- EcoDroid **does not hide itself from games** and does not try to get around
  emulator detection, anti-cheat or hardware attestation.
- It does not modify game files and does not include bots or macros.
- Many games' terms of service forbid emulators or virtual machines. Make sure
  you check the terms of every game you run. Following them is your
  responsibility.

## Requirements (planned)

- Apple silicon Mac (M1 or newer) running macOS 15 or later
- About 10 GB of free disk space for the Android image

## Building

The build instructions will be added once the Avo core has been imported and
audited (see [docs/security-audit.md](docs/security-audit.md)). The planned
toolchain is Xcode 26+, XcodeGen, Rust (stable), CMake and Ninja.

## Roadmap

1. Repository, CI and review process ✅
2. Security audit of the upstream Avo code
3. Import Avo, rebrand, build on Apple silicon
4. Resource monitor (CPU, GPU, RAM, FPS, energy)
5. Eco Mode v1: adaptive FPS, hidden-window present skipping, vCPU budget
6. Per-game compatibility profiles and auto-recovery
7. Signed and notarised release

## Contributing

Contributions are welcome. Please read [CONTRIBUTING.md](CONTRIBUTING.md)
first. In short: branch off `develop`, use Conventional Commits, and open a
pull request against `develop`. Only the maintainer merges into `main`.

Report security issues privately as described in [SECURITY.md](SECURITY.md).

## License

EcoDroid is licensed under the [Apache License 2.0](LICENSE). It is derived
from Avo; see [NOTICE](NOTICE) for attributions. "Avo" is the name of the
upstream project and is not used as EcoDroid branding. Android is a trademark
of Google LLC.

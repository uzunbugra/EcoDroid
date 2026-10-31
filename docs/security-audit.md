# Upstream security audit (Avo)

| | |
|---|---|
| Upstream | [nizwar/avo](https://github.com/nizwar/avo) |
| Commit audited | `6e1e77dc71a22f1369cc0f9aa09b23cbc884e7de` (2026-10-31) |
| Submodule | `vendor/avo_kit` at `e6d4a0a1938c6828c0d31b33d77947b37d60984c` |
| Date | 2026-10-31 |
| Method | Static review only. Nothing was built or run. |

## Decision

**Use, with the conditions below.** We found no malicious behaviour: no
telemetry, no hidden network endpoints, no credential or keychain access, no
obfuscated code. Two things need work before a public release. The update
channel points at the upstream author, and ten helper binaries are embedded
in the source as Base64 and need to be rebuilt from source.

## Context

- The repository was created on 2026-10-16. Of its 325 commits, 317 are by
  one author (`nizwar`). The project is young and has had little outside
  review.
- Licences: Apache-2.0, with BSD-3-Clause for `engine/hvf`. Third-party
  components are listed in its NOTICE and are under permissive licences
  (MIT, Apache-2.0, 0BSD, OFL).

## Findings

| # | Area | Finding | Risk | Action in EcoDroid |
|---|---|---|---|---|
| 1 | Auto-update | Sparkle checks `github.com/nizwar/avo/releases/latest/download/appcast.xml` once a day, and accepts updates signed with the upstream `SUPublicEDKey`. If we kept this, a fork would install the upstream author's builds. | **High** if kept | Replace `SUFeedURL` and `SUPublicEDKey` with our own, or turn updates off until we sign releases. |
| 2 | Embedded binaries | 10 helpers are stored as Base64 in `host/AvoKit/Sources/AvoDaemonCore/*Helper.swift`: 5 dex/APK files (clipboard, fingerprint, links, macstate, widgets) and 5 arm64 `.so` files (GLES, Vulkan, GPU profile, gfxstream sync). We decoded and inspected them. The dex files contain only `com.avodroid.*` classes and `org.json`. The `.so` files contain no network, socket or `exec` symbols; they only load Android graphics libraries. However, a binary blob cannot be reviewed the way source can, and an outside PR (#4) has already changed one. | Medium | Rebuild every helper from `guest/*` with `make helpers` and replace the blobs. Add a CI check that rebuilds them and compares the result. |
| 3 | Persistence | The runtime daemon is installed as a LaunchAgent in `~/Library/LaunchAgents` with `RunAtLoad` and `KeepAlive`, so it starts at login. This is documented, and it is visible under Login Items › Allow in the Background. | Low | Keep it. Rename the label, consider starting it only while the app is open, and document how to uninstall it. |
| 4 | Root | Optional Magisk root (Developer settings). The user turns it on. It downloads Magisk v30.7 from its official GitHub release, checked against a SHA-256 hash. Root is off by default. | Low | Keep it off by default. Don't advertise it. |
| 5 | Device identity | "Identify as" profiles change `Build.*` to imitate real phones (Pixel, OnePlus, …). Avo keeps identifying itself through `ro.boot.avo.*` and the `com.avodroid.feature.MAC` feature, and does not try to pass attestation. | Low (policy) | Keep the honest default and the self-identification. Never remove the self-identification. |
| 6 | Entitlements | `avo-vmm`: `hypervisor`, `audio-input`, `cs.disable-library-validation` (needed to load the gfxstream dylib). App and daemon: camera, microphone and location, used to bridge them into Android. Nothing unexpected. | Low | Review `disable-library-validation` once the renderer is bundled inside the signed app. |
| 7 | Build-time downloads | `build-gfxstream.sh` downloads MoltenVK and checks its SHA-256. `fetch-cuttlefish.sh` downloads AOSP images from `androidbuild-pa.googleapis.com` / `ci.android.com` using a hardcoded Google API key. `bridge-agent/build.sh` downloads the protobuf jar from Maven Central. `build-bluetooth.sh` clones Google's rootcanal. No `curl \| sh`, no `sudo`. | Low | Pin hashes for the protobuf jar and the rootcanal commit. Let the API key be overridden with `AVO_BUILD_API_KEY` (already supported). |
| 8 | Guest agent | The privileged system app has broad platform permissions (install/delete packages, inject input, capture display, read the clipboard in the background). It is used for app windows, input and clipboard bridging. It listens only on AF_VSOCK, the channel between the host and the VM, and not on the network. | Low | Keep it. Make sure every new guest capability goes through vsock only. |
| 9 | Network endpoints | The source only reaches GitHub (releases and API for Google Apps packages, Magisk, Sparkle), Google's Android build servers and `google.com/android/uncertified`. There is no analytics or crash-reporting SDK. | None | — |
| 10 | Dependencies | Rust: 26 crates, including `smoltcp`, `mio`, `libc`, `thiserror` and `heapless`, all well known. Swift: `swift-protobuf` and Sparkle. | Low | `cargo deny` runs in CI. |

## Still to do

- [ ] Run `cargo deny check` and `cargo audit` once Rust is installed locally.
- [ ] Rebuild the helpers from source and compare them with the embedded
      blobs (needs the Android SDK and NDK).
- [ ] Run the first build with network monitoring (`nettop` or Little
      Snitch) and confirm traffic goes only to the hosts listed in finding 9.
- [ ] Replace the Sparkle feed and key (finding 1) before any release build.

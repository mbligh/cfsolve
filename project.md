# cfsolve (Debian 13 build fork)

## Requirements

- Preserve upstream source and the vendored Rust and TLS patches from [chris124567/cfsolve](https://github.com/chris124567/cfsolve).
- Produce a Linux `amd64` CLI executable (`challenge`) built **inside Debian 13**, not merely on an Ubuntu runner.
- Use the toolchain pinned by upstream in `rust-toolchain.toml` and the existing `Cargo.lock` (`cargo build --locked --release --bin challenge`).
- Publish a downloadable build on pushes to `master` and on manual workflow dispatch. For `v*` tag pushes, also attach it to a GitHub Release.
- Include the `profiles/` tree (the CLI requires the profile JSON and recorded browser data at runtime), plus README and licenses, with the executable.

## Implementation

`.github/workflows/debian13.yml` uses the `debian:13` container on a GitHub-hosted `amd64` runner. It installs the C/C++/CMake/libclang/Git dependencies needed by the patched BoringSSL crates, installs the Rust toolchain declared by upstream, builds the `challenge` CLI, checks `--help`, creates a portable `.tar.gz`, records its SHA-256, and uploads both as GitHub Actions artifacts. A separate job publishes them as release assets on version tags. Runtime execution does not require Docker or Rust.

## Build and installation

From the fork on GitHub, open **Actions → Debian 13 binary**, and download the `cfsolve-debian13-amd64` artifact from a successful run. It contains `cfsolve-debian13-amd64.tar.gz` and `SHA256SUMS`.

On a Debian 13 amd64 host:

```sh
mkdir -p ~/cfsolve
cd ~/cfsolve
tar -xzf /path/to/cfsolve-debian13-amd64.tar.gz
./challenge --help
```

Use `--fingerprint profiles/safari-26-ios.json` from the extracted directory when running the CLI.

## Tracking upstream

This is a fork. To merge upstream code later, add the upstream remote and merge from `master` after reviewing conflicts:

```sh
git remote add upstream https://github.com/chris124567/cfsolve.git
git fetch upstream
git merge upstream/master
```

After any upstream update, validate GitHub Actions builds again. Upstream's pinned Rust version and vendored dependencies may change.

- 2026-11-27: Merge upstream `3d2f67ed56df29be6ceba1de60b7ab9077e554c8` (fix for new build), retaining the Debian 13 immutable binary release workflow. The APT package consumes the new release after it is published.

# CEF Automated Builds

Build and package CEF for:

- Windows x86_64
- Linux x86_64
- macOS x86_64
- macOS arm64

This repository uses GitHub Actions and supports two runner modes:

- `shared`: GitHub-hosted runners, for workflow validation
- `self-hosted`: your own runners, for persistent source cache and production builds

## Workflow

The workflow file is [`/.github/workflows/cef-build.yml`](./.github/workflows/cef-build.yml).

Manual trigger (`workflow_dispatch`) inputs:

- `runner_mode`: `shared` or `self-hosted`
- `cef_version`: CEF version/tag/branch (example: `132.3.0`, full tag, or `6533`)
- `publish_release`: whether to create/update GitHub Release and upload binaries

Scheduled trigger runs weekly.

`cef_version` resolution rules:

- numeric input like `6533`: treated as `--branch=6533`
- semantic version input like `145.0.26`: resolves latest matching CEF version from `cef-builds.spotifycdn.com/index.json`
- full CEF version input like `145.0.26+g6ed7554+chromium-145.0.7632.110`: resolves exactly
- `latest`: resolves to latest CEF version from the same index

For non-numeric input, workflow extracts:

- `--branch=<chromium branch>` (for example `7632`)
- `--checkout=<cef git short hash>` (for example `6ed7554`)

When `publish_release=true`:

- workflow creates/updates a release tag in the format `cef-<resolved-cef-version>`
- all platform artifacts are uploaded as `tar.bz2` bundles
- `cef_version` must be version (`latest`, semver-like, or full CEF version), not raw branch number

## Codec build flags (FFmpeg/H264/H265/AAC)

The workflow sets:

- `is_official_build=true`
- `proprietary_codecs=true`
- `ffmpeg_branding=Chrome`
- `enable_platform_hevc=true`
- `enable_hevc_parser_and_hw_decoder=true`

You can override all GN defines using repository variable `CEF_GN_DEFINES`.

Note: workflow uses `--build-target="cefsimple cefclient"` because `--client-distrib` requires `cefsimple`.

## Build acceleration (sccache + git cache)

The workflow enables `sccache` by default and injects `cc_wrapper="sccache"` into GN.
If `sccache` is unavailable on a runner, workflow automatically falls back to normal compilation (no cache) instead of failing.

It also configures `GIT_CACHE_PATH` to speed up `gclient`/`git` fetch.

Default fast-build GN flags:

- `symbol_level=0`
- `blink_symbol_level=0`
- `v8_symbol_level=0`
- `use_jumbo_build=true`

You can override these defaults with repository variable `CEF_FAST_GN_DEFINES`.

Optional repository variable:

- `SCCACHE_CACHE_SIZE` (default: `80G`)

## Self-hosted runner labels

Expected labels per matrix target:

- `self-hosted`, `cef`, `windows`, `x64`
- `self-hosted`, `cef`, `linux`, `x64`
- `self-hosted`, `cef`, `macos`, `x64`
- `self-hosted`, `cef`, `macos`, `arm64`

## Persistent cache directory (self-hosted)

Set repository variable `CEF_CACHE_ROOT` to a persistent path (for example `/data/cef-cache`).

If not set, workflow defaults to `${HOME}/cef-cache` on the runner.

`automate-git.py` will reuse this directory and maximize git/depot_tools cache hits across runs.

For best performance on self-hosted runners:

- put `CEF_CACHE_ROOT` on fast SSD/NVMe
- keep runner machine stable (same path reused over time)
- avoid cleaning `${CEF_CACHE_ROOT}` unless needed

Windows note:

- workflow enables `git config --global core.longpaths true`
- on shared Windows runner, cache root is moved to `${RUNNER_TEMP}/cef-cache` to shorten checkout paths

Linux note:

- workflow installs `libcups2-dev` and `pkg-config` to satisfy `cups-config` required by Chromium printing build files

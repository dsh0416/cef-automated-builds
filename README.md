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
- `cef_branch`: CEF branch number (example: `6533`)

Scheduled trigger runs weekly.

## Codec build flags (FFmpeg/H264/H265/AAC)

The workflow sets:

- `is_official_build=true`
- `proprietary_codecs=true`
- `ffmpeg_branding=Chrome`
- `enable_platform_hevc=true`
- `enable_hevc_parser_and_hw_decoder=true`

You can override all GN defines using repository variable `CEF_GN_DEFINES`.

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

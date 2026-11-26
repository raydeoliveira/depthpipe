---
name: depthpipe
description: >-
  Live 2D-to-3D stereo depth pipeline, Apple Silicon Metal DIBR synthesizer,
  bidirectional MV-HEVC Spatial Video converter CLI and MCP server, with a
  SteamOS Vulkan Z-buffer layer distributed as source. Use when probing Metal/MV-HEVC/ScreenCaptureKit
  capabilities, inspecting ISO BMFF Spatial Video boxes, looking up Game Depth
  Profiles, rendering 2D/RGB-D/pair frames to Full-SBS or lenticular PNGs,
  converting between MV-HEVC and Full-SBS/Half-SBS/TAB/anaglyph video, capturing live
  or simulated frames, or composing UNIX spatial pipes via --from-json -.
---

# Skill: DepthPipe (`depthpipe`)

## Install & Discovery

The v0.1.0 CLI and MCP bundle run on Apple Silicon with macOS 14 or later. The release archive also contains the Vulkan layer source for Linux and SteamOS.

```bash
brew install raydeoliveira/tap/depthpipe
depthpipe --describe
```

## Safe Probe, Inspect & Profile Workflow
1. Probe Apple Silicon Metal GPU, VideoToolbox MV-HEVC, ScreenCaptureKit TCC, and CoreML:
   ```bash
   depthpipe probe --json
   depthpipe probe --sim --json
   ```
2. Inspect an existing `.mov` / `.mp4` container for `hvcC`, `lhvC`, and `vexu` stereo boxes (read-only, free):
   ```bash
   depthpipe inspect spatial.mov --json
   ```
3. Look up deterministic Game Depth Profiles (`near_plane`, `far_plane`, `disparity_px`, `reversed_z`):
   ```bash
   depthpipe profile --app 1091500 --json
   depthpipe profiles --list --json
   ```

## Render, Convert & Capture Pipelines
- Preview with `--dry-run` before writing files:
  ```bash
  depthpipe render --input frame.png --output frame_sbs.png --out-layout full-sbs --dry-run --json
  ```
- Convert bidirectional MV-HEVC <-> Full-SBS `.mov` (guards `--input == --output` and odd dimensions):
  ```bash
  depthpipe convert --input spatial.mov --in-layout mv-hevc --out-layout full-sbs --output sbs.mov --json
  ```
- Capture deterministic simulation fixtures (hardware-free QA) or live ScreenCaptureKit (requires Screen Recording TCC):
  ```bash
  depthpipe capture --sim --fixture gradient-shapes --max-frames 3 --out-layout mv-hevc --output sim_mv.mov --json
  ```

## UNIX Composability (`--from-json -`)
Pipe upstream `edidsmith` `solved_mode` (`fits: true`) or `xrctl` `target_surface` into `depthpipe`, then forward `media_artifact` into `glassweave`:
```bash
edidsmith edid budget --width 3840 --height 1080 --hz 60 --link usbc-4lane --json \
  | depthpipe render --input frame.png --output frame_sbs.png --out-layout full-sbs --from-json - --json \
  | glassweave weave --output woven.png --from-json - --json
```
Upstream `{"ok":false,...}` or `solved_mode.fits == false` fails closed with exit `2`. Explicit CLI flags override piped JSON fields. In non-TTY pipelines, overwriting an existing `--output` file requires `--yes`.

## Native MCP Mode
Run `depthpipe mcp` over stdio (JSON-RPC 2.0, `protocolVersion: "2024-11-05"`) to invoke `probe_pipeline`, `lookup_depth_profile`, `inspect_media`, `render_stereo` (`dry_run: true` default), `convert_media` (`dry_run: true` default), and `capture_frames` (`dry_run: true` default).

## Exit Codes
- `0` ok | `1` runtime failure | `2` invalid usage / validation / bandwidth / upstream error | `3` Metal GPU or source unavailable | `4` Screen Recording TCC permission denied | `5` Pro license required (`depthpipe license activate --key <KEY>`) | `130` `SIGINT`/`SIGTERM`

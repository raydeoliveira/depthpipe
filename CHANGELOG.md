# Changelog

All notable changes to DepthPipe are documented in this file.

## [0.1.0] - 2026-11-21

### Added
- Clean-room Swift 6 Ports & Adapters core and macOS adapters (`MetalDepthEngine` runtime MSL DIBR & 8 stereo output weaves, VideoToolbox/AVFoundation bidirectional MV-HEVC <-> Full-SBS pipeline, pure-Swift ISO BMFF `hvcC`/`lhvC`/`vexu` inspector, `ScreenCaptureKit` TCC-gated capture + `SimCaptureAdapter`, `CoreMLDepthAdapter`, `ProfileCatalog` 13-game DB, and `PolarLicenseAdapter`).
- Clean-room C++20/GLSL SteamOS Vulkan Z-buffer implicit layer (`vulkan-layer/`) with `VkLayer_depthpipe_zbuf.json`, `shaders/dibr_zbuf.comp`, and ASan+UBSan unit/parity/regression tests.
- Modular `DepthPipeCLI` supporting `probe`, `inspect`, `profile`/`profiles`, `render`, `convert`, `capture`, `license`, `mcp` (stdio JSON-RPC 2.0 `2024-11-05`), `--from-json -` UNIX composability, non-TTY `--yes` overwrite guard, and `SIGINT`/`SIGTERM` exit `130` handling.
- Trinity packaging and manifests (`SKILL.md`, `server.json`, `packaging/homebrew/depthpipe.rb`, `packaging/polar/PRODUCT_LISTING.md`, `packaging/llms.txt`, `scripts/package_release.sh`).

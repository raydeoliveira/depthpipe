# Polar.sh Product Listing — DepthPipe Pro (macOS & SteamOS Vulkan Layer)

> Use this copy when creating the DepthPipe Pro product in your Polar.sh dashboard (`polar.sh`).
> Enable Polar's **License Keys** benefit (prefix: `DP3D`, activation limit: `3` devices per buyer, expiration: perpetual).

---

## Product Title
**DepthPipe Pro — Apple Silicon Metal DIBR, MV-HEVC Spatial Video Pipeline & SteamOS Z-Buffer Layer**

## Price & Billing Model
- **$29 USD** — One-time purchase (Perpetual License, v0.x updates included, up to 3 devices).

## Tagline
Synthesize Full-SBS, lenticular, and MV-HEVC Spatial Video on Apple Silicon Metal and VideoToolbox, capture live screens or simulation fixtures, and pair with a clean-room SteamOS Vulkan Z-buffer implicit layer.

## Storefront Description

Converting 2D/RGB-D content, stereo pairs, and MV-HEVC Spatial Video between Apple Vision Pro / macOS and 3D monitors or AR glasses normally requires bloated GUI transcoders or FFmpeg wrappers.

**DepthPipe** provides a single native Apple Silicon Swift 6 CLI and stdio MCP server with runtime-compiled Metal DIBR shaders, native VideoToolbox MV-HEVC encode/decode, `ScreenCaptureKit` capture, optional CoreML depth inference, a 13-game depth profile catalog, and a clean-room C++20/GLSL SteamOS Vulkan Z-buffer implicit layer (`vulkan-layer/`).

### What Is Included in Pro ($29 Perpetual)
- **Metal DIBR & Stereo Weaving (`render`)**: Convert `2d`, `rgbd`, `pair`, `full-sbs`, `half-sbs`, or `tab` PNGs into `full-sbs`, `half-sbs`, `tab`, `row-interlaced`, `col-interlaced`, `checkerboard`, `lenticular`, or `dubois-anaglyph` via runtime-compiled MSL compute kernels.
- **Bidirectional MV-HEVC Video Pipeline (`convert`)**: Transcode MV-HEVC `.mov` (`hvcC`/`lhvC`/`vexu`) to/from Full-SBS, Half-SBS, TAB, interlaced, or anaglyph `.mov`/`.mp4` while preserving audio and enforcing `--input == --output` and even-dimension guards.
- **Live ScreenCaptureKit & Fixture Capture (`capture`)**: Capture display or window 32BGRA buffers through `ScreenCaptureKit` (gated by `CGPreflightScreenCaptureAccess()`, exiting `4` if denied) or deterministic `--sim --fixture` pipelines into PNG or MV-HEVC `.mov`.
- **SteamOS Vulkan Z-Buffer Layer (`vulkan-layer/`)**: Clean-room C++20 dispatch layer + `shaders/dibr_zbuf.comp` GLSL compute shader and `VkLayer_depthpipe_zbuf.json` implicit layer manifest for Linux/SteamOS.
- **Native AI-Agent MCP Server (`depthpipe mcp`)**: Stdio JSON-RPC 2.0 (`2024-11-05`) with `--describe`, `--json`, `--from-json -` UNIX piping, and `dry_run: true` defaults on mutating tools.

### Requirements & Honest Limits
- **macOS Binary**: Requires macOS 14.0+ on Apple Silicon (`arm64`) with a Metal GPU; VideoToolbox MV-HEVC hardware encode/decode depends on Apple Silicon macOS capabilities.
- **SteamOS Vulkan Layer**: `vulkan-layer/` builds `libVkLayer_depthpipe_zbuf.so` on Linux/SteamOS; on macOS it is verified via `clang++ -std=c++20` ASan+UBSan unit, CPU reference parity, and regression tests against minimal Khronos ABI types (not loaded into a macOS MoltenVK ICD).
- **Depth Models & Media**: No large CoreML weights or FFmpeg binaries are bundled. Pass `--model <path.mlpackage>` for CoreML ANE depth inference, or omit `--model` to use the built-in Metal synthetic depth estimator.
- **Activation & Privacy**: Activates via Polar.sh (`depthpipe license activate --key <KEY>`), caches locally with `0600` permissions and 14-day offline grace, and makes zero background network calls.

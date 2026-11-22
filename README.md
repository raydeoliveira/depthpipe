# DepthPipe

Live 2D-to-3D stereo depth pipeline, runtime-compiled Metal DIBR synthesizer, bidirectional MV-HEVC Spatial Video converter, and SteamOS Vulkan Z-buffer implicit layer CLI and stdio MCP server (`877,424` bytes / `857 KB` stripped macOS `arm64` binary, `7.00 ms` min / `9.19 ms` median `--version` cold start on Apple M4 Max, zero external Swift dependencies).

## What It Does and Does Not Do

| Capability | Status | Notes / Requirements |
|---|---|---|
| Hardware & media probing (`probe`, `inspect`) | Supported | Reports Metal GPU, VideoToolbox MV-HEVC encode/decode, ScreenCaptureKit TCC, and CoreML status; parses ISO BMFF `hvcC`, `lhvC`, and `vexu` boxes |
| Game Depth Profile catalog (`profile`, `profiles`) | Supported | 13 deterministic profiles (`near_plane`, `far_plane`, `disparity_px`, `convergence_px`, `depth_invert`, `reversed_z`) keyed by Steam AppID or bundle ID |
| Metal DIBR & stereo weaving (`render`) | Supported | Runtime-compiled MSL compute kernels (`rgbd_dibr_synthesize`, `split_packed_stereo`, `shift_stereo_convergence`, `weave_stereo_pair`) across 8 PNG output layouts (`full-sbs`, `half-sbs`, `tab`, `row-interlaced`, `col-interlaced`, `checkerboard`, `lenticular`, `dubois-anaglyph`) |
| Bidirectional MV-HEVC video (`convert`) | Supported | Audio-preserving VideoToolbox MV-HEVC `.mov` <-> Full-SBS/Half-SBS/TAB/interlaced/anaglyph `.mov`/`.mp4` |
| Live capture & sim fixtures (`capture`) | Supported | `ScreenCaptureKit` 32BGRA capture (gated by `CGPreflightScreenCaptureAccess()`, exit `4` on denial) plus `--sim --fixture` (`gradient-shapes`, `checker-fg`, `portrait-card`, `desktop-sim`, `desktop-pillar`) |
| SteamOS Vulkan Z-buffer layer (`vulkan-layer/`) | Linux source + macOS unit-tested | Builds `libVkLayer_depthpipe_zbuf.so` + `VkLayer_depthpipe_zbuf.json` + `shaders/dibr_zbuf.comp` on Linux/SteamOS; verified on macOS via `clang++ -std=c++20` ASan+UBSan unit, CPU parity, and regression tests against minimal Khronos ABI types (not a macOS MoltenVK ICD layer) |
| Neural weights & FFmpeg | Not bundled | Omit `--model` for built-in Metal synthetic depth, or supply `--model <path.mlpackage>` for CoreML ANE inference; zero bundled FFmpeg binaries |

## Install and Uninstall

```bash
# Build from source (Commercial edition)
cd projects/depthpipe


# Or via Homebrew tap
brew install raydeoliveira/tap/depthpipe
```

Build and test the SteamOS Vulkan layer on Linux/SteamOS (or run its C++20 ASan+UBSan tests on macOS):

```bash
make -C projects/depthpipe/vulkan-layer clean all test
```

Linux Vulkan layer environment variables: `DEPTHPIPE_VK_ENABLE=1`, `DEPTHPIPE_VK_DISABLE=1`, `DEPTHPIPE_BYPASS=1`, `DEPTHPIPE_NEAR_PLANE`, `DEPTHPIPE_FAR_PLANE`, `DEPTHPIPE_DISPARITY_PX`, `DEPTHPIPE_CONVERGENCE_PX`, `DEPTHPIPE_REVERSED_Z`, `DEPTHPIPE_DEPTH_INVERT`.

To uninstall:

```bash
brew uninstall depthpipe
rm -rf ~/.config/depthpipe
```

## Quick Start (Human + `--json` Agent Usage)

```bash
$ depthpipe --version
depthpipe 0.1.0 (commercial, arm64)

$ depthpipe probe --json
{"data":{"capture_backend":"ScreenCaptureKit","coreml_available":true,"gpu_available":true,"gpu_backend":"Metal","gpu_name":"Apple M4 Max","metal_available":true,"mv_hevc_decode_supported":true,"mv_hevc_encode_supported":true,"screen_capture_permitted":true},"error":null,"ok":true,"schema_version":1}

$ depthpipe profile --app 1091500 --json
{"data":{"aliases":["com.cdprojektred.cyberpunk2077","cyberpunk2077"],"app_id":"1091500","convergence_px":0,"depth_invert":false,"disparity_px":26,"far_plane":1000,"id":"cyberpunk-2077","matched":true,"near_plane":0.1,"notes":"REDengine 4 reversed-Z float32 depth buffer; moderate convergence for indoor HUD.","reversed_z":true,"title":"Cyberpunk 2077"},"error":null,"ok":true,"schema_version":1}

$ depthpipe capture --sim --fixture gradient-shapes --max-frames 2 --out-layout full-sbs --output /tmp/dp_cap.png --dry-run --json
{"data":{"depth_source":"metal-synthetic","elapsed_ms":0,"fixture_name":"gradient-shapes","frames_captured":0,"frames_processed":0,"gpu_name":"Apple M4 Max","media_artifact":{"eye_height":240,"eye_width":320,"frames_processed":0,"out_layout":"full-sbs","output_height":240,"output_path":"/tmp/dp_cap.png","output_width":640},"source":{"bundle_id":"com.depthpipe.sim","height":240,"kind":"sim","source_id":1,"title":"Simulated Spatial Desktop (320x240)","width":320},"used_sim":true},"error":null,"ok":true,"schema_version":1}

# UNIX composability: chain edidsmith/xrctl -> depthpipe -> glassweave via --from-json -
$ edidsmith edid budget --width 3840 --height 1080 --hz 60 --link usbc-4lane --json \
  | depthpipe render --input frame.png --output frame_sbs.png --out-layout full-sbs --from-json - --json \
  | glassweave weave --output woven.png --from-json - --json
```

## Safety and Rollback

- **Screen Recording TCC gating**: `capture` without `--sim` or `--fixture` calls `CGPreflightScreenCaptureAccess()` first and exits `4` with `System Settings > Privacy & Security > Screen Recording` instructions when denied.
- **Overwrite and dimension guards**: `render`, `convert`, and `capture` refuse overlapping input/output or colliding `--output`/`--depth-out` paths (including resolved symlinks and case-variants) with exit `2`; HEVC 4:2:0 video outputs reject odd or `< 2x2` dimensions with exit `2`.
- **Non-TTY `--yes` guard**: In non-interactive pipelines, overwriting an existing `--output` or `--depth-out` file fails closed (`exit 2`) unless `--yes` is passed; writing to a new output file in a pipe works without `--yes`.
- **Atomic writes & `--from-json -` fail-closed**: Media files finalize through temporary staging files, and upstream `ok: false` or `solved_mode.fits == false` (`BANDWIDTH_EXCEEDED`) aborts immediately with exit `2`.
- **Signals**: Installs `sigaction` (`sa_flags = 0`) for `SIGINT` and `SIGTERM`, exiting `130`.

## Exit Codes and CLI Reference

- `0`: Success
- `1`: Runtime, GPU, capture, or media failure
- `2`: Invalid usage, NaN/Inf/odd dimensions, non-TTY overwrite without `--yes`, `BANDWIDTH_EXCEEDED`, or upstream `--from-json` `ok: false`
- `3`: Metal GPU or capture source unavailable
- `4`: Screen Recording TCC permission denied
- `130`: `SIGINT` or `SIGTERM` received

Run `depthpipe --help` for human CLI usage, `depthpipe --describe` for the JSON capability manifest, or `depthpipe mcp` for stdio JSON-RPC 2.0 (`2024-11-05`) MCP tools (`probe_pipeline`, `lookup_depth_profile`, `inspect_media`, `render_stereo`, `convert_media`, `capture_frames`).

## License and Support

- Security policy: [SECURITY.md](SECURITY.md)
- Agent skill & MCP manifest: [SKILL.md](SKILL.md), [server.json](server.json)

# DepthPipe

Render stereo images from a 2D image and a supplied depth map, or convert Apple MV-HEVC spatial video and side-by-side video. DepthPipe runs locally on Apple Silicon Macs with macOS 14 or later as a command-line tool and MCP server.

[Downloads and SHA-256 checksums](https://github.com/raydeoliveira/depthpipe/releases/tag/v0.1.0) · [macOS MCP bundle](https://github.com/raydeoliveira/depthpipe/releases/download/v0.1.0/depthpipe-0.1.0-macos-arm64.mcpb) · [DepthPipe Pro](#license-and-support)

## Preview a depth map to stereo render

![A rendered left-right stereo image showing colored shapes shifted between the two views](docs/depth-map-example/stereo.png)

DepthPipe rendered these two 320 × 192 views from one [color image](docs/depth-map-example/color.png) and its [grayscale depth map](docs/depth-map-example/depth.png). The example gives nearby shapes brighter values in the map, so they shift farther between views. The free preview uses the files you supply and requires no model downloads, screen recording or 3D display.

```sh
brew install raydeoliveira/tap/depthpipe
work_dir=$(mktemp -d)
cd "$work_dir"
sample=https://raw.githubusercontent.com/raydeoliveira/depthpipe/main/docs/depth-map-example
curl --fail --location --silent --show-error --remote-name-all "$sample/color.png" "$sample/depth.png"
depthpipe render --input color.png --depth depth.png --output stereo.png --out-layout full-sbs --dry-run --json
```

The release reports `"output_width":640`, `"output_height":192` and `"depth_inferred":false`. The preview writes no `stereo.png`. To write the side-by-side PNG, activate a Pro key and rerun the command without `--dry-run`.

## Inspect an MV-HEVC spatial video

Check that a clip has both stereo views before importing it into another tool. The free command reads the container. Run this in the same sample directory.

```sh
curl --fail --location --silent --show-error --remote-name-all "$sample/spatial.mov"
depthpipe inspect spatial.mov --json
```

The 320 × 240 synthetic clip reports `"is_mv_hevc":true`, `"has_left_eye":true` and `"has_right_eye":true`. You can also [download the sample clip](docs/depth-map-example/spatial.mov) and substitute a local file in the command.

## What It Does and Does Not Do

| Capability | Status | Notes / Requirements |
|---|---|---|
| Hardware & media probing (`probe`, `inspect`) | Supported | Reports Metal GPU, VideoToolbox MV-HEVC encode/decode, ScreenCaptureKit TCC, and CoreML status; parses ISO BMFF `hvcC`, `lhvC`, and `vexu` boxes |
| Game Depth Profile catalog (`profile`, `profiles`) | Supported | 13 deterministic profiles (`near_plane`, `far_plane`, `disparity_px`, `convergence_px`, `depth_invert`, `reversed_z`) keyed by Steam AppID or bundle ID |
| Stereo image rendering (`render`) | Free planning. Full side-by-side export requires Pro. | Use a supplied grayscale depth map or provide a Core ML model for inference. Outputs include full/half side-by-side, top/bottom, interlaced, checkerboard, lenticular and Dubois anaglyph PNG. |
| MV-HEVC spatial video converter (`convert`) | Pro | Convert between MV-HEVC `.mov` and packed stereo video such as full side-by-side. |
| Screen capture (`capture`) | Pro for live capture | Live capture requires Screen Recording permission. Use `--sim --fixture gradient-shapes --dry-run` to preview in simulation. |
| Linux and SteamOS Vulkan layer | Source scaffold | The layer records depth-buffer bookkeeping and forwards frames unchanged. The supported renderer and converter run on Apple Silicon. |
| Depth model and runtime | A supplied map works offline | The macOS binary is 877,424 bytes. Inference from a photo requires your own Core ML model via `--model <path.mlpackage>`. The built-in synthetic depth is for pipeline testing. |

## Install and uninstall

Use the Homebrew command above, or extract the macOS archive after checking its checksum from the [release page](https://github.com/raydeoliveira/depthpipe/releases/tag/v0.1.0). Run `depthpipe --version` to verify the install.

For Linux or SteamOS development, the macOS archive also ships the experimental layer source. Extract the archive, then run `make -C depthpipe-0.1.0-macos-arm64/vulkan-layer all test` to build and test the scaffold. It collects depth-buffer information and leaves displayed frames unchanged.

Use `brew uninstall raydeoliveira/tap/depthpipe` to remove the Homebrew package. If you extracted the archive without installing it, remove the extracted directory.

## Use DepthPipe with an agent

With the CLI installed, add the [agent instructions](SKILL.md) to your project for depth-map previews and spatial video inspection.

```sh
npx skills add raydeoliveira/depthpipe
```

Use the [macOS MCP bundle](https://github.com/raydeoliveira/depthpipe/releases/download/v0.1.0/depthpipe-0.1.0-macos-arm64.mcpb) for an MCPB-compatible client. For a client configured for stdio, use `depthpipe mcp`. Run `depthpipe --describe` for command metadata and `depthpipe profiles --json` for the built-in game-depth profiles. [Glama listing](https://glama.ai/mcp/servers/raydeoliveira/depthpipe)

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
- `5`: Pro license required
- `130`: `SIGINT` or `SIGTERM` received

Run `depthpipe --help` for human CLI usage, `depthpipe --describe` for the JSON capability manifest, or `depthpipe mcp` for stdio JSON-RPC 2.0 (`2024-11-05`) MCP tools (`probe_pipeline`, `lookup_depth_profile`, `inspect_media`, `render_stereo`, `convert_media`, `capture_frames`).

## License and support

The [commercial license](LICENSE) covers free inspection and preview tools plus the Pro rendering and conversion features. [Buy DepthPipe Pro ($29)](https://buy.polar.sh/polar_cl_pM3CJwYQ9ilUfvBxDVLf0iMGnTBJltLssCFFj1YSozm) · [Four-tool bundle ($49)](https://buy.polar.sh/polar_cl_KTtCY3iQC4bzjqp93aC6LCog8UZqzZzuDgkAs3BvF74)

[Report an issue](https://github.com/raydeoliveira/depthpipe/issues) · [Security policy](SECURITY.md) · [Software catalog](https://www.raymonddeoliveira.com/software#depthpipe)

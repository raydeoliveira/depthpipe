# Security Policy — DepthPipe

## Reporting a Vulnerability
Report security vulnerabilities privately via GitHub Security Advisories on `https://github.com/raydeoliveira/depthpipe`.

## Security & Privacy Model
- **Zero external dependencies**: Built strictly against Apple system frameworks (`Metal`, `VideoToolbox`, `AVFoundation`, `ScreenCaptureKit`, `CoreML`, `ImageIO`).
- **Explicit capture permissions**: Checks `CGPreflightScreenCaptureAccess()` before opening live capture streams and exits `4` when permission is denied.
- **File overwrite safety**: Rejects `--input == --output` (including resolved symlinks) and writes outputs atomically via temporary files.

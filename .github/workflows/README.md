# opx-probe

Fast media file inspector for broadcast engineers.

Binary-only release. No source published.

## Download

**[→ Latest release](https://github.com/karhu-tv/opx-probe/releases/latest)**

| Platform | Requirements |
|----------|-------------|
| Linux x86_64 | Ubuntu 22.04+ · `sudo apt install ffmpeg` |
| Windows x86_64 | Windows 10+ · FFmpeg DLLs bundled |

## Usage

```
opx-probe <file> [--json] [--house <format>]

Options:
  --json            JSON output
  --house <format>  Preflight against house format

House formats:
  1080p50   1080p25   1080p2997   1080p30
  1080p24   1080p2398 1080i50     1080i5994
  720p50    720p25

Exit codes:
  0  Clean (preflight passed if --house given)
  1  Preflight failures
  2  Probe error
```

## Examples

```bash
# Inspect a file
opx-probe programme.mxf

# Preflight against house format
opx-probe clip.mp4 --house 1080p50

# JSON output
opx-probe clip.mp4 --json | jq .video.frame_rate_label

# Scriptable preflight
opx-probe clip.mp4 --house 1080p50 && echo "OK" || echo "FAIL"
```

## Sample output

```
opx-probe  programme_ep3.mxf
────────────────────────────────────────────────────────────────────────
Duration   48:22.080

Video
  Codec      MPEG-2 Video  (id 12)
  Frame rate 25 fps
  Resolution 1920×1080
  Frames     72552

Audio
  Codec      PCM s24le  (id 86020)
  Sample rate 48000 Hz
  Channels   2

MXF
  Audio streams  4
  AS-11 sidecar  present
  Programme      Series Title S01E03
  Audio layout   stereo_and_dual_mono

────────────────────────────────────────────────────────────────────────
Preflight  ✓  PASSED
```

---

[karhu.tv](https://karhu.tv) · otso@karhu.tv

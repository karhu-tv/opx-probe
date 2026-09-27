# opx-probe

Fast media file inspector for broadcast engineers.

Binary-only release. No source published.

## Download

**[→ Latest release](https://github.com/karhu-tv/opx-probe/releases/latest)**

| Platform | Requirements |
|----------|-------------|
| Linux x86_64 | Ubuntu 24.04 LTS · `sudo apt install ffmpeg` — FFmpeg 6.x, `libavformat.so.60`. 22.04 ships FFmpeg 4.4 and 26.04 ships FFmpeg 8; neither is supported. Other distributions need FFmpeg 6.x from their own sources |
| Windows x86_64 | Windows 10 (tested Jul 2026) · FFmpeg DLLs bundled |

## Usage

```
opx-probe <file> [--json] [--house <format>]
opx-probe --version

Options:
  --json            JSON output
  --house <format>  Preflight against a target format
  --version, -V     Print version and exit

Target formats (--house):
  1080p50   1080p25   1080p2997   1080p30
  1080p24   1080p2398 1080i50     1080i5994
  720p50    720p25

Preflight takes only the frame rate from the target format. Resolution and
scan type are not compared, so 1080p50 and 720p50 give the same verdict, as do
1080p25, 1080i50 and 720p25 (0.2.0).

Exit codes:
  0  Clean (preflight passed if --house given)
  1  Preflight failures
  2  Probe error
```

## Examples

```bash
# Inspect a file
opx-probe programme.mxf

# Preflight against a target format (frame rate, sample rate, start offset, VFR — not resolution or scan)
opx-probe clip.mp4 --house 1080p50

# JSON output
opx-probe clip.mp4 --json | jq .video.frame_rate_label

# Scriptable preflight
opx-probe clip.mp4 --house 1080p50 && echo "OK" || echo "FAIL"
```

## Sample output

Captured 2026-09-27 with opx-probe 0.2.0 on a 10 s DNxHD bars file, quoted verbatim.
The earlier sample here was not the tool's output (it lacked the per-stream Duration
lines every report prints) and was withdrawn.

```
$ opx-probe bars_dnxhd_1080p25_10s.mxf --house 1080p25
opx-probe  bars_dnxhd_1080p25_10s.mxf
────────────────────────────────────────────────────────────────────────
Duration   0:10.000

Video
  Codec      Avid DNxHD/DNxHR  (id 99)
  Frame rate 25 fps
  Resolution 1920×1080
  Duration   0:10.000

Audio
  Codec      PCM s24le  (id 65548)
  Sample rate 48000 Hz
  Channels   1
  Duration   0:10.000

MXF
  Audio streams  1
  AS-11 sidecar  not found

────────────────────────────────────────────────────────────────────────
Preflight  ✓  PASSED
```

A note on the start-offset check: it flags any stream that does not start at time
zero and reports it as an edit list. MPEG-TS has no edit lists, but its streams
rarely start at zero, so all 9 `.ts` files we tried on 2026-09-27 tripped it.

---

[karhu.tv](https://karhu.tv) · otso@karhu.tv

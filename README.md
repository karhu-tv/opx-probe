# opx-probe

Fast media file inspector for broadcast engineers.

Binary-only release. Source code is not published. One exception, stated so nobody has to find it: from 16 Apr to 28 Jul 2026 this repository carried one source file of the command-line front-end, committed by mistake under `.github/workflows/`. It was removed from the tree on 28 Jul 2026. It remains in the repository's history and in the “Source code” archive of the v0.1.0 release. Stated 29 Sep 2026.

## Download

**[→ Latest release](https://github.com/karhu-tv/opx-probe/releases/latest)**

| Platform | Requirements |
|----------|-------------|
| Linux x86_64 | Ubuntu 24.04 LTS · `sudo apt install ffmpeg` — FFmpeg 6.x, `libavformat.so.60`. 22.04 ships FFmpeg 4.4 and 26.04 ships FFmpeg 8; neither is supported. Other distributions need FFmpeg 6.x from their own sources |

**Windows x86_64: withdrawn.** The Windows zip bundled FFmpeg 6.1.1 DLLs (gyan.dev full_build-shared), licensed GPL version 3 or later. The zip did not include that licence's text or an offer of the FFmpeg source, which the licence requires. Windows downloads withdrawn 29 Sep 2026 for this reason. The Linux build does not bundle FFmpeg.

## Known issues in 0.2.0

*Known issues added 29 Sep 2026.* These items are also listed on the [latest release](https://github.com/karhu-tv/opx-probe/releases/latest) and on [karhu.tv](https://karhu.tv/tools.html#opx-probe-issues).

- **A false “will reject” line.** The line `Rytmi v1 will reject this file` is wrong: Rytmi does not reject such a file. The line prints whenever any stream starts at a non-zero time: every MPEG-TS file we tried, and the Matroska and FLV files we tried on 29 Sep 2026, including a fresh MKV encode, which `--house` also fails.
- **Several files: only the last is probed.** Given several files, 0.2.0 probes only the last and ignores the others without a message, so `opx-probe *.mxf --house 1080i50` can print PASSED and exit 0 over files it never opened. Pass one file per run: `for f in *.mxf; do opx-probe "$f" --house 1080i50 || echo "FAIL $f"; done`.
- **VFR.** VFR is flagged only when FFmpeg reports no frame rate at all. A variable-rate file that declares a nominal rate, such as a typical phone or screen capture or a stream whose rate changes part-way, passes preflight.
- **Truncated files.** Preflight reads the container header, not the media. A truncated file whose header survives passes: on 29 Sep 2026 the first 23 % of a faststart MP4 reported its full 10 s duration and PASSED.
- **Edit lists.** The start-offset check does not read MP4/MOV edit lists. A stream-copy trim that carries a real edit list passes, reports `has_edit_list: false`, and counts the frames its edit list discards.
- **MXF sidecar.** The sidecar is found by file name only: `<name>.xml`, `<name>_shim.xml`, or a `shim.xml` in the same folder, so one `shim.xml` is reported against every MXF beside it. Any well-formed XML counts as a sidecar. AFD, subtitles and rights expiry are read from field names that are not AS-11's, and a missing subtitles field reads as false. AS-11 metadata embedded in the MXF is not read.
- **Exit code 101.** opx-probe ends in a Rust panic with exit code 101, outside the documented 0/1/2, when a file name is not valid UTF-8, and intermittently when stdout is closed early (`| head`).
- **Publisher name.** `--help` names a limited company that is not the publisher, and the archive's README.txt runs the trading name together as one word. The publisher is Bear Media Services LTD, trading as Karhu TV. The `--help` text is fixed in source; the published archive is unchanged.
- **Supported systems.** The README.txt inside the Linux archive says "Ubuntu 24.04+". Of Ubuntu releases, only 24.04 LTS is supported, as this page states.

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
rarely start at zero, so all 9 `.ts` files we tried on 27 Sep 2026 tripped it, as
did the Matroska and FLV files we tried on 29 Sep 2026.

---

[karhu.tv](https://karhu.tv) · otso@karhu.tv

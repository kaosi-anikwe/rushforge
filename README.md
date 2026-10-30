# rushforge

Transcode Samsung phone footage (or any H.264/H.265 clips) into DaVinci Resolve-ready intermediates — GPU-accelerated, one command, no fuss.

**Output format:** DNxHR SQ (visually lossless for 8-bit footage) in `.mov` with 48 kHz PCM audio. Resolve on Linux imports both without complaint.

---

## Why this exists

DaVinci Resolve on Linux has no AAC decoder and can choke on HEVC from phone cameras. The Samsung A56 (and most modern Androids) shoot H.265/HEVC with AAC audio. This script batch-converts your footage to an intermediate codec Resolve handles natively, using your NVIDIA GPU for fast decoding.

---

## Requirements

- Fedora Linux (or any distro with `dnf`)
- NVIDIA GPU with CUDA (tested on Quadro T1000)
- `ffmpeg` with NVENC/CUDA support

```bash
# Swap the stub ffmpeg-free for the full build (requires RPM Fusion)
sudo dnf swap ffmpeg-free ffmpeg --allowerasing

# Confirm CUDA/NVENC support
ffmpeg -encoders 2>&1 | grep nvenc
```

---

## Install

```bash
# Symlink so edits to the repo take effect immediately
ln -sf "$(pwd)/rushforge" ~/.local/bin/rushforge

# ~/.local/bin should already be in $PATH on Fedora; verify with:
echo $PATH | grep -o "$HOME/.local/bin"
```

---

## Usage

```bash
# Transcode all clips in a folder
rushforge /path/to/footage

# Run in the current directory
rushforge
```

Output goes into `resolve_ready/` inside the input folder. A log file is written to `resolve_ready/rushforge.log`.

Before transcoding each file, rushforge probes its codec with `ffprobe`. If the file is already DNxHR + PCM 48kHz it is skipped — no stamp files or sidecar metadata needed. This means you can safely re-run rushforge on the same folder (e.g. after adding new clips) and it will only process what actually needs transcoding. If a previous run was interrupted mid-transcode, the partial output is detected as not matching the target format and the file is re-transcoded cleanly.

---

## GNOME Files (Nautilus) right-click integration

Install both scripts so they appear in the right-click menu:

```bash
mkdir -p ~/.local/share/nautilus/scripts
ln -sf "$(pwd)/nautilus-script/Transcode for Resolve" ~/.local/share/nautilus/scripts/"Transcode for Resolve"
ln -sf "$(pwd)/nautilus-script/Copy Videos by Date" ~/.local/share/nautilus/scripts/"Copy Videos by Date"
```

### Transcode for Resolve

Right-click a folder containing video files → **Scripts** → **Transcode for Resolve**. Transcodes everything in the folder to DNxHR SQ + PCM and drops the output into `resolve_ready/`. A notification fires on start and finish.

### Copy Videos by Date

Designed for phone ingestion. Right-click your DCIM/Camera folder → **Scripts** → **Copy Videos by Date**.

A dialog asks for a time period:
- **Today / Yesterday / This Week** — one click, no typing
- **Custom Range** — two calendar pickers for start and end date

A folder picker then asks where to copy. Files land in a dated subfolder automatically (e.g. `~/Videos/2026-10-30/` or `~/Videos/2026-10-24_to_2026-10-30/`). JPEGs and other non-video files are ignored.

**Typical workflow:**
1. Mount phone, open DCIM/Camera in Files
2. Right-click → **Copy Videos by Date** → pick period → pick destination
3. Open the destination folder, right-click → **Transcode for Resolve**
4. Import `resolve_ready/` into DaVinci Resolve

---

## Storage note

DNxHR SQ is an intermediate codec — not delivery. Expect **40–60 GB per hour** of 1080p footage. Keep your original `.mp4` files until the project is exported and published, then delete the `resolve_ready/` folder to reclaim space.

---

## Tested with

- Samsung Galaxy A56 (4K30 and 1080p60, HEVC)
- Fedora 40+, NVIDIA Quadro T1000, driver 550+
- DaVinci Resolve 19 (Linux)

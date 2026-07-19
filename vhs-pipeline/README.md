# VHS Archive AI Analysis Pipeline

Analyze hours of archive footage with Gemini. Export structured markers to Final Cut Pro.

Built for documentary filmmakers working with analog archives (VHS, Hi8, miniDV, Super8).
No cloud storage of footage. Gemini only processes temporary proxy files, then deletes them.

---

## What it does

```
Archive footage (any format)
        │
        ▼
  Proxy creation     ffmpeg → 480p, 1fps, burned timecodes
        │
        ├──→  Blind pass (optional)     No framework. What does a monteur see cold?
        │
        ├──→  Deep analysis             Your analytical grid. Behavioral patterns.
        │
        ├──→  FCPXML with colored markers  →  Final Cut Pro 12
        └──→  Markdown report              →  Reference document
```

Each analyzed segment becomes a **colored marker** in FCP12:

| Color | Meaning |
|-------|---------|
| Red (to-do) | Strong interest, must review |
| Orange (chapter) | Narrative structure moment |
| Blue | Standard marker |
| Green | Glitch / artifact |

The marker note contains: scene description, behavioral observation, editorial interpretation,
exact transcription if speech is present.

---

## Architecture

```
Archive footage          Proxy (480p, 1fps, BITC)
USB / local / NAS   →   created locally, kept
READ ONLY               ~400MB / hour of footage

         │
         ▼  Gemini Files API (temporary, deleted after each call)
         │
    Phase 1    gemini-2.5-flash        Pre-scan 3min sample per video
         │
    Phase 2a   gemini-3-flash-preview  Blind pass (--blind flag)
         │
    Phase 2b   gemini-3-flash-preview  Deep analysis (your grid)
         │
    Phase 3    gemini-2.5-pro          Optional deep pass on "strong" clips only
         │                            + LOW resolution (3h/call in 1M context)
         │
    Phase 4    local, no API           FCPXML per video + Markdown report
         │
    Phase 5    gemini-2.5-pro          Corpus synthesis (text only, no video)
```

---

## Model selection

Three-tier system. Use what you need:

| Phase | Model | Input | Output | Notes |
|-------|-------|-------|--------|-------|
| Pre-scan | gemini-2.5-flash | $0.30/1M | $2.50/1M | Stable, cheap |
| Main analysis | gemini-3-flash-preview | $0.50/1M | $3.00/1M | Best quality/cost |
| Selected clips | gemini-2.5-pro | $1.25/1M | $10-15/1M | 2M context + low_res |

**Context window vs video length** (at 1fps proxy):

| Resolution | Tokens/sec | 1M context | 2M context |
|-----------|-----------|------------|------------|
| LOW | ~92 | ~3h video | ~6h video |
| MEDIUM | ~175 | ~1.5h | ~3h |
| HIGH | ~263 | ~1h | ~2h |

**Cost estimate (12h of archive footage, all phases): ~$15-22**

---

## Deployment

Three supported environments. All produce identical output.

```
┌─────────────────────────────────────────────────────────────────────┐
│                        DEPLOYMENT OPTIONS                           │
│                                                                     │
│  ┌──────────────┐    ┌──────────────────┐    ┌──────────────────┐  │
│  │   macOS      │    │   Mac Mini M4    │    │   Windows WSL2   │  │
│  │   Desktop    │    │   Headless       │    │   / Linux        │  │
│  │              │    │   (recommended)  │    │                  │  │
│  │  Direct run  │    │  SSH + tmux      │    │  WSL terminal    │  │
│  │  USB local   │    │  USB via SSH     │    │  NAS / network   │  │
│  └──────┬───────┘    └───────┬──────────┘    └────────┬─────────┘  │
│         │                   │                         │            │
│         └───────────────────┴─────────────────────────┘            │
│                             │                                       │
│                    ┌────────▼────────┐                              │
│                    │  Gemini API     │                              │
│                    │  Files API      │                              │
│                    │  (cloud, temp)  │                              │
│                    └─────────────────┘                              │
└─────────────────────────────────────────────────────────────────────┘
```

### Mac Mini M4: headless server (recommended for long runs)

Best setup: run on a dedicated machine via SSH. The pipeline can take hours,
tmux keeps it alive if the connection drops.

```
Your laptop / iPad
       │
       │  SSH / mosh (Tailscale or local network)
       ▼
Mac Mini M4  ──→  USB drive with footage  ──→  Gemini API
       │
       └── tmux session "analysis"
               │
               ├── vhs_analyzer.py running in foreground
               └── progress.json updated after each video
```

```bash
# On Mac Mini: one-time setup
brew install ffmpeg python
pip install google-genai
echo 'export GEMINI_API_KEY="your-key"' >> ~/.zshrc

# Every run: from any device
ssh user@mac-mini-ip
tmux new -s analysis              # or: tmux attach -s analysis
ls /Volumes/                      # verify USB is mounted
python vhs_analyzer.py /Volumes/USB/footage
# Ctrl+B then D to detach, run continues in background
```

### macOS Desktop

```bash
# Install
brew install ffmpeg
pip install google-genai
export GEMINI_API_KEY="your-key"   # https://aistudio.google.com/apikey

# Run
python vhs_analyzer.py /Volumes/USB/footage
```

### Windows (WSL2) or Linux

```bash
# Ubuntu / WSL2
sudo apt install ffmpeg python3-pip
pip install google-genai
export GEMINI_API_KEY="your-key"

# Footage path examples:
# WSL2: /mnt/d/footage/              (D:\ drive)
# Linux: /media/user/USB/            (mounted USB)
# NAS:  /mnt/nas/archive/            (network mount)

python vhs_analyzer.py /mnt/d/footage
```

```powershell
# PowerShell (native Windows, no WSL)
# 1. Download ffmpeg: https://ffmpeg.org/download.html → add ffmpeg\bin to PATH
pip install google-genai
[System.Environment]::SetEnvironmentVariable("GEMINI_API_KEY", "your-key", "User")
python vhs_analyzer.py D:\footage\archive
```

---

## Usage

```bash
# Dry run: list videos without processing
python vhs_analyzer.py /footage --dry-run

# Phase 1 only (pre-scan, cheap)
python vhs_analyzer.py /footage --phase 1

# Full pipeline with blind pass
python vhs_analyzer.py /footage --blind

# Full pipeline without blind pass
python vhs_analyzer.py /footage

# Resume interrupted run
python vhs_analyzer.py /footage --resume

# Export only (JSON already present)
python vhs_analyzer.py /footage --phase 4

# Retry failed videos only
python vhs_analyzer.py /footage --retry-failed
```

---

## Customizing the analysis

Edit `prompts.py` → `SYSTEM_DEEP_ANALYSIS`. Define:
1. What you're looking for (behavioral signals, themes, patterns)
2. Few-shot examples of what each signal looks like
3. Keep the JSON schema, it drives the FCPXML export

**Two-pass approach (recommended to avoid confirmation bias):**

- **Blind pass** (`--blind`): No framework. "What does an editor who knows nothing see?"
  → Segments marked `strong` here become candidates for the deep pass
- **Deep pass**: Your specific analytical grid, applied to everything
- Results are merged by `tc_start`, the blind observation appears in the FCP marker note

---

## Integration with Final Cut Pro 12 and Jumper

**FCP12:** File → Import → XML → your `.fcpxml`

Each video gets its own project with colored markers. The marker note contains the full
Gemini interpretation, visible in the FCP12 inspector when hovering a marker.

**[Jumper](https://jumper.com)** indexes FCP marker notes for semantic search.
Our notes contain the full analysis text → Jumper can search across all analyzed footage
by concept, not just by filename.

Pipeline: `Gemini → FCPXML markers → FCP12 → Jumper semantic search`

---

## Why 1fps proxies?

VHS at 25fps with HIGH resolution = ~263 tokens/frame.
At 1fps proxy + LOW resolution: ~92 tokens/second.
A 2h VHS: 1.7M tokens → 66K tokens. Cost ÷25.

Family archive footage is mostly static scenes with fixed camera.
1fps loses nothing relevant for behavioral observation.

---

## Why burned-in timecodes?

Gemini reads the timecode directly from the image (top left corner).
This means accurate timestamps across chunk boundaries, no post-processing to reconcile
relative times from different chunks.

---

## Storage

```
Originals          READ ONLY. USB / external / NAS. Never modified.
proxies/           480p 1fps with BITC. ~400MB/hour. Kept as reference.
raw_json/          Gemini output per video. ~2MB/video. The key deliverable.
blind_analysis/    Blind pass output. Merged into raw_json during export.
fcpxml/            One .fcpxml per video.
reports/           One .md report per video.
SYNTHESIS.md       Corpus overview (phase 5).
```

No footage is stored in the cloud. The Files API is a temporary upload buffer,
the pipeline deletes every file after each API call.

---

## License

See [LICENSE.md](../LICENSE.md) at the repo root (PolyForm Noncommercial 1.0.0).

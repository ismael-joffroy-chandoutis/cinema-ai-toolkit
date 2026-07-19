<picture><source media="(prefers-color-scheme: dark)" srcset="banner.svg"><img src="banner-light.svg" alt="Cinema AI Toolkit" width="100%"></picture>

**English** · [Français](README.fr.md)

# Cinema AI Toolkit

**Documentary filmmaker's AI toolkit — voice repair, VHS analysis, FCP workflows, OCR for handwritten documents.**

Built for real productions, not demos.

<img src="monde.jpg" alt="cinema-ai-toolkit" width="100%">

[![License: PolyForm Noncommercial 1.0.0](https://img.shields.io/badge/License-PolyForm_Noncommercial-yellow.svg)](LICENSE.md)
![Python](https://img.shields.io/badge/Python-3.10+-blue?style=flat-square&logo=python)
![Vision LLM](https://img.shields.io/badge/Vision_LLM-API-4285F4?style=flat-square)
![Final Cut Pro](https://img.shields.io/badge/Final_Cut_Pro-12-purple?style=flat-square)

---

## Author

[Ismaël Joffroy Chandoutis](https://ismaeljoffroychandoutis.com) — filmmaker and artist. **César 2022**, Cannes, IDFA, Hot Docs, Ars Electronica.

These tools were built during active production of feature documentaries. They solve problems I actually have.

---

## What's inside

Four independent tools, one repo. Each has its own README with full documentation.

```
cinema-ai-toolkit/
├── voice-pipeline/    # Voice repair for documentary subjects
├── vhs-pipeline/      # VHS/Hi8/miniDV analysis with a vision LLM → FCP markers
├── fcp-workflow/       # Final Cut Pro auteur workflow + agentic scripting integration
├── prison-writing/    # OCR + graphological analysis for handwritten documents
└── ETHICS.md          # Ethics statement for documentary AI tools
```

---

## Tools

### [Voice Pipeline](voice-pipeline/)

Voice direction for documentary filmmakers working with non-professional subjects. The subject's voice is authentic and irreplaceable — this pipeline repairs the performance, not the voice.

```
RAW RECORDINGS → Denoise → Isolate → Diarize → Segment → Normalize
                          → Enhance → Inpaint → Fix delivery → Voice clone ready
```

**Stack:** DeepFilterNet 3, Resemble Enhance, ElevenLabs, Sesame CSM, Chatterbox

---

### [VHS Pipeline](vhs-pipeline/)

Analyze hours of analog archive footage (VHS, Hi8, miniDV, Super8) with a vision LLM. Export colored markers directly to Final Cut Pro 12.

```
Archive footage → ffmpeg proxy → vision LLM analysis → FCPXML colored markers → FCP 12
```

| Marker Color | Meaning |
|------|---------|
| Red | Strong interest — must review |
| Orange | Narrative structure moment |
| Blue | Standard marker |
| Green | Glitch / artifact |

**Stack:** Vision LLM (fast), ffmpeg, FCPXML

---

### [FCP Workflow](fcp-workflow/)

A-to-Z workflow for auteur cinema with Final Cut Pro, agentic scripting, and open source tools.

**Philosophy:** Hack everything. Local first. Plain text is king. Automate the boring parts.

---

### [Prison Writing Analyzer](prison-writing/)

OCR + graphological analysis + data mining from photographs of handwritten documents and prison correspondence. Built for documentary research.

For each image:
1. **Transcription** — word-for-word OCR with illegibility markers
2. **Classification** — letter, prison email, psychiatric report, legal document...
3. **Graphological analysis** — pressure, slant, regularity, legibility
4. **Data mining** — persons, locations, dates, themes, emotional state

**Stack:** fast vision LLM (primary) + high-tier vision LLM (recheck low-confidence)

| Model | Handwriting accuracy | Cost |
|-------|---------------------|------|
| Vision LLM (fast) | ~90% | $0.50/1M tokens |
| GPT-5 | ~90%+ | $$$ |
| Tesseract (local) | ~64% | Free |

---

## Ethics

All tools in this repo follow the ethics statement in [ETHICS.md](ETHICS.md). Documentary AI tools operate on real people's lives. The technology is never neutral.

---

## Install

Each tool has its own requirements. See the README in each directory.

```bash
# Voice pipeline
cd voice-pipeline && pip install -r requirements.txt

# VHS pipeline
cd vhs-pipeline && pip install -r requirements.txt
```

---

**Consolidated from:** cinema-voice-pipeline · vhs-ai-pipeline · [fcp-auteur-workflow](https://github.com/ismael-joffroy-chandoutis/fcp-auteur-workflow) · prison-writing-analyzer

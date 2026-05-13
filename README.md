# Batch Video Generator

<p align="center">
  <b>AI Batch Video Generation — Claude Code Skill</b><br>
  Three-mode orchestration for Dreamina CLI, Dreamina International Web, and Grok browser
</p>

<p align="center">
  <img src="https://img.shields.io/badge/claude_code-skill-8A2BE2" alt="Claude Code Skill">
  <img src="https://img.shields.io/badge/license-MIT-green" alt="License MIT">
  <img src="https://img.shields.io/badge/dreamina-v2.0-blue" alt="Dreamina 2.0">
</p>

---

## Overview

A **Claude Code skill** for orchestrating batch AI video generation across multiple tools. Automatically selects the right engine based on your content and handles the full lifecycle: submission, monitoring, download, and failover retry.

### Three Modes

| Mode | Purpose | Engine |
|------|---------|--------|
| **A — Batch Generation** | Multiple independent prompt+image → video | Dreamina CLI / Grok browser |
| **B — Storyboard Script** | Scripted multi-scene → sequential video → concatenation | Dreamina CLI |
| **C — Dreamina International Web** | Web UI automation via Playwright CDP | dreamina.capcut.com |

---

## Features

- **Multi-engine orchestration** — Automatically routes between Dreamina CLI, Grok, and Dreamina International Web
- **Smart failover** — Detects failures and retries with the alternative tool
- **Prompt inversion** — Optional semantic inversion for failed tasks via the prompt-optimizer skill
- **Storyboard-to-video** — Parse formatted scripts, generate sequentially, concatenate with ffmpeg
- **Automated monitoring** — Polls generation status at configurable intervals, downloads on completion
- **Credit management** — Checks available credits before starting
- **Web automation** — Full Playwright CDP-based control of dreamina.capcut.com

---

## Quick Start

### Prerequisites

- Claude Code CLI
- [Dreamina CLI](https://dreamina.cn/) installed at `~/.local/bin/dreamina`
- (Optional) Google Chrome with `--remote-debugging-port=9222` for Mode C
- (Optional) Python 3 + Playwright for Grok automation

### Installation

**Option 1: Install via Claude Code plugin**

```bash
claude plugin add batch-video-generator
```

**Option 2: Manual install**

```bash
cd ~/.claude/skills/
git clone https://github.com/527998482-jpg/batch-video-generator.git
```

---

## Usage Guide

### Mode A: Batch Generation

Use when you have a **list of prompts ± reference images** and want to generate multiple independent videos.

**Workflow:**
1. Choose tool (Dreamina or Grok) and confirm specs
2. Submit all tasks in parallel
3. Monitor until completion (polling every 60s)
4. Download results + retry failures with the other tool

### Mode B: Storyboard Script

Use when you have a **formatted script** with numbered scenes and want one combined video.

```
01｜Waterfall cascading, mist rising｜Push up from low-angle mountain view
02｜Sunlight piercing canopy, god rays｜Pan right across the forest
```

**Workflow:**
1. Parse storyboard (delimiter formats: ｜, |, ||, tabs)
2. Submit tasks one-at-a-time (Dreamina concurrency limit: 1)
3. Download each as it completes, named 01.mp4, 02.mp4...
4. Concatenate all via ffmpeg stream copy

### Mode C: Dreamina International Web

Use when your account works on the **web UI** but not via CLI (e.g., artisan accounts). Requires Chrome with CDP.

```bash
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome \
  --remote-debugging-port=9222 --user-data-dir=/tmp/chrome_profile &
```

Then the skill will:
1. Connect via Playwright `connect_over_cdp()`
2. Navigate to dreamina.capcut.com
3. Upload reference image, set model/duration/ratio
4. Submit and monitor until video ready

---

## File Structure

```
batch-video-generator/
├── SKILL.md                # Claude Code skill definition
├── README.md               # This documentation
├── dreamina_monitor.py     # CLI polling + download automation
└── LICENSE                 # MIT
```

---

## Configuration Reference

### Dreamina CLI Parameters

| Parameter | Options | Default | Notes |
|-----------|---------|---------|-------|
| model_version | seedance2.0_vip / seedance2.0 / seedance2.0fast | seedance2.0 | _vip = highest quality |
| ratio | 1:1 / 3:4 / 16:9 / 4:3 / 9:16 / 21:9 | 16:9 | Depends on subcommand |
| duration | 4–15 (integer seconds) | 5 | Per model limit may vary |
| resolution | 720p | 720p | seedance2.0 series only |
| subcommand | multimodal2video / text2video / image2video | — | Depends on input type |

### Dreamina International Web Parameters

| Parameter | Options |
|-----------|---------|
| Model | Dreamina Seedance 2.0 / 2.0 Fast / 1.5 Pro / 1.0 / Sora 2 |
| Duration | 4s / 8s / 10s / 12s |
| Aspect Ratio | 21:9 / 16:9 / 4:3 / 1:1 / 3:4 / 9:16 |
| Resolution | 720P / 1080P |
| Credit Cost (8s) | ~152 (Fast) / ~192 (Standard) |

### Grok Parameters

| Parameter | Options |
|-----------|---------|
| Duration | 6s / 10s |
| Resolution | 480p / 720p |
| Aspect Ratio | 2:3 / 3:2 / 1:1 / 9:16 / 16:9 |

---

## Tool Selection Guide

| Scenario | Recommended Tool | Reason |
|----------|----------------|--------|
| Creative content, clean themes | Dreamina CLI | Faster, lower cost, batch submission |
| Sensitive / historical / military content | Grok | Lighter content moderation |
| CLI auth failing (artisan accounts) | Dreamina International Web | Web UI alternative |
| Multi-scene storyboard with concatenation | Dreamina CLI | Sequential submission + ffmpeg concat |
| Maximum quality | Dreamina seedance2.0_vip | Highest quality output |

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| `ExceedConcurrencyLimit` | Dreamina allows 1 task at a time. Submit sequentially. |
| `post-TNS check did not pass` | Content blocked. Try Grok instead or invert prompts. |
| WebSocket connection failed | Ensure Chrome is running with `--remote-debugging-port=9222`. |
| Queue stuck at same position | Wait — Dreamina queue advances ~65 positions/min. |
| Download timeout | Retry the download; Grok can use CDP base64 pipe fallback. |

---

## License

MIT — see [LICENSE](LICENSE).

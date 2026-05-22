---
name: mulerouter
description: Generates images, videos, audio, speech, and music using MuleRouter or MuleRun multimodal APIs. Text-to-Image, Image-to-Image, Text-to-Video, Image-to-Video, Reference-to-Video, Video-to-Video, video editing (VACE, keyframe interpolation), Text-to-Speech, Text-to-Music. Use when the user wants to generate, edit, or transform images, videos, speech, or music using AI models like Wan2.6, Veo3, Nano Banana Pro, Sora2, Midjourney, Kling V3, Kling V3 Omni, MiniMax Speech 2.8, MiniMax Music 2.5.
compatibility: Requires Python 3.10+, uv, MULEROUTER_API_KEY env var, and one of MULEROUTER_BASE_URL or MULEROUTER_SITE env var. Needs network access to api.mulerouter.ai or api.mulerun.com. The API key is sent in Authorization headers to the configured endpoint.
homepage: https://github.com/openmule/mulerouter-skills
allowed-tools: Bash(uv run *) Bash(uv sync *) Bash(npx mulerouter*) Bash(npm install*) Read
metadata:
  clawdbot:
    requires:
      env: ["MULEROUTER_API_KEY"]
      env_one_of: ["MULEROUTER_BASE_URL", "MULEROUTER_SITE"]
      bins: ["uv", "python3"]
    primaryEnv: "MULEROUTER_API_KEY"
    install: "uv sync"
    files: ["scripts/*", "models/*", "core/*", "pyproject.toml"]
---

# MuleRouter API

Generate images and videos using MuleRouter or MuleRun multimodal APIs.

## Required Environment Variables

This skill requires the following environment variables to be set before use:

| Variable | Required | Description |
|----------|----------|-------------|
| `MULEROUTER_API_KEY` | **Yes** | API key for authentication ([get one here](https://www.mulerouter.ai/app/api-keys?utm_source=github_claude_plugin)) |
| `MULEROUTER_BASE_URL` | **Yes*** | Custom API base URL (e.g., `https://api.mulerouter.ai`). Takes priority over SITE. |
| `MULEROUTER_SITE` | **Yes*** | API site: `mulerouter` or `mulerun`. Used if BASE_URL is not set. |

*At least one of `MULEROUTER_BASE_URL` or `MULEROUTER_SITE` must be set.

The API key is included in `Authorization: Bearer` headers when making network calls to the configured API endpoint.

**If any of these variables are missing, the scripts will fail with a configuration error.** Check the Configuration section below to set them up.

## Configuration Check

Before running any commands, verify the environment is configured:

### Step 1: Check for existing configuration

Run the built-in config check script:

```bash
uv run python -c "from core.config import load_config; load_config(); print('Configuration OK')"
```

If this prints "Configuration OK", skip to **Step 3**. If it raises a `ValueError`, proceed to Step 2.

### Step 2: Configure if needed

**If the variables above are not set**, ask the user to provide their API key and preferred endpoint.

**Create a `.env` file** in the skill's working directory:

```env
# Option 1: Use custom base URL (takes priority over SITE)
MULEROUTER_BASE_URL=https://api.mulerouter.ai
MULEROUTER_API_KEY=your-api-key

# Option 2: Use site (if BASE_URL not set)
# MULEROUTER_SITE=mulerun
# MULEROUTER_API_KEY=your-api-key
```

**Note:** `MULEROUTER_BASE_URL` takes priority over `MULEROUTER_SITE`. If both are set, `MULEROUTER_BASE_URL` is used.

**Note:** The skill only loads variables prefixed with `MULEROUTER_` from the `.env` file. Other variables in the file are ignored.

**Important:** Do NOT use `export` shell commands to set credentials. Use a `.env` file or ensure the variables are already present in your shell environment before invoking the skill.

### Step 3: Using `uv` to run scripts

The skill uses `uv` for dependency management and execution. Make sure `uv` is installed and available in your PATH.

Run `uv sync` to install dependencies.

## Quick Start

### 1. List available models

```bash
uv run python scripts/list_models.py
```

### 2. Check model parameters

```bash
uv run python models/alibaba/wan2.6-t2v/generation.py --list-params
```

### 3. Generate content

**Text-to-Video:**
```bash
uv run python models/alibaba/wan2.6-t2v/generation.py --prompt "A cat walking through a garden"
```

**Text-to-Image:**
```bash
uv run python models/alibaba/wan2.6-t2i/generation.py --prompt "A serene mountain lake"
```

**Image-to-Video:**
```bash
uv run python models/alibaba/wan2.6-i2v/generation.py --prompt "Gentle zoom in" --image "https://example.com/photo.jpg" #remote image url
```
```bash
uv run python models/alibaba/wan2.6-i2v/generation.py --prompt "Gentle zoom in" --image "/path/to/local/image.png" #local image path
```

**Text-to-Speech (voice-id is required):**
```bash
uv run python models/minimax/speech-2.8-turbo/generation.py --prompt "Hello world, welcome to the future of AI." --voice-id "Charming_Lady"
```

**Text-to-Music:**
```bash
uv run python models/minimax/music-2.5/generation.py --prompt "[verse]\nHello world\n[chorus]\nLa la la"
```

## Image Input

For image parameters (`--image`, `--images`, etc.), **prefer local file paths** over base64.

```bash
# Preferred: local file path (auto-converted to base64)
--image /tmp/photo.png

--images ["/tmp/photo.png"]
```

Local file paths are validated before reading: only files with recognized image extensions (`.png`, `.jpg`, `.jpeg`, `.gif`, `.bmp`, `.webp`, `.tiff`, `.tif`, `.svg`, `.ico`, `.heic`, `.heif`, `.avif`) are accepted. Paths pointing to sensitive system directories or non-image files are rejected. Valid image files are converted to base64 and sent to the API, avoiding command-line length limits that occur with raw base64 strings.

## Workflow

1. Check configuration: verify `MULEROUTER_API_KEY` and either `MULEROUTER_BASE_URL` or `MULEROUTER_SITE` are set
2. Install dependencies: run `uv sync`
3. Run `uv run python scripts/list_models.py` to discover available models
4. Run `uv run python models/<path>/<action>.py --list-params` to see parameters
5. Execute with appropriate parameters
6. Parse output URLs from results

## Model Selection

When listing models, each model's **tags** (e.g., `[SOTA]`) are displayed by default next to its name. Tags help identify model characteristics at a glance — for example, `SOTA` indicates a state-of-the-art model.

You can also filter models by tag using `--tag`:
```bash
uv run python scripts/list_models.py --tag SOTA
```

**If you are unsure which model to use**, present the available options to the user and let them choose. Use the `AskUserQuestion` tool (or equivalent interactive prompt) to ask the user which model they prefer. For example, if the user asks to "generate an image" without specifying a model, list the relevant image generation models with their tags and descriptions, and ask the user to pick one.

## Tips
1. For an image generation model, a suggested timeout is 5 minutes.
2. For a video generation model, a suggested timeout is 15 minutes.
3. For TTS models (speech-2.8-hd/turbo), `--voice-id` is required. Use `--list-params` to see available voices, or refer to [MINIMAX_VOICES.md](references/MINIMAX_VOICES.md) for the full voice catalog.

## Seedance via npm CLI (newer models)

The Seedance 2.0 / 2.0-fast series (ByteDance video models) is invoked through the **npm-distributed `mulerouter` CLI**, not via `uv run python ...`. All other existing models continue to use the Python entry points above.

> ⚠ Currently seedance is only available on the subsystem site (`api.mulerun.com`). Add `--site mulerun` explicitly. The standalone site (`api.mulerouter.ai`) does not yet route these endpoints (404).

### Installation

```bash
npm install -g mulerouter
# Or run ad-hoc without installing:
npx -y mulerouter@latest --help
```

### Environment Variables

Reuses the same variables as the Python skill: `MULEROUTER_API_KEY` plus `MULEROUTER_SITE` (or `MULEROUTER_BASE_URL`). `.env` loading semantics are identical.

### Discover endpoints

```bash
mulerouter list --provider bytedance
mulerouter params bytedance/seedance-2.0/text-to-video
```

### Examples (6 endpoints × std/fast)

T2V — text-to-video:
```bash
mulerouter run bytedance/seedance-2.0/text-to-video \
  --site mulerun \
  --prompt "A cat walking through a snowy garden" \
  --resolution 1080p --duration 5

# fast variant (max 720p, does not accept camera_fixed/watermark)
mulerouter run bytedance/seedance-2.0-fast/text-to-video \
  --site mulerun \
  --prompt "A cat walking" --resolution 720p --duration 4
```

I2V — image-to-video (local paths are auto-converted to base64; optional last frame):
```bash
mulerouter run bytedance/seedance-2.0/image-to-video \
  --site mulerun \
  --prompt "Gentle zoom in" \
  --image /tmp/first.png \
  --last-frame-image /tmp/last.png
```

R2V — reference-to-video (multi-modal references; at least one of images/videos/audios required):
```bash
mulerouter run bytedance/seedance-2.0/reference-to-video \
  --site mulerun \
  --prompt "Cinematic montage" \
  --images '["/tmp/ref1.png","/tmp/ref2.png"]' \
  --videos '["https://example.com/clip.mp4"]'
```

> R2V notes: `--videos` only accepts **HTTPS URLs** (no http, no base64); `--audios` cannot be used alone — it must be combined with images or videos.

### Async workflow (--no-wait + status)

```bash
# 1) Submit without waiting
mulerouter run bytedance/seedance-2.0/text-to-video \
  --site mulerun \
  --prompt "..." --no-wait --json
# → {"task_id":"...","api_path":"/vendors/bytedance/v1/seedance-2.0/text-to-video/generation",...}

# 2) Poll using api_path (poll URL = ${api_path}/${task_id}, same convention as alibaba/wan etc.)
mulerouter status /vendors/bytedance/v1/seedance-2.0/text-to-video/generation <task-id> --site mulerun

# 3) Block until terminal state
mulerouter status /vendors/bytedance/v1/seedance-2.0/text-to-video/generation <task-id> --site mulerun --wait
```

### Tips

- For video tasks, use `--max-wait 900` (default) or longer. The fast variant typically takes 1-3 min; std + 1080p can take 5-10 min.
- `--duration` is a discrete set `{-1, 4..15}`. `-1` lets the model choose; other integers must fall in 4..15 (**2 or 3 seconds are not accepted**).
- `--seed` range is `-1..4294967295`; omit for a random seed.
- Do not pass `--model` — the mule-router gateway injects it automatically based on the URL segment.

## References

- [REFERENCE.md](references/REFERENCE.md) - API configuration and CLI options
- [MODELS.md](references/MODELS.md) - Complete model specifications

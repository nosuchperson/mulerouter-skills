---
name: mulerouter
description: Generates images, videos, audio, speech, and music using MuleRouter or MuleRun multimodal APIs. Text-to-Image, Image-to-Image, Text-to-Video, Image-to-Video, Reference-to-Video, Video-to-Video, video editing (VACE, keyframe interpolation), Text-to-Speech, Text-to-Music. Use when the user wants to generate, edit, or transform images, videos, speech, or music using AI models like Wan2.6, Veo3, Nano Banana Pro, Sora2, Midjourney, Kling V3, Kling V3 Omni, MiniMax Speech 2.8, MiniMax Music 2.5, ByteDance Seedance 2.0.
compatibility: Requires Node.js 18+, the `mulerouter` npm CLI, MULEROUTER_API_KEY env var, and one of MULEROUTER_BASE_URL or MULEROUTER_SITE env var. Needs network access to api.mulerouter.ai or api.mulerun.com. The API key is sent in Authorization headers to the configured endpoint.
homepage: https://github.com/openmule/mulerouter-skills
allowed-tools: Bash(mulerouter *) Bash(npx mulerouter*) Bash(npm install*) Read
metadata:
  clawdbot:
    requires:
      env: ["MULEROUTER_API_KEY"]
      env_one_of: ["MULEROUTER_BASE_URL", "MULEROUTER_SITE"]
      bins: ["node", "npm"]
    primaryEnv: "MULEROUTER_API_KEY"
    install: "npm install -g mulerouter"
    files: ["references/*"]
---

# MuleRouter API

Generate images, videos, speech, and music via the **`mulerouter` npm CLI**, which fronts the MuleRouter / MuleRun multimodal API gateways.

There is no Python entry point any more — every model in this skill is invoked through the CLI's `mulerouter run <provider>/<model>/<action> ...` form. The CLI handles task submission, polling, image base64 conversion, and result extraction.

## Required Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `MULEROUTER_API_KEY` | **Yes** | API key for authentication ([get one here](https://www.mulerouter.ai/app/api-keys?utm_source=github_claude_plugin)) |
| `MULEROUTER_BASE_URL` | **Yes\*** | Custom API base URL (e.g. `https://api.mulerouter.ai`). Takes priority over SITE. |
| `MULEROUTER_SITE` | **Yes\*** | API site: `mulerouter` or `mulerun`. Used only if `MULEROUTER_BASE_URL` is not set. |

\*At least one of `MULEROUTER_BASE_URL` or `MULEROUTER_SITE` must be set.

The API key is sent in an `Authorization: Bearer ...` header to the configured endpoint.

**Note:** The CLI only loads variables prefixed with `MULEROUTER_` from any `.env` file in the working directory. Other variables in the file are ignored.

**Important:** Do NOT use `export` shell commands to set credentials inline. Use a `.env` file or have the variables already present in your shell environment before invoking the skill.

## Installation

```bash
# global install (recommended)
npm install -g mulerouter

# or run ad-hoc without installing
npx -y mulerouter@latest --help
```

Verify:

```bash
mulerouter --version
```

## Configuration Check

Run a no-cost discovery command — it loads config the same way `run` does and fails fast if env is missing:

```bash
mulerouter list --limit 1
```

If you see endpoints, configuration is OK. If it errors with a config message, create a `.env` file:

```env
MULEROUTER_API_KEY=your-api-key

# pick ONE of the two below
MULEROUTER_BASE_URL=https://api.mulerouter.ai
# MULEROUTER_SITE=mulerouter   # or: mulerun
```

`MULEROUTER_BASE_URL` takes priority over `MULEROUTER_SITE` when both are present.

## Quick Start

### 1. List available endpoints

```bash
mulerouter list                         # all endpoints
mulerouter list --provider alibaba      # filter by provider
mulerouter list --site mulerun          # filter by site
mulerouter list --output-type video --tag SOTA
```

### 2. Inspect parameters for an endpoint

```bash
mulerouter params alibaba/wan2.6-t2v/generation
```

### 3. Generate content

```bash
# text-to-video
mulerouter run alibaba/wan2.6-t2v/generation \
  --prompt "A cat walking through a garden"

# text-to-image
mulerouter run alibaba/wan2.6-t2i/generation \
  --prompt "A serene mountain lake"

# image-to-video (URL or local file path)
mulerouter run alibaba/wan2.6-i2v/generation \
  --prompt "Gentle zoom in" \
  --image /tmp/photo.png

# text-to-speech (voice-id required)
mulerouter run minimax/speech-2.8-turbo/generation \
  --site mulerun \
  --prompt "Hello world, welcome to the future of AI." \
  --voice-id Charming_Lady
```

## Image Input

Image parameters (`--image`, `--images`, `--first-frame`, `--last-frame`, `--first-frame-url`, `--last-frame-url`, `--ref-images-url`, `--reference-images`) accept any of:

- **Local file path** (preferred) — auto-validated for image extension (`.png`, `.jpg`, `.jpeg`, `.gif`, `.bmp`, `.webp`, `.tiff`, `.tif`, `.svg`, `.ico`, `.heic`, `.heif`, `.avif`), then read and base64-encoded by the CLI.
- **HTTPS URL** — passed through unchanged.
- **`data:image/...;base64,...` URI** — passed through unchanged.

Always prefer local paths over hand-pasted base64 to avoid command-line length limits.

For array-valued image params, pass a JSON literal:

```bash
--images '["/tmp/a.png","/tmp/b.png","https://example.com/c.png"]'
```

## Async Workflow (`--no-wait` + `status`)

Every endpoint except `midjourney/diffusion/generation` runs asynchronously. By default `mulerouter run` polls until the task reaches a terminal state. To submit and return immediately:

```bash
# 1) submit without waiting
mulerouter run alibaba/wan2.6-t2v/generation \
  --prompt "..." --no-wait --json
# → {"task_id":"...","api_path":"/vendors/alibaba/v1/wan2.6-t2v/generation",...}

# 2) check status once
mulerouter status /vendors/alibaba/v1/wan2.6-t2v/generation <task-id>

# 3) or block until terminal
mulerouter status /vendors/alibaba/v1/wan2.6-t2v/generation <task-id> --wait
```

Polling defaults: `--poll-interval 20` (seconds), `--max-wait 900` (seconds). Tune for long video jobs (`--max-wait 1800`) or fast image jobs (`--poll-interval 5`).

The `--site` flag must match the site the task was originally submitted to (some endpoints are only routed on one site — see the per-model sections below).

## Models

Notation in this section:

- **Site**: which gateway routes this endpoint. `both` → omit `--site` or pick either. `mulerun-only` → must add `--site mulerun`. `mulerouter-only` → must add `--site mulerouter` (or omit if your default is mulerouter).
- **API path**: used as the first argument to `mulerouter status` for async polling.
- **Required** / **Key optional**: flag names map to API field names by replacing `-` with `_`.
- Never pass `--model` unless the row explicitly says so (the gateway injects it from the URL).

---

### Alibaba — 15 endpoints

All Alibaba endpoints accept `--safety-filter` (boolean, default `true`) and `--seed` (integer); they are not repeated in every stanza.

#### `alibaba/wan2.1-vace-plus/generation`
- **Site**: both — `/vendors/alibaba/v1/wan2.1-vace-plus/generation`
- **Required**: `--model wan2.1-vace-plus` (⚠ **legacy quirk** — this is the only endpoint where `--model` is required), `--function <fn>` (one of `outpainting`, `video_extend`, `reference_generation`, `interpolation`, `inpainting`), `--prompt <text>`
- **Key optional**: `--negative-prompt`, `--ref-images-url '[...]'`, `--video-url`, `--mask-image-url`, `--mask-video-url`, `--mask-type`, `--first-frame-url`, `--last-frame-url`, `--first-clip-url`, `--last-clip-url`, `--strength <num>`, `--expand-ratio <num>`, `--expand-mode`, `--top-scale`/`--bottom-scale`/`--left-scale`/`--right-scale` (outpainting edge scales), `--size`, `--duration <int> (default=5)`, `--prompt-extend (default=true)`
- **Example**:
  ```bash
  mulerouter run alibaba/wan2.1-vace-plus/generation \
    --model wan2.1-vace-plus --function reference_generation \
    --prompt "Animate this character running" \
    --ref-images-url '["https://example.com/char.png"]' --duration 5
  ```

#### `alibaba/wan2.1-kf2v-plus/generation`
- **Site**: both — `/vendors/alibaba/v1/wan2.1-kf2v-plus/generation`
- **Required**: `--prompt`, `--image` (first keyframe), `--last-frame` (last keyframe)
- **Key optional**: `--negative-prompt`, `--template`, `--resolution (default=720P)`, `--duration <int> (default=5)`, `--prompt-extend (default=true)`
- **Example**:
  ```bash
  mulerouter run alibaba/wan2.1-kf2v-plus/generation \
    --prompt "Morph smoothly" \
    --image /tmp/first.png --last-frame /tmp/last.png \
    --duration 5
  ```

#### `alibaba/wan2.2-t2v-plus/generation`
- **Site**: both — `/vendors/alibaba/v1/wan2.2-t2v-plus/generation`
- **Required**: `--prompt`
- **Key optional**: `--negative-prompt`, `--size` (free-form `WxH`), `--duration <int> (default=5)`, `--prompt-extend (default=true)`

#### `alibaba/wan2.2-i2v-plus/generation`
- **Site**: both — `/vendors/alibaba/v1/wan2.2-i2v-plus/generation`
- **Required**: `--prompt`, `--image`
- **Key optional**: `--negative-prompt`, `--resolution {480P,1080P}`, `--duration <int> (default=5)`, `--prompt-extend (default=true)`

#### `alibaba/wan2.2-i2v-flash/generation`
- **Site**: both — `/vendors/alibaba/v1/wan2.2-i2v-flash/generation`
- **Required**: `--prompt`, `--image`
- **Key optional**: `--negative-prompt`, `--resolution {480P,720P}` (no 1080P on flash), `--duration <int> (default=5)`, `--prompt-extend (default=true)`

#### `alibaba/wan2.5-t2v-preview/generation`
- **Site**: both — `/vendors/alibaba/v1/wan2.5-t2v-preview/generation`
- **Required**: `--prompt`
- **Key optional**: `--negative-prompt`, `--size`, `--duration {5,10} (default=5)`, `--prompt-extend (default=true)`, `--audio (default=false)`, `--audio-url <wav/mp3 url>`

#### `alibaba/wan2.5-i2v-preview/generation`
- **Site**: both — `/vendors/alibaba/v1/wan2.5-i2v-preview/generation`
- **Required**: `--prompt`, `--image`
- **Key optional**: `--negative-prompt`, `--resolution {480P,720P,1080P}`, `--duration {5,10} (default=5)`, `--prompt-extend (default=true)`, `--audio (default=false)`, `--audio-url`

#### `alibaba/wan2.5-t2i-preview/generation`
- **Site**: both — `/vendors/alibaba/v1/wan2.5-t2i-preview/generation`
- **Required**: `--prompt`
- **Key optional**: `--negative-prompt`, `--size`, `--n <int> (default=4)`, `--prompt-extend (default=true)`

#### `alibaba/wan2.5-i2i-preview/generation`
- **Site**: both — `/vendors/alibaba/v1/wan2.5-i2i-preview/generation`
- **Required**: `--prompt`, `--images '[...]'` (max 2 images)
- **Key optional**: `--negative-prompt`, `--size`, `--n <int> (default=4)`, `--prompt-extend (default=true)`

#### `alibaba/wan2.6-t2v/generation`   `[SOTA]`
- **Site**: both — `/vendors/alibaba/v1/wan2.6-t2v/generation`
- **Required**: `--prompt`
- **Key optional**: `--negative-prompt`, `--size {1280*720,960*960,720*1280,1920*1080,1080*1920,2048*1080,1080*2048,1080*1080} (default=1280*720)`, `--duration {5,10,15} (default=5)`, `--prompt-extend (default=true)`, `--multi-shots (default=false)`, `--audio (default=false)`, `--audio-url`

#### `alibaba/wan2.6-i2v/generation`   `[SOTA]`
- **Site**: both — `/vendors/alibaba/v1/wan2.6-i2v/generation`
- **Required**: `--image` (prompt is **optional** for i2v)
- **Key optional**: `--prompt`, `--negative-prompt`, `--resolution {480P,720P,1080P} (default=720P)`, `--duration {5,10,15} (default=5)`, `--prompt-extend (default=true)`, `--multi-shots (default=false)`, `--audio (default=false)`, `--audio-url`

#### `alibaba/wan2.6-t2i/generation`
- **Site**: both — `/vendors/alibaba/v1/wan2.6-t2i/generation`
- **Required**: `--prompt`
- **Key optional**: `--negative-prompt`, `--size (default=1280*1280)`, `--n <int 1..4> (default=1)`, `--prompt-extend (default=true)`

#### `alibaba/wan2.6-image/generation`
- **Site**: both — `/vendors/alibaba/v1/wan2.6-image/generation`
- **Required**: `--prompt`, `--images '[...]'`
- **Key optional**: `--negative-prompt`, `--size`, `--n <int> (default=1)`

#### `alibaba/happy-horse-1-0-t2v/generation`
- **Site**: **mulerun-only** — `/vendors/alibaba/v1/happy-horse-1-0-t2v/generation`
- **Required**: `--prompt` (max 2500 chars)
- **Key optional**: `--resolution {720P,1080P} (default=1080P)`, `--duration <int 3..15> (default=5)`
- **Example**:
  ```bash
  mulerouter run alibaba/happy-horse-1-0-t2v/generation --site mulerun \
    --prompt "A galloping horse on the beach" --resolution 1080P --duration 6
  ```

#### `alibaba/happy-horse-1-0-i2v/generation`
- **Site**: **mulerun-only** — `/vendors/alibaba/v1/happy-horse-1-0-i2v/generation`
- **Required**: `--image` (first frame; prompt is optional)
- **Key optional**: `--prompt` (max 2500 chars), `--resolution {720P,1080P} (default=1080P)`, `--duration <int 3..15> (default=5)`

---

### Google — 5 endpoints (4 models)

#### `google/nano-banana/generation`
- **Site**: **mulerun-only** — `/vendors/google/v1/nano-banana/generation`
- **Required**: `--prompt`
- **Key optional**: `--aspect-ratio {1:1,3:4,4:3,9:16,16:9,2:3,3:2,9:21,21:9} (default=1:1)`
- **Example**:
  ```bash
  mulerouter run google/nano-banana/generation --site mulerun \
    --prompt "A serene mountain lake at sunrise" --aspect-ratio 16:9
  ```

> The companion `nano-banana/edit` action is also available at `/vendors/google/v1/nano-banana/edit` (params: `--prompt*`, `--images*`, `--aspect-ratio`). Same site (mulerun-only).

#### `google/nano-banana-2/generation`   `[SOTA]`
- **Site**: both — `/vendors/google/v1/nano-banana-2/generation`
- **Required**: `--prompt`
- **Key optional**: `--aspect-ratio {1:1,3:4,4:3,9:16,16:9,2:3,3:2,9:21,21:9,1:2,2:1,4:5,5:4,5:8} (default=1:1)`, `--resolution {1K,2K,4K}`, `--web-search`

> Also exposes `google/nano-banana-2/edit` — same flags plus `--images '[...]'` (1..14 images required).

#### `google/nano-banana-pro/generation`   `[SOTA]`
- **Site**: both — `/vendors/google/v1/nano-banana-pro/generation`
- **Required**: `--prompt`
- **Key optional**: `--aspect-ratio {1:1,3:4,4:3,9:16,16:9,2:3,3:2,9:21,21:9,4:5} (default=1:1)`, `--resolution {1K,2K}`

#### `google/nano-banana-pro/edit`   `[SOTA]`
- **Site**: both — `/vendors/google/v1/nano-banana-pro/edit`
- **Required**: `--prompt`, `--images '[...]'` (1–10 images; URLs or local paths)
- **Key optional**: `--aspect-ratio {1:1,3:4,4:3,9:16,16:9,2:3,3:2,9:21,21:9,4:5} (default=1:1)`, `--resolution {1K,2K}`

#### `google/veo3/generation`   `[SOTA]`
- **Site**: **mulerun-only** — `/vendors/google/v1/veo/generation` (path is `veo`, not `veo3`)
- **Required**: `--prompt`
- **Key optional**: `--model {veo-3.1,veo-3.1-fast,veo-3} (default=veo-3.1)` (⚠ veo3 is the **only** other endpoint where `--model` is a real flag — used to pick the variant), `--negative-prompt`, `--image` (first frame), `--last-frame`, `--reference-images '[...]'` (max 3, for style), `--aspect-ratio {16:9,9:16}`, `--resolution {720p,1080p}`, `--duration {4,6,8} (default=8)`
- **Example**:
  ```bash
  mulerouter run google/veo3/generation --site mulerun \
    --prompt "A drone shot over a coastal cliff" \
    --model veo-3.1 --aspect-ratio 16:9 --duration 8
  ```

---

### KlingAI — 7 endpoints

All Kling endpoints accept `--multi-shot` / `--shot-type` for shot control and `--negative-prompt` (max 2500 chars). Prompts in Kling max out at 2500 chars.

#### `klingai/kling-v3-t2v/generation`   `[SOTA]`
- **Site**: both — `/vendors/klingai/v1/kling-v3/text-to-video/generation`
- **Required**: one of `--prompt` or `--multi-prompt '[{"prompt":"shot1","duration":N},...]'`
- **Key optional**: `--negative-prompt`, `--mode {std,pro} (default=std)`, `--multi-shot {false,true} (default=false)`, `--shot-type {customize,intelligence} (default=customize)`, `--aspect-ratio {16:9,9:16,1:1} (default=16:9)`, `--duration <int 3..15> (default=5)`, `--sound {off,on} (default=off)`

#### `klingai/kling-v3-i2v/generation`   `[SOTA]`
- **Site**: both — `/vendors/klingai/v1/kling-v3/image-to-video/generation`
- **Required**: at least one of `--first-frame` or `--last-frame`
- **Key optional**: `--prompt`, `--elements '[{...}]'` (max 3, reference via `<<<element_N>>>` in prompt), `--negative-prompt`, `--mode {std,pro} (default=std)`, `--multi-shot {false,true} (default=false)`, `--shot-type {customize,intelligence}`, `--multi-prompt '[{"prompt":"...","duration":N}]'`, `--duration <int 3..15> (default=5)`, `--sound {off,on} (default=off)`
- **Example**:
  ```bash
  mulerouter run klingai/kling-v3-i2v/generation \
    --first-frame /tmp/start.png --last-frame /tmp/end.png \
    --prompt "Slow zoom" --duration 5
  ```

#### `klingai/kling-v3-omni-t2v/generation`   `[SOTA]`
- **Site**: both — `/vendors/klingai/v1/kling-v3-omni/text-to-video/generation`
- **Required**: one of `--prompt` or `--multi-prompt '[...]'` (max 6 shots)
- **Key optional**: `--negative-prompt`, `--sound {off,on} (default=off)`, `--mode {std,pro} (default=pro)`, `--aspect-ratio {16:9,9:16,1:1} (default=16:9)`, `--duration <int 3..15> (default=5)`, `--multi-shot`, `--shot-type {customize,intelligence}`

#### `klingai/kling-v3-omni-i2v/generation`   `[SOTA]`
- **Site**: both — `/vendors/klingai/v1/kling-v3-omni/image-to-video/generation`
- **Required**: at least one of `--first-frame` or `--last-frame` (last-frame requires first-frame). If neither, `--aspect-ratio` must be set.
- **Key optional**: `--prompt`, `--multi-prompt`, `--negative-prompt`, `--sound {off,on} (default=off)`, `--mode {std,pro}`, `--duration <int 3..15>`, `--aspect-ratio {16:9,9:16,1:1}`, `--multi-shot`, `--shot-type`

#### `klingai/kling-v3-omni-ref2v/generation`   `[SOTA]`
- **Site**: both — `/vendors/klingai/v1/kling-v3-omni/reference-image-to-video/generation`
- **Required**: at least one reference input (combination of `--images '[...]'`, `--elements '[...]'`, `--first-frame`, `--last-frame`); total count of elements+images+frames ≤ 7. Use `<<<element_N>>>` in `--prompt` to reference individual elements.
- **Key optional**: `--prompt` (max 2500), `--multi-prompt` (max 6 shots), `--negative-prompt`, `--sound {off,on} (default=off)`, `--mode {std,pro} (default=pro)`, `--aspect-ratio {16:9,9:16,1:1} (default=16:9)`, `--duration <int 3..15> (default=5)`, `--multi-shot`, `--shot-type`
- **Example**:
  ```bash
  mulerouter run klingai/kling-v3-omni-ref2v/generation \
    --prompt "Cinematic montage of <<<element_1>>>" \
    --images '["/tmp/ref1.png","/tmp/ref2.png"]' \
    --duration 5
  ```

#### `klingai/kling-v3-omni-v2v/generation`   `[SOTA]`
- **Site**: both — `/vendors/klingai/v1/kling-v3-omni/reference-video-to-video/generation`
- **Required**: `--prompt` (reference the input video with `@Video1`), `--video '[{"video_url":"https://...","refer_type":"feature","keep_original_sound":"no"}]'` (exactly 1 video)
- **Key optional**: `--negative-prompt`, `--first-frame`, `--last-frame`, `--images '[...]'`, `--elements '[...]'` (total elements+images+frames ≤ 4), `--mode {std,pro} (default=pro)`, `--aspect-ratio {16:9,9:16,1:1}` (auto-inferred from video if omitted), `--duration <int 3..15> (default=5, auto-inferred if omitted)`

#### `klingai/kling-v3-omni-v2v-edit/generation`   `[SOTA]`
- **Site**: both — `/vendors/klingai/v1/kling-v3-omni/video-to-video/edit`
- **Required**: `--prompt` (reference video with `@Video1`), `--video '[{"video_url":"https://...","refer_type":"base","keep_original_sound":"no"}]'` (exactly 1 video)
- **Key optional**: `--negative-prompt`, `--images '[...]'`, `--elements '[...]'` (total elements+images ≤ 4), `--mode {std,pro} (default=pro)` (std=720P output, pro=1080P)

---

### Midjourney — 2 endpoints

#### `midjourney/diffusion/generation`   `[SOTA]`
- **Site**: both — `/vendors/midjourney/v1/tob/diffusion`
- **Required**: `--prompt` (max 8192 chars; include `--ar 16:9`, `--v 7`, etc. as Midjourney inline syntax inside the prompt string)
- **Note**: This endpoint is **synchronous** — `mulerouter run` returns the result directly without polling. `--no-wait` / `mulerouter status` are not needed.
- **Example**:
  ```bash
  mulerouter run midjourney/diffusion/generation \
    --prompt "A neon-lit cyberpunk alley --ar 16:9 --stylize 250"
  ```

#### `midjourney/video/generation`   `[SOTA]`
- **Site**: both — `/vendors/midjourney/v1/tob/video-diffusion`
- **Required**: `--prompt` (max 8192). For image-to-video, embed the image URL inside the prompt string, e.g. `"description https://example.com/image.jpg"`.
- **Key optional**: `--video-type {0,1} (default=0)` — `0`=480p, `1`=720p
- **Example**:
  ```bash
  mulerouter run midjourney/video/generation \
    --prompt "https://example.com/portrait.jpg subtle head turn --motion low" \
    --video-type 1
  ```

---

### MiniMax — 4 endpoints (all `--site mulerun`)

`speech-2.8-hd` and `speech-2.8-turbo` share the same parameter set. Flat flags below are transformed by the CLI into the nested `voice_setting` / `audio_setting` request body.

#### `minimax/speech-2.8-hd/generation`
- **Site**: **mulerun-only** — `/vendors/minimax/v1/speech-2.8-hd/text-to-speech/generation`
- **Required**: `--prompt` (text to speak, 1–50000 chars), `--voice-id <id>` (see [references/MINIMAX_VOICES.md](references/MINIMAX_VOICES.md) for the full voice catalog)
- **Key optional**: `--speed <0.5..2.0>`, `--vol <0.01..10.0>`, `--pitch <int -12..12>`, `--emotion {happy,sad,angry,fearful,disgusted,surprised,neutral}`, `--language-boost <code>` (e.g. `zh`, `en`, `ja`, `ko`, `es`, `fr`, `de`, `ru`, `ar`, …), `--output-format {url,hex}`, `--audio-format {mp3,pcm,flac}`, `--sample-rate <Hz>`, `--bitrate <bps>`, `--english-normalization`

#### `minimax/speech-2.8-turbo/generation`
- **Site**: **mulerun-only** — `/vendors/minimax/v1/speech-2.8-turbo/text-to-speech/generation`
- **Required / optional**: identical to `speech-2.8-hd` above.
- **Example**:
  ```bash
  mulerouter run minimax/speech-2.8-turbo/generation --site mulerun \
    --prompt "Hello world." --voice-id Charming_Lady \
    --emotion happy --speed 1.1 --audio-format mp3
  ```

#### `minimax/music-2.0/generation`
- **Site**: **mulerun-only** — `/vendors/minimax/v1/music-2.0/text-to-music/generation`
- **Required**: `--lyrics-prompt` (10–3000 chars; use `[verse]` / `[chorus]` tags to structure)
- **Key optional**: `--prompt` (style description, max 2000), `--audio-format`, `--sample-rate`, `--bitrate`
- **Example**:
  ```bash
  mulerouter run minimax/music-2.0/generation --site mulerun \
    --prompt "uplifting indie pop, female vocal" \
    --lyrics-prompt $'[verse]\nWalking down the street\n[chorus]\nLa la la la'
  ```

#### `minimax/music-2.5/generation`
- **Site**: **mulerun-only** — `/vendors/minimax/v1/music-2.5/text-to-music/generation`
- **Required**: none in isolation — provide either `--lyrics-prompt` (lyrics with tags) OR `--lyrics-optimizer` plus `--prompt` (style) to let the model write lyrics.
- **Key optional**: `--prompt` (style, max 2000), `--lyrics-prompt`, `--lyrics-optimizer`, `--audio-format`, `--sample-rate`, `--bitrate`

---

### OpenAI — 3 endpoints (2 models)

#### `openai/gpt-image-2/generation`   `[SOTA]`
- **Site**: **mulerouter-only** — `/vendors/openai/v1/gpt-image-2/generation`
- **Required**: `--prompt`
- **Key optional**: `--quality {high,medium,low,auto} (default=high)`, `--size {1024x1024,1536x1024,1024x1536,2048x2048,2048x1152,3840x2160,2160x3840,auto} (default=auto)`, `--n <int 1..4> (default=1)`, `--format {png,jpeg,webp} (default=png)`

#### `openai/gpt-image-2/edit`   `[SOTA]`
- **Site**: **mulerouter-only** — `/vendors/openai/v1/gpt-image-2/edit`
- **Required**: `--prompt`, `--images '[...]'` (1+ images, URLs or local paths)
- **Key optional**: `--size` (same enum as generation, default `auto`), `--n <int 1..4> (default=1)`, `--mask <url-or-path>`, `--format {png,jpeg,webp} (default=png)`

#### `openai/sora2/generation`   `[SOTA]`

> ⚠ **Unavailable until npm publish.** `openai/sora2/generation` requires `mulerouter` CLI built from commit `2dc5b60` (`mulerouter-cli` branch `mc-auto/sess-02d06240`) or later. The endpoint is not yet routed by published npm versions. Until a new `mulerouter@x.y.z` is published, calls will fail with "unknown endpoint". Track release status at the repo. The docs below describe the intended invocation once available.

- **Site**: **mulerun-only** — `/vendors/openai/v1/sora-2/generation` (path uses hyphenated `sora-2`)
- **Required**: `--prompt` (max 2000 chars)
- **Key optional**: `--size {1280x720,720x1280} (default=1280x720)`, `--seconds {4,8,12} (default=4)`, `--image <url-or-path>` (optional first-frame for I2V; max 10 MB; jpeg/png/webp)
- **Example** (once available):
  ```bash
  mulerouter run openai/sora2/generation --site mulerun \
    --prompt "A koi pond in autumn rain" --size 1280x720 --seconds 8
  ```

---

## Seedance via npm CLI (ByteDance)

ByteDance Seedance 2.0 / 2.0-fast — 6 endpoints (`text-to-video`, `image-to-video`, `reference-to-video` × `seedance-2.0` and `seedance-2.0-fast`).

> ⚠ Seedance is currently only routed on `--site mulerun`. The standalone site (`api.mulerouter.ai`) does not yet route these endpoints (404).

```bash
mulerouter list --provider bytedance
mulerouter params bytedance/seedance-2.0/text-to-video
```

T2V:
```bash
mulerouter run bytedance/seedance-2.0/text-to-video --site mulerun \
  --prompt "A cat walking through a snowy garden" \
  --resolution 1080p --duration 5

# fast variant (max 720p; does NOT accept camera_fixed/watermark)
mulerouter run bytedance/seedance-2.0-fast/text-to-video --site mulerun \
  --prompt "A cat walking" --resolution 720p --duration 4
```

I2V (local paths auto-base64; optional `--last-frame-image`):
```bash
mulerouter run bytedance/seedance-2.0/image-to-video --site mulerun \
  --prompt "Gentle zoom in" \
  --image /tmp/first.png --last-frame-image /tmp/last.png
```

R2V (multi-modal references):
```bash
mulerouter run bytedance/seedance-2.0/reference-to-video --site mulerun \
  --prompt "Cinematic montage" \
  --images '["/tmp/ref1.png","/tmp/ref2.png"]' \
  --videos '["https://example.com/clip.mp4"]'
```

R2V notes: `--videos` only accepts **HTTPS URLs** (no http, no base64); `--audios` cannot be used alone — combine with images or videos. `--duration` is a discrete set `{-1, 4..15}` (no 2 or 3); `--seed` range is `-1..4294967295`.

---

## Model Selection

When listing models, each model's **tags** (e.g. `[SOTA]`) are displayed by default. Filter with `--tag`:

```bash
mulerouter list --tag SOTA
```

**If you are unsure which model to use**, present the relevant options to the user and let them choose. Use the `AskUserQuestion` tool (or equivalent interactive prompt). For example, if the user asks to "generate an image" without specifying a model, list image-generation models with their tags and descriptions and ask the user to pick one.

## Tips

1. **Suggested timeouts** — image: 5 min; video: 15 min; speech/music: 3 min. Tune via `--max-wait <seconds>`.
2. **TTS models** (`speech-2.8-hd`, `speech-2.8-turbo`) require `--voice-id`. Run `mulerouter params minimax/speech-2.8-turbo/generation` for parameter details, or refer to [MINIMAX_VOICES.md](references/MINIMAX_VOICES.md).
3. **Never pass `--model`** except for two specific endpoints:
   - `alibaba/wan2.1-vace-plus/generation` (always pass `--model wan2.1-vace-plus`)
   - `google/veo3/generation` (use `--model` to pick variant: `veo-3.1` / `veo-3.1-fast` / `veo-3`)
4. **Do not paste raw base64** on the command line — use a local file path so the CLI base64-encodes for you (`--image /tmp/x.png`).
5. **JSON-typed flags** (arrays/objects) must be a single-quoted JSON literal: `--images '["a.png","b.png"]'`, `--multi-prompt '[{"prompt":"shot1","duration":3}]'`.
6. **Async polling**: any `mulerouter run` invocation can be split into submit + status with `--no-wait --json` followed by `mulerouter status <api_path> <task_id> [--wait]`. The `api_path` is in the `--json` output and matches the API path shown in each model section above.

## References

- [REFERENCE.md](references/REFERENCE.md) — CLI reference (flags, subcommands, env vars)
- [MODELS.md](references/MODELS.md) — Full model catalog with availability matrix
- [MINIMAX_VOICES.md](references/MINIMAX_VOICES.md) — MiniMax TTS voice IDs

# MuleRouter Agent Skill

Agent Skill for calling MuleRouter / MuleRun multimodal APIs to generate images, videos, speech, and music. The skill is a thin documentation layer over the [`mulerouter` npm CLI](https://www.npmjs.com/package/mulerouter), which handles all network calls, image base64 conversion, and async task polling.

## Features

- **Multiple sites** — supports both [MuleRouter](https://mulerouter.ai) and [MuleRun](https://mulerun.com) gateways
- **37+ endpoints** — Wan2.1/2.2/2.5/2.6, Happy Horse, Seedance 2.0, Nano Banana / Pro, Veo3, GPT-Image-2, Kling V3 / V3 Omni, Midjourney, MiniMax Speech 2.8 + Music 2.5
- **Single CLI surface** — `mulerouter run <provider>/<model>/<action> --flag value …`
- **Easy configuration** — environment variables or `.env` file
- **Async task handling** — automatic polling, or split into `--no-wait` + `mulerouter status` for long jobs
- **AI-friendly discovery** — `mulerouter list` / `mulerouter params <endpoint>` enumerate everything

## Installation

### Step 1 — Install the CLI

```bash
npm install -g mulerouter
# or run ad-hoc without installing
npx -y mulerouter@latest --help
```

Requires Node.js 18 or later.

### Step 2 — Install the skill

#### Via Claude CLI

```bash
claude plugin marketplace add openmule/mulerouter-skills
claude plugin install mulerouter-skills
```

#### Via Claude Code session

```
/plugin marketplace add openmule/mulerouter-skills
/plugin install mulerouter-skills
```

After installation, **restart Claude Code** to load the new skill.

## Configuration

### Required Environment Variables

| Variable | Description |
|----------|-------------|
| `MULEROUTER_API_KEY` | API key for authentication ([get one here](https://www.mulerouter.ai/app/api-keys?utm_source=github_claude_plugin)) |

### API endpoint (one required)

| Variable | Description | Priority |
|----------|-------------|----------|
| `MULEROUTER_BASE_URL` | Custom API base URL (e.g. `https://api.mulerouter.ai`) | higher |
| `MULEROUTER_SITE` | API site: `mulerouter` or `mulerun` | lower |

If both are set, `MULEROUTER_BASE_URL` wins.

### Setup options

#### Option A — shell env

```bash
export MULEROUTER_API_KEY="your-api-key"
export MULEROUTER_BASE_URL="https://api.mulerouter.ai"
# or instead: export MULEROUTER_SITE=mulerouter
```

#### Option B — `.env` file

```env
MULEROUTER_API_KEY=your-api-key

# pick ONE
MULEROUTER_BASE_URL=https://api.mulerouter.ai
# MULEROUTER_SITE=mulerouter   # or: mulerun
```

The CLI only loads variables prefixed with `MULEROUTER_` from `.env`. Other variables are ignored.

### Verify

```bash
mulerouter --version
mulerouter config              # show current configuration / diagnose env
```

## Quick Start

Once installed and configured, just ask Claude to use MuleRouter to generate something — the skill self-documents available models. Manually you can also call the CLI directly:

```bash
# enumerate endpoints
mulerouter list --tag SOTA

# inspect parameters for one endpoint
mulerouter params alibaba/wan2.6-t2v/generation

# generate a video
mulerouter run alibaba/wan2.6-t2v/generation \
  --prompt "A cat walking through a garden"

# text-to-speech (mulerun-only model)
mulerouter run minimax/speech-2.8-turbo/generation --site mulerun \
  --prompt "Hello world." --voice-id Charming_Lady
```

See [`skills/mulerouter-skills/SKILL.md`](skills/mulerouter-skills/SKILL.md) for full per-endpoint documentation and [`skills/mulerouter-skills/references/MODELS.md`](skills/mulerouter-skills/references/MODELS.md) for the model catalog.

## Project Structure

```
.
├── .claude-plugin/
│   └── marketplace.json
├── README.md
├── LICENSE
└── skills/
    └── mulerouter-skills/
        ├── SKILL.md                    # Agent skill entry point (per-model docs)
        ├── README.md                   # (this file)
        ├── .env.example
        └── references/
            ├── REFERENCE.md            # CLI subcommands, flags, lifecycle
            ├── MODELS.md               # Model catalog (37+ endpoints)
            └── MINIMAX_VOICES.md       # MiniMax TTS voice IDs
```

All runtime logic lives in the `mulerouter` npm CLI.

## License

MIT

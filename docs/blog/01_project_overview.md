# ACE-Step on Mac: Project Overview

## Screenshot-Driven Development

A tweet by [@AmbsdOP](https://x.com/AmbsdOP/status/2018735590930518175) showed ACE-Step 1.5 running locally on a Mac -- a Suno-level AI music generation model, no cloud, no subscription. The response was simple: screenshot the tweet, hand it to Claude Code, and say "make me one too."

~20 minutes later: a full AI music production studio running at `localhost:3000`.

## What Got Built

A local deployment combining two open-source projects:

- **ACE-Step 1.5** -- Open-source AI music generation model (MIT license). Diffusion transformer architecture for text-to-music synthesis.
- **ace-step-ui** by [@AmbsdOP](https://x.com/AmbsdOP) -- Professional React/TypeScript frontend with song library, audio editor, and stem separation.

Both running on Apple Silicon with the **MLX backend** for native Metal GPU acceleration.

**GitHub:** https://github.com/aviz85/ace-step

## Architecture

```
                    Browser (:3000)
                         |
                    Vite Dev Server (React/TS)
                         |
                    Express.js Backend (:3001)
                    ├── REST API (songs, playlists, auth, generate)
                    ├── SQLite database
                    ├── Gradio client (primary)
                    └── Python spawn (fallback)
                         |
                    ACE-Step API (:8001)
                    ├── Gradio server
                    ├── DiT model (diffusion transformer)
                    ├── LM model (language model for lyrics/structure)
                    └── MLX backend (Metal GPU)
```

## Three-Process Startup

The `start-all.sh` script launches three services sequentially:

1. **ACE-Step API** (port 8001) -- Python/Gradio server running the ML model with MLX backend
2. **Express Backend** (port 3001) -- Node.js API server with SQLite, job queue, audio storage
3. **Vite Frontend** (port 3000) -- React development server proxying API calls to the backend

## Key Capabilities

- Text-to-music generation from prompts and style descriptions
- Lyrics editor with vocal language selection
- Multiple generation modes: text2music, cover, audio2audio, repaint
- Batch generation (up to 16 variations)
- Audio editor integration (AudioMass)
- Stem separation (Demucs)
- Song library with search, playlists, favorites
- Social features (user profiles, sharing, oEmbed)
- 100% offline after initial ~10GB model download
- LAN access enabled (0.0.0.0 binding)

## Model Details

ACE-Step 1.5 ships with multiple DiT model variants:

| Model | Description |
|-------|-------------|
| `acestep-v15-turbo` | Default, optimized for speed |
| `acestep-v15-base` | Base model |
| `acestep-v15-sft` | Supervised fine-tuned |
| `acestep-v15-turbo-shift1` | Turbo with shift=1 |
| `acestep-v15-turbo-shift3` | Turbo with shift=3 |
| `acestep-v15-turbo-continuous` | Continuous generation |

Models are stored in `ACE-Step-1.5/checkpoints/` and auto-downloaded on first use.

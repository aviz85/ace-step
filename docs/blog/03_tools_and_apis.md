# Tools, APIs, and Tech Stack

## Core Stack

### ML Backend (Python)

| Component | Role |
|-----------|------|
| **ACE-Step 1.5** | Diffusion transformer model for music generation |
| **MLX** | Apple's ML framework for native Metal GPU acceleration |
| **PyTorch** | Tensor operations (fallback/compatibility) |
| **Gradio** | Web API server wrapping the model (port 8001) |
| **uv** | Python package manager (replaces pip/venv) |
| **FFmpeg / ffprobe** | Audio format conversion and duration detection |

### Application Backend (Node.js)

| Component | Role |
|-----------|------|
| **Express.js** | HTTP API server (port 3001) |
| **better-sqlite3** | Embedded database for songs, users, playlists, jobs |
| **@gradio/client** | TypeScript client for the Gradio API |
| **multer** | File upload handling (audio reference tracks) |
| **jsonwebtoken** | JWT-based local authentication |
| **helmet** | HTTP security headers |
| **node-cron** | Scheduled cleanup jobs |
| **tsx** | TypeScript execution without build step |

### Frontend (React/TypeScript)

| Component | Role |
|-----------|------|
| **React 19** | UI framework |
| **Vite 6** | Dev server and bundler |
| **lucide-react** | Icon library |
| **@ffmpeg/ffmpeg** | Client-side audio processing (WASM) |
| **AudioMass** | Embedded audio editor |
| **Demucs Web** | Browser-based stem separation (ONNX runtime) |

## API Architecture

### External APIs

- **ACE-Step Gradio API** (`localhost:8001`) -- Primary generation interface. The Express backend connects via `@gradio/client` and calls `/generation_wrapper` with 50 positional arguments.
- **Pexels API** (optional) -- Stock photos/videos for music video backgrounds. API key configurable per-user.
- **Google GenAI** (optional) -- Gemini integration for enhanced features.

### Internal REST API

The Express server exposes these route groups:

```
/api/auth          -- JWT signup/login
/api/songs         -- CRUD for generated songs
/api/generate      -- Submit jobs, poll status, upload audio
/api/users         -- Profiles, followers
/api/playlists     -- Playlist management
/api/search        -- Full-text search across songs/creators/playlists
/api/reference-tracks  -- Reference audio management
/api/lora          -- LoRA model management
/api/training      -- Model training jobs
/api/contact       -- Contact form
```

### Generation Endpoint Detail

`POST /api/generate` accepts a rich parameter set:

- **Mode:** simple (description) or custom (style + lyrics)
- **Music params:** BPM, key/scale, time signature, vocal language, duration
- **DiT params:** inference steps, guidance scale, seed, shift, inference method
- **LM params:** temperature, CFG scale, top-k, top-p, negative prompt
- **Expert params:** reference audio, source audio, audio codes, repainting range, cover strength, task type
- **CoT features:** thinking mode, enhance mode, metadata generation, caption rewriting

The backend maps these to 50 positional Gradio arguments via `buildGradioArgs()`.

## Dual Generation Strategy

The backend implements a fallback chain:

1. **Gradio client** (primary) -- Connects to the running Gradio server, calls `predict()`, downloads result files via filesystem copy or HTTP.
2. **Python spawn** (fallback) -- If Gradio is unavailable, spawns a Python subprocess running `simple_generate.py` with CLI arguments. Parses JSON output from stdout.

Both paths produce audio files stored in `server/public/audio/` and referenced by URL in the SQLite database.

## DevOps

| Tool | Purpose |
|------|---------|
| `start-all.sh` | Launches all three services with PID management |
| `stop-all.sh` | Graceful shutdown via stored PIDs |
| `setup.sh` | First-time setup (deps, env, db migration) |
| `logs/` | Separate log files per service |

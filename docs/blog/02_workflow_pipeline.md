# Workflow Pipeline: Screenshot to Music Studio

## The Input

A single screenshot of [this tweet](https://x.com/AmbsdOP/status/2018735590930518175) by @AmbsdOP, plus the instruction: "make me one too."

## What Claude Code Did (Autonomously)

### Phase 1: Repository Setup
1. Cloned `ACE-Step-1.5` from GitHub
2. Cloned `ace-step-ui` (the frontend by @AmbsdOP)
3. Detected the project structure: Python backend + Node.js frontend

### Phase 2: Python Environment
1. Created Python virtual environment with `uv`
2. Ran `uv sync` to install all Python dependencies
3. Detected Apple Silicon and configured MLX backend
4. Set `ACESTEP_LM_BACKEND=mlx` for Metal GPU acceleration

### Phase 3: Node.js Environment
1. Ran `npm install` for the Vite/React frontend
2. Ran `npm install` in `server/` for the Express backend
3. Configured `.env` with correct paths and ports

### Phase 4: Model Download
1. First launch triggered automatic model download (~10GB)
2. Models downloaded from HuggingFace to `ACE-Step-1.5/checkpoints/`
3. Includes DiT weights, LM weights, tokenizer, and vocoder

### Phase 5: Configuration & Launch
1. Created startup scripts (`start-all.sh`, `stop-all.sh`)
2. Configured Vite proxy to forward `/api` and `/audio` to the backend
3. Set up the three-service launch sequence
4. Opened browser to `http://localhost:3000`

## The Bug

On first generation attempt, the backend failed with a path resolution error. The `ACESTEP_PATH` environment variable was pointing to the wrong location.

**Root cause:** The `resolveAceStepPath()` function in `server/src/services/acestep.ts` resolves relative paths from `process.cwd()`, but the startup script changes directories before launching the backend. The path `../ACE-Step-1.5` resolved correctly from the project root but not from the `server/` subdirectory.

**Fix:** Set `ACESTEP_PATH` as an absolute path in the environment, or ensure the backend is always launched from the project root.

## Timeline

```
T+0:00  Screenshot provided to Claude Code
T+0:30  Repos cloned
T+2:00  Python deps installed via uv
T+4:00  Node deps installed
T+5:00  Configuration files created
T+6:00  First launch attempt
T+8:00  Model download begins (~10GB)
T+18:00 Models ready, services restarted
T+19:00 First generation attempt -- path bug hit
T+20:00 Bug fixed, first song generated
```

## Generation Flow (Runtime)

```
User types prompt in browser
        |
POST /api/generate
        |
Express creates job in SQLite, submits to job queue
        |
Job queue processes sequentially (GPU = 1 job at a time)
        |
    [Gradio available?]
    /               \
  YES               NO
   |                 |
buildGradioArgs()   spawn Python process
   |                 |
client.predict()    simple_generate.py
   |                 |
Download audio      Copy audio from output dir
from Gradio path    to public/audio/
        |               |
        +-------+-------+
                |
Store in SQLite, return audio URLs
                |
Frontend polls GET /api/generate/status/:jobId
                |
Audio plays in browser
```

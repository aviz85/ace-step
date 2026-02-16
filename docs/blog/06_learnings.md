# Learnings

## Technical Learnings

### 1. MLX Is the Key for Apple Silicon ML

The critical configuration that makes this work on Mac is the MLX backend. Without it, the model falls back to CPU-only PyTorch, which is unusably slow for music generation.

The magic line in `start-all.sh`:
```bash
ACESTEP_LM_BACKEND=mlx uv run acestep --backend mlx
```

MLX provides native Metal GPU acceleration, making generation times practical on consumer hardware.

### 2. Gradio as a Local API Gateway

ACE-Step uses Gradio not just as a web UI framework but as a full API server. The Express backend connects to it via `@gradio/client` and calls endpoints like `/generation_wrapper` with 50 positional arguments.

This architecture means:
- The ML model runs in its own process with its own memory management
- The Node.js backend acts as an orchestrator, not a model host
- The Gradio server handles model loading, GPU scheduling, and inference
- If Gradio is unavailable, the backend can fall back to spawning Python directly

### 3. Path Resolution Is the #1 Bug Source

The only bug encountered was a path resolution issue. `ACESTEP_PATH` was set as a relative path (`../ACE-Step-1.5`), which broke when the working directory changed during startup.

Lesson: In multi-process architectures where services start from different directories, always use absolute paths in environment variables.

### 4. uv Is Dramatically Faster Than pip

The agent chose `uv` for Python package management. For a project with ~10GB of ML dependencies (PyTorch, MLX, transformers, etc.), the speed difference is significant. `uv sync` handles virtual environment creation and dependency resolution in one command.

### 5. The 50-Argument Gradio Interface

ACE-Step's Gradio endpoint accepts 50 positional arguments. The `buildGradioArgs()` function in the backend maps a clean TypeScript interface to these positional args. This is fragile -- any Gradio API change breaks the mapping -- but it works because both components are version-locked locally.

### 6. Model Download Is the Real Bottleneck

The ~10GB model download (DiT weights, LM weights, tokenizer, vocoder) takes longer than everything else combined. First-run experience could be improved with:
- Progress indication during download
- Pre-downloaded model bundles
- Incremental model loading

## Process Learnings

### 7. Screenshots Are Valid Specifications

The entire project was specified by a screenshot of a tweet. The agent extracted:
- The project name (ACE-Step 1.5)
- The required components (backend + UI)
- The target platform (Mac)
- The expected result (local music generation)

No written specification, no architecture document, no requirements list. A visual reference was sufficient.

### 8. Agent Error Recovery Works

When the path bug was hit, the human's contribution was sending the error message back to the agent. The agent:
1. Read the error trace
2. Identified the root cause (relative path resolution)
3. Applied the fix
4. Verified it worked

This is a realistic workflow: the human acts as a sensor (observing and reporting), the agent acts as the debugger and fixer.

### 9. Multi-Service Architecture Is Agent-Friendly

The three-service architecture (Python ML server, Node.js backend, Vite frontend) is actually well-suited for agent setup because:
- Each service has clear boundaries and independent configuration
- Startup is sequential with observable health checks
- Failures are isolated -- one service crashing doesn't take down the others
- The agent can reason about each service independently

### 10. The 80/20 of Local AI

Getting from "nothing" to "working demo" is fast. The remaining polish (custom models, fine-tuning, production hardening) is where the real time goes. For personal/creative use, the initial setup is sufficient.

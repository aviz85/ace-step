# Automation Created

## What Was Automated

The entire project setup -- from an empty directory to a running AI music studio -- was automated by Claude Code based on a single screenshot input. This section documents the specific automation artifacts created during the process.

## Startup Automation

### `start-all.sh`

Three-phase sequential startup with health checks:

```
[1/3] ACE-Step API → wait 5s → verify PID alive
[2/3] Express Backend → wait 3s → verify PID alive
[3/3] Vite Frontend → wait 2s → verify PID alive
→ Save PIDs to logs/*.pid
→ Open browser
→ Print all URLs (localhost + LAN)
```

Key details:
- Sets `ACESTEP_LM_BACKEND=mlx` for Metal GPU
- Detects LAN IP for cross-device access
- Creates `logs/` directory for per-service log files
- Stores PIDs for clean shutdown
- Auto-opens browser after all services confirm running

### `stop-all.sh`

Reads stored PIDs from `logs/*.pid`, sends SIGTERM to each, cleans up PID files.

### `setup.sh`

First-time setup automation:
- Validates ACE-Step directory exists with Python venv
- Resolves absolute path for `ACESTEP_PATH`
- Generates `.env` file with correct paths and ports
- Installs frontend and server npm dependencies
- Runs database migration

## Backend Automation

### Job Queue (`server/src/services/acestep.ts`)

Sequential GPU job processing:
- GPU can only handle one generation at a time
- Jobs are queued with position tracking
- Automatic cleanup of jobs older than 1 hour (every 10 minutes)
- Dual-path execution: Gradio primary, Python spawn fallback

### Path Resolution

Cross-platform Python path finder (`resolvePythonPath()`):
- Checks `PYTHON_PATH` env var override
- Checks portable installation (`python_embeded/`)
- Scans common venv directories: `env/`, `.venv/`, `venv/`
- Windows/macOS/Linux compatible

### Database Migration (`server/src/db/migrate.ts`)

Auto-runs on server start. Creates tables for:
- Users and authentication
- Songs with full metadata
- Generation jobs with status tracking
- Playlists and playlist-song relationships
- Followers
- Reference tracks

### Scheduled Cleanup (`node-cron`)

Daily at 3 AM:
- `runCleanupJob()` -- removes orphaned data
- `cleanupDeletedSongs()` -- purges soft-deleted songs and their audio files

## Frontend Automation

### Vite Proxy Configuration

Automatic API routing without CORS issues:

```typescript
proxy: {
  '/api':    → http://127.0.0.1:3001
  '/audio':  → http://127.0.0.1:3001
  '/editor': → http://127.0.0.1:3001
  '/blog':   → http://127.0.0.1:3001
}
```

### Generation Status Polling

Frontend polls `GET /api/generate/status/:jobId` to track:
- Queue position
- ETA in seconds
- Generation stage description
- Progress percentage
- Final audio URLs on completion

## What Could Be Automated Next

| Mechanism | What It Does |
|-----------|-------------|
| **launchd plist** | Auto-start on macOS login |
| **Claude Code skill** | `/music "dreamy lo-fi piano"` generates directly from CLI |
| **Webhook** | POST to an endpoint triggers generation, returns audio URL |
| **Cron-based training** | Scheduled LoRA fine-tuning on liked songs |

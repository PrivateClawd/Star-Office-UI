# Star Office UI — Feature Overview

Star Office UI is a "pixel office" visualization interface that renders the status of AI assistants / multiple OpenClaw visitors as a small office scene viewable on the web (including mobile).

## What You See
- Pixel-art office background (top-down view)
- Characters (Star + visitors) move to different areas based on their status
- Names and bubbles display the current status/thoughts (customizable mapping)
- Mobile-friendly — works for demos, livestreams, and external presentations

## Core Features

### 1) Single Agent (Local Star) Status Rendering
- Backend reads `state.json` and serves `GET /status`
- Frontend polls `/status` and renders Star's position based on `state`
- Provides `set_state.py` for quick status switching

### 2) Multi-Visitor (Multi-Lobster) Office Join
- Visitors join via `POST /join-agent` and receive an `agentId`
- Visitors push their status via `POST /agent-push`
- Frontend fetches the visitor list via `GET /agents` and renders them

### 3) Join Key System
- Supports fixed, reusable join keys (e.g. `ocj_starteam01~08`)
- Per-key concurrency limit (default: 3)
- Controls "who can enter the office" and "how many lobsters per key"

### 4) Status-to-Area Mapping (Unified Logic)
- idle → breakroom
- writing / researching / executing / syncing → writing (workspace)
- error → error (bug corner)

### 5) Visitor Animations & Performance Optimization
- Visitor characters use animated sprites
- WebP assets supported (smaller size, faster loading)

### 6) Non-Overlapping Name/Bubble Layout
- Real visitors and demo visitors have separate layout logic
- Non-demo visitor names and bubbles are shifted upward
- Bubbles anchor above names to avoid covering them

### 7) Demo Mode (Optional)
- `?demo=1` to show demo visitors (hidden by default)
- Demo and real visitors don't interfere with each other

## Main Endpoints (Backend)
- `GET /`: Frontend page
- `GET /status`: Single agent status (backward compatible)
- `GET /agents`: Multi-agent list (for visitor rendering)
- `POST /join-agent`: Visitor joins
- `POST /agent-push`: Visitor pushes status
- `POST /leave-agent`: Visitor leaves
- `GET /health`: Health check

## Security & Privacy
- Do not put private information in `detail` (it gets rendered and can be fetched)
- Before open-sourcing: clean up logs, runtime files, join keys, tunnel output, etc.

## Art Asset License (Required)
- Code can be open-sourced, but art assets (backgrounds, characters, animations) are copyrighted by the original author/studio.
- Art assets are for learning and demo purposes only — **commercial use is prohibited**.

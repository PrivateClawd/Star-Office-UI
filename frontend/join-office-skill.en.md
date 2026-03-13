# Join Star Office - Visitor Agent Skill

## Description
Join a pixel-art office dashboard so your lobster has a workstation on the board, showing its work status in real time.

## Prerequisites
- You need a join key, format: `ocj_xxx`
- Your user has authorized you to push status
- You can access `https://office.hyacinth.im`

## Quick Start

### Step 1: Confirm authorization with your user
> "I will periodically send my work status (idle/writing/researching/executing/syncing/error) to the pixel office dashboard. Only status words and short descriptions are pushed — no private content. Can be stopped at any time. Do you authorize this?"

### Step 2: Download the push script
```bash
curl -o office-agent-push.py https://office.hyacinth.im/static/office-agent-push.py
```

### Step 3: Configure and run
Open `office-agent-push.py` and fill in these 3 variables:

```python
JOIN_KEY = "your-join-key"        # e.g. ocj_starteam02
AGENT_NAME = "your-name"          # e.g. "Alice's Lobster"
OFFICE_URL = "https://office.hyacinth.im"
```

Then run:
```bash
python3 office-agent-push.py
```

### Step 4: Done!
The script will automatically:
- Join the office (auto-approved, no waiting)
- Read your local status and push it every 15 seconds
- Your lobster will appear on the office dashboard, moving to different areas based on its status

## Status-to-Area Mapping
| Status | Office Area | Description |
|--------|-------------|-------------|
| idle | Breakroom (sofa) | Standing by / task complete |
| writing | Workspace (desk) | Writing code / documents |
| researching | Workspace | Searching / researching |
| executing | Workspace | Running tasks |
| syncing | Workspace | Syncing data |
| error | Bug Corner | Error / exception |

## Local Status Discovery Priority
The script auto-discovers your status source in this order (no manual config needed):
1. `state.json` (local OpenClaw workspace, auto-discovers multiple candidate paths)
2. `http://127.0.0.1:19000/status` (local HTTP endpoint)
3. Default fallback: idle

If your status file path is unusual, specify it with an environment variable:
```bash
OFFICE_LOCAL_STATE_FILE=/your/path/state.json python3 office-agent-push.py
```

## Stop Pushing
- `Ctrl+C` to stop the script
- The script will automatically leave the office on exit

## Notes
- Only status words and short descriptions are pushed — no private content
- Authorization expires after 24h, requires re-joining
- If you receive 403 (key expired) or 404 (removed), the script auto-stops
- Each key supports up to 100 concurrent lobsters online

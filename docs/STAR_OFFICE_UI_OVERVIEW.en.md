# Star Office UI — Feature Overview

Star Office UI is a pixel-art office dashboard that visualizes AI agent work status in real time. Agents move between office zones (breakroom, desk, bug corner) based on their state. The dashboard supports multi-agent rendering, trilingual UI (CN/EN/JP), customizable art assets, AI-generated room backgrounds (Gemini), and mobile-responsive layout.

## What You See

- Pixel-art office background (top-down RPG style, 1280×720 canvas)
- Characters move between areas based on their status — idle agents sit on the sofa, working agents sit at the desk, errored agents go to the bug corner
- Names and speech bubbles display current status and activity
- Interactive decorations: click plants, poster, cat, or flowers to cycle their animations
- Mobile-friendly — works on phones and tablets for quick status checks

## Core Features

### 1) Agent Status Rendering

The backend reads agent state and serves it via `GET /status` (main agent) and `GET /agents` (all agents). The frontend polls these endpoints every 2–2.5 seconds and renders agents at the correct office zone.

Six states map to three office areas:

| State | Office Area | Description |
|-------|-------------|-------------|
| `idle` | Breakroom (sofa) | Standing by, no active task |
| `writing` | Desk (workspace) | Composing / organizing documents |
| `researching` | Desk (workspace) | Searching for information |
| `executing` | Desk (workspace) | Running a task |
| `syncing` | Desk (workspace) | Syncing / deploying |
| `error` | Bug corner | Something went wrong |

State normalization aliases: `working`/`busy`/`write` → `writing`, `run`/`running`/`execute`/`exec` → `executing`, `sync` → `syncing`, `research`/`search` → `researching`. Unknown states default to `idle`.

Working states auto-reset to `idle` after a configurable TTL (default 300 seconds).

### 2) Multi-Agent Office

Multiple agents can appear in the same office simultaneously. Each agent has:

- Independent avatar (main star sprite or one of 6 guest avatar styles)
- Name tag
- Status dot colored by auth status: green (approved), yellow (pending), red (rejected), gray (offline)
- Position assigned by area + slot index to prevent overlap

Remote agents join via `POST /join-agent` with a join key, then push status updates via `POST /agent-push` every 15 seconds. The frontend renders all agents from the `GET /agents` endpoint.

### 3) Join Key System

Controls who can enter the office. Keys are stored in `join-keys.json` (auto-generated from `join-keys.sample.json` on first start).

| Property | Type | Description |
|----------|------|-------------|
| `key` | string | Join key string (e.g. `ocj_starteam01`) |
| `reusable` | boolean | Can be reused after agent leaves |
| `maxConcurrent` | number | Max concurrent agents per key (default 3) |
| `expiresAt` | ISO 8601 / null | Key expiration date |

Default keys: `ocj_starteam01` through `ocj_starteam08`, each allowing up to 3 concurrent agents. "Active" is defined as `authStatus == "approved"` AND last push/update within 300 seconds.

### 4) Asset Management

The right-side asset drawer (320px wide, 92vw on mobile) provides full control over office art:

- **Asset list** — view all frontend assets with visibility toggles
- **Position editing** — adjust X/Y coordinates and scale for any asset
- **Upload** — replace assets with custom artwork (PNG, WEBP, GIF, SVG, AVIF, JPG)
- **Spritesheet conversion** — auto-convert animated GIF/WEBP to spritesheets on upload
- **Restore** — revert to `.default` snapshot or `.bak` backup
- **Password protection** — drawer requires authentication (default password: `1234`)

### 5) AI Background Generation (Gemini)

Generate RPG-style pixel-art backgrounds using the Gemini API:

- **Two speed modes:** Fast (1152×648, quick turnaround) and Quality (1280×720, full resolution)
- **Random themes:** 8-bit dungeon guild room, stardew-valley tavern, nordic fantasy tavern, magitech workshop, elven forest inn, pixel cyber tavern, desert caravan inn, snow mountain lodge
- **Custom prompts:** enter your own description for fully custom backgrounds
- **Async generation:** background is generated in a worker thread to avoid timeouts; the frontend polls for progress
- **Model options:** `nanobanana-pro` (nano-banana-pro-preview) or `nanobanana-2` (gemini-2.5-flash-image)

Configure the Gemini API key and model via the drawer's Gemini settings section or the `POST /config/gemini` endpoint.

### 6) Home Favorites

Save up to 30 generated or custom backgrounds as favorites:

- **Save current** — snapshot the active background
- **Apply** — switch to any saved favorite
- **Delete** — remove favorites you no longer want

### 7) Yesterday Memo

Automatically reads the most recent daily memo from `../memory/YYYY-MM-DD.md`, sanitizes sensitive content (OpenID tokens, file paths, IPs, emails, phone numbers), and displays it as a "Yesterday's Memo" card. Long content is truncated with a random wisdom quote appended.

### 8) Trilingual UI (i18n)

Three languages available: Chinese (`zh`), English (`en`), Japanese (`ja`).

- Language preference saved in `localStorage` as `uiLang`
- Switch via language buttons in the bottom-right corner
- All UI strings, bubbles, loading messages, and status text update instantly
- The `I18N` object and `t(key)` function handle translations

### 9) Mobile Layout

Responsive design triggered at `max-width: 900px` or `pointer: coarse`:

- Game canvas: 100vw × 66.666vh (upper portion)
- Control panels: 100vw × 33.334vh (stacked below)
- Touch-friendly: 44px minimum button height
- Asset drawer: 92vw with backdrop overlay and body scroll lock
- Two-column guest agent grid

### 10) Animations

| Key | Source | Frame Size |
|-----|--------|-----------|
| `star_idle` | star-idle-spritesheet | 128×128 |
| `star_working` | star-working-spritesheet-grid | 230×144 |
| `star_researching` | star-researching-spritesheet | 128×105 |
| `sofa_busy` | sofa-busy-spritesheet | 256×256 |
| `error_bug` | error-bug-spritesheet-grid | 180×180 |
| `sync_anim` | sync-animation-spritesheet-grid | 256×256 |
| `cats` | cats-spritesheet | 160×160 |
| `plants` | plants-spritesheet | 160×160 |
| `posters` | posters-spritesheet | 160×160 |
| `coffee_machine` | coffee-machine-spritesheet | 230×230 |
| `serverroom` | serverroom-spritesheet | 180×251 |
| `flowers` | flowers-spritesheet | 65×65 |

Guest avatars: `guest_anim_1.webp` through `guest_anim_6.webp` and `guest_role_1.png` through `guest_role_6.png`.

### 11) Desktop Pet Mode (Optional)

An Electron-based desktop wrapper (`desktop-pet/`) turns the office into a transparent desktop pet window. Starts the Python backend automatically and loads the UI in a frameless window.

## API Endpoints

### Core State

| Method | Path | Description |
|--------|------|-------------|
| GET | `/status` | Main agent state |
| POST | `/set_state` | Set main agent state |
| GET | `/agents` | All agents with auto-cleanup |
| GET | `/health` | Health check |

### Multi-Agent

| Method | Path | Description |
|--------|------|-------------|
| POST | `/join-agent` | Guest agent joins the office |
| POST | `/agent-push` | Guest agent pushes status update |
| POST | `/leave-agent` | Guest agent leaves |
| POST | `/agent-approve` | Approve a pending guest |
| POST | `/agent-reject` | Reject a pending guest |

### Content

| Method | Path | Description |
|--------|------|-------------|
| GET | `/yesterday-memo` | Sanitized yesterday memo |

### Asset Management

| Method | Path | Description |
|--------|------|-------------|
| POST | `/assets/auth` | Authenticate for the asset drawer |
| GET | `/assets/auth/status` | Check drawer auth status |
| GET | `/assets/list` | List all frontend assets |
| GET | `/assets/positions` | Get asset positions |
| POST | `/assets/positions` | Set asset position |
| GET | `/assets/defaults` | Get default positions |
| POST | `/assets/defaults` | Set default position |
| POST | `/assets/upload` | Upload a new asset |
| POST | `/assets/restore-default` | Restore asset from `.default` |
| POST | `/assets/restore-prev` | Restore asset from `.bak` |

### AI Background Generation

| Method | Path | Description |
|--------|------|-------------|
| POST | `/assets/generate-rpg-background` | Start async background generation |
| GET | `/assets/generate-rpg-background/poll` | Poll generation progress |
| POST | `/assets/restore-reference-background` | Restore original reference background |
| POST | `/assets/restore-last-generated-background` | Restore last AI-generated background |

### Home Favorites

| Method | Path | Description |
|--------|------|-------------|
| GET | `/assets/home-favorites/list` | List saved favorites (max 30) |
| POST | `/assets/home-favorites/save-current` | Save current background |
| POST | `/assets/home-favorites/apply` | Apply a saved favorite |
| POST | `/assets/home-favorites/delete` | Delete a favorite |

### Configuration

| Method | Path | Description |
|--------|------|-------------|
| GET | `/config/gemini` | Get Gemini API config (masked key, model) |
| POST | `/config/gemini` | Set Gemini API key and model |

### Pages

| Method | Path | Description |
|--------|------|-------------|
| GET | `/` | Main pixel office UI |
| GET | `/join` | Agent join form |
| GET | `/invite` | Invite instructions |
| GET | `/electron-standalone` | Electron standalone variant |

## Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `STAR_BACKEND_PORT` | `19000` | Flask server port |
| `FLASK_SECRET_KEY` / `STAR_OFFICE_SECRET` | `star-office-dev-secret` | Flask session secret (production: ≥ 24 chars) |
| `ASSET_DRAWER_PASS` | `1234` | Asset drawer password (production: ≥ 8 chars, not `1234`) |
| `GEMINI_API_KEY` / `GOOGLE_API_KEY` | — | Gemini API key for image generation |
| `GEMINI_MODEL` | `nanobanana-pro` | Gemini model for image generation |
| `AUTO_ROTATE_HOME_ON_PAGE_OPEN` | `0` | Rotate home background on page open |
| `AUTO_ROTATE_MIN_INTERVAL_SECONDS` | `60` | Min interval between auto-rotations |
| `STAR_OFFICE_STATE_FILE` | `state.json` | Path to main state file |
| `STAR_OFFICE_ENV` / `FLASK_ENV` | — | Set to `production` for security hardening |
| `OPENCLAW_WORKSPACE` | auto-detected | OpenClaw workspace path for memo discovery |

## Security

- **Asset drawer authentication:** password-based, session-stored (`asset_editor_authed`), HttpOnly + SameSite=Lax cookies, 12-hour session lifetime
- **Production mode:** `FLASK_SECRET_KEY` must be ≥ 24 chars with no weak markers; `ASSET_DRAWER_PASS` must be ≥ 8 chars and not `1234`; startup fails if checks fail
- **Runtime config:** `runtime-config.json` is `chmod 0600`
- **Content sanitization:** yesterday memos are stripped of OpenID tokens, file paths, IPs, emails, and phone numbers

## PrivateClawd Integration

When Star Office is deployed as part of [PrivateClawd](https://privateclawd.com), additional integration features are enabled:

- **Multi-tenancy:** State endpoints (`/status`, `/agents`) are intercepted by the Next.js proxy and return per-user data from PostgreSQL instead of the Flask file-based state
- **Write endpoint blocking:** `/set_state`, `/agent-push`, `/join-agent`, `/leave-agent` return no-op success responses since agent states are managed by the bot lifecycle
- **Authenticated reverse proxy:** All requests go through `/api/star-office/proxy/` with full session authentication
- **URL rewriting:** Root-relative paths in the HTML are rewritten to the proxy prefix so the iframe works correctly
- **Bot status sync:** Bot lifecycle events (deploy, start, stop, error) automatically update the corresponding Star Office agent state via `syncStarOfficeAgent()`
- **Per-agent opt-in:** Only agents with the `star-office` skill appear in the office
- **Dashboard integration:** Sidebar link, quick-launch card on the main dashboard, and iframe view with toolbar (agent count, refresh, fullscreen, new tab)

## Art Asset License

- **Code:** MIT license
- **Art assets:** No commercial use — character/scene assets must be replaced with original artwork for commercial deployment
- Main character sprites use copyright-free cat artwork
- Guest character animations use free assets from [LimeZu](https://limezu.itch.io/animated-mini-characters-2-platform-free) — retain attribution in redistributions

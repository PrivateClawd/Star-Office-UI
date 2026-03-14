# Star Office UI — New Features (v2 Release & PrivateClawd Integration)

## Phase 1: v2 Rewrite (2026-03-01)

### 1. Multi-Agent Visitor System
- Multiple remote OpenClaw agents can join the same office simultaneously.
- Visitors have independent avatars, names, status, areas, and speech bubbles.
- Dynamic online/offline status with real-time updates.
- 6 guest avatar styles with animated sprites (`guest_anim_1~6`).

### 2. Join Key System Upgrade
- Upgraded from "single-use keys" to "fixed reusable keys".
- Default keys: `ocj_starteam01` ~ `ocj_starteam08`.
- Per-key concurrency limit `maxConcurrent` (default: 3).
- Key expiration support via `expiresAt` field.

### 3. Concurrency Control (Race Condition Fixed)
- Fixed race condition during concurrent joins.
- The 4th concurrent join on the same key is correctly rejected (HTTP 429).

### 4. Visitor Status Mapping & Area Rendering
- `idle` → breakroom, `writing/researching/executing/syncing` → writing, `error` → error.
- State normalization aliases (e.g. `working` → `writing`, `run` → `executing`).
- Visitor bubble text syncs with status — no more mismatches.

### 5. Visitor Animations & Asset Optimization
- Visitors upgraded from static images to animated sprites (pixel art style).
- `guest_anim_1~6` available in WebP format, reducing file size.
- Guest role icons: `guest_role_1~6` in PNG.

### 6. Name & Bubble Display Optimization
- Non-demo visitor names and bubbles shifted upward to prevent character overlap.
- Bubble anchor point changed to name-based positioning, ensuring "bubble always above name".

### 7. Mobile Display
- Responsive layout at `max-width: 900px` or `pointer: coarse`.
- Game canvas: 100vw × 66.666vh; control panels: 100vw × 33.334vh.
- Touch-friendly: 44px minimum button height, 2-column guest grid.
- Asset drawer: 92vw with backdrop overlay and body scroll lock.

### 8. Remote Push Script Improvements
- Supports reading from state file and pushing status to the office.
- Added status source diagnostic logging (to debug "why always idle").
- Fixed `AGENT_NAME` environment variable override timing issue.

### 9. Trilingual UI (i18n)
- Three languages: Chinese (zh), English (en), Japanese (ja).
- `I18N` object with all UI strings; `t(key)` translation function.
- Language preference persisted in `localStorage`.
- Language switcher buttons in the bottom-right of the UI.

## Phase 2: Asset Management & AI Generation (2026-03-01 ~ 03-05)

### 10. Asset Drawer (Sidebar)
- Right-side panel (320px wide) with password protection.
- Full asset list with visibility toggles and position editing (X, Y, scale).
- Upload custom assets with automatic spritesheet conversion for animated files.
- Restore assets to `.default` snapshot or `.bak` backup.
- Mobile: 92vw width, backdrop overlay, scroll lock.

### 11. AI Background Generation (Gemini)
- Generate RPG-style pixel-art backgrounds via the Gemini API.
- Two speed modes: Fast (1152×648) and Quality (1280×720).
- 8 random themes: dungeon guild room, stardew-valley tavern, nordic fantasy tavern, magitech workshop, elven forest inn, pixel cyber tavern, desert caravan inn, snow mountain lodge.
- Custom prompts supported for fully personalized backgrounds.
- Async generation in worker threads to avoid Cloudflare 524 timeouts.
- Model options: `nanobanana-pro` and `nanobanana-2`.

### 12. Home Favorites
- Save up to 30 generated or custom backgrounds as favorites.
- Apply, delete, or save-current operations.

### 13. Gemini Configuration UI
- Configure API key and model from the asset drawer.
- Masked key display for security.
- Runtime config stored in `runtime-config.json` (chmod 0600).

## Phase 3: Security & Stability (2026-03-04 ~ 03-06)

### 14. Production Security Hardening
- `FLASK_SECRET_KEY` must be ≥ 24 chars in production.
- `ASSET_DRAWER_PASS` must be ≥ 8 chars and not `1234`.
- Startup fails immediately if these checks fail.
- Session config: HttpOnly, SameSite=Lax, Secure in production, 12h lifetime.

### 15. CDN Cache Fix
- Explicit `no-store` headers on 404 responses to prevent CDN caching of missing assets.

### 16. Backend Module Refactoring
- Backend split into focused modules (`memo_utils.py`, etc.) for maintainability.

### 17. Auto-Idle TTL
- Working states auto-reset to `idle` after `ttl_seconds` (default 300s) if stale.
- Prevents agents from appearing permanently "busy" after a crash.

### 18. Port Change
- Default port changed from 18791 to 19000 to avoid OpenClaw Browser Control port conflicts.

## Phase 4: PrivateClawd Integration (2026-03 ~)

### 19. Multi-Tenancy via Proxy
- Star Office runs as a Docker Compose sidecar service.
- Next.js authenticated reverse proxy at `/api/star-office/proxy/`.
- State endpoints (`/status`, `/agents`) intercepted to return per-user data from PostgreSQL.
- Write endpoints (`/set_state`, `/agent-push`, `/join-agent`, `/leave-agent`) return no-op success.
- Static assets, config, and generation endpoints forwarded to the Flask backend.

### 20. URL Rewriting
- All root-relative URLs in the HTML response are rewritten to the proxy prefix.
- Patterns: `/static/` → `/api/star-office/proxy/static/`, `fetch('/` → `fetch('/api/star-office/proxy/`.

### 21. Cookie Forwarding
- Proxy forwards `cookie` headers to Flask and `set-cookie` headers back.
- Enables the asset drawer session authentication to work through the iframe.

### 22. Prisma Data Model
- `StarOfficeAgent` model with `userId`, `botId`, `agentName`, `status`, `statusMessage`, `avatarKey`.
- Unique constraint on `[userId, botId]`.
- `StarOfficeStatus` enum: `idle`, `writing`, `researching`, `executing`, `syncing`, `error`.

### 23. Bot Status Sync
- `syncStarOfficeAgent()` called on deploy, start, stop, error, and status drift.
- Maps `BotStatus` → `StarOfficeStatus`: `running` → `writing`, `deploying/starting/stopping` → `syncing`, `stopped` → `idle`, `error` → `error`.
- `draft` and `deleting` bots are hidden (agent record deleted).

### 24. Per-Agent Opt-In
- Only agents with the `star-office` skill enabled appear in the office.
- Guided empty state when no agents have the skill.

### 25. Dashboard Integration
- Sidebar navigation link ("Star Office" with Building2 icon).
- Quick-launch card on the main dashboard for users with deployed agents.
- `StarOfficeView` component: iframe with toolbar (agent count badge, refresh, fullscreen, new tab).

### 26. Docker Configuration
- `Dockerfile.star-office`: Python 3.12-slim, Flask + Pillow.
- Docker Compose service with health check, volume persistence, and `depends_on` for startup ordering.
- Environment variable passthrough: `STAR_BACKEND_PORT`, `FLASK_SECRET_KEY`, `ASSET_DRAWER_PASS`.

### 27. Yesterday Memo Integration
- Reads from `../memory/YYYY-MM-DD.md` relative to the star-office root.
- Content sanitization: redacts OpenID tokens, file paths, IPs, emails, phone numbers.
- Truncation with random wisdom quotes for long content.

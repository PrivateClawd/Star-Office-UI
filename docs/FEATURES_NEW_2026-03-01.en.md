# Star Office UI — New Features (This Phase)

## 1. Multi-Lobster Visitor System
- Multiple remote OpenClaw agents can join the same office simultaneously.
- Visitors have independent avatars, names, status, areas, and speech bubbles.
- Dynamic online/offline status with real-time updates.

## 2. Join Key System Upgrade
- Upgraded from "single-use keys" to "fixed reusable keys".
- Default keys: `ocj_starteam01` ~ `ocj_starteam08`.
- Security controls retained: per-key concurrency limit `maxConcurrent` (default: 3).

## 3. Concurrency Control (Race Condition Fixed)
- Fixed race condition during concurrent joins.
- The 4th concurrent join on the same key is correctly rejected (HTTP 429).

## 4. Visitor Status Mapping & Area Rendering
- `idle -> breakroom`
- `writing/researching/executing/syncing -> writing`
- `error -> error`
- Visitor bubble text syncs with status — no more mismatches.

## 5. Visitor Animations & Asset Optimization
- Visitors upgraded from static images to animated sprites (pixel art style).
- `guest_anim_1~6` now available in WebP, reducing file size.

## 6. Name & Bubble Display Optimization
- Non-demo visitor names and bubbles shifted upward to prevent character overlap.
- Bubble anchor point changed to name-based positioning, ensuring "bubble always above name".

## 7. Mobile Display
- Pages are accessible and viewable on mobile devices.
- Basic mobile layout adaptation for demo/presentation scenarios.

## 8. Remote Push Script Improvements
- Supports reading from state file and pushing status to the office.
- Added status source diagnostic logging (to debug "why always idle").
- Fixed AGENT_NAME environment variable override timing issue.

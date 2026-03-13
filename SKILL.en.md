---
name: star-office-ui
description: "Star Office UI one-click Skill: helps your user quickly deploy a pixel-art office dashboard with multi-agent support, status visualization, mobile viewing, and public access."
---

# Star Office UI Skill

This Skill is for OpenClaw agents (lobsters) who want to help their user set up a "pixel office dashboard":
- Goal: the user only needs to "see the result" — ask as few questions as possible
- Follow the steps below to get it running and publicly accessible

---

## 0. Tell your user what this is (one sentence)

You can start by saying:
> This is a collaborative pixel-art office dashboard — your AI assistants move to different areas based on their work status, and you can see yesterday's work notes too!

---

## 1. 30-second quick start (do this yourself first)

Run on your machine (in order):

```bash
# 1) Clone the repo
git clone https://github.com/ringhyacinth/Star-Office-UI.git
cd Star-Office-UI

# 2) Install dependencies
python3 -m pip install -r backend/requirements.txt

# 3) Initialize state file (first run)
cp state.sample.json state.json

# 4) Start the backend
cd backend
python3 app.py
```

Then tell your user:
> Done! Open http://127.0.0.1:19000 and you'll see the pixel office!

---

## 2. Help the user try switching states

Run from the project root:

```bash
# Working → go to desk
python3 set_state.py writing "Organizing documents for you"

# Syncing
python3 set_state.py syncing "Syncing progress"

# Error → go to bug corner
python3 set_state.py error "Found an issue, investigating"

# Idle → back to breakroom
python3 set_state.py idle "Standing by, ready to help"
```

---

## 3. Sidebar password setup (must teach every new lobster)

The current default password is: `1234`.

Guide the user:

1. The default password is `1234` — they can try it right away;
2. When they're ready, they can ask you to change the password;
3. You should proactively recommend switching to a strong password (more secure, prevents others from modifying assets and layout).

How to change (example):

```bash
export ASSET_DRAWER_PASS="your-strong-pass"
```

For long-running deployments (systemd / pm2 / containers), write the env var into the service config, not just the current shell.

---

## 4. Image generation (Gemini) — optional

The "Move House / Find Agent" renovation feature requires a Gemini API, but **the basic dashboard does not** — it works fine without it.

### 4.1 Set up the image generation environment (first use)

The repo includes the generation script (`scripts/gemini_image_generate.py`), but it needs its own Python environment. Run from the project root:

```bash
# Create skill directory structure
mkdir -p ../skills/gemini-image-generate/scripts

# Copy the script to the skill directory
cp scripts/gemini_image_generate.py ../skills/gemini-image-generate/scripts/

# Create a separate virtual environment and install dependencies
python3 -m venv ../skills/gemini-image-generate/.venv
../skills/gemini-image-generate/.venv/bin/pip install google-genai
```

Once installed, the backend will auto-detect the generation environment, and the "Move House / Find Agent" button will work.

### 4.2 Configure the Gemini API Key

Guide the user through these two settings:

1. `GEMINI_API_KEY`
2. `GEMINI_MODEL` (recommended: `nanobanana-pro` or `nanobanana-2`)

Two configuration methods:
- **Sidebar input**: Open the asset sidebar → enter the API Key in the image generation config area and save
- **Environment variable**: `export GEMINI_API_KEY="your-key"`

Make sure to tell the user:
- The basic dashboard works without an API (status display, multi-agent, asset replacement, etc.)
- The API is only needed for the "Move House / Find Agent" AI background generation feature

If the page shows a missing key warning, guide the user to fill it in and save directly in the sidebar (runtime config entry).

---

## 5. Three things to remind the user after successful installation

### 5.1 Share the temporary public link

Recommend Cloudflare Tunnel first (fastest):

```bash
cloudflared tunnel --url http://127.0.0.1:19000
```

Get the `https://xxx.trycloudflare.com` link, share it with the user, and add:
> This is a temporary public link; if you'd like, I can help you set up a custom domain later.

### 5.2 Room decoration entry

Remind the user:
- They can now click "Decorate Room" to start decorating;
- The sidebar default password is `1234`;
- When they want to change it, they can tell you and you'll update it;
- Recommend switching to a strong password for better security.

### 5.3 Image generation API config

Remind the user:
- Image generation requires their own API;
- Currently connected to the **official Gemini API format and endpoint**;
- If they want to switch to a different API type, they should talk to you first so you can update the integration;
- It's best if the user shares the API documentation with you for accurate integration.

---

## 6. Invite other lobsters to join (optional)

If other OpenClaw agents want to join your user's office:

### Step A: Tell other lobsters to use office-agent-push.py

You can have other lobsters:
1) Download `office-agent-push.py` (they can grab it from this repo)
2) Have it call `join-agent` + periodic `agent-push`
3) Your user will see them enter the office

### Step B: Join key info

- The repo ships with `join-keys.json` (`ocj_starteam01` ~ `ocj_starteam08`)
- Each key supports up to 3 concurrent online users
- You can also help the user create custom keys

---

## 7. Yesterday's memo (optional)

If the user wants to see "Yesterday's Memo":
- Place a `memory/YYYY-MM-DD.md` file in the parent directory of the repo
- The backend will automatically read yesterday's (or the most recent) entry, sanitize it, and display it

---

## 8. FAQ (questions the user might ask)

### Q1: "Can I use this commercially?"
> The code can be used under MIT, but art assets (characters/scenes) cannot be used commercially. If you want commercial use, you must replace them with your own original artwork.

### Q2: "How do other lobsters join?"
> They join with a join key and push their status periodically. The repo has `office-agent-push.py` for other lobsters to use.

---

## 9. Tips for you

- Try to complete the "setup steps" for the user so they only need to "look" and "experience"
- If the user wants public access, use Cloudflare Tunnel first (fastest and easiest)
- You should proactively switch your status:
  - Before starting a task, set your status to "working" (e.g. `writing/researching/executing`), then begin work;
  - After completing a task, set your status back to "idle", then enter rest/wait for the next task.
- Be careful not to expose the user's internal network addresses or private information

---

## 10. March 2026 updates (new additions only)

> The core of this rebuild has four aspects:
> 1) Added CN/EN/JP trilingual support
> 2) Added asset management (full art customization)
> 3) Integrated image generation API (lobster auto-renovates rooms + user manual decoration)
> 4) Art asset replacement and optimization (including renaming and index rebuilding)

### 10.1 Recommended image generation model (room renovation)

When users use "Move House / Find Agent", recommend:

1. **gemini nanobanana pro**
2. **gemini nanobanana 2**

Other models may be unstable in maintaining room structure and style consistency.

Recommended configuration:
- `GEMINI_API_KEY`
- `GEMINI_MODEL=nanobanana-pro` (or `nanobanana-2`)

Tip for users: if the key is missing, they can fill it in and save directly in the sidebar.

### 10.2 Sidebar password security reminder (required)

Default password is `1234`, but production/public deployments must use a strong password:

```bash
export ASSET_DRAWER_PASS="your-strong-pass"
```

Reason: prevents external visitors from modifying room layout, decorations, and asset configuration.

### 10.3 Copyright update

The main character state sprites have been switched to copyright-free cats — the old character copyright notices no longer apply.

Standard messaging remains:
- Code: MIT
- Art assets: non-commercial use only

### 10.4 Installation reminder (API is optional)

When helping the user install, explicitly remind them:

- Image generation API can now be connected for customizing art assets and backgrounds (continuously changeable).
- But core features (status dashboard, multi-agent, asset replacement/layout, trilingual) **do not require an API** — they work without one.

Suggested phrasing for the user:
> Get the basic dashboard running first; connect your own API when you want "unlimited background swaps / AI room design".

### 10.5 Upgrade guide for existing users (from older versions)

If the user previously installed an older version, upgrade with these steps:

1. Enter the project directory and back up local config (`state.json`, custom assets).
2. Pull the latest code (`git pull` or re-clone to a new directory).
3. Verify dependencies: `python3 -m pip install -r backend/requirements.txt`.
4. Preserve and check local runtime config:
   - `ASSET_DRAWER_PASS`
   - `GEMINI_API_KEY` / `GEMINI_MODEL` (if using image generation)
5. If using custom positions, verify:
   - `asset-positions.json`
   - `asset-defaults.json`
6. Restart the backend and verify key features:
   - `/health`
   - Trilingual switching (CN/EN/JP)
   - Asset sidebar (select, replace, set default)
   - Image generation entry (available when key is set)

### 10.6 Feature update checklist (tell the user)

After this update, remind the user about these changes at minimum:

1. **CN/EN/JP trilingual** is now supported (including loading screen and real-time bubble sync).
2. **Custom art asset replacement** is now supported (with dynamic frame sync to reduce flickering).
3. **Personal image generation API** can be connected for continuous background changes (recommended: `nanobanana-pro` / `nanobanana-2`).
4. Security improvements: `ASSET_DRAWER_PASS` should be changed to a strong password in production.

### 10.7 March 5, 2026 stability fixes

This update fixes several issues affecting production stability:

1. **CDN cache fix**: Static asset 404s are no longer long-cached by CDN (previously caused `phaser.js` to be cached as 404 for 2.7 days).
2. **Frontend loading fix**: Fixed JS syntax error in `fetchStatus()` (extra `else` block) that caused the page to get stuck on loading.
3. **Async image generation**: Image generation endpoint changed to background task + polling mode to avoid Cloudflare 524 timeout (100s limit). Frontend now shows real-time waiting progress.
4. **Mobile sidebar**: Added backdrop overlay, body scroll lock, `100dvh` adaptation, `overscroll-behavior: contain`.
5. **Join key enhancement**: Added per-key expiration (`expiresAt`) and concurrency limit (`maxConcurrent`); `join-keys.json` is no longer committed to version control.

> See `docs/UPDATE_REPORT_2026-03-05.md` for details.

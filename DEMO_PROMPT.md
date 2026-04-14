# 🎯 LIVE DEMO PROMPT
# Paste this exactly into Claude Code to kick off the build.
# Do not modify it live — it's already been tested to produce the right output.

---

Read CLAUDE.md and agents.md first. Then build the full YouTube Channel Analyser app exactly as specified.

Build in this order:
1. `package.json` with dependencies: express, axios, dotenv, @anthropic-ai/sdk
2. `server.js` — Express backend with YouTube API calls and /analyze endpoint
3. `prompt.md` — Claude analysis prompt with all 8 report sections
4. `index.html` — Single file dark-theme UI with channel ID input, skeleton loader, and report cards

Requirements:
- The app must run with `node server.js` on port 3000
- No build step, no TypeScript, no React
- All API keys from .env only
- The /analyze endpoint must fetch: channel stats, last 10 videos, top 20 comments from 3 most viewed videos, and transcripts from top 3 videos
- UI must show a pulsing skeleton loader while the analysis runs
- Each report section renders as its own card
- Handle errors gracefully — show a friendly message in the UI if something fails

When done, tell me exactly what command to run to start the app.

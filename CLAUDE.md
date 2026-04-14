# CLAUDE.md — YouTube Channel Analyser

## What This Is
A local web app that takes a YouTube Channel ID, fetches channel stats, top videos, comments, and transcripts via the YouTube Data API v3, and generates a deep analysis report rendered in a clean UI.

## Stack
- **Backend:** Node.js (Express) — single `server.js` file
- **Frontend:** Single `index.html` file with vanilla JS + Tailwind CDN
- **API:** YouTube Data API v3
- **AI Analysis:** Claude API (claude-sonnet-4-20250514)
- **No build step.** No React. No bundler. Runs with `node server.js`.

## Project Structure
```
youtube-analyser/
├── server.js        ← Express backend, all YouTube API calls
├── index.html       ← Frontend UI, single file
├── prompt.md        ← Claude prompt template for analysis
├── .env             ← API keys (never commit)
└── .env.example     ← Key names only
```

## What the App Does
1. User enters a YouTube Channel ID in the UI
2. Backend fetches:
   - Channel stats (name, subs, total views, video count)
   - Last 10 videos (title, views, likes, comments count)
   - Top 20 comments from the 3 most viewed videos
   - Transcripts (captions) from the top 3 videos if available
3. All data is sent to Claude with the prompt from prompt.md
4. Claude returns a structured analysis report
5. Report renders in the UI — no page reload

## API Endpoints
- `GET /` → serves index.html
- `POST /analyze` → accepts `{ channelId }`, returns analysis report

## Environment Variables
- `YOUTUBE_API_KEY` — YouTube Data API v3 key
- `ANTHROPIC_API_KEY` — Anthropic API key

## Conventions
- No TypeScript. Plain JS only.
- No npm frameworks beyond `express`, `axios`, `dotenv`, and `@anthropic-ai/sdk`
- All YouTube API calls in `server.js` — never from the frontend
- Keep `index.html` self-contained — no external JS files
- Error handling on every API call — show friendly errors in the UI, never crash

## Never Do
- Never expose API keys to the frontend
- Never use a database — this is stateless
- Never add a build step

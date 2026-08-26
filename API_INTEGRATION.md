# CricketLive World — Real API Integration

## What changed

All fake/demo cricket data has been removed. Every screen now fetches real data:

| Screen | Real data source |
|---|---|
| Live Matches, Upcoming, Completed | `currentMatches` (CricAPI) |
| Match Details → Overview | `match_info` |
| Match Details → Scorecard | `match_scorecard` |
| Match Details → Commentary | *(honest "not available" — see below)* |
| Match Details → Stats | *(honest "not available" — see below)* |
| Series list & points/schedule | `series`, `series_info` |
| Teams | `countries` |
| Players & search | `players`, `players_info` |
| Global search | `series` + `players` search |

Nothing is invented. Where the free API tier doesn't provide a field (ball-by-ball commentary, advanced live stats like required run rate), the app shows a clearly worded "not available" state instead of a fake number — see `CommentaryTab()` and `StatsTab()` in `app.js`.

Two things I deliberately **removed** rather than fake:
- The old "International" vs "League" filter chips — CricAPI doesn't reliably flag this, so guessing from series names would've been invented data. Format filters (T20/ODI/Test/Women's) are still there because those come straight from the API.
- The "Videos" tab on team pages — CricAPI has no video content, so I removed it instead of showing made-up highlight titles.

## 1. Recommended API: CricAPI

**Why:** best balance of coverage and a genuinely usable free tier for a project like this.

- **Free tier:** 100 requests/day, includes `currentMatches`, `match_info`, `match_scorecard`, `series`, `players`, `countries` — everything this app's screens need except ball-by-ball commentary and live run-rate stats.
- **Paid tiers:** raise the daily request cap substantially and add the ball-by-ball/fantasy endpoints. Pricing and exact limits are on their site and change over time, so check https://cricapi.com/ directly before committing.
- **Sign up:** https://cricapi.com/ → free account → copy your API key from the dashboard.

If you outgrow CricAPI later, only `server/server.js` needs to change (the `proxyToCricAPI` calls and route handlers) — the frontend never touches the provider directly, so swapping providers doesn't touch `app.js` at all.

## 2. Where to add your API key — exactly

**Only one place, and it's on the server, never in the app:**

```
CricketLiveWorld/server/.env.example   ← copy this file
CricketLiveWorld/server/.env           ← paste your real key here (create this file)
```

```bash
cd CricketLiveWorld/server
cp .env.example .env
```

Then open `.env` and replace the placeholder:

```
CRICKET_API_KEY=paste_your_real_cricapi_key_here
```

`.env` is listed in `.gitignore` — it will never be committed or bundled into the app. The frontend (`www/js/app.js`) has no `API_KEY` field at all; it only knows your backend's URL (`CONFIG.API_BASE_URL`).

## 3. How to run and test the API

### Start the backend
```bash
cd CricketLiveWorld/server
npm install
npm start
```
You should see:
```
CricketLive World backend running on http://localhost:3000
Health check: http://localhost:3000/api/health
```

### Test it directly (before touching the app)
```bash
# 1. Confirm the server is up and the key is loaded
curl http://localhost:3000/api/health
# → {"ok":true,"keyConfigured":true}

# 2. Pull real live/upcoming/completed matches
curl http://localhost:3000/api/current-matches

# 3. Pull real series
curl http://localhost:3000/api/series

# 4. Search real players
curl "http://localhost:3000/api/players?search=Kohli"

# 5. Real countries/teams
curl http://localhost:3000/api/countries
```
If `keyConfigured` is `false`, or you get a 400/500 error mentioning the key, double check `server/.env` has the real key and restart `npm start`.

For a match's detail endpoints you need a real match `id` — grab one from the `current-matches` response above, then:
```bash
curl "http://localhost:3000/api/match-info?id=PASTE_ID_HERE"
curl "http://localhost:3000/api/match-scorecard?id=PASTE_ID_HERE"
```

### Point the app at the backend
In `www/js/app.js`, `CONFIG.API_BASE_URL` is already set to `http://localhost:3000/api` for local testing. Open `www/index.html` in a browser (or serve the `www` folder) and the Home screen should show real matches, or an honest empty state if nothing is live right now.

### Testing on a real Android device / emulator
`localhost` on the phone means the phone itself, not your computer, so:
- **Emulator:** use `http://10.0.2.2:3000/api` (Android emulator's alias for your host machine).
- **Real device on the same Wi-Fi:** find your computer's LAN IP (e.g. `192.168.1.23`) and use `http://192.168.1.23:3000/api`.
- Update `CONFIG.API_BASE_URL` in `app.js` to match, then `npx cap sync` before rebuilding.

### Before publishing
`localhost`/LAN addresses only work for testing. For a real release, deploy the `server/` folder somewhere (Render, Railway, Fly.io, a small VPS, etc.), set `CRICKET_API_KEY` as an environment variable there (not in a committed `.env`), and point `CONFIG.API_BASE_URL` at that deployed URL, e.g. `https://cricketlive-backend.onrender.com/api`.

## 4. What the backend already handles for you

Per the original spec, `server/server.js` includes:
- **Secure key handling** — key only ever lives in `process.env`, never sent to the client.
- **Caching** — 60s in-memory cache per endpoint+query, so repeated screen visits don't burn through your daily request quota.
- **Rate-limit protection** — both a basic per-IP limiter on your own backend, and a clean 429 response if CricAPI's own rate limit is hit.
- **Error handling** — network failures, bad responses, and provider errors all return clear JSON errors (`{ "error": "..." }`) instead of crashing or leaking internals.

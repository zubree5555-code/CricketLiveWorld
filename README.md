# CricketLive World

A production-architecture, worldwide live cricket application.

> ⚠️ **No live data yet.** This project currently ships with a `MockCricketDataProvider`
> used only for development. It returns clearly-labeled fake data so you can build
> and test the UI. **No real live scores exist until you sign up with a licensed
> cricket data API provider** (examples: SportRadar Cricket API, CricAPI, Entity
> Sport, RapidAPI Cricket vendors — you must check licensing terms and pricing
> yourself) and configure it as described in Stage 5.

## What this project is

- `frontend/` — the Next.js website/app users see.
- `backend/` — the Node.js API + real-time server. This is what talks to the
  database and (eventually) the licensed cricket data provider.
- `shared/` — TypeScript types used by both frontend and backend, so they never
  disagree about what a "Match" object looks like.
- `docs/` — architecture notes, kept up to date as we build.

## How the stages work

We are building this in 12 stages (see `docs/roadmap.md`). Each stage adds real,
runnable code — never placeholder pseudocode for core functionality. Data-provider
and payment-provider code is real, but requires **you** to paste in your own API
keys once you have accounts with those providers — we'll tell you exactly where
each time it comes up.

## Running the project locally (once Stage 2+ is done)

You'll need installed on your computer:
1. **Node.js** (version 20 or later) — https://nodejs.org
2. **Docker Desktop** — https://www.docker.com/products/docker-desktop (runs Postgres + Redis locally without installing them by hand)

Commands will be given stage-by-stage. For now, Stage 1 just sets up the folders
and config files — there's nothing to "run" yet. That starts in Stage 2.

# FAnRoC 2026 Scoring App

Responsive PWA scoring prototype for FAnRoC. Local-first build stores data in the browser (`localStorage`) and supports JSON backup/restore, CSV team import/export, random alliance schedule generation, scorekeeping, rankings, and a spectator-board view.

## Current deployment status
- Frontend can be published as a static Site.
- Current build is single-device/local storage only. It does not yet sync between devices, expose a public live spectator endpoint, authenticate users, or automatically back up remotely.
- Do not use this build as the only authoritative scoring system for a tournament until shared backend persistence is configured.

## Deploy
This project uses `dist/` as its static output. See `DEPLOY_GUIDE_VI.md`.

## Local preview
From the project root, run `python3 -m http.server 8000 --directory dist` and open `http://localhost:8000`.

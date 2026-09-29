# CrowdFlow AI

CrowdFlow AI is a dark operational dashboard that helps station operators monitor crowd levels, forecast capacity risk, choose one recommended response, and draft public announcements.

## Run & Operate

- `pnpm --filter @workspace/api-server run dev` — run the API server (port 5000)
- `pnpm --filter @workspace/crowdflow-ai run dev` — run the CrowdFlow frontend
- `pnpm run typecheck` — full typecheck across all packages
- `pnpm run build` — typecheck + build all packages
- `pnpm --filter @workspace/api-spec run codegen` — regenerate API hooks and Zod schemas from the OpenAPI spec
- `pnpm --filter @workspace/db run push` — push DB schema changes (dev only)
- Required env: `DATABASE_URL` — Postgres connection string

## Stack

- pnpm workspaces, Node.js 24, TypeScript 5.9
- API: Express 5
- DB: PostgreSQL + Drizzle ORM
- Validation: Zod (`zod/v4`), `drizzle-zod`
- API codegen: Orval (from OpenAPI spec)
- Build: esbuild (CJS bundle)

## Where things live

- `artifacts/crowdflow-ai/src/App.tsx` — dashboard routes, scenario state, deterministic forecast/risk logic, and Gemini announcement generation
- `artifacts/crowdflow-ai/src/index.css` — CrowdFlow dark lavender/purple theme tokens and responsive layout styles
- `artifacts/api-server` — shared API server scaffold; the current CrowdFlow prototype keeps scenario data client-side
- `attached_assets` — original CrowdFlow PRD and supplied visual reference archive

## Architecture decisions

- Forecast and risk calculations remain deterministic in the browser; Gemini is used only for optional announcement wording.
- The Gemini key is stored only in the browser's local storage because the user supplies their own free API key.
- Scenario controls, presets, action review, announcements, and settings share one client-side state so the prototype works without a backend connection.
- The interface follows the supplied CrowdFlow dark navy, lavender, purple, and pink visual system rather than a generic blue admin theme.

## Product

- Overview with current crowd, forecast, occupancy, next vehicle, action, and announcement surfaces
- Live Crowd and Forecast views with platform monitoring and what-if controls
- Actions view with one recommended response and an operator review history
- Announcements view with English, Tamil, and Hindi drafts, copy, read-aloud, regenerate, and Gemini-assisted generation
- Settings view with station defaults, configurable risk thresholds, and local Gemini API key/model storage

## User preferences

_Populate as you build — explicit user instructions worth remembering across sessions._

## Gotchas

- The Gemini key is intentionally client-side for this prototype; do not move it into logs or source control.
- If no key is saved or the Gemini request fails, the UI keeps the deterministic forecast/risk result and shows a clearly labeled fallback announcement.
- `pnpm --filter @workspace/crowdflow-ai run typecheck` is the fast frontend validation command.

## Pointers

- See the `pnpm-workspace` skill for workspace structure, TypeScript setup, and package details

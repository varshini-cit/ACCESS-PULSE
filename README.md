# AccessPulse

Live accessibility intelligence for indoor navigation: profile-aware step-free
routing, live facility status, a verified-report workflow, and instant
recalculation when a lift, ramp or corridor goes out of service.

This project was generated with [Lovable](https://lovable.dev) and the existing
UI, design system and screens are preserved.

## Run it

```sh
npm i
npm run dev       # http://localhost:8080
```

The app runs with no external services in **demo mode**: the bundled sample venue
is used and data is stored in IndexedDB on this device.

```sh
npm run test      # vitest (routing, reports, store workflow)
npm run lint      # eslint
npm run typecheck # tsc --noEmit
npm run build     # production build
```

## Backend (Supabase)

1. Create a Supabase project.
2. Run [`supabase/schema.sql`](supabase/schema.sql) in the SQL editor.
3. Copy `.env.example` to `.env` and fill in the Supabase keys plus
   `FACILITY_MANAGER_CODE`.
4. `npm run seed` to load the sample venue graph.

Full details, schema description, authorization model and limitations:
[docs/BACKEND.md](docs/BACKEND.md).

| Variable | Secret | Used by |
| --- | --- | --- |
| `SUPABASE_URL` | no | server functions, seeder |
| `SUPABASE_SERVICE_ROLE_KEY` | yes | server functions only |
| `SUPABASE_ANON_KEY` | no | browser Realtime (RLS read-only) |
| `FACILITY_MANAGER_CODE` | yes | manager sign-in, checked server-side |
| `VENUE_ID` | no | optional, defaults to the sample venue |

## Demo walkthrough

1. **Get started → Wheelchair User.**
2. **Scan QR Code** → *Use simulated checkpoint* → a checkpoint (or scan a test
   code printed from the same screen). Your location updates and the route is
   recalculated.
3. Pick a destination on the map or from search. The route is step-free,
   excludes blocked facilities and shows distance and walking time.
4. **Report an Issue** → *Lift / elevator issue* → Lift A → submit.
5. **Reports** → **Verify** the report. Confirming sets Lift A to *blocked*.
6. If the blocked facility was on the active route, AccessPulse opens the
   reroute screen with the new route (via Lift B and the East Ramp) — or warns
   that no accessible alternative exists and to ask venue staff.
7. **Voice guidance** and **AR** (clearly labelled simulated) use the
   recalculated directions.
8. **Offline** shows the cached map, cached statuses, last sync time and queues
   new reports until a backend confirms them.

## Architecture

- `src/routing/engine.ts` — Dijkstra over the venue graph, pure functions.
- `src/data/` — bundled sample venue graph, accessibility profiles.
- `src/services/app-store.tsx` — one React context: profile, location, statuses,
  reports, trip, settings, connectivity, backend mode.
- `src/services/domain.ts` — report validation, verification and status-change
  rules (pure, unit tested).
- `src/backend/` — TanStack Start server functions + Supabase repository.
  Service-role credentials never reach the browser.
- `src/services/realtime.ts` — Supabase Realtime subscription with a polling
  fallback.
- `src/services/venue-service.ts` / `src/services/idb.ts` — offline cache.

## Status

**Fully working:** indoor routing with all five profiles, blocked/maintenance/
crowded handling, multi-floor routes, turn-by-turn and landmark directions, QR
checkpoint scanning (camera or simulated) with test codes, issue reporting with
validation and duplicate prevention, manager verification/rejection, facility
status updates with an audit trail, dynamic rerouting with notification, voice
guidance, simulated AR overlay, offline cache and queued report sync, Supabase
persistence and Realtime updates.

**Simulated / limited:** AR is a simulated overlay (no spatial tracking); manager
authorization is a shared server-side code; room furniture on the floor plan is
sample data; reports in demo mode never leave the device. See
[docs/BACKEND.md](docs/BACKEND.md#8-known-limitations).

## Built with

- TanStack Start / TanStack Router / React 19 / TypeScript
- Tailwind CSS v4, Radix UI, Motion, Sonner
- Supabase (PostgreSQL + Realtime), IndexedDB, Web Speech API, jsQR

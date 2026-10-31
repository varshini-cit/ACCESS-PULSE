<!-- LOVABLE:BEGIN -->
> [!IMPORTANT]
> This project is connected to [Lovable](https://lovable.dev). Avoid rewriting
> published git history — force pushing, or rebasing/amending/squashing commits
> that are already pushed — as it rewrites history on Lovable's side and the
> user will likely lose their project history.
>
> Commits you push to the connected branch sync back to Lovable and show up in
> the editor, so keep the branch in a working state.
<!-- LOVABLE:END -->

## Architecture rules
- Routing math lives only in `src/routing/` as pure functions (Dijkstra, non-negative weights, blocked/maintenance facility edges excluded); UI never computes routes itself — keeps the engine testable (`src/routing/engine.test.ts`).
- Venue graph and sample data live in `src/data/`; all reads/writes go through `src/services/venue-service.ts` so a backend can replace local data without UI changes.
- Global app state (profile, location, facility statuses, reports, trip, settings, online/offline) is one React context in `src/services/app-store.tsx`; prefs persist to localStorage, reports/statuses/map cache to IndexedDB (`src/services/idb.ts`).
- Screens inside the sidebar/bottom-nav shell are `src/routes/_shell.*.tsx`; full-screen experiences (splash, profile, scan, AR, voice, reroute, offline) are top-level routes. Never name a route file `*.route.tsx` (reserved suffix).
- User reports never change facility status directly; only verification (or a facility-manager status change) does, which may trigger a reroute event.
- Simulated modes (QR checkpoint picker, AR without camera, offline toggle) must be visibly labelled as simulated.

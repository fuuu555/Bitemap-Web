# Stage 9 Local PWA and Cloud Release Design

## Goal

Deliver BiteMap's PWA in two independently reviewable parts. First, make the local application installable and verify it on a phone while the API and data stores remain on the development machine. Then deploy the application and its data services to a secure cloud environment.

## Chosen approach

Use a local-first sequence. The first part validates the install experience, static shell caching, and service worker updates without requiring a cloud database. The second part establishes the public HTTPS environment and production operations. This keeps PWA work testable before production infrastructure is selected.

## Stage 9A: Local PWA

- Add a Web App Manifest, app icons, theme metadata, and standalone presentation.
- Cache only the app shell and approved static assets. Provide an offline notice and a controlled service worker update flow.
- Keep API responses, authenticated/private user data, map tiles, and chat messages out of the service worker cache.
- Install on the development computer through `localhost`. For phone acceptance, serve the local frontend and API through a trusted HTTPS address reachable by the phone; PostgreSQL/PostGIS and Redis remain local.
- Continue to require connectivity for application data. Offline reads and queued writes are out of scope. Web Push is also out of scope for this part.

### Acceptance

- A supported desktop browser can install the local app and open it in standalone mode.
- A phone can open the HTTPS local service, add it to the home screen, sign in, and load the map and main pages.
- When disconnected, the app shows an offline notice; after reconnection it can load current data.
- Service worker updates are visible to users, old cache versions are removed, and private or real-time data is not cached.

## Stage 9B: Cloud release

- Deploy the Next.js frontend and FastAPI backend behind HTTPS. Route REST and WebSocket traffic through a same-origin proxy or explicitly configured secure endpoints.
- Deploy PostgreSQL with PostGIS support and Redis. Keep database access limited to the backend; do not expose the database to browsers.
- Configure production secrets, secure cookies, security headers, rate limits, and external-service quotas.
- Establish migrations, rollback procedures, backups, and a restore exercise. Treat local development data and production data separately; use an explicit import process if selected data should be transferred.
- Add structured logs and error/trace monitoring. Confirm external data and geocoding terms before serving production traffic.
- Add Object Storage and Celery only when production enables file uploads or background processing. Treat Web Push as an optional extension.
- Review moderation and reporting readiness before opening user-generated content to the public.

### Acceptance

- A supported phone browser can install the PWA from the production HTTPS site.
- Main flows for login, map/location, restaurants, meals, and chat pass end-to-end checks in the staging environment; WebSocket reconnect recovers authoritative state.
- Service worker updates do not strand users on an old version and do not cache private or real-time data.
- Production database backup and restore are exercised, and migrations have a documented upgrade/rollback path.
- Production secrets, rate limits, security headers, and error monitoring are active.

## Out of scope

- Offline access to live API data, offline writes, and background synchronization.
- Web Push as a launch requirement.
- Completing unrelated Stage 1–8 product backlog items as a prerequisite for PWA installation.

## Self-review

- The local phone test explicitly requires HTTPS reachability; the local database remains off the public browser path.
- Static caching and live/authenticated data are separated to avoid stale or sensitive responses being stored by the service worker.
- Optional storage, worker, and Push work is tied to product features that need it rather than making it a prerequisite for a basic cloud PWA.

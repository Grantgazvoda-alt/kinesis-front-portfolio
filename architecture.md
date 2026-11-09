# Kinesis Front — Architecture

## Atelier shell

Client-side 3D workspace built on TanStack Start with React 19. The router is resolved by a named `getRouter` export from `src/router.tsx`, matching the installed TanStack Start conventions. The root route mounts an `AuthProvider`, a `PreviewHostBridge` (lets the Grok preview chrome drive the app; noops when not embedded), and the app outlet.

State lives in Zustand. Styling is Tailwind v4 with a base layer setting cursor pointers on interactive elements.

## 3D layer

Three.js via React Three Fiber handles the viewport: head mesh rendering, écorché deformation, performer animation, and lighting. The facial muscle layer sits between the mesh topology and the deformation output — cartoon mode uses squash-stretch heuristics, lifelike mode uses the muscle map directly.

## Photogrammetry pipeline

Sixteen-camera hemisphere capture feeds:

1. **FAST** feature detection — speed-optimized keypoints on dense surfaces
2. **Descriptor matching** across camera pairs
3. **Sparse reconstruction** — initial structure from motion
4. **Bundle adjustment** — refines camera poses before meshing
5. **Dense stereo** — depth maps per baseline
6. **Mesh generation** — surface reconstruction from dense cloud
7. **Texture baking** — project source images onto the mesh

Hemisphere placement maximizes baseline diversity for dense stereo coverage.

## Photo-avatar path

Single-image head reconstruction. Facial landmark priors drive mesh topology — the system infers a plausible 3D head from one 2D source rather than requiring multi-view capture.

## Lip sync

Three independent algorithms selectable per performer style:

- **Viseme-driven** — discrete mouth shapes mapped to phoneme classes. Lowest latency, lowest fidelity.
- **Audio-feature-driven** — continuous feature extraction from the audio stream driving blend shapes. Higher fidelity, higher latency.
- **Hybrid** — viseme anchors with audio-feature interpolation between them. Best balance for real-time performers.

## Seven-desk swarm

Parallel reconstruction across seven departments, each producing a stage artifact the next consumes:

| Desk | Output |
| --- | --- |
| Geometry | Base mesh |
| Texture | UV maps and baked textures |
| Topology | Cleaned, retopologized mesh |
| Facial rig | Bone and weight maps |
| Lip sync | Viseme or blend-shape tracks |
| Lighting | Environment and key light setup |
| Final assembly | Composited performer ready for animation |

Merge resolves conflicts by **weighted confidence**, not last-write-wins. Each desk stamps its artifact with a confidence score derived from input quality and reconstruction residuals.

## Auth and data

- Better Auth runs the app's own session at `/api/auth/*`, federating to a shared Grok auth broker for Google and X. Local email-password is opt-in via `src/lib/auth/email-password.ts` — the broker federation file is never rewritten.
- `useCurrentUser()` reads the session; `SignedIn` / `SignedOut` / `UserButton` gate the UI.
- Every `createServerFn` is wrapped in `authMiddleware`, which hands the handler a verified `context.userId` resolved from the same-origin session. All queries are scoped by that user ID. Client-sent IDs are never trusted.
- Connector and AppData APIs are backend-only: browser → app's `createServerFn` → app backend SDK → connectors host. No connector JWTs or gate secrets ever enter client state.

## Deploy contracts

- `npm run build` must succeed under Vercel's process — dev-only deps, server-only Node APIs at import time, runtime filesystem writes, and hard-coded ports are bugs.
- Nitro preset `vercel` is gated on `command === "build" || isPreview` so it never runs in dev (it would open a second port and break the single-port 8080 live preview).
- `vite preview` is pinned to loopback `127.0.0.1:8081` — the built output can never be picked up as the user's live preview.
- `:8081` is QA-only. `npm run preview:restart` frees the port before serving so a stale build is never smoked.
- The PWA plugin (`grokPwaPlugin()`) and the Home Screen install page must survive any config edit.
- Blank-deploy failure mode watched for: `Failed to load module script … MIME type "text/html"` — asset base path must resolve `/assets/*` correctly.

## Browser QA

Agent-driven, not user-driven. Mandatory smoke pass: Playwright helper loads desktop (1280×800) and mobile (390×844), screenshots both, prints a JSON verdict with per-viewport console errors, body text, and horizontal overflow. Interactive passes use `agent-browser` with batched command flows and throwing `eval`s that turn checks into exit-code verdicts. Depth beyond the smoke pass is judgment — a landing page needs the smoke pass; a game with WASD controls needs sign verification (A left / D right from a chase cam).

## Session lifecycle

- Fresh workspace: scaffold, then write `startup.sh`.
- Hibernate/revive: the platform re-runs `/workspace/startup.sh`; everything else comes back as it was. A revive with no `startup.sh` leaves nothing listening on 8080.
- Reboot/recreate: may wipe app files back to template — re-scaffold and restore `startup.sh` before verifying.
- Vite HMR pushes source edits instantly. Restart the dev server only for `vite.config` or dependency changes, and update `startup.sh` in the same turn.
- After edits, re-run the smoke pass; re-read screenshots only when the edit touched visible UI.
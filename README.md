# Kinesis Front — Live 3D Animation Atelier

Portfolio for the front-end atelier of Kinesis, Official Intelligence's live 3D animation system. This repo documents architecture and methodology. The implementation stays proprietary.

## What it does

- **Photo-to-avatar**: upload a single photo, generate a 3D head mesh
- **Photogrammetry**: 16-camera hemisphere capture with FAST detection, sparse and dense reconstruction, mesh generation, texture baking
- **Lip sync**: three independent algorithms — viseme-driven, audio-feature-driven, hybrid
- **Facial écorché**: muscle-based deformation layer driving both cartoon squash-stretch and lifelike performer modes
- **Seven-desk swarm**: parallel reconstruction across geometry, texture, topology, facial rig, lip sync, lighting, and final assembly

## Stack

TanStack Start · React 19 · Three.js / React Three Fiber · Zustand · Tailwind v4

## Deploy

Vercel via Nitro. Gated live preview on port 8080, QA-only preview on 8081 that never contaminates the live session. Auth federates through a shared broker for Google and X, with local email-password as an opt-in. Every server function is gated behind auth middleware resolving a verified user ID — never trusting a client-sent one.

## Public materials

- `architecture.md` — system structure, swarm orchestration, deploy contracts
- `methodology.md` — photogrammetry, photo-avatar, lip-sync, and swarm methodology
- `pipeline-stages.md` — the reconstruction stage chain from capture to final assembly
- `PROPRIETARY-LICENSE.md` — implementation remains closed

## Signal for reviewers

This is a production 3D pipeline running entirely in the browser — not a Three.js demo. The swarm merge resolves conflicts by weighted confidence, not last-write-wins. The dual preview-port contract prevents stale builds from ever serving as live. Auth middleware is enforced at the server-function layer with zero client-trusted identity.
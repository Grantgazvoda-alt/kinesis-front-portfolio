# Kinesis Front — Methodology

## Photogrammetry

**Hemisphere placement.** Sixteen cameras arranged on a hemisphere maximize baseline diversity. Each pair's baseline angle determines depth precision for dense stereo — a narrow baseline gives precision but poor coverage; a wide baseline gives coverage but noisy depth. The hemisphere distributes baselines across the full range so no surface region is starved.

**FAST detection.** Features from Accelerated Segment Test runs on dense keypoint grids. It's not the most repeatable detector, but it's an order of magnitude faster than SIFT/ORB on the same hardware, which matters when you're extracting keypoints across sixteen views in real time. The speed buys you denser sampling; the density compensates for individual keypoint fragility.

**Descriptor matching.** Cross-view matching uses the extracted descriptors with a ratio test to reject ambiguous matches. Matches that survive feed the sparse reconstruction.

**Sparse reconstruction.** Structure from motion solves for camera poses and a sparse point cloud simultaneously. This stage is where gross errors surface — a camera with bad intrinsics or a mismatched pair produces outliers that bundle adjustment must absorb.

**Bundle adjustment.** Nonlinear least squares refines all camera poses and 3D points jointly, minimizing reprojection error across every observation. This is the quality gate: if residuals stay high after adjustment, the capture or matching had a fundamental problem and downstream stages inherit it.

**Dense stereo.** Per-baseline depth maps from rectified pairs. Multiple baselines are fused — near baselines for precision on fine detail, far baselines for coverage on textureless regions.

**Mesh generation.** Dense cloud to surface via Poisson or Delaunay reconstruction. Topology cleanup removes non-manifold edges and degenerate triangles before texturing.

**Texture baking.** Source images project onto the mesh with view-dependent weighting — faces most frontal to a camera get that camera's texels. Seams are minimized by clustering faces into UV islands with low distortion.

## Photo-avatar

Single-image head reconstruction. Facial landmark priors (eyes, nose, mouth, jawline, brow) anchor the mesh topology so the system doesn't have to infer structure from shading alone. The prior gives the optimizer a starting point close to human anatomy; the image evidence deforms it toward the specific face. Failure mode: profile or extreme-angle photos lack landmark visibility, so frontal captures are required.

## Lip sync

**Viseme-driven.** Audio is classified into discrete phoneme classes, each mapped to a canonical mouth shape (viseme). Transitions are interpolated. Pros: deterministic, low latency, works on any audio. Cons: mouth shapes are quantized — subtle coarticulation is lost, so it reads as slightly mechanical on close-up performers.

**Audio-feature-driven.** Continuous features (MFCCs, spectral flux, energy envelopes) map directly to blend-shape weights through a learned or heuristic function. Pros: captures coarticulation and micro-movements. Cons: latency from feature extraction, and feature-to-shape mapping can overshoot on noisy audio.

**Hybrid.** Viseme anchors provide the coarse structure; audio features interpolate between them. The anchors prevent the feature-driven path from drifting; the features fill in the motion the visemes can't express. Best real-time balance for live performers.

Selection is per-performer: cartoon modes favor viseme-driven (stylized mouths tolerate quantization), lifelike modes favor hybrid.

## Écorché deformation

The facial muscle layer sits between mesh topology and deformation output. Each muscle is a region with an origin, insertion, and action line. Activating a muscle pulls vertices along that line with falloff from the insertion point.

**Cartoon mode** uses simplified muscle groups with exaggerated action ranges — squash and stretch are explicit parameters, not emergent from physics. This is the right abstraction for stylized performers where anatomical accuracy would fight the art direction.

**Lifelike mode** uses the full muscle map with anatomically plausible ranges. Deformation follows from muscle activation patterns derived from the performance (audio for speech, animation curves for expression). The risk is uncanny-valley stiffness if activation timing doesn't match audio onset — which is why lip sync and écorché share a common timebase.

## Swarm merge

Seven desks produce artifacts in parallel. The merge step doesn't just concatenate — it resolves conflicts. When two desks disagree (e.g., geometry desk's edge vs. topology desk's retopo), the resolution is **weighted confidence**: each desk's artifact carries a score from input quality and reconstruction residuals, and the higher-confidence claim wins with the lower-confidence desk's output as a soft constraint. Last-write-wins would let a noisy desk silently overwrite a clean one.

## Auth methodology

Defense in depth at the identity layer: the browser never holds a trusted ID. The session is same-origin, resolved server-side by `authMiddleware` into a verified `context.userId` that scopes every query. Client-sent IDs are ignored entirely. Connector secrets stay in the backend SDK path — the browser talks only to the app's own server functions.

## Deploy methodology

Build-time identity: the PWA injector bakes manifest tags at `vite build`, so the deployed Nitro server can inject without reading the workspace filesystem. The dual-port contract (8080 live, 8081 QA) is enforced mechanically — `preview:restart` kills whatever holds 8081 before serving, so a stale build can never be mistaken for live. The nitro preset is build-gated so dev never opens the second port.

## QA methodology

Agent-driven, not human-driven. The smoke pass is mandatory: desktop and mobile screenshots plus a JSON verdict covering console errors, body text, and overflow. Interactive checks use batched `agent-browser` flows with throwing evals — falsy is not failure, throwing is. A 404 page reports a checkmark too, so every check asserts on content, not just load status. Depth matches the app: landing page gets smoke; WASD game gets control-sign verification.
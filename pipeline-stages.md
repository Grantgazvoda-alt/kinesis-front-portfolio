# Kinesis Front — Pipeline Stages

End-to-end reconstruction chain from capture to final assembly.

## Stage 1 — Capture

Sixteen cameras on a hemisphere. Each camera records a synchronized frame sequence. Hemisphere geometry guarantees overlapping fields of view with diverse baselines — no surface is seen from only one angle.

**Output:** synchronized multi-view image sequence with camera intrinsics.

## Stage 2 — Feature extraction

FAST detection across all views. Dense keypoint grids on surfaces with texture; sparser sampling on uniform regions where features would be unstable anyway.

**Output:** per-view keypoint lists with descriptors.

## Stage 3 — Matching

Cross-view descriptor matching with ratio test. Ambiguous matches rejected. Surviving matches form the observation set for reconstruction.

**Output:** matched keypoint correspondences across camera pairs.

## Stage 4 — Sparse reconstruction

Structure from motion solves camera poses and a sparse point cloud. Initial pose estimates come from the matched pairs; the solver iterates to global consistency.

**Output:** camera pose graph + sparse point cloud.

## Stage 5 — Bundle adjustment

Nonlinear refinement of all poses and points, minimizing total reprojection error. Residuals are the quality metric: high residuals after convergence mean the capture had a fundamental problem.

**Output:** refined poses + quality residuals per camera.

## Stage 6 — Dense reconstruction

Per-baseline stereo depth maps from rectified pairs. Multiple baselines fused — near for precision, far for coverage. Fusion weights by baseline geometry and matching confidence.

**Output:** dense point cloud.

## Stage 7 — Meshing

Surface reconstruction from the dense cloud. Poisson or Delaunay. Non-manifold cleanup, degenerate triangle removal, hole filling.

**Output:** watertight mesh.

## Stage 8 — Texturing

Source images projected onto the mesh. View-dependent weighting favors frontal cameras per face. UV unwrapping clusters faces into low-distortion islands. Seam minimization across island boundaries.

**Output:** textured mesh.

## Stage 9 — Swarm parallel pass

Seven desks consume the textured mesh and produce parallel artifacts: geometry refinement, texture enhancement, topology cleanup, facial rig, lip-sync tracks, lighting, final assembly. Each stamps a confidence score.

**Output:** seven stage artifacts with confidence metadata.

## Stage 10 — Merge

Weighted-confidence resolution across desk outputs. Higher-confidence claims win; lower-confidence output becomes a soft constraint. The merge is the only stage that sees all seven desks at once.

**Output:** merged performer asset.

## Stage 11 — Final assembly

Composited performer: mesh + texture + rig + lip-sync + lighting, ready for animation. Export formats include glTF for web delivery.

## Photo-avatar shortcut

When multi-view capture isn't available: Stage 1 is replaced by single-image intake. Facial landmark priors substitute for the pose graph. Stages 2–8 run in reduced form — no dense stereo, landmark-driven topology instead. Stages 9–11 are unchanged.

The shortcut trades geometric precision for accessibility. It's the right call for quick performer generation; the full pipeline is the right call for production assets.
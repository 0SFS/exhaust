# Exhaust depth projection — 27 October 2026

Status: **closed as held, unshipped** at the user’s wrap-up request. The candidate
passes numerical/full-helper validation and wins the isolated microbenchmark.
The required ready-aircraft GPU/depth-image comparison remains unproven, so
production source and tests remain unchanged. No push or deployment was made.

Baseline is clean exhaust `6a9bd23c859d8c05439a9c4a87cbbcbd16f26a50`, source
SHA-256 `5b6786bd7118c92626bec64d2f337ec0e6bc1fddbb2e1fe445d95fc47c80f4ae`.
It includes the newer non-shadow-dressing exclusion (`f7d994e`) and retained
light buffers (`304264f`) in addition to ancestor refresh batching (`b4ba2c9`).
[Preparation](../2026-10-26/depth.md) records the original frozen candidate.

## Measured projection-only experiments

Scratch and raw results live in ignored `build/perf-depth-20261026/`.
`projection.mjs` imports source-extracted baseline/candidate functions. The
baseline retains its returned array literal; the reuse candidate passes one
six-number output array belonging to the current caller. A second variant adds
first-corner extrema seeding but still reuses the output array.

These are short Node projection-only microbenchmarks, without GPU rendering,
material preparation, actual world-matrix/bounds updates, browser profiling or
energy measurement. Fresh arrays may be optimized away by V8, while writes to
reused storage remain observable. The result rejects these concrete candidates;
it does not establish that allocation dominates the real depth pass. Timing
pairs were serialized under the root coordinator's exclusive measurement slot.

### Reuse result storage

12,000 deterministic differential cases passed, including 426 invalid projections,
plus nested projection/storage checks. Returned numeric fields use `Object.is`.
Each timing arm projected 150 bounds over 1,200 frames per round; arm order
alternated over 12 pairs.

| Pair | Baseline ms | Candidate ms | Candidate / baseline |
| ---: | ---: | ---: | ---: |
| 1 | 5.2123 | 6.4842 | 1.244 |
| 2 | 5.3309 | 6.4072 | 1.202 |
| 3 | 5.2705 | 6.1537 | 1.168 |
| 4 | 5.3012 | 6.1909 | 1.168 |
| 5 | 5.3328 | 6.0270 | 1.130 |
| 6 | 5.3319 | 6.1601 | 1.155 |
| 7 | 5.2217 | 6.5459 | 1.254 |
| 8 | 5.2558 | 6.4455 | 1.226 |
| 9 | 5.2425 | 6.4320 | 1.227 |
| 10 | 5.2738 | 6.4729 | 1.227 |
| 11 | 5.2359 | 6.5595 | 1.253 |
| 12 | 5.2943 | 6.1579 | 1.163 |

Median arm duration: 5.2738 → 6.4320 ms.
Candidate is slower in every pair. It was rejected before full-helper/browser work.

### Reuse result storage and seed the first extrema

12,000 deterministic differential cases passed, including 426 invalid projections,
plus nested projection/storage checks. Returned numeric fields use `Object.is`.
Each timing arm projected 150 bounds over 1,200 frames per round; arm order
alternated over 12 pairs.

| Pair | Baseline ms | Candidate ms | Candidate / baseline |
| ---: | ---: | ---: | ---: |
| 1 | 5.2205 | 6.4764 | 1.241 |
| 2 | 5.3160 | 6.3385 | 1.192 |
| 3 | 5.3723 | 6.4421 | 1.199 |
| 4 | 5.4087 | 6.4888 | 1.200 |
| 5 | 5.3454 | 6.3297 | 1.184 |
| 6 | 5.3171 | 6.3590 | 1.196 |
| 7 | 5.3199 | 6.4792 | 1.218 |
| 8 | 5.4217 | 6.3577 | 1.173 |
| 9 | 5.3452 | 6.2786 | 1.175 |
| 10 | 5.3260 | 6.4548 | 1.212 |
| 11 | 5.3000 | 6.3624 | 1.200 |
| 12 | 5.3555 | 6.2702 | 1.171 |

Median arm duration: 5.3452 → 6.3624 ms.
Candidate is slower in every pair. It was rejected before full-helper/browser work.

Raw JSON identities are `projection-results-candidate.json` and
`projection-results-seed-first.json`. No browser arms, source maps, CPU profiles
or depth images were generated for these rejected candidates.

## Independent projection variants without result reuse

Every variant below starts from the same unchanged baseline and retains its
fresh output array. Each comparison uses 12 alternating pairs, 150 mesh bounds
and 12,000 frames per arm per pair. All share the 12,000 differential cases
(426 invalid), plus 36 explicit signed-zero/near-plane/infinity/empty-corner
fixtures. Numeric outputs use `Object.is`.

| Variant | Baseline median ms | Candidate median ms | Per-pair ratio range | Decision |
| --- | ---: | ---: | ---: | --- |
| Indexed traversal | 73.2938 | 63.8823 | 0.868–0.882 | Superseded by combined candidate |
| Seed first corner, indexed traversal | 74.8109 | 65.4539 | 0.870–0.877 | Superseded by combined candidate |
| Comparisons for forward-distance extrema | 73.9365 | 77.4114 | 1.031–1.052 | Reject: slower in every pair |
| Snapshot matrix coefficients after bounds lookup | 74.6330 | 46.1548 | 0.616–0.620 | Superseded by combined candidate |
| Snapshot coefficients plus indexed traversal | 74.3498 | 44.8895 | 0.600–0.606 | Retain for browser validation |

The combined candidate reads the same 12 matrix values once after world refresh
and `getBoundingInfo`, then uses an indexed loop over the same corners. It
retains every multiply/add/divide expression, finite check, extrema operation,
returned value and corner order. No cross-call cache or scheduling changes.
The specific redundant operation is reloading 12 matrix coefficients for each
of eight corners rather than once for the projection.

Paired combined-candidate evidence:

| Pair | Baseline ms | Candidate ms | Candidate / baseline |
| ---: | ---: | ---: | ---: |
| 1 | 74.5971 | 44.9455 | 0.603 |
| 2 | 74.4255 | 44.9556 | 0.604 |
| 3 | 73.8955 | 44.7771 | 0.606 |
| 4 | 74.3274 | 44.6748 | 0.601 |
| 5 | 74.3069 | 44.8633 | 0.604 |
| 6 | 74.3255 | 44.8895 | 0.604 |
| 7 | 74.4493 | 44.9388 | 0.604 |
| 8 | 74.5521 | 44.8030 | 0.601 |
| 9 | 74.2999 | 44.8483 | 0.604 |
| 10 | 74.3498 | 44.9027 | 0.604 |
| 11 | 74.4903 | 44.6691 | 0.600 |
| 12 | 74.3400 | 45.0317 | 0.606 |

Raw JSON files are `projection-results-<label>.json` in the scratch directory.
The combined candidate saves about 39.6% of this isolated projection workload;
this is not an end-to-end depth-pass/FPS claim.

## Full-helper and callback validation

- `mutations.mjs`, with `DEPTH_CANDIDATE=hoist-indexed`, passed exact
  matrix/call-log checks when world refresh, bounding lookup or a nested
  projection mutates the matrix before corner projection. Results:
  `mutations-results-hoist-indexed.json`.
- Existing eight exhaust-depth tests plus two new callback fixtures pass against
  the frozen combined candidate. The callback fixtures check that coefficient
  reads occur after world-refresh/bounds callbacks, and preserve the render-pass
  restoration. Log: `focused-hoist-indexed.log` (10 tests).
- `differential.test.ts` compares the complete baseline and combined helper over
  400 moving frames. Render-list membership/order, readiness calls and results,
  publication counts and render-request counts match. Every mesh world-matrix
  entry matches with `Object.is`. Coverage includes edits in the same render ID,
  billboard/frozen hierarchies, instances/morph managers, perspective/orthographic
  cameras and camera crossings, visibility/enabled flags, material alpha/depth
  flags, readiness polling and pass restoration. Log:
  `differential-hoist-indexed.log` (one test).

An earlier fixture attempt did not dirty its transform before expecting a
world-update callback and compared Babylon's observed array directly to an
unobserved empty array. The fixture now dirties the transform and asserts length;
the corrected suite passes. This changed no production code.

The reachable loop contract was inspected in installed Babylon 8.56.2:
`Maths/math.vector.js` implements Vector3 x/y/z getters as direct `_x`/`_y`/`_z`
reads; `Culling/boundingBox.js` creates `vectorsWorld` with eight ordinary
Vector3 instances. Coefficients are captured after callback-capable world/bounds
updates. There are no ordinary callbacks during the arithmetic loop. Arbitrary
custom coordinate accessors or concurrently rewritten matrix storage are outside
this validation scope.

## Frozen browser candidate and remaining checks

`build/perf-depth-20261026/engineExhaustDepth.hoist-indexed.ts` is the full source
candidate. Its SHA-256 is `ec25447760e8308f7f4d15e53c736536233b297a31c9c5fc509afeedcdc113ba`.
`engineExhaustDepth.with-callback-tests.ts` holds the proposed focused test file;
its SHA-256 is `7e62a459723b896bb3d37bc42a3730a5482d5a1d29d552f657d220918177a710`.
`inputs.json` records these alongside the original baseline identities.

Root coordinates any production application, lint/typecheck, pinned F-35B
browser arms with comparable model readiness, unchanged depth/color images,
source-map/profile identities and Git mutations. The final section below records subsequent frozen browser arms and the
incomplete visual validation. The frozen full-helper suite alone does not
establish GPU output identity or whole-frame improvement.


## Final queued full-helper validation

The prepared actual-Babylon benchmark ran during wrap-up, serialized with all
other load. Each arm had its own scene with 150 boxes, 1,000 warmup frames and
8 alternating pairs × 6,000 moving-root frames. Checksums and render-list counts
matched in every pair. This exercises the complete helper, including world and
bounds work, with NullEngine and readiness forced ready; it does not draw GPU
images or load the actual F-35B model.

| Comparison | Left median ms | Right median ms | Right/left range |
| --- | ---: | ---: | ---: |
| Two byte-identical baselines | 267.410 | 262.788 | 0.970–1.011 |
| Baseline vs hoist+indexed | 260.547 | 251.679 | 0.957–0.989 |

The A/B median reduction is 3.4%, much smaller than the 39.6% projection-only
result. The unchanged A/A modules also differed by 1.7%, limiting inference
about the small complete-helper gain. All raw paired rounds are retained in
`full-pass-aa-left-aa-right.json` and `full-pass-baseline-hoist-indexed.json`
inside ignored `build/perf-depth-20261026/`. Strict candidate TypeScript and
focused ESLint both passed (`typecheck.log`, `lint.log`).

The root built frozen `oct27-base` and `oct27-depth` flight arms from the same
904-file source snapshot. Initial F-35B calibration had no exhaust-depth rows in
the top function tables. The diagnostic candidate run proved aircraft assets
were still loading before its first profile and only became ready after
27,964 ms. This invalidates it as a controlled ready-aircraft comparison; the
second window being ready does not retroactively validate the first. Neither
run establishes candidate savings or visual identity. Both completed and their
Chrome clones were cleaned.

Current loading outcome `ready` does not mean the aircraft mesh is ready.
Check `osfsLoadingDiagnostics().phases.assets.state === "ready"` and reject
`fallback`/`unavailable` detail. Default F-35B throttle is 0.48, engineRunning is
true, exhaust and depth are enabled, contribution is combined. No throttle or
engine-model setting was silently changed to make the workload exercise depth.
`window.osfsJsbsimBuild` identifies the aircraft and engine model; DOM
`[data-engine]` identifies running/starting/stopped without exposing SDK handles.
A readiness-gated runner was prepared, but no additional exhaust browser cohort
or image fixture was started after the request to wrap up. The optional margin
hoist also remains unmeasured and unshipped.

The source patch below preserves the strongest completed candidate for a future
explicit continuation. Required remaining evidence is a comparable ready F-35B
cohort plus unchanged depth/color images. Do not treat this patch as approved
for production based solely on the isolated arithmetic speedup.

```diff
--- a/src/engineExhaustDepth.ts
+++ b/src/engineExhaustDepth.ts
@@ -61,2 +61,8 @@
-  for (const { x, y, z } of mesh.getBoundingInfo().boundingBox.vectorsWorld) {
-    const w = x * m[3] + y * m[7] + z * m[11] + m[15];
+  const points = mesh.getBoundingInfo().boundingBox.vectorsWorld;
+  // Bounds refresh can run synchronous matrix callbacks; read coefficients afterwards.
+  // Babylon's corner vectors have plain numeric getters throughout this loop.
+  const m0 = m[0], m1 = m[1], m3 = m[3], m4 = m[4], m5 = m[5], m7 = m[7];
+  const m8 = m[8], m9 = m[9], m11 = m[11], m12 = m[12], m13 = m[13], m15 = m[15];
+  for (let index = 0; index < points.length; index++) {
+    const { x, y, z } = points[index];
+    const w = x * m3 + y * m7 + z * m11 + m15;
@@ -65,2 +71,2 @@
-    const px = (x * m[0] + y * m[4] + z * m[8] + m[12]) / w;
-    const py = (x * m[1] + y * m[5] + z * m[9] + m[13]) / w;
+    const px = (x * m0 + y * m4 + z * m8 + m12) / w;
+    const py = (x * m1 + y * m5 + z * m9 + m13) / w;
```

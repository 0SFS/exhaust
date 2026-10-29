# Exhaust synchronize — 27 October 2026

Shipped the flow-domain string cache in `createEngineExhaust.ts`. Reverted depth-options `Vector2` reuse and volume `Vector3` reuse. No browser run, no push.

Baseline source SHA-256 `b80176799fe3fbd13b726e87cafff731fdcb55534eb7035e7626019aa337cd69`. Scratch and raw pairs are in ignored `build/perf-b-20261027/`. Timing wraps `synchronize` only, in bench copies, with `process.hrtime`. `update` still forces the anchor world matrix outside that timer. Twelve pairs, arm order alternating, `nice -n 19`, `gc` between arms. Outputs were compared with `Object.is` on mesh pose, serialized uniforms, light, and the source-field buffer, plus two camera binds. Baseline, domain cache, vector reuse, depth-vector reuse, and the combination all matched.

The workload is one F135 gas support. Stable calls return the cached snapshot. Creeping calls change throat radius, exit radius, and nozzle position, so each tick builds a new domain object and rebakes the field.

## Stable plume, 120,000 synchronizes

Identical copies were not symmetric: the second copy's median was 4.2% faster (155.944 → 149.761 ms, pair ratios 0.794–0.975). A part had to land outside that band.

| Part | Median ms | Median ratio | Pair ratios | Decision |
| --- | ---: | ---: | ---: | --- |
| Identical copy | 155.944 → 149.761 | 0.958 | 0.794–0.975 | Noise band |
| Domain identity cache | 156.669 → 62.704 | 0.401 | 0.257–0.415 | Ship |
| Volume vector reuse | 170.905 → 165.971 | 0.982 | 0.727–1.005 | Revert |
| Depth vector reuse | 175.772 → 173.547 | 0.952 | 0.894–1.793 | Revert |
| All three together | 162.729 → 67.265 | 0.411 | 0.362–0.427 | Same win as the cache |

The 0.257 domain pair is a 245 ms baseline stall against a normal 63 ms candidate. The other eleven pairs sit at 0.385–0.415.

## Creeping aperture, 80 synchronizes

Identical copies: 24.141 → 24.042 ms, median ratio 0.999, pair ratios 0.958–1.075.

| Part | Median ratio | Pair ratios | Decision |
| --- | ---: | ---: | --- |
| Domain identity cache | 1.007 | 0.970–1.033 | No creep win; no regression past noise |
| Volume vector reuse | 1.002 | 0.867–1.022 | Revert |
| Depth vector reuse | 0.998 | 0.942–1.025 | Revert |
| All three together | 1.008 | 0.883–1.767 | No extra win |

A new domain object still stringifies. The cache is keyed by object identity, not revision and not deep equality, because a posed support can bump revision while keeping the same domain object.

## Draw bind

Reusing the depth-options vector did not show up in `synchronize`. On 8,000 binds the identical-copy median ratio was 0.983 (pair ratios 0.811–1.595) and the depth candidate was 0.981 (0.752–1.280). Reverted with the volume vectors.

Focused vitest: `createEngineExhaust.test.ts` and `createEngineExhaustDomain.test.ts`, 40 passed. The bound effect still receives the current camera.

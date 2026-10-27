# Exhaust depth projection preparation — 26 October 2026

Resumed on 27 October: see [measured continuation](../2026-10-27/depth.md).

Status at the original checkpoint: **prepared, unrun and unvalidated**. No production source or tests changed.
No demonstrated performance improvement, exactness result, browser comparison or
visual result is claimed. No commit, push or deployment was made.

The exhaust checkout was clean at `6a9bd23c859d8c05439a9c4a87cbbcbd16f26a50`.
It already includes `b4ba2c9` (refresh each moved ancestor once), `f7d994e`
(exclude aircraft dressing flagged out of shadow casting), and `304264f`
(retain exhaust light buffers across cutoff and rewind). The October 22 profile
costs therefore describe an older workload, rather than this checkout.

## Hypothesis and isolated candidate

`projectedBounds` returns a new six-number array for every mesh that passes its
finite/front-of-camera checks. `pendingNow` consumes each mesh's result before
projecting the next mesh. A scratch candidate passes one result array owned by
the current `pendingNow` invocation and overwrites it after each successful
projection. The plume retains its independent result array. This could avoid
per-mesh array allocation; its extra writes and branch may erase the benefit.

The candidate changes only `src/engineExhaustDepth.ts` in a frozen scratch copy.
It retains projection expressions, corner order, world-matrix refresh, finite
checks, camera-crossing retention, list membership/order/publication, readiness
calls, render-pass restoration and scheduling. Result storage belongs to each
invocation, so a synchronous nested invocation cannot overwrite another
invocation's storage. An invalid projection leaves the scratch buffer untouched.
These are design properties awaiting validation, not test results.

Both copies and the source-extracted projection modules are retained under the
ignored `exhaust/build/perf-depth-20261026/` directory:

- `engineExhaustDepth.baseline.ts` and `engineExhaustDepth.candidate.ts`.
- `baseline-projection.mjs`, `candidate-projection.mjs`, `projection.mjs`.
- `inputs.json` records the full-copy SHA-256 identities.
- `vitest.config.mts` is a minimal focused-test configuration using installed fly
  dependencies, with its cache inside exhaust's `build/`.

The baseline source SHA-256 is
`5b6786bd7118c92626bec64d2f337ec0e6bc1fddbb2e1fe445d95fc47c80f4ae`;
the candidate SHA-256 is
`c912a5600504e0c93c67eb4c1e634cd9042e19e6872aef20b2957c3a76c96d52`.

## Prepared validation and continuation

The projection harness prepares 12,000 deterministic ordinary/invalid cases and
compares all six result fields with `Object.is`, checks a nested projection with
separate buffers, and checks unchanged invalid-projection storage. It then
alternates baseline/candidate order across 12 timing rounds of 1,200 frames,
150 synthetic mesh bounds per frame, and 32 perspective/orthographic views.
It writes bounded results to `projection-results.json`. Its projection-only
numbers would isolate arithmetic/allocation; they would not establish full depth
pass, GPU, FPS or power savings.

After coordinating the machine-wide exclusive measurement slot, from exhaust:

```sh
cd /Users/felg/gh/0sfs/exhaust
TMPDIR="$PWD/build/perf-depth-20261026/tmp" \
  node build/perf-depth-20261026/projection.mjs
```

If there is no clear gain, retain this negative result and leave production
source unchanged. If a gain warrants proceeding, first expand full-helper
baseline/candidate differential checks to same-render-id hierarchy changes,
frozen/billboard nodes, deformed/instanced meshes, camera changes, invalid bounds,
material readiness and render-pass restoration. The existing focused tests can
be run against production source with:

```sh
TMPDIR="$PWD/build/perf-depth-20261026/tmp" \
  ../fly/node_modules/.bin/vitest run --configLoader native \
  --config build/perf-depth-20261026/vitest.config.mts
```

That command alone does not test the scratch candidate. Only a demonstrated
exact gain should be applied to production, followed by focused lint/typecheck
and an isolated pinned F-35B browser arm with comparable model readiness and
unchanged depth images. Freeze every override, serialize builds and browser
runs, and clean Chrome clones after each browser run. No browser arms, source
maps or profile identities exist for this candidate yet.

Execution was deferred because the shared measurement lock belongs to an older
session and clearing it requires user approval under the current lock workflow.
The coordinating parent agent owns that approval and measurement scheduling.

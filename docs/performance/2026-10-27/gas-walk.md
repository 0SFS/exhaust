# Gas-support hierarchy walk — 27 October 2026

Status: **shipped**. `update()` no longer multiplies the bound chain on a
stable aperture. No push or commit was made.

The previous source multiplied every bound frame up to the engine root on
every call, including cache hits. The walk is skipped only when the aperture
fields are `Object.is` to the last accepted values and each walked node's
parent, quaternion reference, quaternion components, position, scaling, and
euler components match the stamp taken with the last identical engine-relative
matrix. Quaternion components are in the stamp because nozzle vectoring
rewrites one quaternion object in place. Reparent, in-place aim, position, and
scale changes still bump the revision. A creeping aperture still walks and
rebuilds; that cost is not counted as saved. Exterior length and spread reuse
the accepted matrix.

The frozen baseline is scratch
`build/gas-walk-20261027/engineGasSupport.baseline.ts`, SHA-256
`a1696a690e543f7292027a9f12771001ac7b355e2819dff055a4c68c304c16db`.

## Stable-aperture measurement

`nice -n 19` Node run, raw JSON in `build/gas-walk-20261027/results.json`.
The installed F135 mesh is buried under 8 ancestors (engine world-matrix
depth 11). Chain steps from the engine root are 0, 1, 2, 3, and 4 for the
engine, three bearings, and the nozzle. Each arm called `update` 130,641
times with one reused aperture (`throatRadius` 0.4, `throatZ` 0.24,
`exitRadius` 0.48, `exitZ` 0.5), exterior length 6, and slope 0.07. Arm order
alternated over 12 pairs. A second 12-pair block timed two copies of the
unchanged baseline.

Section JSON matched the baseline on the stable pose and after an in-place
bearing-quaternion write. Both revisions bumped, and restoring the component
bumped again.

| Pair | Baseline ms | Candidate ms | Candidate / baseline |
| ---: | ---: | ---: | ---: |
| 1 | 112.8676 | 83.4527 | 0.739 |
| 2 | 106.7004 | 78.3854 | 0.735 |
| 3 | 106.8000 | 79.1196 | 0.741 |
| 4 | 106.4833 | 77.2598 | 0.726 |
| 5 | 106.2940 | 76.8958 | 0.723 |
| 6 | 106.6000 | 76.9813 | 0.722 |
| 7 | 106.4949 | 76.7629 | 0.721 |
| 8 | 106.5600 | 77.1452 | 0.724 |
| 9 | 106.7063 | 76.9307 | 0.721 |
| 10 | 106.9894 | 76.8993 | 0.719 |
| 11 | 109.9837 | 79.8019 | 0.726 |
| 12 | 106.8770 | 77.4445 | 0.725 |

Median arm duration: 106.7033 → 77.2025 ms (0.817 → 0.591 µs per update).
Median ratio 0.724. All 12 pairs were faster; the slowest ratio was 0.741.

Unchanged-baseline median absolute deviation was 0.67% (median ratio 1.007).
One baseline pair stalled to 1.434 (152.41 vs 106.28 ms). The candidate
cluster, 0.719 to 0.741, stays clear of that spike and of the 2% bar.

## Gate

`nice -n 19 ./node_modules/.bin/vitest run --configLoader native --maxWorkers=50% src/flight/aircraft/engineGasSupport.test.ts` from `fly`: 2 passed.

eslint from the org root with `fly/eslint.config.js` on `exhaust/src/engineGasSupport.ts` and `fly/src/flight/aircraft/engineGasSupport.test.ts`: clean.

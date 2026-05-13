# Maze Builder Progress

Last updated: 2026-05-13

## Scope

Current work focuses on the TypeScript/Vite generator and viewer in `src/`. The main risk area is rail direction/connection correctness for L90/R90/U90/D90 rails under combined Pitch/Roll/Yaw transforms.

## Completed

### Generator UI

- Added configurable checkpoint count.
- Added configurable max self-spin count.
- Added collapsible Generator, Stats, Rail Detail, and Generation Log panels.
- Made the left sidebar scroll when content exceeds viewport height.
- Replaced the large log expand/collapse button with a small arrow toggle.
- Added hover explanations for Generator labels.
- Added click-to-lock Rail Detail.
- Updated viewer number sprites to use a JetBrains Mono first font stack.
- Updated Rail Library grouping to parse RowName segments as part, direction, descriptor, and size, with card names sourced from CN/EN config names when available and direction labels as fallback.

### Checkpoint Logic

- Checkpoint count is configurable with minimum 0.
- When checkpoint count is greater than 0, generator tracks segment difficulty.
- On checkpoint threshold, generator backtracks one rail and attempts to place a fork rail with at least two exits.
- One fork exit places checkpoint, another can continue maze generation.
- Stats and `MapMeta` include segment difficulties.

### Self-Spin Logic

- Added `maxSpins` generator option.
- Default `maxSpins` is 0.
- Non-zero `spinRot` is disallowed by default.
- Generator tracks `SpinCount`.
- Backtracking and rollback deduct consumed spin count.
- Stats and `MapMeta` include `SpinCount` and `MaxSpins`.

### Seed Logic

- Seed is now a complete generation configuration, not only random entropy.
- Current format is `bm01-random-difficulty-checkpoints-spins-bounds`.
- All seed fields are lowercase base36.
- Inputting a valid seed updates Generator controls and regenerates.
- Random seed button generates a full random seed/configuration.
- Generate button keeps current configuration but changes the random part.
- Legacy `BM1-...` parsing is still accepted, but new seeds use `bm01-...`.

### Direction and Footprint Work

- Added known asset overrides:
  - `BP_Curve_L90_X4_Y4_Z1_Rail`
  - `BP_Curve_L90_Borderless_O_X2_Y2_Z1_Rail`
- Reworked export direction so `Exit_Dir_Abs` is derived from `Exit_Rot_Abs`, not from position offset.
- Reworked placement footprint calculation toward the correct model:
  - generate local footprint first,
  - rotate local footprint with full `Rot_Abs`,
  - translate by `Pos_Rev`.
- Added `calculateLocalOccupiedCells`.
- Added `calculateOccupiedCellsWithRotAbs`.
- `placeRailV2` now uses `calculateOccupiedCellsWithRotAbs` for collision and bounds.
- Removed the incorrect `BP_Curve_R90_X3_Y3_Z1_Rail` footprint override so R90 X3 occupied cells stay on the same side as its local exit before and after full `Rot_Abs`.

### Documentation

- Split repository docs so Maze Builder has its own `README.md`, `AGENTS.md`, and `PROGRESS.md`.
- Consolidated seed and coordinate-system notes into stable Maze Builder docs.

## Verification

Latest known required checks:

```bash
cd web-maze-builder
npm test
npm run build
```

Historical status before doc split: 16 passing tests.

## Important Context

Direction/connection bugs were caused by mixed transform logic:

- footprint used old `rotIdx + rollIdx`;
- exit position used `Rot_Abs`;
- exit direction was sometimes inferred from offset.

The intended invariant is:

```text
local config -> full Rot_Abs transform -> world logical result
```

Everything should use `Pos_Rev + Rot_Abs` as the single authoritative rail pose.

## Known Risks / Next Work

- Some legacy tests still use `calculateOccupiedCells`, which now delegates to `calculateOccupiedCellsWithRotAbs`.
- Local footprint rules are still name-pattern based.
- Asset-level overrides are temporary; long term, CSV should encode true footprint side/shape explicitly.
- The viewer draws proxy boxes, not real mesh geometry.
- Visual disagreements with UE meshes may still happen if local footprint data is inaccurate.

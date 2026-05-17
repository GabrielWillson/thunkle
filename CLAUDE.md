# Thunkle

Single-file browser puzzle game. Everything lives in [thunkle_9.html](thunkle_9.html) — HTML, CSS, and JS in one file. No build step, no dependencies beyond a Google Font.

## The game

A 3×3 grid of pixel-art "patterns" (each pattern is a P×P bitmap, P=3 normal / P=4 in Miserable). The bottom-right cell is blank — the player toggles its pixels to fill in the missing pattern.

The grid follows a hidden rule along either rows or columns:

> `pattern_A  [OP]  pattern_B  =  pattern_C`

…applied bitwise pixel-by-pixel. The same operation and direction is used for every row (or every column) in the puzzle. Each row/column is independently generated, so the player must infer the rule from the two complete rows/columns and apply it to solve the third.

The result of the operation is always the right column (if direction is `row`) or the bottom row (if direction is `col`). The missing cell is always the bottom-right.

## Operations

Six bitwise ops, all defined in `applyOp` ([thunkle_9.html:712](thunkle_9.html:712)):

- `AND`, `OR` — Quick mode only
- `XOR`, `NAND`, `NOR`, `XNOR` — added in Challenge / Miserable

## Modes

Configured in `MODES` ([thunkle_9.html:580](thunkle_9.html:580)):

| Mode       | Ops              | Pattern size | Direction hint |
|------------|------------------|-------------|----------------|
| Quick      | AND, OR          | 3×3         | available      |
| Challenge  | all 6            | 3×3         | none           |
| Miserable  | all 6            | 4×4         | none           |
| Daily      | Challenge config, seeded by date | 3×3 | none |

## Scoring (TQ)

- Each completed Daily gets a score: `(expected_time / your_time) × 100`, where `expected_time` is `DAILY_EXPECT` = 120s.
- **TQ** (Thunkle Quotient) = rounded average of the last 5 Daily scores.
- Practice modes show a score (with their own expected times in `PRACTICE_EXPECT`) but do not feed TQ.
- History persists in `localStorage` under `thunkle_tq` and `thunkle_daily`.

## Puzzle generation

- `genPair` ([thunkle_9.html:730](thunkle_9.html:730)) brute-forces two input patterns whose result has between 15% and 85% on-pixels (so the answer isn't trivially all on/off). Per-op target densities live in `OP_DENSITY`.
- `buildGrid` fills each row or column with an independent `genPair` for the chosen op.
- Daily uses a seeded `mulberry32` RNG keyed off the date so everyone gets the same puzzle.

## Flow

`startGame` / `startDaily` → `buildPuzzleGrid` (renders) → player clicks pixels → `onPixelClick` toggles → `checkAnswer` compares against `S.answer` → `showResult`. Give-up reveals the solution grid via `buildSolutionReveal`.

State lives in the global `S` object, rebuilt per puzzle.

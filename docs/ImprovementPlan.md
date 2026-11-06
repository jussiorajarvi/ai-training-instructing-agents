# Improvement Plan — `app/` (Smelly Tic Tac Toe)

Date: 2026-11-06
Input: [ReviewFindings.md](ReviewFindings.md) (findings are referenced as **F1–F10**)

## Goal and ground rules

The `app/` project is a code-smells kata. Its README says to *"identify the smells and then slowly refactor the code. Remember to keep the tests passing at all times."* The plan follows that:

- **Small steps.** Each step is one commit that ends with `npm test` and `npm run lint` passing.
- **Separate refactoring from behaviour change.** Phases 1–3 change structure only, and every existing test must pass unchanged. Phase 4 changes behaviour, and every change there starts with a failing test.
- **Keep the public API.** `new Game()`, `Play(symbol, x, y)` and `Winner()` keep their names and signatures until Phase 5, so the tests keep compiling and the refactor stays verifiable.
- **Revert freely.** If a step breaks tests and the fix isn't obvious, `git checkout` back to the last green commit.

Coordinate convention in the current code: `x` is the **row** and `y` is the **column** (`TileAt(0, 0..2)` is the top row). The plan keeps that convention.

## Phase 0 — Safety net (tests only, no production changes)

Make sure the refactor can't silently change behaviour.

| Step | Change | Notes |
|------|--------|-------|
| 0.1 | Run `npm ci && npm test && npm run lint` and record the baseline | Lint is expected to flag the unused `tile` (F7) |
| 0.2 | Add characterization tests for current behaviour that isn't covered yet: 'X' can win in each row (already covered), a move on an occupied tile throws, alternate turns are enforced | Covers the paths the refactor touches |
| 0.3 | Add **pending** tests (`it.todo` / `it.skip`) for the known bugs: column wins, diagonal wins, play after a win, out-of-range coordinates, invalid symbols, draw | Documents F1–F6 without breaking the build; they are switched on in Phase 4 |

**Done when:** the suite is green and the todo list shows every known gap.

## Phase 1 — Cosmetic cleanups (no structural change)

| Step | Change | Smell / Finding |
|------|--------|-----------------|
| 1.1 | Delete the unused `tile` object in `AddTileAt` | Dead code (F7) |
| 1.2 | Make `AddTileAt` call `TileAt(x, y)` instead of repeating `find()` | Duplicated code (F7) |
| 1.3 | Delete the wrong or redundant comments; where a comment explains intent, turn it into a well-named method instead (see 2.x) | Comments (F9) |
| 1.4 | Use `===` / `!==` everywhere; replace `var` with `const` in the tests | Consistency |

**Done when:** lint is clean.

## Phase 2 — Refactor the smells (behaviour unchanged)

Order matters: each step makes the next one simpler.

| Step | Change | Smell / Finding |
|------|--------|-----------------|
| 2.1 | Add constants or a type for symbols: `type Player = 'X' \| 'O'` and `const EMPTY = ' '`, used everywhere instead of string literals | Primitive obsession |
| 2.2 | Add a `Position` value (`{ row, col }`) or keep `(x, y)` but put the coordinate checking in one place on `Board` | Data clump, long parameter list |
| 2.3 | Move the "is this tile taken?" question onto `Board`: `board.isEmpty(x, y)` and `board.symbolAt(x, y)`, so `Game` no longer reaches through `TileAt(...).Symbol` | Message chain, feature envy, data class (`Tile`) |
| 2.4 | Extract the move validation in `Play()` into named private methods: `validateFirstMove`, `validateTurnOrder`, `validatePositionFree` | Long method, comments |
| 2.5 | Replace the three copied row blocks in `Winner()` with a `WINNING_LINES` table and one loop. **Put only the 3 rows in the table**, so behaviour stays the same | Duplicated code, long method, shotgun surgery (F8) |
| 2.6 | Move line evaluation (`lineOwner(line)`) onto `Board`; `Game.Winner()` becomes a short delegation | Feature envy, divergent change |
| 2.7 | Replace `Board`'s flat `Tile[]` plus linear `find()` with a 3×3 array indexed `[x][y]`; `Tile` can then go away, or stay internal | Lazy class / data class, O(1) lookup (F10) |

**Done when:** all original tests pass unchanged, `Game.ts` has no `!` non-null assertions, and `Winner()` is about 5 lines.

## Phase 3 — Optional split into separate files

When Phase 2 is finished and only if it helps readability: move `Board` (and `Player`/`Position` types) into `Board.ts`, keeping `Game.ts` as the game rules. Add a small `Board.test.ts` for the board on its own.

## Phase 4 — Fix the bugs (behaviour changes, test first)

Every step: switch on the matching pending test from 0.3, see it fail, then make the smallest change that makes it pass.

| Step | Test to switch on | Fix | Finding |
|------|-------------------|-----|---------|
| 4.1 | Column wins (3 columns × X/O) | Add the 3 column lines to `WINNING_LINES` | F1 |
| 4.2 | Diagonal wins (2 diagonals × X/O) | Add the 2 diagonal lines | F1 |
| 4.3 | Out-of-range coordinates throw `Invalid position` | Bounds check in `Board` (one place, from 2.2) | F3 |
| 4.4 | Invalid symbols (`'Z'`, `' '`, `'x'`) throw | Validate `symbol` is a `Player` at runtime in `Play()` | F4 |
| 4.5 | `Play()` after a win throws (e.g. `Game over`) | Check `Winner()` (or a cached state) before accepting a move | F2 |
| 4.6 | A full board with no winner reports a draw | Add a game-status query (see design decision below) | F5 |
| 4.7 | First move on a valid tile with `_lastSymbol` handling | Replace the `' '` sentinel for "no move yet" with `Player \| null`, so the first-move branch no longer skips the occupied check | F4 |

**Design decision needed for 4.6:** `Winner()` currently returns `' '` for both "in progress" and "draw". Options:

- **A (recommended):** keep `Winner()` as it is and add `Status(): 'in-progress' | 'won' | 'draw'` (or `IsDraw()`). No existing caller breaks.
- **B:** change `Winner()` to return `Player | 'draw' | null`. Cleaner, but breaks callers and needs the existing tests updated.

**Done when:** no tests are pending, and every finding F1–F6 is covered by at least one test.

## Phase 5 — Optional API polish (breaking)

Only if breaking the API is acceptable, after Phase 4:

- Rename to camelCase (`play`, `winner`), the usual TypeScript style.
- Type the `symbol` parameter as `Player` so the compiler catches invalid symbols.
- Use specific error types or messages, and assert on them in tests (`toThrow('Invalid next player')`) rather than a bare `toThrow()`.

## Tooling (can be done any time)

- Add `npm run check` (`tsc --noEmit && eslint && jest`) so each step can be checked with one command.
- Enable `strict` in `tsconfig.json` if it isn't already; this works best after 2.3 and 2.7, which remove the `!` assertions.
- `@types/node` is pinned at 7.x, which is very old. Upgrade it in its own commit.

## Map from findings to steps

| Finding | Addressed in |
|---------|--------------|
| F1 Column/diagonal wins missing | 0.3, 2.5, 4.1, 4.2 |
| F2 Play after win | 0.3, 4.5 |
| F3 Out-of-range crash | 0.3, 2.2, 2.7, 4.3 |
| F4 Symbol not validated | 0.3, 2.1, 4.4, 4.7, 5 |
| F5 No draw state | 0.3, 4.6 |
| F6 Test gaps | 0.2, 0.3, Phase 4 |
| F7 Dead code / duplicate lookup | 1.1, 1.2 |
| F8 `Winner()` duplication | 2.5, 2.6 |
| F9 Wrong comments | 1.3, 2.4 |
| F10 Flat list with linear search | 2.7 |

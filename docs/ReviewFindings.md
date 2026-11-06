# Code Review Findings — `app/`

Reviewed: `app/src/Game.ts` and `app/src/Game.test.ts` (full codebase; no changes under `app/` since the initial commit).
Date: 2026-11-06

| # | Severity | Location | Finding |
|---|----------|----------|---------|
| 1 | High | [Game.ts:27](../app/src/Game.ts#L27) | `Winner()` only checks rows, never columns or diagonals |
| 2 | High | [Game.ts:5](../app/src/Game.ts#L5) | `Play()` accepts moves after the game is won |
| 3 | High | [Game.ts:18](../app/src/Game.ts#L18) | Out-of-range coordinates crash with a TypeError |
| 4 | Medium | [Game.ts:9](../app/src/Game.ts#L9) | Player symbol is never validated |
| 5 | Medium | [Game.ts:66](../app/src/Game.ts#L66) | No draw / game-over state |
| 6 | Medium | [Game.test.ts:105](../app/src/Game.test.ts#L105) | Test gaps: columns, diagonals, post-win play, bounds, symbols |
| 7 | Low | [Game.ts:99](../app/src/Game.ts#L99) | Dead code and duplicated lookup in `AddTileAt` |
| 8 | Low | [Game.ts:28](../app/src/Game.ts#L28) | `Winner()` copies one block three times |
| 9 | Low | [Game.ts:53](../app/src/Game.ts#L53) | Wrong comments |
| 10 | Low | [Game.ts:79](../app/src/Game.ts#L79) | Board uses a flat list with linear search |

## Correctness

### 1. `Winner()` only checks rows — [Game.ts:27](../app/src/Game.ts#L27)
`Winner()` checks the three rows and never checks columns or diagonals.

**Failure scenario:** X plays (0,0),(1,0),(2,0) and O plays (0,1),(1,1). X has a full column, but `Winner()` returns `' '`, so no winner is reported. A diagonal (0,0),(1,1),(2,2) is missed the same way.

### 2. Moves accepted after a win — [Game.ts:5](../app/src/Game.ts#L5)
`Play()` never checks whether the game has already been won.

**Failure scenario:** X completes the top row and `Winner()` returns `'X'`. O can still call `Play('O',2,2)` and later moves with no error. The board keeps changing after the game should be over, and O could also complete a row, so the result depends on move order.

### 3. Out-of-range coordinates crash — [Game.ts:18](../app/src/Game.ts#L18)
Coordinates are never range-checked. For out-of-range values, `TileAt`/`AddTileAt` get `undefined` back and then read it through a non-null assertion.

**Failure scenario:** `Play('X', 3, 0)` as the first move throws `Cannot set properties of undefined` in `AddTileAt`. On a later move, `TileAt(5,5).Symbol` throws `Cannot read properties of undefined`. Neither case raises the intended `Invalid position` error.

### 4. Player symbol not validated — [Game.ts:9](../app/src/Game.ts#L9)
The only symbol check rejects `'O'` as the first move. Any other string is accepted, including `' '` (the empty marker).

**Failure scenario:** `Play('Z',0,0)` is accepted. `Play(' ',0,0)` sets `_lastSymbol` to `' '`, which leaves the game in its "first move" state, so the occupied-tile check is skipped and moves can repeat indefinitely. Mixing `'x'` and `'X'` also passes the alternation check.

### 5. No draw / game-over state — [Game.ts:66](../app/src/Game.ts#L66)
When the board fills with no winner, `Winner()` returns `' '`, the same value as a game still in progress.

**Failure scenario:** After 9 moves with no three-in-a-row, callers cannot tell a draw from an unfinished game. A 10th `Play` throws `Invalid position` instead of reporting that the game is over.

## Tests

### 6. Test coverage gaps — [Game.test.ts:105](../app/src/Game.test.ts#L105)
The tests cover only row wins and the three invalid-move errors. Nothing tests column or diagonal wins, play after a win, out-of-range coordinates, or invalid symbols.

**Impact:** The whole suite passes even though finding #1 is a real bug. A refactor that broke any of the untested paths would pass too.

## Maintainability / Simplification

### 7. Dead code and duplicated lookup in `AddTileAt` — [Game.ts:99](../app/src/Game.ts#L99)
`AddTileAt` builds a `tile` object that is never used. Its lookup copies `TileAt`'s `find()` instead of calling `TileAt`.

**Impact:** An object is allocated on every move for nothing, and eslint reports `no-unused-vars`. Because the lookup exists in two places, a fix to one (such as bounds handling) can be missed in the other.

### 8. Repeated blocks in `Winner()` — [Game.ts:28](../app/src/Game.ts#L28)
One 10-line block appears three times with only the row index changed, and `Winner()` calls `TileAt` (a linear search) up to 30 times per check.

**Suggestion:** Loop over a list of the 8 winning lines. That replaces about 40 lines and also fixes finding #1.

### 9. Wrong comments — [Game.ts:53](../app/src/Game.ts#L53)
The row-2 and row-3 checks both say "first row", the row-3 check also says "middle row", and "if player is X" sits above a check for `'O'`.

**Impact:** The comments mislead anyone editing this code.

### 10. Flat board with linear search — [Game.ts:79](../app/src/Game.ts#L79)
`Board` keeps the 9 tiles in a flat list and finds each one by linear search.

**Suggestion:** Index a 3x3 array by `[x][y]`. Lookups become O(1) and the bounds check becomes simple (supports #3).

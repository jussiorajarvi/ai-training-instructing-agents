# CLAUDE.md

The code lives in `app/`. It is the "Smelly Tic Tac Toe" code-smells kata: a small TypeScript library (`app/src/Game.ts`) with Jest tests (`app/src/Game.test.ts`). The goal is to refactor the smells out while keeping the tests green.

## Tech stack

- TypeScript 5.9 (`strict`, `noImplicitAny`), compiled to ES6 / CommonJS
- Node.js library: no UI, no entry point, no runtime dependencies in use
- Jest 30 + ts-jest; tests are `*.test.ts` under `app/src/`
- ESLint 10 + typescript-eslint (type-checked); `eqeqeq` and `no-var` are errors

## Commands

Run from `app/`:

- `npm test`: run the tests
- `npm run lint`: lint `src/**/*.ts` (the existing code still has lint errors, so check that your change adds none)
- `npm run build`: compile with `tsc`

## Architecture rules

1. **`Game` holds the rules; `Board` holds the grid.** Turn order, move validation and win detection belong in `Game`. Tile storage and lookup belong in `Board`. Don't let `Game` reach into tile internals (`TileAt(...).Symbol`); add a `Board` method that answers the question instead.
2. **Refactor in small steps that keep the tests passing.** Each change is small and ends with `npm test` passing. Never mix a restructure with a behaviour change. A restructure must keep behaviour the same; a behaviour change starts with a failing test.
3. **Keep the domain rules in one place.** Player symbols, the empty marker `' '`, the board size and the winning lines are each defined once (types or constants), never repeated as string literals. Checks like bounds and "is this tile free" live in one place.

## Do not change

1. **The public API of `Game`:** `new Game()`, `Play(symbol: string, x: number, y: number): void`, `Winner(): string`. Keep the PascalCase names, the parameter order (`x` = row, `y` = column), throwing on an invalid move, and `' '` meaning "no winner".
2. **The existing tests in `app/src/Game.test.ts`:** don't edit, delete or weaken the existing test cases; they check that the refactor kept behaviour the same. Adding new tests is fine.

## Related docs

- `docs/ReviewFindings.md`: code review findings
- `docs/ImprovementPlan.md`: step-by-step refactoring plan

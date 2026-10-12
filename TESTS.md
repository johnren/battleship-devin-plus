# Tests run while building Battleship (johnren/battleship-devin)

These are the tests from building v1 in [battleship-devin](https://github.com/johnren/battleship-devin). v2 (this repo) has the same 58 unit and 9 end-to-end tests. Its `e2e/game.spec.ts` also checks the "Your fleet" and "Enemy fleet" board headings. Sections 3–5 were run against v1 and have not been re-run on v2.

## Totals

| Category | Count | Result |
|---|---|---|
| Unit tests (Vitest), in the repo | 58 tests in 5 files | 58 passed |
| End-to-end tests (Playwright), in the repo | 3 tests × 3 browsers = 9 runs | 7 passed, 2 skipped (touch tests on Firefox) |
| Seeded full games played by a script | 7 games | 2 Victory, 5 Defeat, all reached game over |
| Manual play-test by the testing agent (real browser, recorded) | 19 checks | 18 passed, 1 failed (since fixed) |
| One-off scripted checks during development | 5 | all passed after fixes |
| CI runs on GitHub Actions (lint, unit, build, e2e) | 3 runs | 3 passed |

The automated suite in the repo is **67 tests**: 58 unit and 9 end-to-end. Each CI run executes all of them, along with lint (`npm run lint`) and the strict type-check plus production build (`npm run build`).

## 1. Unit tests (Vitest) — `npm test`

All in `src/game/`. They test pure game logic, with no browser.


### `placement.test.ts` — ship placement rules (14 tests)

| # | Group | Test |
|---|---|---|
| 1 | shipCells | extends right for horizontal and down for vertical |
| 2 | checkPlacement edges | accepts ships flush against every edge |
| 3 | checkPlacement edges | rejects ships past the right edge |
| 4 | checkPlacement edges | rejects ships past the bottom edge |
| 5 | checkPlacement edges | rejects ships starting off the top or left edge |
| 6 | checkPlacement edges | does not wrap from column 10 to column 1 |
| 7 | checkPlacement edges | does not wrap from row J to row A |
| 8 | overlaps | rejects a crossing ship |
| 9 | overlaps | rejects a ship sharing an end cell |
| 10 | overlaps | allows touching ships |
| 11 | placeShip | adds a valid ship and returns a new array |
| 12 | placeShip | returns null for invalid or duplicate placement |
| 13 | randomFleet | always produces a complete, valid, non-overlapping fleet |
| 14 | randomFleet | is repeatable for a seed |

### `firing.test.ts` — firing, sinking, winning (14 tests)

| # | Group | Test |
|---|---|---|
| 15 | fire | reports a miss on water |
| 16 | fire | reports a hit with the ship name |
| 17 | fire | rejects a repeated shot on a miss or a hit |
| 18 | fire | rejects shots off the board |
| 19 | fire | does not mutate the input board |
| 20 | sunk detection | sinks the Carrier only on its last cell |
| 21 | sunk detection | sinks the Battleship only on its last cell |
| 22 | sunk detection | sinks the Cruiser only on its last cell |
| 23 | sunk detection | sinks the Submarine only on its last cell |
| 24 | sunk detection | sinks the Destroyer only on its last cell |
| 25 | sunk detection | sinks a ship when hit out of order |
| 26 | win detection | is not won until all 17 cells are hit |
| 27 | win detection | misses never win |
| 28 | win detection | an empty board is never won |

### `ai.test.ts` — computer opponent (8 tests)

| # | Group | Test |
|---|---|---|
| 29 | computer opponent: 100 simulated games | never repeats, never fires off the board, never hunts with an unsunk ship hit |
| 30 | computer opponent: 100 simulated games | is repeatable for the same seed |
| 31 | computer opponent: strategy | hunts on a checkerboard |
| 32 | computer opponent: strategy | tries up, down, left, right after a first hit |
| 33 | computer opponent: strategy | skips neighbours that are off the board |
| 34 | computer opponent: strategy | follows a line of hits in both directions |
| 35 | computer opponent: strategy | keeps targeting leftover hits after a sink |
| 36 | computer opponent: strategy | falls back to neighbours when a line is blocked at both ends |

### `game.test.ts` — game reducer: turns, game over, Play Again (18 tests)

| # | Group | Test |
|---|---|---|
| 37 | initial state | places a full random computer fleet, repeatable by seed |
| 38 | placement | places the selected ship and selects the next one |
| 39 | placement | rotates between horizontal and vertical |
| 40 | placement | ignores invalid placements |
| 41 | placement | cannot start until all five ships are placed |
| 42 | placement | randomize places a full fleet and reset clears it |
| 43 | placement | randomize does not change the computer fleet or its shot sequence |
| 44 | turns | player fires first, then the computer, alternating |
| 45 | turns | a hit does not grant an extra turn |
| 46 | turns | repeated shots are rejected without using a turn |
| 47 | turns | rapid repeated clicks fire only one shot |
| 48 | turns | cannot fire before the game starts or during the computer turn |
| 49 | turns | ignores computer moves with a stale turn id |
| 50 | turns | keeps only the latest log entries, newest first |
| 51 | game over | player wins by sinking every ship |
| 52 | game over | computer wins when the player only misses |
| 53 | play again | fully resets state |
| 54 | play again | cancels a pending computer move |

### `rng.test.ts` — seeded random numbers (4 tests)

| # | Group | Test |
|---|---|---|
| 55 | createRng | is repeatable for the same seed |
| 56 | createRng | differs for different seeds |
| 57 | createRng | continues the same sequence from a saved state |
| 58 | createRng | int stays within range |

## 2. End-to-end tests (Playwright) — `npm run test:e2e`

These build the app, serve it on port 4173, and drive it in Chromium, Firefox and WebKit.

| # | File | Test | What it checks | Chromium | Firefox | WebKit |
|---|---|---|---|---|---|---|
| 59 | `e2e/game.spec.ts` | plays a full seeded game to game over | `?seed=123`: Start disabled → Randomize → Start → a hunt/target player fires one shot per turn until the game-over dialog (Victory/Defeat, shots, accuracy) → Play Again returns to placement | passed | passed | passed |
| 60 | `e2e/touch-placement.spec.ts` | touch: first tap previews, second tap on the same cell places | At 375 px with touch: one tap shows a 5-cell green preview, a second tap places the Carrier (regression test for bug #1) | passed | skipped (no touch emulation) | passed |
| 61 | `e2e/touch-layout.spec.ts` | touch: cells are at least 44 px and only the grid scrolls sideways | At 375 px with touch: cells ≥ 44 × 44 px on both boards, the page has no horizontal overflow, and tapping a cell scrolled into view still previews (regression test for bug #2) | passed | skipped (no touch emulation) | passed |

## 3. Seeded full games played by a script (Chromium)

Each game used the same hunt/target player as the e2e test, which reads only what's visible on the enemy grid.

| Seed | Result | Shots fired | Hit accuracy |
|---|---|---|---|
| 123 | Victory | 55 | 31% |
| 1 | Defeat | 40 | 25% |
| 2 | Defeat | 53 | 25% |
| 3 | Victory | 59 | 29% |
| 4 | Defeat | 59 | 24% |
| 5 | Defeat | 58 | 28% |
| 6 | Defeat | 51 | 22% |

An earlier scripted game (seed 123, firing cells in order instead of hunting) ended in Defeat after 60 shots at 18% accuracy. Play Again then returned to an empty placement screen.

## 4. Manual play-test by the testing agent (recorded)

Real mouse and keyboard input in Chrome, plus 375 px touch emulation.

| # | Check | Result |
|---|---|---|
| 1 | Start disabled until all five ships are placed | passed |
| 2 | Green preview for valid placements, red for invalid | passed |
| 3 | Overlapping, off-board and wrapping placements rejected | passed |
| 4 | Vertical Destroyer placed at I10–J10 (bottom-right corner) | passed |
| 5 | Rotate via button, R, and Space; Space neither scrolls the page nor presses the focused button | passed |
| 6 | Randomize builds a full fleet; Reset clears it | passed |
| 7 | Player fires first; computer replies after about 600 ms (measured 602.1 ms) | passed |
| 8 | Both grids locked while the enemy is firing | passed |
| 9 | Re-firing at a used cell and rapid clicks never add an extra turn | passed |
| 10 | Hit, miss and sunk markers | passed |
| 11 | Fleet status panels | passed |
| 12 | Message log shows the latest 5 shots, newest first | passed |
| 13 | Tab shows the accent focus outline on cells; Enter fires exactly once | passed |
| 14 | Full game to Victory: 58 shots, 17 hits, 29% (matches the board) | passed |
| 15 | Play Again clears ships, shots, damage and log; no stray computer shot 1.2 s later | passed |
| 16 | At 375 px the boards stack with no horizontal page overflow | passed |
| 17 | Touch: first tap previews, second tap places; tapping a different cell moves the preview | passed |
| 18 | Mobile firing at column 10: one shot, one computer reply | passed |
| 19 | Touch cells at least 44 px tall (SPEC.md) | **failed** (29.4 px), fixed afterwards (bug #2) |

Not covered by hand: reaching a Defeat (the scripted games did), and resetting while a computer move is pending (Play Again only appears at game over; unit test 54 covers it).

## 5. One-off scripted checks during development

| # | Check | Result |
|---|---|---|
| 1 | Logged pointer and mouse events during a touch tap | Showed a `mouseleave` arriving after the tap and clearing the preview; this was bug #1 |
| 2 | Mouse hover shows the preview and moving away clears it (after the bug #1 fix) | passed |
| 3 | Layout measurements at 375 px in two touch device setups (after the bug #2 fix) | Cells 44 × 44 px; the grid scrolls inside its board (492 px of content in 351 px); the page width stays 375 px |
| 4 | Desktop screenshot after the bug #2 fix | Layout unchanged |
| 5 | Sunk-ship flash, flagged by Devin Review | The animation was running right after a sink and gone 700 ms later; it's disabled under reduced motion |

## Bugs the tests found

1. **Touch preview cleared after the first tap.** Found by manual touch testing; regression test 60.
2. **Touch cells 29.4 px instead of 44 px.** Found by the testing agent's play-test; regression test 61.
3. **Sunk ships never flashed.** Found by Devin Review on PR #4 and checked with a one-off script (no automated test).

Bugs 1 and 2 are written up in `BUGS.md`.

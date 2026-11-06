---
name: vyzhivshiy-arcade
description: Work on the "Выживший" browser arcade (arcade/ in survival-open). Use when changing gameplay, HUD, menus, balance, enemy AI or visuals of this game, or when running its tests and browser playtests.
---

# Выживший — arcade work guide

Browser space arcade. Vanilla ES modules, DOM/CSS UI plus a canvas playfield.
No engine, no build step, no bundler. `arcade/index.html` is the entry point.

## Hard rules

- **Never edit the upstream repository.** Source of truth for the published game is
  `vesninsergeybk-afk/survival-open`, branch `main`, published to
  `https://vesninsergeybk-afk.github.io/survival-open/arcade/`. Work in a local clone
  or a separate fork; do not push to upstream without the owner's explicit request.
- **Never publish a second arcade build.** One Pages site only, one URL. See
  `docs/PUBLICATION_POLICY.md`. A fork must not enable Pages.
- **Do not touch `Timeweb/site/games/survivor/`** — historical copy, not a release.
- Keep `main` green: every change must survive the full test suite.

## Local loop (verified facts)

```bash
node --test "arcade/tests/*.test.mjs"     # 1234 tests, ~18 s, no npm install needed
node --test arcade/tests                  # DOES NOT WORK — treats dir as a module
```
Quotes around the glob are required. Node 22 runs everything with zero dependencies.

Static server for manual play (no deps, repo root):
```bash
node serve-audit.cjs . 8123      # http://localhost:8123/arcade/
```

## Size of the thing (measured 2026-11-06)

| Item | Count |
| --- | --- |
| JS/CSS files in `arcade/` | 131 |
| Test files | 130 (1234 test cases) |
| Design docs (`arcade/*.md`, `docs/*.md`) | 38 + 60 |
| `game.js` | ~4 000 lines / 271 KB |
| `base-panel.js` | ~1 000 lines / 104 KB |
| `menu-system.css` | ~1 000 lines / 91 KB |

Do not "just rewrite" this in another engine. The UI is DOM-first: `index.html` has
75 `div`, 26 `button`, 114 `id`, and only 2 `canvas`; `game.js` calls `querySelector`
58 times and `getContext` once. Porting to Phaser would mean re-implementing the whole
menu layer in canvas.

## Where things live

| Area | Files |
| --- | --- |
| Main loop, state, docking | `game.js` |
| HUD rendering | `hud.js`, `hud.css`, `hud-art.css`, `mobile-hud.css` |
| Station terminal (5 tabs) | `base-panel.js`, `station-*.css`, `menu-system.css` |
| Modal screens (opening, death, continuation) | `menu-screens.js` |
| Hangar / loadout / development | `hangar-panel.js`, `ship-loadout.js`, `ship-weapons.js`, `development-panel.js` |
| Map, contours, territory | `map-panel.js`, `contour-*.js`, `territory-system.js`, `regional-map.js` |
| Enemies | `ai.js` (`ENEMY_TYPES`), `enemy-*.js` |
| Balance constants | `balance.js` (`RULES`, `expForNext`, `openingThreat`) |
| Save / checkpoint | `run-checkpoint.js` |

## Key balance facts

- Player starts with `hp:110, maxHp:110, shield:0` (`game.js`, player object).
- Enemy damage: scout 9, fighter 12, flanker 13, healer 5, berserker 18, gunship 13.
- After each hit the player gets `invul` 0.55 s + 0.12 s per dodge rank — that caps DPS.
- `openingThreat(time)` stages pressure: 0 enemies to 9 s, cap 4 to 36 s, cap 7 to 75 s,
  cap 11 to 118 s, cap 16 to 155 s, cap 21 to 205 s, then slow growth to 70.
- Early XP thresholds: `[42, 68, 105, 155, 230, 330, 475]`, later `75 * 1.36^(level-1)`.
- Docking at an active station writes a checkpoint; death without one wipes the run.

## Menu structure (and why it feels heavy)

5 top tabs (`Главная`, `Задания`, `Корабль`, `Развитие`, `Станция`), plus 3 hangar
sub-views and 5 development categories. Measured control counts per tab:

| Tab | Buttons | Disabled | Headings | Words |
| --- | --- | --- | --- | --- |
| Главная | 17 | 2 | 5 | 246 |
| Задания | 4 | 0 | 2 | 125 |
| Корабль | 18 | 3 | 2 | 116 |
| Развитие | 22 | 6 | 4 | 310 |
| Станция | 7 | 4 | 6 | 318 |

Design rules the project already set for itself (`docs/MENU_FUNCTION_REGISTRY_2026_11_05.md`):
no more than two decisions before the primary action, buying and browsing are separate
buttons, every blocked action states the exact reason, screen switching must not change
game state.

## Playtest method that actually works

Playwright MCP needs a browser. Chrome is not installed in this environment; Edge is,
so the extension runs with `--browser=msedge`.

- Synthetic `KeyboardEvent` dispatch from `page.evaluate` is **not** reliably consumed by
  the game input layer. Use `page.keyboard.down()/up()` (real input) instead.
- Screenshots are not readable by every model; audit through `innerText`, ARIA snapshots
  and DOM measurements instead.
- Useful hooks: station distance is in the text as `СТАНЦИЯ · N М`; docking range is
  `RULES.baseInteract` = 165; the ship has inertia, so tap thrust, do not hold it.
- The opening choice modal appears at ~00:15 and **pauses the world** while it is open.

## Reading files on Windows

Repo files are UTF-8. `type` and bare `Get-Content` mangle Cyrillic in a cp866 console;
set `[Console]::OutputEncoding = [System.Text.Encoding]::UTF8` and read with
`-Encoding UTF8`. Quoting `$_` inside `powershell -Command "..."` from cmd breaks —
write a `.ps1` file instead of inlining loops.

## Documentation worth reading before a gameplay change

- `arcade/WHOLE-GAME-REVIEW-2026-11-04.md` — the owner's 31-point requirement registry.
- `docs/FIRST_TEN_MINUTES.md` — the implemented opening scenario and its limits.
- `docs/MENU_FUNCTION_REGISTRY_2026_11_05.md` — every working menu command.
- `docs/PUBLICATION_POLICY.md` — publishing rules (non-negotiable).

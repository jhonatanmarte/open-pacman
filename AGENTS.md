# AGENTS.md

Pac-Man-like game in No build tooling, no framework. Main purpose is learning **spec-driven development** (see README and the skills in `.agents/skills/`).

## Run / verify

- No package.json, tests, linter, bundler, or dev server. Run by opening `src/index.html` in a browser (the arcade game loop runs via `requestAnimationFrame`).
- No automated verification exists: "verify" means open the HTML in a browser and play (cannot be confirmed from CLI).
- After JS edits, hard-refresh the browser tab; there is no HMR/watcher.

## Module wiring (critical)

Scripts are NOT ES modules. They share state exclusively through **`window` globals** and must load in this exact order in `src/index.html`:

1. `js/maze.js` → exports `MAZE`, `TUNNEL_ROW`, `PACMAN_START`, `GHOST_STARTS`
2. `js/game.js` – consumes those; exports `createGame`, `update` (and `DIRS`)
3. `js/render.js` – exports `draw`
4. `js/main.js` – owns the loop (`update`/`draw`, both globals), input, and overlays

If you add a file or a cross-file dependency, wire it via `window.*` and update the `<script>` order in `index.html`. Global-only means no `import`/`export`; avoid classic `const` collisions across files.

## Grid model (maze.js)

- `MAZE_STR` is 31 strings of 28 chars; parsed to numeric `MAZE`. Legend: `#=1 wall`, `.=2 dot`, `-=3 door(pen gate)`, space=0 passable. Cell (x,y), x∈[0,27], y∈[0,30].
- `MAZE` (and its source) is treated as **pristine/immutable**. Each game copies it into `game.grid` (maze.js:19); eaten dots mutate `game.grid`, never `MAZE`. Preserve this invariant when editing.
- State machine: `game.state` ∈ `start | playing | won | lost`.
- Movement uses fractional cell positions aligned to whole cells; do not break the `aligned()`/`canMove()`/`wrapTunnel()` logic.

## Workflow & conventions

- This repo exists to practice **spec-driven development**. Big changes should go through the two skills in `.agents/skills/ + spec-impl` (workflow: write a spec to `specs/NN-slug.md`, get user approval, then implement on a `spec-NN-slug` branch). The `specs/` directory does not exist yet — the first spec would be `01-`.
- `spec` skill reads `README.md`/`AGENTS.md`/previous specs to match **language and wording of existing specs**. README and code comments are written in **Spanish**; match existing docs' language when authoring specs.
- Code style (follow existing files): two-space indent, single quotes, `const`-first, and `functionName( a, b )` style with a space before parentheses. JS comments in Spanish.

## Which files change for a feature

- Game rules/state → `src/js/game.js`
- Maze/grid layout → `src/js/maze.js`
- Canvas drawing → `src/js/render.js`
- Loop, input, screens → `src/js/main.js` / `src/index.html`
- Look/feel → `src/css/style.css`
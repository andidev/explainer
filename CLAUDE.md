# Explainer: notes for AI coding agents

Explainer is a very simple Draw.io-like tool for explaining things: boxes with centred text,
connected by lines. The entire app is `index.html` (inline CSS and JS, no build step). Open it in
a browser to run it.

## Decisions log (required)

`DECISIONS.md` records every product and design decision with a permanent ID (`D-001`, `D-002`, …).
It describes how the app must behave.

- Read `DECISIONS.md` before changing behaviour. Only `Active` (and the still-valid parts of
  `Amended by`) decisions are in force.
- **Every time the user asks for a feature or change, record it in `DECISIONS.md` in the same
  change**: add a new entry with the next `D-NNN` ID, using the template at the top of the file.
- If the request overrides an earlier decision, set the new entry's `Supersedes:` (or `Amends:`)
  line and update the old entry's `Status:` to `Superseded by D-NNN` (or `Amended by D-NNN`).
  Never delete or renumber entries.
- Reference decision IDs in commit messages.
- If a request is ambiguous or contradicts an Active decision, say so and confirm before
  overriding it.

## Code map (index.html)

- State: `state = { nextId, boxes:[{id,x,y,w,h,text}], links:[{id,from,fa,to,ta,arrow,curve}] }`.
  Anchors (`fa`/`ta`) are keys of `A`: nw, n, ne, e, se, s, sw, w.
- History: `commit()` snapshots `state` for undo/redo; call it after every user-visible change.
- Gestures: viewport `pointerdown` dispatches to `downCanvas`, `downBox`, `downHandle`, `startPan`;
  the active gesture lives in `gesture` and must end via `endGesture`/`cancelGesture`.
- Lines: `linkGeom` → `route` (orthogonal path search) or `arcCurve`; drawn in `renderLinks`.
- Persistence: localStorage (`explainer.*` keys) plus the URL hash via `pack`/`encodeBoard`.
  If you add state or a setting, include it in `pack`/`unpack` or `currentSettings`/`applySettings`,
  and keep old links loading (the format is versioned).
- Theme: colours are CSS tokens on `:root`, redefined for dark mode. Use tokens, never literals.
- Shortcuts panel (`#help`) must be updated when shortcuts change.

## Repository files

- `README.md` is for people: what Explainer is and how to use it. Keep its shortcut table in sync
  with the in-app shortcuts panel.
- `docs/` holds the logo (`logo-light.png`, `logo-dark.png`) and README screenshots.
- `AGENTS.md` points other coding agents to this file.

## Checks before committing

There are no automated tests. Open `index.html` in a browser and exercise what you changed, plus:
draw a box and type, connect two boxes, undo/redo, refresh (board survives), dark mode.
